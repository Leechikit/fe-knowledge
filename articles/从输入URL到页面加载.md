# 从输入URL到页面加载发生什么

### 1. 浏览器接受URL

URL包含的信息：协议、网络地址:端口号、资源路径、查询字符串？、片段标识符#

### 2. 浏览器查找当前URL是否存在缓存，并比较缓存是否过期。

如果请求的页面在缓存中且未过期，则直接进行第8步。

缓存分为彻底缓存和缓存协商，这里的确认是否过期是指彻底缓存（缓存失效之前不再需要跟服务器交互）。

<details>
<summary>DETAIL</summary>

#### 彻底缓存的机制（http首部字段）：cache-control，Expires

资源http响应状态码在不强刷的情况下为200(from cache)

* Expires：是一个绝对时间，即服务器时间。浏览器检查当前时间，如果还没到失效时间就直接使用缓存文件。但是该方法存在一个问题：服务器时间与客户端时间可能不一致。因此该字段已经很少使用，现在基本用cache-control进行判断。
* cache-control：
	- max-age：cache-control中的max-age保存一个相对时间。例如Cache-Control: max-age = 484200，表示浏览器收到文件后，缓存在484200s内均有效。 如果同时存在cache-control和Expires，浏览器总是优先使用cache-control。
	- no-cache：使用缓存前必须和服务器进行确认，也就是需要发起请求。
	- no-store：不缓存

#### 缓存协商。对应http首部字段：last-modified，Etag

当缓存过期时，浏览器会向服务器发起请求询问资源是否真正过期，这就是缓存协商。

资源http响应状态码在不强刷的情况下为304

* last-modified：是第一次请求资源时，服务器返回的字段，表示最后一次更新的时间。下一次浏览器请求资源时就发送if-modified-since字段。服务器用本地Last-modified时间与if-modified-since时间比较，如果不一致则认为缓存已过期并返回新资源给浏览器；如果时间一致则发送304状态码，让浏览器继续使用缓存。当然，用该方法也存在问题，比如修改时间有变化但实际内容没有变化，而服务器却再次将资源发送给浏览器。所以，使用Etag进行判断更好。
* Etag：资源的实体标识（哈希字符串），当资源内容更新时，Etag会改变。服务器会判断Etag是否发生变化，如果变化则返回新资源，否则返回304。

> 缓存协商的过程需要发起一起HTTP请求，如果返回304则继续使用缓存。对于移动端一次请求还是有代价的，所以我们需要避免304。

> 对于很少进行更改的静态文件，可以在文件名中加入版本号，如get.v1.js，并且把Cache-Control的max-age设置成一年半载，这样就不会发送请求。

> 需要注意的是，当这些文件更新的时候，我们需要更新其版本号，这样浏览器才会到服务器下载新资源。

