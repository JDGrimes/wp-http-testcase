WP HTTP Testcase
================

PHPUnit testcase for testing code that uses WordPress's `WP_Http` class.

# Usage

## Mocking Responses

If you use `wp_remote_request()` or other wrappers for `WP_Http` methods in your
code, this makes it difficult to test, especially if the remote server may not be
reachable from your testing environment. This testcase solves this by letting you
mock the remote responses.

To use it in your code, you need to first include the `wp-http-testcase.php` file in
your PHPUnit bootstrap file.

Then, in your tests that involve `WP_Http`, you need to extend `WP_HTTP_UnitTestCase`
instead of `WP_HTTP_UnitTestCase` as you normally would.

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
