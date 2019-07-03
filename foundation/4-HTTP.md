# HTTP协议类

问题：

- HTTP协议的主要特点
- HTTP报文的组成部分
- HTTP方法
- POST和GET的区别
- HTTP状态码
- 什么是持久连接
- 什么是管线化

------

### HTTP协议的主要特点

- 简单快速
- 灵活
- 无连接
- 无状态

### HTTP报文的组成部分

### HTTP方法

| 常用方法 | 作用         |
| -------- | ------------ |
| GET      | 获取资源     |
| POST     | 传输资源     |
| PUT      | 更新资源     |
| DELETE   | 删除资源     |
| HEAD     | 获得报文首部 |



- `GET、POST、PUT、DELETE、PATCH、HEAD、OPTIONS、TRACE `
- 请求报文：请求行 + 请求头 + 空行 + 请求体

![](https://github.com/zhuoooo/markdown/blob/master/images/request.png?raw=true)

- 响应报文：状态行 + 响应头 + 空行 + 响应体

![](https://github.com/zhuoooo/markdown/blob/master/images/response.png?raw=true)

### POST和GET的区别：

- **GET在浏览器回退时是无害的，而POST会再次提交请求**
- GET产生的URL地址可以被收藏，而POST不可以
- **GET请求会被浏览器主动缓存，而POST不会，除非手动设置**
- GET请求只能进行URL编码，而POST支持多种编码方式
- **GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留**
- **GET请求在URL中传送的参数是有长度限制的，而POST没有限制**
- 对参数的数据类型，GET只接受ASCII字符，而POST没有限制
- GET比POST不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息
- **GET参数通过URL传递，POST放在Request body中**

### HTTP状态码

- 1xx：指示信息 - 表示请求已接收，继续处理
- 2xx：成功 - 表示请求已被成功接收
- 3xx：重定向 - 要完成请求必须进行更近一步的操作
- 4xx：客户端错误 - 请求有语法错误或请求无法实现
- 5xx：服务器错误 - 服务器未能实现合法的请求
- 一些常见的状态码：
  1. 200 OK：客户端请求成功
  2. 206 Partial Content：客户端发送了一个带有Range头的GET请求，服务器完成了它
  3. 301 Moved Permanently：所请求的页面已经转移至新的URL（永久）
  4. 302 Found：所请求的页面已经临时转移至新的URL
  5. 304 Not Modified：客户端有缓冲的文档并发出了一个条件性的请求，服务器告诉客户，原来缓冲的文档还可以继续使用
  6. 400 Bad Request：客户端请求有语法错误，不能被服务器所理解
  7. 401 Unauthorized：请求未经授权，这个状态码必须和WWW-Authenticate报头域一起使用
  8. 403 Forbidden：对被请求页面的访问被禁止
  9. 404 Not Found：请求资源不存在
  10. 500 Internal Server Error：服务器发生不可预期的错误，原来的缓冲文档还可以继续使用
  11. 503 Server Unavailable：请求未完成，服务器临时过载或宕机，一段时间后可能恢复正常

### 什么是持久连接

- HTTP协议采用“请求-应答”模式，当使用普通模式，即非Keep-Alive模式时，每个请求/应答客户和服务器都要新建一个连接，完成之后立即断开连接（HTTP协议为无连接的协议）
- 当使用Keep-Alive模式（又称持久连接、连接重用）时，Keep-Alive功能使客户端到服务器端的连接持久有效，当出现对服务器的后继请求时，Keep-Alive功能避免了建立或者重新建立连接
- 该功能是在HTTP 1.1之后才支持的

### 什么是管线化

- 在持久连接的情况下，某个连接上消息的传递类似于

  请求1 -> 响应1 -> 请求2 -> 响应2 -> 请求3 -> 响应3

- 某个连接上的消息变成类似于

  请求1 -> 请求2 -> 请求3 -> 响应1 -> 响应2 -> 响应3 

- 特点：

  1. 管线化机制通过持久连接完成，仅HTTP/1.1支持此技术
  2. 只有GET和HEAD请求可以进行管线化，而POST则有所限制
  3. 初次创建连接时不应启动管线化机制，因为对方（服务器）不一定支持HTTP/1.1 版本的协议