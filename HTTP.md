### 1 HTTP - inter

> + HTTP协议构建在TCP/IP协议之上，是应用层协议，默认端口80
>
> + HTTP无连接无状态
>

#### 1.1 请求报文

> HTTP 请求分为三个部分：状态行、请求头、消息主体

```http
method request-URL version
request headers

entity body
```

##### GET

> GET用于信息获取，且应该是安全和幂等（对同一URL多个请求返回相同结果）

```http
 GET /books/?sex=man&name=Professional HTTP/1.1
 Host: www.example.com
 User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)
 Gecko/20050225 Firefox/1.0.1
 Connection: Keep-Alive
```



##### POST

> HTTP 协议中规定 POST 提交的数据必须在 body 部分中，但是协议中没有规定数据使用哪种编码方式或者数据格式，数据发送出去还要服务端解析才有意义
>
> **服务端通常根据请求头（headers）中Content-Type字段来获知请求的消息主题是用何种编码方式的**



+ application/x-www-form-urlencoded（表单提交默认方式）

```http
 POST /index.html HTTP/1.1
 Host: www.example.com
 User-Agent: Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.7.6)
 Gecko/20050225 Firefox/1.0.1
 Content-Type: application/x-www-form-urlencoded
 Content-Length: 40
 Connection: Keep-Alive

 sex=man&name=Professional 
```

+ multipart/form-data

    > 使用表单上传文件时，表单enctype必须为multipart/form-data
    >
    > boudary用于分隔不同字段，每部分都以--boundary开始，紧接着是内容描述信息，然后是回车，最后是字段具体内容（文本或二进制），如果是文件还包含文件名和文件类型

```http
POST http://www.example.com HTTP/1.1
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA

------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="text"

title
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="file"; filename="chrome.png"
Content-Type: image/png

PNG ... content of chrome.png ...
------WebKitFormBoundaryrGKCBY7qhFd3TrwA--
```



##### Tips

> - GET 可提交的数据量受到URL长度的限制，HTTP 协议规范没有对 URL 长度进行限制。这个限制是特定的浏览器及服务器对它的限制
> - 理论上讲，POST 是没有大小限制的，HTTP 协议规范也没有进行大小限制，出于安全考虑，服务器软件在实现时会做一定限制（比如Nginx限制2M）
> - 参考上面的报文示例，可以发现 GET 和 POST 数据内容是一模一样的，只是位置不同，一个在 URL 里，一个在 HTTP 包的包体里
>
> 
>
> 随着越来越多的 Web 站点，尤其是 WebApp，全部使用 Ajax 进行数据交互之后，我们完全可以定义新的数据提交方式，例如 `application/json`，`text/xml`，乃至 `application/x-protobuf` 这种二进制格式，只要服务器可以根据 `Content-Type` 和 `Content-Encoding` 正确地解析出请求，都是没有问题的。



#### 1.2 响应报文

> 1. 状态行
> 2. 响应头
> 3. 响应体
>
> 
>
> 常见的状态码有如下几种：
>
> - `200 OK` 客户端请求成功
> - `301 Moved Permanently` 请求永久重定向
> - `302 Moved Temporarily` 请求临时重定向
> - `304 Not Modified` 文件未修改，可以直接使用缓存的文件。
> - `400 Bad Request` 由于客户端请求有语法错误，不能被服务器所理解。
> - `401 Unauthorized` 请求未经授权。这个状态代码必须和WWW-Authenticate报头域一起使用
> - `403 Forbidden` 服务器收到请求，但是拒绝提供服务。服务器通常会在响应正文中给出不提供服务的原因
> - `404 Not Found` 请求的资源不存在，例如，输入了错误的URL
> - `500 Internal Server Error` 服务器发生不可预期的错误，导致无法完成客户端的请求。
> - `503 Service Unavailable` 服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常。

``` http
HTTP/1.1 200 OK

headers

response body
```

```http
HTTP/1.1 200 OK

Server:Apache Tomcat/5.0.12
Date:Mon,6Oct2003 13:23:42 GMT
Content-Length:112

<html>...
```



#### 1.3 条件GET

> 条件GET是HTTP协议为减少不必要的带宽浪费提出的一种方案
>
> 1. 客户端已经访问过某页面，并打算再次访问该页面
> 2. 客户端询问自上次访问网站时间后是否更改了页面，如果服务器无更新则使用本地缓存即可，如果服务器显示已更新则发送更新給用户

客户端第一次请求，会返回`If-Modified-Since`，根据该字段判断是否更新

```http
 GET / HTTP/1.1  
 Host: www.sina.com.cn:80  
 If-Modified-Since:Thu, 4 Feb 2010 20:39:13 GMT  
 Connection: Close  
```

如果无更新，服务器返回`304 Not Modified`，浏览器可使用上次获取的文件（缓存）

```http
HTTP/1.0 304 Not Modified  
Date: Thu, 04 Feb 2010 12:38:41 GMT  
Content-Type: text/html  
Expires: Thu, 04 Feb 2010 12:39:41 GMT  
Last-Modified: Thu, 04 Feb 2010 12:29:04 GMT  
Age: 28  
X-Cache: HIT from sy32-21.sina.com.cn  
Connection: close 
```



