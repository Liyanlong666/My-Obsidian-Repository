# HTTP 常见状态码有哪些？
**按状态码分类整理如下，每类列出最常被问到的核心状态码，并配一句“场景口令”帮助记忆。**

## 1xx 信息性（握手阶段的“收到！”）
| 代码 | 口令 | 含义与场景 |
| --- | --- | --- |
| 100 Continue | “继续灌” | 客户端先发送请求首部（含 `Expect: 100-continue`），收到 100 再发包体（RFC 7231），适用于大文件分块上传。 |
| 101 Switching Protocols | “换频道” | WebSocket 升级、HTTP/1.1 → HTTP/2 时常见。 |


---

## 2xx 成功（服务器“OK！”）
| 代码 | 口令 | 含义与典型用法 |
| --- | --- | --- |
| 200 OK | “一切安好” | 最常见；GET/POST 都可能返回。 |
| 201 Created | “新建完成” | POST `/users` 创建用户；响应中一般给 `Location` 头。 |
| 202 Accepted | “我收下先” | 异步任务排队，如上传转码。 |
| 204 No Content | “办完了，没料” | 删除成功、不需要返回体。 |
| 206 Partial Content | “分段寄” | 断点续传，`Range: bytes=...` |


---

## 3xx 重定向（服务器“去那边”）
| 代码 | 口令 | 含义与差异点 |
| --- | --- | --- |
| 301 Moved Permanently | “搬家永久” | 浏览器会缓存；SEO 友好。 |
| 302 Found | “临时搬家” | 老版浏览器照样改成 GET；早期最滥用。 |
| 303 See Other | “换 GET 拿” | POST 后重定向到 GET 资源（支付回跳常见）。 |
| 304 Not Modified | “缓存命中” | `If-None-Match` / `If-Modified-Since` 协商缓存。 |
| 307 Temporary Redirect | “临时搬家但保持方法” | 强制客户端使用原请求方法（如 POST 仍为 POST），与 302 的兼容性差异需注意旧代理行为。 |
| 308 Permanent Redirect | “永久搬家且保持方法” | 301 + 保留方法；HTTP/2 推广。 |


---

## 4xx 客户端错误（服务器“你错了”）
| 代码 | 口令 | 典型场景 |
| --- | --- | --- |
| 400 Bad Request | “报文烂了” | JSON 语法错、请求头过大等。 |
| 401 Unauthorized | “先登录” | 缺失/失效 Token；配合 `WWW-Authenticate`。 |
| 403 Forbidden | “我认得你，但不给” | 鉴权通过但无权限；IP 黑名单。 |
| 404 Not Found | “地址错了” | 经典“404 页面”。 |
| 405 Method Not Allowed | “动手方式错” | PUT 到只允许 GET 的 URL。 |
| 408 Request Timeout | “你太慢了” | 客户端未在时限内发完整请求。 |
| 409 Conflict | “版本冲突” | 编辑冲突、资源重复创建。 |
| 410 Gone | “永别了” | 资源永久删除，不会再有。 |
| 413 Payload Too Large | “包太大” | 上传超过限制。 |
| 415 Unsupported Media Type | “格式不懂” | `Content-Type` 不被接受。 |
| 429 Too Many Requests | “别刷了” | 限流/防刷必备。 |


---

## 5xx 服务端错误（服务器“我挂了”）
| 代码 | 口令 | 说明与排查方向 |
| --- | --- | --- |
| 500 Internal Server Error | “后台炸了” | 日志第一时间看 stack trace。 |
| 501 Not Implemented | “功能未上” | 服务器不支持当前方法。 |
| 502 Bad Gateway | “网关炸了” | 上游服务无响应（如超时、协议错误）或反向代理配置错误（如 DNS 解析失败）。 |
| 503 Service Unavailable | “临时停业” | 服务暂时不可用（如维护、限流），需通过 `Retry-After` 头告知客户端重试时间。 |
| 504 Gateway Timeout | “上游超时” | 反向代理等待后端 > timeout。 |
| 505 HTTP Version Not Supported | “版本太古” | 服务器不支持请求里的 HTTP 版本。 |


---

## 一张图背口诀
```markdown
1xx 我收到了；2xx 全搞定；  
3xx 去别处；4xx 你先改；  
5xx 我先修。
```

---

## 面试加分 Tips
1. **记住**：“401 要带 `WWW-Authenticate`，403 则不带”——很多人会混。
2. **当说到**：断点续传，要提 **206 + **`Range`**/**`Content-Range`。
3. **搭 API 时**：常用 **201 + **`Location` 返回新资源地址，胜过一股脑儿给 200。

---

希望这份整理对你理解、记忆和应对面试/工作中常见的 HTTP 状态码有所帮助！

