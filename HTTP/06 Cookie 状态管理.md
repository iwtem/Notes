## Cookie 的状态管理

HTTP 是无状态协议，它不对之前发生过的请求和响应的状态进行管理。也就是说，无法根据之前的状态进行本次的请求处理。

假设要求登录认证的 Web 页面本身无法进行状态的管理（不记录已登录的状态），那么每次跳转新页面就要再次登录，或者要在每次请求报文中附加参数来管理登录状态。

无状态协议不必保存状态，自然可减少服务器的CPU及内存资源的消耗。但是为了解决保存状态的问题，于是引入了 **Cookie** 技术。Cookie技术通过在请求和响应报文中写入Cookie信息来控制客户端的状态。

Cookie 会根据从服务器端发送的响应报文内的一个叫做 Set-Cookie 的首部字段信息，通知客户端保存 Cookie。当下次客户端再往该服务器发送请求时，客户端会自动在请求报文中加入 Cookie 值后发送出去。

服务器端发现客户端发送过来的 Cookie 后，会去检查究竟是从哪一个客户端发来的连接请求，然后对比服务器上的记录，最后得到之前的状态信息。



**第1次，没有Cookie信息状态下的请求**

![无Cookie的请求](./images/no-cookie.jpeg)

**第2次，以后（存有Cookie信息状态）的请求**

![有Cookie的请求](./images/has-cookie.jpeg)

**Cookie 交互的场景，请求报文与响应报文**

```
①请求报文（没有Cookie信息的状态）
GET /reader/ HTTP/1.1
Host: hackr.jp
＊首部字段内没有Cookie的相关信息

②响应报文（服务器端生成Cookie信息）
HTTP/1.1200 OK
Date: Thu, 12 Jul 2012 07:12:20 GMT
Server: Apache
＜Set-Cookie: sid=1342077140226724; path=/; expires=Wed, =&gt;10-Oct-12 07:12:20 GMT＞
Content-Type: text/plain; charset=UTF-8

③请求报文（自动发送保存着的Cookie信息）
GET /image/ HTTP/1.1
Host: hackr.jp
Cookie: sid=1342077140226724    
```