#### 1.4 持久连接

> 非 Keep-Alive 模式时，每个请求/应答客户和服务器都要新建一个连接，完成之后立即断开连接（HTTP 协议为无连接的协议）；当使用 Keep-Alive 模式（又称持久连接、连接重用）时，Keep-Alive 功能使客户端到服务器端的连接持续有效，当出现对服务器的后继请求时，Keep-Alive 功能避免了建立或者重新建立连接。
>
> 如果客户端浏览器支持 Keep-Alive ，那么就在HTTP请求头中添加一个字段 Connection: Keep-Alive，这样客户端和服务器之间的HTTP连接就会被保持，不会断开（超过 Keep-Alive 规定的时间，意外断电等情况除外）
>
> 在 HTTP 1.1 版本中，默认情况下所有连接都被保持，如果加入 "Connection: close" 才关闭。目前大部分浏览器都使用 HTTP 1.1 协议，也就是说默认都会发起 Keep-Alive 的连接请求
>
> 
>
> + HTTP 长连接不可能一直保持，例如 `Keep-Alive: timeout=5, max=100`，表示这个TCP通道可以保持5秒，max=100，表示这个长连接最多接收100次请求就断开。
> + 长连接如何判断传输结束？
>     1. 判断传输数据是否达到了Content-Length 指示的大小；
>     2. 动态生成的文件没有 Content-Length ，它是分块传输（chunked），这时候就要根据 chunked 编码的数据在最后有一个空 chunked 块，表明本次传输数据结束



##### Transfer-Encoding

> 目前只有一种取值：chunked
>
> 如果HTTP消息（请求或应答消息）Tranfer-Encoding为`chunked`，那么消息体由数量未定的块组成，并以最后一个大小为0的块为结束。

```http
HTTP/1.1 200 OK
Content-Type: text/plain
Transfer-Encoding: chunked

25
This is the data in the first chunk

1A
and this is the second one
0
```

> tips:
>
> chunked 的优势在于，服务器端可以边生成内容边发送，无需事先生成全部的内容。HTTP/2 不支持 Transfer-Encoding: chunked，因为 HTTP/2 有自己的 streaming 传输方式（Source：[MDN - Transfer-Encoding](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Transfer-Encoding)）。



#### 1.5 HTTP Pipelining

> **HTTP管线化**
>
> 默认情况下 HTTP 协议中每个传输层连接只能承载一个 HTTP 请求和响应，浏览器会在收到上一个请求的响应之后，再发送下一个请求。在使用持久连接的情况下，某个连接上消息的传递类似于`请求1 -> 响应1 -> 请求2 -> 响应2 -> 请求3 -> 响应3`。
>
> HTTP Pipelining（管线化）是将多个 HTTP 请求整批提交的技术，在传送过程中不需等待服务端的回应。使用 HTTP Pipelining 技术之后，某个连接上的消息变成了类似这样`请求1 -> 请求2 -> 请求3 -> 响应1 -> 响应2 -> 响应3`。
>
> 注意以下几点：
>
> - 管线化机制通过持久连接（persistent connection）完成，仅 HTTP/1.1 支持（HTTP/1.0不支持）
> - 只有 GET 和 HEAD 请求可以进行管线化，而 POST 则有所限制
> - 初次创建连接时不应启动管线机制，因为对方（服务器）不一定支持 HTTP/1.1 版本的协议
> - 管线化不会影响响应到来的顺序，如上面的例子所示，响应返回的顺序并未改变
> - HTTP /1.1 要求服务器端支持管线化，但并不要求服务器端也对响应进行管线化处理，只是要求对于管线化的请求不失败即可
> - 由于上面提到的服务器端问题，开启管线化很可能并不会带来大幅度的性能提升，而且很多服务器端和代理程序对管线化的支持并不好，因此==现代浏览器如 Chrome 和 Firefox 默认并未开启管线化支持==



#### 1.6 会话追踪

什么是会话？

客户端打开与服务器的连接发出请求到服务器响应客户端请求的全过程称之为会话。

为什么需要会话跟踪？

HTTP协议是“无状态协议”，无法保存客户的信息，这样每次请求都需要判断是否同一个用户，所以需要会话跟踪

> 会话跟踪常用方法：
>
> 1. URL重写，在URL结尾添加附加数据标识该会话以识别不同用户
>
> 2. 隐藏表单域，将会话ID以隐藏元素提交到服务器
>
> 3. Cookie
>
>     Cookie是Web服务器发送给客户端的一小段信息，在客户端可以保存，客户端请求时可读取该信息发送到服务器。
>
>     Cookie可以被客户端禁用
>
> 4. Session
>
>     服务端创建一个session对象，产生一个sessionID用于标识不同用户
>
>     session依赖Cookie，如果Cookie被禁用那么session也会失效



#### 1.7 跨站攻击

