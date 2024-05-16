[httpbin.org](http://httpbin.org) 是一个简单的 HTTP 请求和响应服务，它用于测试 HTTP 库、HTTP 客户端和其他与 HTTP 相关的功能。这个服务提供了多种测试端点，可以用来检查 HTTP 请求的各种属性和行为。

# 支持的测试功能

- **/get**: 返回请求的查询参数和其他信息。
- **/post**: 用于测试 POST 请求，返回请求体和头部信息。
- **/put**: 用于测试 PUT 请求。
- **/delete**: 用于测试 DELETE 请求。
- **/headers**: 显示当前请求的 HTTP 头部。
- **/status/:code**: 返回用户指定的状态码。
- **/response-headers**: 返回用户指定的一组响应头。
- **/redirect/:n**: 重定向到另一个端点 n 次。
- **/delay/:n**: 延迟 n 秒后响应请求。
- **/cookies**: 返回请求中的 cookies。
- **/cookies/set**: 设置 cookies 并重定向到 /cookies 显示设置的 cookies。
- **/basic-auth/:user/:passwd**: 测试基本认证。
- **/stream/:n**: 流式传输 n 个 JSON 响应。
- **/drip**: 以定制的方式 drip 数据。
- **/range/1024**: 返回数据的一部分，支持 Range 头部的请求。

---

此外，httpbin.org 还支持运行在本地环境中，可以通过 Docker 运行：
```bash
$ docker run -p 80:80 kennethreitz/httpbin
```

这些功能使得 `httpbin.org` 成为开发和测试 HTTP 客户端、库或服务时的有用工具。