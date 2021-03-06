## tornado-httpclient-mock
A library for mocking requests in Tornado HTTP client 

[![Build Status](https://travis-ci.org/SuminAndrew/tornado-httpclient-mock.svg?branch=master)](https://travis-ci.org/SuminAndrew/tornado-httpclient-mock)
[![codecov](https://codecov.io/gh/SuminAndrew/tornado-httpclient-mock/branch/master/graph/badge.svg)](https://codecov.io/gh/SuminAndrew/tornado-httpclient-mock)

This library can be used to patch `AsyncHTTPClient.fetch_impl` method to stub requests to external resources.

`tornado_mock.httpclient.patch_http_client` does this patching on `AsyncHTTPClient` instance.
After that you can use `tornado_mock.httpclient.set_stub` method to setup an appropriate responses to the requests.

```python
from tornado import gen
from tornado.httpclient import AsyncHTTPClient
from tornado.testing import AsyncHTTPTestCase
from tornado.web import Application, RequestHandler

from tornado_mock.httpclient import get_response_stub, patch_http_client, set_stub


class TestHandler(RequestHandler):
    @gen.coroutine
    def get(self, *args, **kwargs):
        response1 = yield self.application.http_client.fetch('http://example.com/path?query')
        self.write(response1.body)

        response2 = yield self.application.http_client.fetch('http://test.com/path')
        self.finish(response2.body)


class HTTPClientMockTest(AsyncHTTPTestCase):
    def get_app(self):
        app = Application([('/test', TestHandler)])
        self.app_http_client = app.http_client = AsyncHTTPClient(force_instance=True)
        return app

    def test_mock(self):
        patch_http_client(self.app_http_client)

        def get_response(request):
            return get_response_stub(request, buffer='STUB2')

        set_stub(self.app_http_client, 'http://example.com/path', response_body='STUB1')
        set_stub(self.app_http_client, 'http://test.com/path', response_function=get_response)

        self.assertEqual(self.fetch('/test').body, b'STUB1STUB2')
```

For more information see docstrings in [httpclient.py](tornado_mock/httpclient.py).