> 1. CSRF（Cross-site request forgery，跨站请求伪造）
>
> **==只要协议，主机，端口其中有一个不相同即是跨站请求==**
>
> `http://www.baidu.com:80`
>
> `https://www.baidu.com:80`
>
> 如果是跨站请求，客户端会先发送OPTIONS方法去请求服务器是否允许跨站请求，如果允许才发送具体请求，这里会请求两次。
>
> 如何防止：
>
> + 关键操作只接受POST
>
> + Token（随机性，一次性）
>
> 
>
> 2. XSS（Cross Site Scripting，跨站脚本攻击）
>
>     如何防止：**对用户输入的HTML进行转义**





### 3 summarize

1. http版本
    Http于1990年问世，那时的HTTP并没有作为正式的标准被建立。这时的HTTP其实含有HTTP/
    1.0之前版本的意思，因此被称为HTTP/0.9。

HTTP正式作为标准被公布是在1996年的5月，版本被命名为HTTP/1.0，并记载于RFC1945。

1997年1月公布的HTTP/1.1是目前主流的HTTP协议版本。

HTTP2.0主要基于SPDY协定。它由互联网工程任务组（IETF）的Hypertext Transfer Protocol Bis（httpbis）工作小组进行开发。[2]该组织于2014年12月将HTTP/2标准提议递交至IESG进行讨论[3]，于2015年2月17日被批准。

2. http各个版本之间的区别
    HTTP协议的出现主要是为了解决文本传输的难题。

HTTP是一种不保存状态，即无状态（stateless）协议。HTTP协议自身不对请求和响应之间的通信状态进行保存。也就是说在HTTP这个级别，协议对于发送过的请求或响应都不做持久化处理。

HTTP/1.1虽然是无状态协议，但为了实现期望的保持状态功能，于是引入了Cookie技术。有了Cookie再用HTTP协议通信，就可以管理状态了。



HTTP主要有这些不足：

+ 通信使用明文（不加密），内容可能会被窃听；
+ 不验证通信方的身份，因此有可能遭遇伪装；
+ 无法证明报文的完整性，所以有可能已遭破坏。

Https就是为了解决这些问题而存在。

==**HTTP + 加密 + 认证  + 完整性保护 = HTTPS。**==

HTTP标准的瓶颈：

+ 一条连接上只可发送一个请求。
+ 请求只能从客户端开始。客户端不可以接收除响应以外的指令。
+ 请求/响应首部未经压缩就发送。首部信息越多延迟越大。
+ 发送冗长的首部。每次互相发送相同的首部造成的浪费较多。
+ 可任意选择数据压缩格式。非强制压缩发送

为了解决这些问题就出现了SPDY协议。Http2.0基于SPDY协议实现的。

3. ==http协议格式==
    ==请求报文：请求报头+空行+实体==
    ==响应报文：响应报头+空行+实体==
    ==请求报头：请求行+报头（请求首部+通用首部+实体首部）==
    ==响应报头：响应状态行+报头（响应首部+通用首部+实体首部）==
    ==报头首部类型：请求首部、响应首部、通用首部、实体首部==



4. 代理、网关、隧道的区别

代理是黄牛，倒买倒卖，手上沾点油。不改动，服务端一定使用的是Http。比如一般的CDN服务器。

网关是一个人工服务员，对外提供统一的服务，客户不用与各种内部接口人一一对接。网关的上游服务器可能使用的是ftp、mail等非http协议。比如三大电影运营商。

隧道也是一个秘密的暗道，其他人没法窥探里面的内容。比如shadowsocket

5. CA认证机构
    https协议会使用非对称加密，但非对称加密中公开密钥存在的身份认证问题，即怎么证明我是我的问题。这个时候就出现了第三者：CA认证机构。所有需要使用数字签名的网站，就把身份认证的事情交给CA来完成。

那么CA认证机构也出现一个尴尬的问题。怎么证明自己是自己的问题。你CA是假的怎么办？那么这个时候的解决办法就是将所有顶级的CA的公钥在浏览器里面都内置一份。对的，每个浏览器在出厂的时候，都会存一些CA认证机构的照片，这样就假不了了。

6. 关于SSL
    SSL技术最初是由浏览器开发商网景通信公司率先倡导的，开发过SSL3.0之前的版本。目前主导权已转移到IETF（Internet Engineering Task Force,Internet工程任务组）的手中。
    IETF以SSL3.0为基准，后又制定了TLS1.0、TLS1.1和TLS1.2。TSL是以SSL为原型开发的协议，有时会统一称该协议为SSL。当前主流的版本是SSL3.0和TLS1.0

SSL的慢分两种。一种是指通信慢。另一种是指由于大量消耗CPU及内存等资源，导致处理速度变慢。

和使用HTTP相比，网络负载可能会变慢2到100倍



7. 服务器端考虑的几个问题
    怎么省流量？怎么减少机器资源的消耗？怎么保证缓存一致性？怎么保证可靠性？怎么保证拓展性，保证各个业务分离？简言之，快、稳、省、准。

8. 关于Web完全
    There is a door，there is a way！
    有和外部交互的地方，那么就存在安全问题的可能。只要外部能够把数据送到你服务端，那么就存在安全风险的可能。具体就可能是怎么把数据送进去的问题了。

