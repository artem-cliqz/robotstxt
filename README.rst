What
====

This is a robots.txt exclusion protocol implementation for Go language (golang).


Build
=====

To build and run tests run `go test` in source directory.


Usage
=====

As usual, no special installation is required, just

    import "github.com/temoto/robotstxt"

run `go get` and you're ready.

1. Parse
^^^^^^^^

First of all, you need to parse robots.txt data. You can do it with
functions `FromBytes(body []byte) (*RobotsData, error)` or same for `string`::

    robots, err := robotstxt.FromBytes([]byte("User-agent: *\nDisallow:"))
    robots, err := robotstxt.FromString("User-agent: *\nDisallow:")

As of 2012-10-03, `FromBytes` is the most efficient method, everything else
is a wrapper for this core function.

There are few convenient constructors for various purposes:

* `FromResponse(*http.Response) (*RobotsData, error)` to init robots data
from HTTP response. It *does not* call `response.Body.Close()`::

    robots, err := robotstxt.FromResponse(resp)
    resp.Body.Close()
    if err != nil {
        log.Println("Error parsing robots.txt:", err.Error())
    }

* `FromStatusAndBytes(statusCode int, body []byte) (*RobotsData, error)` or
`FromStatusAndString` if you prefer to read bytes (string) yourself.
Passing status code applies following logic in line with Google's interpretation
of robots.txt files:

    * status 2xx  -> parse body with `FromBytes` and apply rules listed there.
    * status 4xx  -> allow all (even 401/403, as recommended by Google).
    * other (5xx) -> disallow all, consider this a temporary unavailability.

2. Query
^^^^^^^^

Parsing robots.txt content builds a kind of logic database, which you can
query with `(r *RobotsData) TestAgent(url, agent string) (bool)`.

Explicit passing of agent is useful if you want to query for different agents. For
single agent users there is an efficient option: `RobotsData.FindGroup(userAgent string)`
returns a structure with `.Test(path string)` method and `.CrawlDelay time.Duration`.

Simple query with explicit user agent. Each call will scan all rules.

::

    allow := robots.TestAgent("/", "FooBot")

Or query several paths against same user agent for performance.

::

    group := robots.FindGroup("BarBot")
    group.Test("/")
    group.Test("/download.mp3")
    group.Test("/news/article-2012-1")


3. Choose pattern matcher
^^^^^^^^
`regexp.Regexp` object caches one matching machine per concurrently matched goroutine.
Every machine allocates 32+KB for every matched expression.
With broad crawls and robotx.txt caching it becomes a problem.

Alternative matcher that supports only "*" and "$" symbols can be used by setting
`RobotsPatternMatcherType = GLOB_MATCHER`

Who
===

Honorable contributors (in undefined order):

    * Ilya Grigorik (igrigorik)
    * Martin Angers (PuerkitoBio)
    * Micha Gorelick (mynameisfiber)

Initial commit and other: Sergey Shepelev temotor@gmail.com
