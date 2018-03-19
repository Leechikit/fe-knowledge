# HTTPS和HTTP

## HTTPS协议

HTTP报文是包裹在TCP报文中发送的，服务器端收到TCP报文时会解包提取出HTTP报文。但是这个过程中存在一定的风险，HTTP报文是明文，如果中间被截取的话会存在一些信息泄露的风险。那么在进入TCP报文之前对HTTP做一次加密就可以解决这个问题了。HTTPS协议的本质就是HTTP + SSL(or TLS)。在HTTP报文进入TCP报文之前，先使用SSL对HTTP报文进行加密。从网络的层级结构看它位于HTTP协议与TCP协议之间。

![](https://sfault-image.b0.upaiyun.com/359/891/3598916885-5608f6c220945_articlex)

HTTPS在传输数据之前需要客户端与服务器进行一个握手(TLS/SSL握手)，在握手过程中将确立双方加密传输数据的密码信息。

HTTPS相比于HTTP，虽然提供了安全保证，但是势必会带来一些时间上的损耗，如握手和加密等过程，是否使用HTTPS需要根据具体情况在安全和性能方面做出权衡。

## HTTP缺点

* 通信使用明文（不加密），内容可能会被窃听；
* 不验证通信方的身份，因此有可能遭遇伪装；
* 无法证明报文的完整性，所以有可能已遭到篡改。

## HTTP + 加密 + 认证 + 完整性保护 = HTTPS

![](https://user-gold-cdn.xitu.io/2018/3/13/1621e18a22a107c7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

1. 客户端通过发送Client Hello报文开始SSL通信，报文中包含客户端支持的SSL的指定版本、加密组件（Cipher Suite）列表（所使用的加密算法及密钥长度等）；
2. 服务器可进行SSL通信时，会以Server Hello报文作为应答，和客户端一样，在报文中包含SSL版本及加密组件。服务器的加密组件内容是从接收到的客户端加密组件内筛选出来的；
3. 之后服务器发送Certificate报文，报文中包含公开密钥证书；
4. 最后服务器发送Server Hello Done报文通知客户端，最初阶段的SSL握手协商部分结束；
5. SSL第一次握手结束后，客户端以Client Key Exchange报文作为应答，报文中包含通信加密中使用的一种被称为Pre-mastersecret的随机密码串，该报文已用步骤3中的公开密钥进行加密；
6. 接着客户端继续发送Change Cipher Spec报文，该报文会提示服务器，在此报文之后的通信会采用Pre-master secret密钥进行加密；
7. 客户端发送Finished报文，该报文包含链接至今全部报文的整体校验值，这次握手协商是否能够成功，要以服务器是否能够正确解密该报文作为判定标准；
8. 服务器同样发送Change Cipher Spec报文；
9. 服务器同样发送Finished报文；
10. 服务器和客户端的Finshed报文交换完毕后，SSL连接就算建立完成。当然，通信会受到SSL的保护，从此处开始进行应用层协议的通信，即发送HTTP请求；
11. 应用层协议通信，即发送HTTP响应；
12. 最后由客户端断开连接，断开连接时，发送close_notify报文，上图做了一些省略，这步之后再发送TCP FIN报文来关闭与TCP的通信；