有的可能是通过在客户端入手，把私货夹带进入；而有的是路上动手，中间人劫持；还有人是直接在服务端动手，破解你root权限；发一份钓鱼邮件，先送到你内网再说，你点开了，那么这个雷就埋好了。

当然上述都是主动劫持，还有被动劫持。那么就是对用户下手。

发一发钓鱼网站，添加XSS跨域攻击，把用户的信息在不知不觉中上传到黑客的网站上。





### 4 Note

#### 1.2 HTTP诞生

HTTP/0.9。

HTTP/1.0 HTTP正式作为标准被公布是在1996年的5月，版本被命名为HTTP/1.0，并记载于RFC1945。虽说是初期标准，但该协议标准至今仍被广泛使用在服务器端。

HTTP/1.1 1997年1月公布的HTTP/1.1是目前主流的HTTP协议版本



#### 1.3 TCP/IP
TCP/IP协议簇中存在各式各样的内容：

从电缆的规格到IP地址的选定方法、寻找异地用户的方法、双方建立通信的顺序，以及Web页面显示需要处理的步骤，等等。 
发送端在层与层之间传输数据时，每经过一层时必定会被打上一个该层所属的首部信息。反之，接收端在层与层传输数据时，每经过一层时会把对应的首部消去。

#### 1.4 IP、TCP和DNS

IP、TCP和DNS与HTTP关系密切

IP协议的作用是把各种数据包传送给对方。而要保证确实传送到对方那里，则需要满足各类条件。其中两个重要的条件是IP地址和MAC地址（Media Access Control Address）。 IP地址指明了节点被分配到的地址，MAC地址是指网卡所属的固定地址。

使用ARP协议凭借MAC地址进行通信 IP间的通信依赖MAC地址。

TCP协议为了更容易传送大数据才把数据分割，而且TCP协议能够确认数据最终是否送达到对方。

三次握手：==发送端首先发送一个带SYN标志的数据包给对方。接收端收到后，回传一个带有SYN + ACK标志的数据包以示传达确认信息。最后，发送端再回传一个带ACK标志的数据包，代表“握手”结束。==

#### 1.7 URI和URL
Identifier 表示可标识的对象。也称为标识符。 综上所述，URI就是由某个协议方案表示的资源的定位标识符。

协议方案是指访问资源所使用的协议类型名称。

 采用HTTP协议时，协议方案就是http。除此之外，还有ftp、mailto、telnet、file等。
URI用字符串标识某一互联网资源，而URL表示资源的地点（互联网上所处的位置）。可见URL是URI的子集。
使用绝对URI必须指定待访问的服务器地址。

地址可以是类似hackr.jp这种DNS可解析的名称，或是192.168.1.1这类IPv4地址名，还可以是[0:0:0:0:0:0:0:1]这样用方括号括起来的IPv6地址名。

==一些用来制定HTTP协议技术标准的文档，它们被称为RFC（Request for Comments，征求修正意见书）==

#### 2.2 请求与响应

通过请求和响应的交换达成通信

请求报文是由请求方法、请求URI、协议版本、可选的请求首部字段和内容实体构成的。
响应报文基本上由协议版本、状态码（表示请求成功或失败的数字代码）、用以解释状态码的原因短语、可选的响应首部字段以及实体主体构成。

#### 2.3 无状态协议

HTTP是一种不保存状态，即无状态（stateless）协议。HTTP协议自身不对请求和响应之间的通信状态进行保存。

#### 2.5 HTTP方法

告知服务器意图的HTTP方法

| 方法    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| GET     | 请求指定的资源，并返回响应主体                               |
| HEAD    | 类似于GET，但不返回响应主体，用于获取报头                    |
| POST    | 向指定资源提交数据（如表单提交、文件上传），请求数据包含在请求体中，POST可能导致新在资源创建 / 已有资源修改（`非幂等`） |
| PUT     | 从客户端向服务器传送数据取代指定的文档的内容                 |
| DELETE  | 请求服务器删除指定的资源                                     |
| CONNECT | HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器       |
| OPTIONS | 允许客户端查看服务器的性能                                   |
| TRACE   | 回显服务器收到的请求，用于测试与诊断                         |
| PATCH   | 是对PUT的补充，用来对已有资源进行局部更新                    |



#### 2.6 常见状态码

> 200 - 请求成功
>
> 301 - 资源被永久转移到其它URI
>
> 404 - 请求的资源不存在
>
> 500 - 服务器错误
>
>  
>
> 1xx：信息。服务器收到请求，需要请求者继续执行操作
>
> 2xx：成功。请求被成功处理
>
> 3xx：重定向。需要进一步的操作以完成请求
>
> 4xx：客户端错误。请求包含语法错误或无法完成请求
>
> 5xx：服务器错误。服务器在处理请求过程中发生了错误





#### 2.7 持久连接
HTTP1.1默认持久连接，特点是只要任意一端没有明确提出断开连接，则保持TCP连接状态。

``` properties
# 显式断开连接
Connect: close
```

