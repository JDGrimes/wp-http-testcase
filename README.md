WP HTTP Testcase
================

PHPUnit testcase for testing code that uses WordPress's `WP_Http` class.

If you use `wp_remote_request()` or other wrappers for `WP_Http` methods in your
code, this makes it difficult to test, especially if the remote server may not be
reachable from your testing environment. This testcase solves this by letting you
route your requests to a different host address, use a cached set of
responses, or just mock the remote responses by supplying artificial ones.

# Installation

You can install this package using composer:

```bash
composer require --dev jdgrimes/wp-http-testcase
```

# Usage

To use it in your code, you need to first include the `wp-http-testcase.php` file in
your PHPUnit bootstrap file. If you will be using the host routing and response
caching features, you will need to call `WP_HTTP_TestCase::init()` in your
bootstrap file.

Then, in your tests that involve `WP_Http`, you need to extend `WP_HTTP_TestCase`
instead of `WP_UnitTestCase` as you normally would.

## Mocking Responses

### Using Response Caching

The best way of testing, when possible, it to set up a mock host to handle the
requests. In some cases, you may want or need to actually send the requests through
to the real server, and that can be done as well. Which of these you do will depend
on the nature of the requests, and what side-effects they produce on the recipient
host.

#### Setting Up a Test Host

For example, if you are testing a plugin that makes requests to an API provided
by another plugin or other software, you probably don't want or need to test this on
a live site. Instead, you can set up a test site, or use a local server that is part
of your development environment. There you can install the software that handles
the requests. Once this is done, you can run your tests against that test site like
this:

```bash
WP_HTTP_TC_HOST=localhost phpunit
```

Just replace `localhost` with the hostname of the local server. Note that the
`WP_HTTP_TC_*` flags can be defined as PHP constants, or as bash environment
variables as above. The latter will take precedence.

#### Enabling Caching

Of course this will be much slower than most other unit tests, because the requests
are bound to take a bit of time. That is where caching comes in. When caching is
enabled, the response to each request is cached the first time it is run, and the
cached version is used in the future. This means that your tests can remain lightning
fast.

To enable caching, just add this to your bootstrap:

```php
define( 'WP_HTTP_TC_USE_CACHING', true );
```

You'll probably also want to specify the directory to save the cache in, via
`WP_HTTP_TC_CACHE_DIR`. You can utilize multiple cache groups and switch between them
using `WP_HTTP_TC_CACHE_GROUP`.

#### Using the Live Host

There is the second case though, where you are unable to set up a test server. An
example where this would be the case would be if your plugin makes requests to the
API provided by GitHub. Depending on the situation, it may be feasible to actually
make the requests to the "live" recipient. The main issue again is that the requests
will make the tests take a long time to complete. There is also the possibility that
the API isn't always accessible from your testing environment, or that your tests
will end up pounding the API too hard and you'll get blocked. This is where caching
can help you. You only need to run your tests against the "live" API once in a while,
and the rest of the time you can test using the cached responses.

### Supplying Artificial Responses

Of course, there may be times when it isn't possible to create a test server, and it
isn't feasible to run against the live server either. In this case, you may want to
hard-code artificial responses into your tests. Here is how you can do that:

Before calling the code that will invoke the HTTP request, you need to set the
function to mock the responses like so:

```php
$this->http_responder = array( $this, 'mock_server_response' );
```

The HTTP responder function will be passed two arguments, the request arguments and
the URL the request was intended for.

```php
protected function mock_server_response( $request, $url ) {

	return array( 'body' => 'Test response.' );
}
```

For a full list of the `$request` and response arguments, see
[`WP_Http::request()`](http://developer.wordpress.org/reference/classes/wp_http/
request/#source-code)

## Testing Requests

You may also wish to test that your code is making requests as expected. You can do
this by checking the value of `$this->http_requests`, which is an array of requests.
Each entry in the array stores the request arguments (`'request'` key) and URL
(`'url'` key).

To check that a request was made, you could do something like this:

```php
$this->assertCount( 1, $this->http_requests );
$this->assertEquals( 'http://example.com/', $this->http_requests[0]['url'] );
```

When you just want to test the request and don't care about the response, you can
short-circuit the request before it is made, by setting the response mocker to be
`__return_true()`:

```php
$this->http_responder = '__return_true';

my_prefix_make_request();

$this->assertCount( 1, $this->http_requests );
```