![](https://upload-images.jianshu.io/upload_images/3160413-fb75e5af66606680.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/692)
</details>

### 3. DNS解析域名

如果网络地址不是一个 IP 地址，通过DNS解析域名返回一个IP地址

<details>
<summary>DETAIL</summary>

DNS数据库是域名和IP地址相互映射的一个分布式数据库，DNS协议用来将域名转换为IP地址，它运行在UDP协议之上。UDP无需连接，时效性更好。

#### DNS查询

* 操作系统会先检查本地hosts文件是否有这个网址映射关系，如果有就调用这个IP地址映射，完成域名解析。
* 否则，查找本地DNS解析器缓存，如果查找到则返回。
* 否则，查找本地DNS服务器，如果查找到则返回。
* 否则，1）未用转发模式，按根域服务器 ->顶级域,.com->第二层域，example.com ->子域，www.example.com的顺序找到IP地址。2）用转发模式，按上一级DNS服务器->上上级->....逐级向上查询找到IP地址。

</details>

### 4. 浏览器与服务器通过三次握手建立TCP连接

<details>
<summary>DETAIL</summary>

![](https://upload-images.jianshu.io/upload_images/3160413-5a9f596e6bde10c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

#### 为什么需要进行三次握手，而不是两次握手？

原因是两次握手不可靠。比如，浏览器发送一个连接请求包A，但包A在半路上堵车了，浏览器就认为包A丢失了，所以重新发生一个请求包B给服务器。服务器收到请求，建立连接。两端进行通信，结束后关闭连接。但是这时候，包A到达了服务器，服务器不知道这是一个无效的包，所以进行响应。这时两次握手已经完成，两端就建立起一个无效的连接。但浏览器认为自己没发出请求，所以不会回应，这样就让服务器白白等待回应，浪费了服务器资源。而三次握手的机制下，浏览器知道自己并没有请求连接，会发送拒绝包给服务器，服务器收到回应后也会结束这次无效的连接。

#### TCP/IP协议族里可分为应用层、传输层、网络层和数据层

##### 应用层

应用层决定了向用户提供应用服务时通信的活动。TCP/IP协议族内预存了各类通用的应用服务。比如FTP（File Transfer Protocol，文件传输协议）和DNS（Domain Name Sysytem，域名系统），HTTP协议也处于该层。

##### 传输层

传输层对上层应用层，提供处于网络连接中的两台计算机之间的数据传输。在传输层有两个性质不同的协议：TCP（Transmission Control Protocol，传输控制协议）和UDP（User Data Protocol，用户数据报协议）。

##### 网络层

网络层用来处理在网络上流动的数据包。数据包是网络传输的最小数据单位，该层规定了通过怎样的路径（所谓的传输路线）到达对方计算机，比把数据包传送给对方。

##### 链路层（又名数据链路层，网络接口层）

用来处理链接网络的硬件部分。包括控制操作系统、硬件的设备驱动、NIC（Network Interface Card，网络适配器，即网卡），及光纤等物理可见的部分。

</details>

### 5. 浏览器向服务器发送HTTP请求。

完整的HTTP请求包含请求起始行、请求头部、请求主体三部分。

数据经过应用层、传输层、网络层、物理层逐层封装，传输到下一个目的地。

<details>
<summary>DETAIL</summary>

![](https://upload-images.jianshu.io/upload_images/5695795-50fb4ff95551f317.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

#### 发送HTTP大体流程

1. 客户端在应用层（HTTP协议）发出一个想看某个Web页面的HTTP请求；
2. 为了传输方便，在传输层（TCP协议）把从应用层收到的数据（HTTP请求报文）进行分割，并在各个报文上打上标记序号及端口号后转发给网络层；
3. 在网络层（IP协议，Internet Protocol），增加作为通信目的地的MAC地址后转发给链路层。这样一来，发往网络的通信请求就准备齐全了；
4. 接收端的服务器在链路层收到数据，按序往上层发送，一直到应用层。当传输到应用层，才能算真正接收到由客户端发送过来的HTTP请求。

![](https://user-gold-cdn.xitu.io/2018/3/13/1621d3cfd3474953)

</details>

### 6. 服务器收到请求，从它的文档空间中查找资源并返回HTTP响应。

服务器在收到浏览器发送的HTTP请求之后，会将收到的HTTP报文封装成HTTP的Request对象，并通过不同的Web服务器进行处理，处理完的结果以HTTP的Response对象返回，主要包括状态码，响应头，响应报文三个部分。

### 7. 浏览器接受 HTTP 响应，检查 HTTP header 里的状态码，并做出不同的处理方式。

<details>
<summary>DETAIL</summary>

状态码主要包括以下部分
* 1xx：指示信息–表示请求已接收，继续处理。
* 2xx：成功–表示请求已被成功接收、理解、接受。
* 3xx：重定向–要完成请求必须进行更进一步的操作。
* 4xx：客户端错误–请求有语法错误或请求无法实现。
* 5xx：服务器端错误–服务器未能实现合法的请求。

>200 OK (from cache)  是浏览器没有跟服务器确认，直接用了浏览器缓存；
>
>304 Not Modified 是浏览器和服务器多确认了一次缓存有效性，再用的缓存。

响应头主要由Cache-Control、 Connection、Date、Pragma等组成。

响应体为服务器返回给浏览器的信息，主要由HTML，css，js，图片文件组成。

</details>

### 8. 如果是可以缓存的，这个响应则会被存储起来。

根据首部字段判断是否进行缓存。例如，Cache-Control, no-cache(每次使用缓存前和服务器确认)，no-store 绝对禁止缓存

### 9. 解码

解析HTML，查看[浏览器渲染流程](https://github.com/Leechikit/fe-knowledge/issues/9)

### 10. 渲染

查看[浏览器渲染流程](https://github.com/Leechikit/fe-knowledge/issues/9)

### 11. 关闭TCP连接或继续保持连接

通过四次挥手关闭连接(FIN ACK, ACK, FIN ACK, ACK)。

<details>
<summary>DETAIL</summary>

![](https://upload-images.jianshu.io/upload_images/3160413-1d8c843335a5f419.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

为什么需要进行四次挥手？

第一次挥手是浏览器发完数据后，发送FIN请求断开连接。第二次挥手是服务器发送ACK表示同意，如果在这一次服务器也发送FIN请求断开连接似乎也没有不妥，但考虑到服务器可能还有数据要发送，所以服务器发送FIN应该放在第三次挥手中。这样浏览器需要返回ACK表示同意，也就是第四次挥手。

简而言之，一端断开连接需要两次挥手（请求和回应），两端断开连接就需要四次挥手了。

</details>