管线化技术出现后，不用等待响应亦可直接发送下一个请求。 这样就能够做到同时并行发送多个请求，而不需要一个接一个地等待响应了。

#### 2.8 Cookie

使用Cookie的状态管理

Cookie会根据从服务器端发送的响应报文内的一个叫做Set-Cookie的首部字段信息，通知客户端保存Cookie。当下次客户端再往该服务器发送请求时，客户端会自动在请求报文中加入Cookie值后发送出去。

#### 3.1 HTTP报文
用于HTTP协议交互的信息被称为HTTP报文。

请求端（客户端）的HTTP报文叫做请求报文，响应端（服务器端）的叫做响应报文。

HTTP报文本身是由多行（用CR+LF作换行符）数据构成的字符串文本

#### 3.2 报文结构

请求报文及响应报文的结构

请求行 包含用于请求的方法，请求URI和HTTP版本。 

状态行 包含表明响应结果的状态码，原因短语和HTTP版本。

 首部字段 包含表示请求和响应的各种条件和属性的各类首部。 

一般有4种首部，分别是：通用首部、请求首部、响应首部和实体首部。 其他可能包含HTTP的RFC里未定义的首部（Cookie等）。

#### 3.3 编码

编码提升传输速率

+ gzip 由文件压缩程序gzip（GNU zip）生成的编码格式（RFC1952），采用Lempel-Ziv算法（LZ77）及32位循环冗余校验（Cyclic Redundancy Check，CRC）。

+ compress 由UNIX文件压缩程序compress生成的编码格式，采用Lempel-Ziv-Welch算法（LZW）。
+ deflate 组合使用zlib格式（RFC1950）及由deflate压缩算法（RFC1951）生成的编码格式
+ identity 不执行压缩或不会变化的默认编码格式

通常，报文主体等于实体主体。只有当传输中进行编码操作时，实体主体的内容发生变化，才导致它和报文主体产生差异。

#### 3.4 多部分数据

 发送多种数据的多部分对象集合，multipart-formdata

MIME（Multipurpose Internet Mail Extensions，多用途因特网邮件扩展）机制
，HTTP协议中也采纳了多部分对象集合，发送的一份报文主体内可含有多类型实体。

==通常是在图片或文本文件等上传时使用。==

#### 3.5 范围请求

获取部分内容的范围请求

如果下载过程中遇到网络中断的情况，那就必须重头开始。为了解决上述问题，需要一种可恢复的机制。所谓恢复是指能从之前下载中断处恢复下载。 要实现该功能需要指定下载的实体范围。像这样，指定范围发送的请求叫做范围请求（Range Request）。
针对范围请求，响应会返回状态码为206 Partial Content的响应报文。另外，对于多重范围的范围请求，响应会在首部字段Content-Type标明multipart/byteranges后返回响应报文。 如果服务器端无法响应范围请求，则会返回状态码200 OK和完整的实体内容

#### 4.2 2XX成功

该状态码表示客户端进行了范围请求，而服务器成功执行了这部分的GET请求。响应报文中包含由Content-Range指定范围的实体内容。

#### 4.3 3XX重定向
301永久性重定向。

该状态码表示请求的资源已被分配了新的URI，以后应使用资源现在所指的URI
和301 Moved Permanently状态码相似，但302状态码代表的资源不是被永久移动，只是临时性质的
当301、302、303响应状态码返回时，几乎所有的浏览器都会把POST改成GET，并删除请求报文内的主体，之后请求会自动再次发送。

 301、302标准是禁止将POST方法改变成GET方法的，但实际使用时大家都会这么做。
304 Not Modified（服务器端资源未改变，可直接使用客户端未过期的缓存）

#### 4.4 4XX客户端错误
该状态码表示请求报文中存在语法错误。当错误发生时，需修改请求的内容后再次发送请求。
该状态码表示发送的请求需要有通过HTTP认证（BASIC认证、DIGEST认证）的认证信息
该状态码表明对请求资源的访问被服务器拒绝了。服务器端没有必要给出拒绝的详细理由，但如果想作说明的话，可以在实体的主体部分对原因进行描述，这样就能让用户看到了。 未获得文件系统的访问授权，访问权限出现某些问题（从未授权的发送源IP地址试图访问）等列举的情况都可能是发生403的原因。

#### 4.5 5XX服务器错误
该状态码表明服务器端在执行请求时发生了错误。也有可能是Web应用存在的bug或某些临时的故障。
该状态码表明服务器暂时处于超负载或正在进行停机维护，现在无法处理请求。

#### 5.2 代理、网关、隧道

通信数据转发程序

代理：接收客户端发送的请求后转发给其他服务器。代理不改变请求URI，会直接发送给前方持有资源的目标服务器。 持有资源实体的服务器被称为源服务器。从源服务器返回的响应经过代理服务器后再传给客户端。

每次通过代理服务器转发请求或响应时，会追加写入Via首部信息

使用代理服务器的理由有：

+ 利用缓存技术（稍后讲解）减少网络带宽的流量
+ 网站的访问控制（负载均衡）
+ 获取访问日志，等等。



 ==网关 ：网关可以将HTTP协议转化成其他协议，传送给目标服务器处理，同时可以将非http协议的响应结果再转化成http协议==，利用网关可以由HTTP请求转化为其他协议通信




