# 请求头

# 响应头

## Location

```bash
##Location (指示客户端应该跳转到的新的URL)
```

常见场景：

1. 重定向（3xx）

当服务器返回 301、302、303、307、308 等状态码时，Location 告诉浏览器或客户端，资源已经移动，请访问这个新地址。

例如：

```http
HTTP/1.1 302 Found
Location: https://example.com/new-page
```

浏览器会自动跳转到：`https://example.com/new-page`

2. 资源创建成功（201 Created）

当客户端发送 POST 创建了一个新资源，服务器返回 201 Created 时，Location 通常指向新创建资源的 URL。

例如：

```http
HTTP/1.1 201 Created
Location: /users/123
```

这表示新用户资源可以通过 /users/123 访问。

3. 其他 3xx 场景

303 See Other ：常表示 POST 后重定向到结果页。

307 / 308：要求保持原请求方法重定向。

300 Mutiple Choices：可能列出多个可选位置。

简单说：location 就是服务器告诉客户端“接下来去这个地址”。他的具体行为由状态码决定，客户端（浏览器、HTTP库等）通常会按规范自动处理。

## Server

服务器用来处理这个请求的软件名称和版本信息。生产环境下建议隐藏或伪造；

```bash
Server: nginx/1.24.0
Server: Apache/2.4.58 (Ubuntu)
Server: cloudflare
Server: Microsoft-IIS/10.0
Server: BigIP （F5 Networks 的 BIG-IP 设备）
```

# http通用头

## Connection

常见的两个值：

1、Connection:：keep-alive

- 表示这次请求处理完后，tcp连接先不要关，后续请求可以复用这条链接；

- 在 HTTP/1.0 里要显式写：HTTP / 1.0 默认就是长连接，所以通常不用

写，但很多服务器仍会回一个 keep-alive 表示确认。

- 好处：省掉反复建立 TCP 或 TLS 连接的开销，性能更好。

2、Connection：close

表示这次请求处理完后，直接关闭连接不复用。

常见于：服务器要下线，连接数打满，客户端明确不想复用、或出现错误时。