使用Connect属性可开启隧道，隧道使用ssl，tsl加密传输

#### 5.3 缓存
缓存是指代理服务器或客户端本地磁盘内保存的资源副本。利用缓存可减少对源服务器的访问，因此也就节省了通信流量和通信时间。 

缓存服务器是代理服务器的一种，并归类在缓存代理类型中
浏览器缓存如果有效，就不必再向服务器请求相同的资源了，可以直接从本地磁盘内读取。 另外，和缓存服务器相同的一点是，当判定缓存过期后，会向源服务器确认资源的有效性。若判断浏览器缓存失效，浏览器会再次请求新资源。



#### 6.2 HTTP首部字段
使用首部字段是为了给浏览器和服务器提供报文主体大小、所使用的语言、认证信息等内容。
当HTTP报文首部中出现了两个或两个以上具有相同首部字段名时会怎么样？这种情况在规范内尚未明确，根据浏览器内部处理逻辑的不同，结果可能并不一致。有些浏览器会优先处理第一次出现的首部字段，而有些则会优先处理最后出现的首部字段。
在HTTP协议通信交互中使用到的首部字段，不限于RFC2616中定义的47种首部字段。还有Cookie、Set-Cookie和Content-Disposition等在其他RFC中定义的首部字段，它们的使用频率也很高
HTTP首部字段将定义成缓存代理和非缓存代理的行为

#### 6.3 HTTP/1.1 通用首部字段
Cache-Control: only-if-cached 使用only-if-cached指令表示客户端仅在缓存服务器本地缓存目标资源的情况下才会要求其返回。换言之，该指令要求缓存服务器不重新加载响应，也不会再次确认资源有效性。若发生请求缓存服务器的本地缓存无响应，则返回状态码504 Gateway Timeout。
HTTP/1.1版本的默认连接都是持久连接。为此，客户端会在持久连接上连续发送请求。当服务器端想明确断开连接时，则指定Connection首部字段的值为Close。
首部字段Via不仅用于追踪报文的转发，还可避免请求回环的发生。所以必须在经过代理时附加该首部字段内容

#### 6.4 请求首部字段
文本文件 text/html, text/plain, text/css ... application/xhtml+xml, application/xml ... 

●图片文件 image/jpeg, image/gif, image/png ...

●视频文件 video/mpeg, video/quicktime ...

●应用程序使用的二进制文件 application/octet-stream, application/zip ...


形如If-xxx这种样式的请求首部字段，都可称为条件请求。服务器接收到附带条件的请求后，只有判断指定条件为真时，才会执行请求。

通过TRACE方法或OPTIONS方法，发送包含首部字段Max-Forwards的请求时，该字段以十进制整数形式指定可经过的服务器最大数目。服务器在往下一个服务器转发请求之前，会将Max-Forwards的值减1后重新赋值。当服务器接收到Max-Forwards值为0的请求时，则不再进行转发，而是直接返回响应。
接收到附带Range首部字段请求的服务器，会在处理请求之后返回状态码为206 Partial Content的响应。无法处理该范围请求时，则会返回状态码200 OK的响应及全部资源。

#### 6.5 响应首部字段

字段ETag能告知客户端实体标识。它是一种可将资源以字符串形式做唯一性标识的方式。服务器会为每份资源分配对应的ETag值。

==使用首部字段Location可以将响应接收方引导至某个与请求URI位置不同的资源。==

``` js
Location: http://www.baidu.com
```

首部字段Retry-After告知客户端应该在多久之后再次发送请求。主要配合状态码503 Service Unavailable响应，或3xx Redirect响应一起使用。

当代理服务器接收到带有Vary首部字段指定获取资源的请求时，如果使用的Accept-Language字段的值相同，那么就直接从缓存返回响应。反之，则需要先从源服务器端获取资源后才能作为响应返回

首部字段WWW-Authenticate用于HTTP访问认证。它会告知客户端适用于访问请求URI所指定资源的认证方案（Basic或是Digest）和带参数提示的质询（challenge）。状态码401 Unauthorized响应中，肯定带有首部字段WWW-Authenticate。



#### 6.6 实体首部字段

``` js
Allow: GET, HEAD
# 实体的编码方式
Content-Encoding: gzip
Content-Language: zh-CN
# 资源大小15000字节
Content-Length: 15000
Content-Location: http://www.baidu.com/index.htm
# 一致性校验
Content-MD5: 0GHkZDUwN...
# 客户端范围请求响应
Content-Range
# 实体对象的媒体类型
Content-Type: text/html;charset=UTF-8
# 资源失效期
Expires: Wed, 04 Jul 2012 ...
Last-Modified: Wed, ...
```



#### 6.7 Cookie

Cookie标准：

+ Netscape
+ RFC2109
+ RFC2965（Set-Cookie2；Cookie2）
+ RFC6265

将网景公司制定的标准作为业界事实标准（De facto standard），重新定义Cookie标准后的产物。

目前使用最广泛的Cookie标准却不是RFC中定义的任何一个。而是在网景公司制定的标准上进行扩展后的产物。

``` js
Set-Cookie: status=enable; expires=...;path=/;domain=.hackr.jp;
Cookie: status=enable
```



#### 7.1 加密

HTTP的隐患：

+ 无法确定请求发送至目标的Web服务器是否是按真实意图返回响应的那台服务器。有可能是已伪装的Web服务器。
+ 无法确定响应返回到的客户端是否是按真实意图接收响应的那个客户端。有可能是已伪装的客户端。
+ 无法确定正在通信的对方是否具备访问权限。因为某些Web服务器上保存着重要的信息，只想发给特定用户通信的权限
+ 无法判定请求是来自何方、出自谁手。
+  即使是无意义的请求也会照单全收。无法阻止海量请求下的DoS攻击（Denial of Service，拒绝服务攻击）。



HTTP协议中没有加密机制，但可以通过和SSL（Secure Socket Layer，安全套接层）或TLS（Transport LayerSecurity，安全传输层协议）的组合使用，加密HTTP的通信内容。

用SSL建立安全通信线路之后，就可以在这条线路上进行HTTP通信了。==与SSL组合使用的HTTP被称为HTTPS（HTTP Secure，超文本传输安全协议）**HTTP over SSL**==

==SSL不仅提供加密处理，而且还使用了一种被称为证书的手段，可用于确定方。==



#### 7.2 HTTPS

==**HTTP+加密+认证+完整性保护=HTTPS**==

**HTTPS并非是应用层的一种新协议**。只是HTTP通信接口部分用SSL（Secure Socket Layer，安全套接层）和TLS（Transport Layer Security，安全传输层协议）协议代替而已。

通常，HTTP直接和TCP通信。当使用SSL时，则演变成先和SSL通信，再由SSL和TCP通信了。简言之，所谓HTTPS，其实就是身披SSL协议这层外壳的HTTP。



**SSL更新到3.0时，IETF（互联网任务工程组）对SSL3.0进行标准化（添加少数机制，几乎与SSL3.0无异），标准化后更名为TLS1.0（Transport Layer Security），可以说TLS是SSL3.1**

SSL是独立于HTTP的协议，所以不只是HTTP协议，==其他运行在应用层的SMTP和Telnet等协议均可配合SSL协议使用==

| HTTP | HTTP |
| ---- | ---- |
| TCP  | SSL  |
| IP   | TCP  |
|      | IP   |



#### 7.3 非对称加密

公开密钥加密使用一对非对称的密钥。

一把叫做私有密钥（private key），另一把叫做公开密钥（public key）。顾名思义，私有密钥不能让其他任何人知道，而公开密钥则可以随意发布，任何人都可以获得。

发送密文的一方使用对方的公钥进行加密，对方收到被加密的信息后用自己的私钥解密。

==加密通常采用大整数的模幂运算（单向函数），那么解密过程就是对离散对数求值（如果能快速的对大整数因式分解，那么破解还存在希望，但目前不行）==

``` shell
对称加密：（请考虑颜色混合示例）
共享数字  B
Client： Y
Server： X
Client 计算 B的Y次方 模 M 发送給 Server
Server 计算 B的X次方 模 M 发送給 Client
p1 = Server加上自己的X：(B的Y次方 模 M)的X次方
p2 = Client加上自己的Y：（B的X次方 模 M）的Y次方
p1 == p2

非对称加密：RSA
...
```



#### 7.4 自签名证书

公钥证书也可叫做数字证书或直接称为证书



如果使用OpenSSL这套开源程序，每个人都可以构建一套属于自己的认证机构，从而自己给自己颁发服务器证书。但该服务器证书在互联网上不可作为证书使用。独立构建的认证机构叫做自认证机构，由自认证机构颁发的“无用”证书也被戏称为自签名证书。浏览器访问该服务器时，会显示“无法确认连接安全性”或“该网站的安全证书存在问题”等警告消息。



#### 8.4 HTTP认证方式

+ BASIC（基本认证）
+ DIGEST（摘要认证）
+ SSL客户端认证
+ FormBase（基于表单认证）



#### 11.5 彩虹表

由明文密码与之对应的散列值构成的数据表，在穷举破解过程中缩短时间



### 3 状态码

|      | 类别                             | 描述                   |
| ---- | -------------------------------- | ---------------------- |
| 1XX  | Informational（信息性状态码）    | 请求正在处理           |
| 2XX  | Success（成功状态码）            | 请求正常处理完毕       |
| 3XX  | Redirection（重定向状态码）      | 需要附加操作以完成请求 |
| 4XX  | Client Error（客户端错误状态码） | 服务器无法处理请求     |
| 5XX  | Server Error（服务端错误状态码） | 服务器处理请求错误     |

+ 200 OK： 正常处理

+ 204 No Content：处理成功，但没有资源可返回

+ 206 Partial Content：客户端进行了范围请求，服务器成功执行这部分的GET请求。响应报文中包含由Content-Range指定范围的实体内容。



### 4 SSL3.0

\>> SSL加密技术是为保护敏感数据在传送过程中的安全，而设置的加密技术。
SSL（Security Socket Layer）是Netscape公司所提出的安全保密协议，在浏览器（如Internet Explorer、Netscape Navigator）和Web服务器（如Netscape的Netscape Enterprise Server、ColdFusion Server等等）之间构造安全通道来进行数据传输，SSL运行在TCP/IP层之上、应用层之下，为应用程序提供加密数据通道，它采用了RC4、MD5以及RSA等加密算法，使用40 位的密钥，适用于商业信息的加密。
SSL加密并不保护数据中心本身，而是确保了SSL加密设备的数据中心安全，可以监控企业中来往于数据中心的最终用户流量。
注:Netscape公司开发了HTTPS协议并内置于其浏览器中，HTTPS实际上就是HTTP over SSL，它使用默认端口443，而不是像HTTP那样使用端口80来和TCP/IP进行通信。HTTPS协议使用SSL在发送方把原始数据进行加密，然后在接受方进行解密，加密和解密需要发送方和接受方通过交换共知的密钥来实现，从而确保所传送的数据不会轻易被网络黑客截获和解密。但是，需要注意的是，加密和解密过程需要耗费系统大量的开销，严重降低机器的性能（相关测试数据表明使用HTTPS协议传输数据的工作效率只有使用HTTP协议传输的十分之一。）
SSL数据加密的应用:SSL 数字证书。根据可信强度，SSL数字证书可以分为以下几种：
域名型 SSL 证书（DVSSL）
企业型 SSL 证书（OVSSL）
增强型 SSL 证书（EVSSL）
企业 PKI 管理



#### 9.2 SPDY加快网站速度

speedy是HTTP/2.0的前身，开发组全程参与HTTP/2.0制定过程

google开发的基于TCP的会话层协议，以最小化网络延迟提升网络速度

不是一种新协议， 而是在HTTP之前做了一层会话层，HTTP/2.0基于SPDY设计，是SPDY的升级版

**HTTP/2.0：HTTP —> SPDY—> TLS—>TCP—>IP**

**HTTP/1.0：HTTP	—>	TCP	—>	IP**

+ 多路复用流（单一TCP连接无限制处理多个HTTP请求）
+ 请求优先级
+ header压缩（HTTP首部）
+ 推送功能（支持服务器主动向客户端推送数据）
+ 服务器提示功能（服务器主动提示客户端请求所需资源）



#### 9.3 WebSocket

全双工通信协议，API由W3C定标准。

客户端与服务器通信过程中，可互相发送JSON，XML，HTML或图片等任意格式数据。

+ 推送功能
+ 通信量减少（首部小）
+ 握手请求（为实现WebSocket，需使用HTTP的首部Upgrade字段，告知服务器通信协议改变）

``` properties
# 请求行
GET /chat HTTP/1.1
Host: server.com
# 通信协议改变
Upgrade: websocket
Connection: Upgrade
# 握手过程必要的键值
Sec-WebSocket-Key: dGh1IHN...
# 子协议，定义连接的名称
Sec-WebSocket-Protocol: chat, superchat
```

+ 握手响应

``` properties
# 对握手响应，返回状态码101 Switching Protocols
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: ...
Sec-WebSocket-Protocol: chat
```

**成功握手后建立WebSocket连接，通信不再使用HTTP的数据帧，而采用WebSocket独立的数据帧**

``` js
# WebSocket API
let socket = new WebSocket('ws://game.example.com:12010/updates');
# 每10ms发送一次数据
socket.open = function() {
    setInterval(() => {
        if (socket.bufferedAmount == 0)
            socket.send(getUpdateData());
    }, 10)
}
```



#### 9.4 HTTP/2.0

HTTP/2.0围绕7个技术讨论

| 压缩                     | SPDY、Friendly              |
| ------------------------ | --------------------------- |
| 多路复用                 | SPDY                        |
| TLS义务化                | Sppedy + Mobility           |
| Client Pull，Server Push | Speedy + Mobility           |
| 流量控制                 | SPDY                        |
| WebSocket                | Speedy + Mobility           |
| 协商                     | Speedy + Mobility，Friendly |



#### 10.3 CGI

CGI（Common Gateway Interface，通用网关接口）：

​	==指Web服务器在接收到客户端发送的请求后转发给程序的一组机制，==在CGI作用下，程序会对请求内容做出相应动作，比如创建HTML等动态内容

​	使用CGI的程序通常是Perl、PHP、Ruby，C

实现[维基百科](https://zh.wikipedia.org/wiki/維基百科)编辑的CGI程序的一个例子：首先用户代理程序向这个CGI程序请求某个名称的条目，如果该条目页面存在，CGI程序就会去获取那个条目页面的原始数据，然后把它转换成[HTML](https://zh.wikipedia.org/wiki/HTML)并把结果输出给浏览器；如果该条目页面不存在，CGI程序则会提示用户新建一个页面。所有维基操作都是通过这个CGI程序来处理的。



#### 10.4 Servlet

轻量服务程序

Servlet常驻内存，Servlet运行环境叫Servlet容器 / Web容器