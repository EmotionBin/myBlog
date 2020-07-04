# 关于https

## http协议

浏览器和服务器之间通信是通过`HTTP`协议，`HTTP`协议是`Hyper Text Transfer Protocol`（超文本传输协议），它是基于`TCP/IP`通信协议来传递数据的，目前比较常用的是`HTTP/1.1`，`HTTP`协议永远都是客户端发起请求，服务器回送响应。  

![Y9QYAU.png](https://s1.ax1x.com/2020/05/04/Y9QYAU.png)

----

### 三次握手

三次握手是指建立一个 `TCP` 连接时，需要客户端和服务器总共发送3个包。  

![Yi8JB9.png](https://s1.ax1x.com/2020/05/05/Yi8JB9.png)

**1. 第一次握手([SYN], Seq = x)**

- 客户端发送一个`SYN`标记的包，`Seq`初始序列号x，发送完成后客户端进入`SYN_SEND`状态。  

**2. 第二次握手([SYN,ACK], Seq = y, ACK = x + 1)**

- 服务器返回确认包(`ACK`)应答，同时还要发送一个`SYN`包回去。`ACK = x + 1`,表示确认收到(客户端发来的`Seq`值 + 1)，`Seq = y`, 表示让客户端确认是否能收到。发送完成后服务端进入`SYN_RCVD`状态。  

**3. 第三次握手([ACK], ACK = y + 1)**

- 客户端再次发送确认包(`ACK`),`ACK = y + 1`, 表示确认收到服务器的包（服务端发来的`Seq`值 + 1）。客户端发送完毕后，进入`ESTABLISHED`状态，服务端接收到这个包，也进入`ESTABLISHED`状态, `TCP`握手结束。  

可以用一种形象的方式来解释:  
 
 - 女朋友向程序员提出约会(Seq = x)的建议，然后女朋友进入`SYN_SEND`状态。
 - 程序员收到后同意了去约会(ACK = x + 1), 然后向女朋友建议去吃麻辣烫吧(Seq = y)，程序员进入`SYN_RCVD`状态。
 - 女朋友收到建议后，勉为其难的答应了，然后告诉程序员说 那好吧(ACK = y + 1)。女朋友就进入了`ESTABLISHED`状态， 程序员也进入了 `ESTABLISHED`状态，整个约会讨论结束。

---

### 四次挥手

`TCP`连接的断开需要发送四个包，所以称为四次挥手。  

![YFS79K.png](https://s1.ax1x.com/2020/05/05/YFS79K.png)

**1. 第一次挥手（[FIN], Seq = x）**

- 客户端发送一个FIN标记的包，告诉服务器需要关闭连接，表示自己不用发送数据了，但是还可以接收数据。发送完成后，客户端进入`FIN_WAIT_1`状态。

**2. 第二次挥手 ([ACK], ACK = x + 1)**

- 服务端发送一个`ACK`的确认包，告诉客户端接收到关闭的请求，但是还没有准备好。发送完成后，服务端进入`CLOSE_WAIT`状态，客户端收到这个包后，进入`FIN_WAIT_2`，等待服务器关闭连接。

**3. 第三次挥手 ([FIN], Seq = y)**

- 服务端准备好关闭连接时，发送`FIN`标记的包，告诉客户端准备关闭了。发送完成后，服务端进入`LAST_ACK`状态，等待客户端确认。

**4. 第四次挥手 ([ACK], ACK = y + 1)**

- 客户端接收到服务端的关闭请求，再发送`ACK`标记的确认包，进入`TIME_WAIT`状态，等待服务端可能请求重传的`ACK`包。
- 服务端接收到`ACK`包后，关闭连接，进入`CLOSED`状态。
- 客户端在等待固定时间(两个最大段生命周期)后，没有接收到服务的`ACK`包，认为服务器已关闭连接，自己也关闭连接，进入`CLOSED`状态。

四次挥手连接看起来挺复杂，其实认真看看也挺简单的，同样的比喻:  

- 女朋友向程序员提出分手
- 程序员告诉女朋友说，我知道了，但是要考虑一下
- 程序员考虑清楚后，跟女朋友说那就分手吧
- 女朋友接收到程序员的消息后，然后还在等程序员发挽留的消息，然而等了两天后没等到，就认为程序员是真的不会再发消息来了，于是就拉黑删除程序员，关闭连接了

虽然传统的`HTTP`协议解决了客户端和服务器通信的问题，但是`HTTP`协议在传输数据的时候，是以明文的形式来传输数据的，这就导致了在需要向服务端传输一些敏感的数据时，非常的不安全。例如，把用户的密码传输给服务器时，密码是明文的，谁都可以看到，这样还叫密码吗？所以，为了安全起见，就提出了在传输数据的时候加密传输。  

----

## 对称加密

对称加密其实就是同一个密钥可以同时用作信息的加密和解密。废话不多说，来看个例子，假设现在有两个算法f1和f2，f1用于加密数据，f2用于解密数据，下图会模拟使用对称加密的方式进行客户端与服务器的通信。  

![Y9UvJx.png](https://s1.ax1x.com/2020/05/04/Y9UvJx.png)

1. 客户端想要给服务器发送数据data，这次不再是明文传输，而是使用了对称加密算法f1将数据data加密为x，即`f1(k，data) = x`，将x传输给服务器
2. 服务器受到数据x后，使用对称加密算法f2进行解密，即`f2(k，x) = data`
3. 服务器想要回给客服端data1，也使用了对称加密算法f1对数据data1进行加密，即`f1(k，data1) = y`，将y传输给客户端
4. 客户端收到数据y后，使用对称加密算法f2解密数据y，即`f2(k，y) = data1`

整个通信过程完成，数据不再以明文的形式传输，实现了加密通信。但是这里的秘钥k只能有一个，为什么秘钥k只能有一个，因为服务器去定制秘钥k的时候，它不可能知道有多少个客户端，不可能为每个客户端都量身定制一个秘钥k，如果有一亿个客户端呢，**只有一个秘钥k就导致了每个人都知道这个秘钥k，这样一来，加密了等于没加密**，所以只使用对称加密也是不可取的。  

----

## 非对称加密

非对称加密会有两个秘钥，一个是公钥一个是私钥，公钥与私钥是一对的，用公钥对数据进行加密，只有用对应的私钥才能解密，用私钥对数据进行加密，只有用对应的公钥才能解密。再来看个例子，假设算法f是非对称加密算法，`pk`是公钥，`sk`是私钥，`pk`和`sk`都存储在服务器中。  

![Y90GtA.png](https://s1.ax1x.com/2020/05/04/Y90GtA.png)

1. 客户端向服务端发送请求获取公钥`pk`
2. 服务器收到客服端请求，把`pk`给客户端
3. 客户端用公钥`pk`加密想要发送的数据data，即`f(pk,data) = y`，将数据y传送给服务器
4. 服务器收到数据y，用私钥`sk`解密y，即`f(sk,y) = data`，获取到客户端发送的数据
5. 服务器要给客户端回信息data1，用私钥`sk`进行加密，即`f(sk,data1) = y1`，将y1回送给客户端
6. 客户端收到信息y1后，用公钥`pk`进行解密，即`f(pk,y1) = data1`

整个通信过程完成，使用了非对称加密，这种通信方式有没有漏洞呢，其实也是有的，同样的，`pk`和`sk`也只有一对，都存储在服务器，客户端向服务器索要公钥`pk`的时候，服务器并不知道客户端是谁，是好人还是坏人，所以每个人都可以拿到公钥`pk`，拿到`pk`后，客户端在发送数据的时候，经过了公钥`pk`加密，只要私钥`sk`不泄露，那么就是安全的，但是在服务器回信息的时候，使用的是私钥`sk`进行数据加密，前面说了每个人都可以拿到公钥`pk`，那么**服务器回信息的时候，每个人都可以用拿到的公钥pk对信息进行解密**，所以这个过程在服务器回信息的时候是不安全的，只使用非对称加密也是不可取的。  

----

## 对称加密 + 非对称加密

如果只使用对称加密或者只使用非对称加密，都是不安全的，那么可不可以把两者都结合在一起，既使用对称加密又使用非对称加密。再来看个例子，假设现在有对称加密算法F，非对称加密算法f，公钥`pk`和私钥`sk`一对，都存储在服务器。  

![Y9qsL8.png](https://s1.ax1x.com/2020/05/04/Y9qsL8.png)

1. 客户端向服务端发送请求获取公钥`pk`
2. 服务器收到客服端请求，把`pk`给客户端
3. 客户端生成随机字符串num1，用公钥pk进行非对称加密后为y，把y传输给服务器
4. 服务器收到数据y，用私钥`sk`进行解密，得到字符串num1，此时服务器回应ok，这里的ok是抽象表示，就是告诉客户端我收到了你发过来的数据。**从此开始，客户端与服务器后来的通信，都会用这个字符串num1作为秘钥进行对称加密**
5. 客户端将要发送的数据data以num1作为秘钥进行对称加密，即`F(num1，data) = x`，把x传输给服务器
6. 服务器收到数据x后进行解密，即`F(num1，x) = data`，拿到数据data
7. 服务器要给客户端回信息data1,将data1进行加密，即`F(num1,data1) = x1`，把x1传输给客户端
8. 客户端收到信息x1，将x1解密，即`F(num1,x1) = data1`，拿到数据data1

这个完整的通信过程结合了对称加密与非对称加密，看起来天衣无缝的通讯方式是不是已经很安全了呢，再看完下面这个例子就会明白。  

![Y9L1Yj.png](https://s1.ax1x.com/2020/05/04/Y9L1Yj.png)

在客户端向服务器发起请求的过程中，假设黑客直接介入，黑客也有自己的公钥`pk'`和私钥`sk'`。  

1. 客户端向服务端发送请求获取公钥`pk`，黑客介入，黑客很贼，它假装自己是一个无辜的良民，向服务器发送请求获取公钥`pk`
2. 服务器收到请求，把`pk`返回，黑客又继续介入，成功拿到服务器公钥`pk`，将自己的公钥`pk'`给客户端，客户端拿到的是假的公钥`pk'`，但是它并不知道
3. 客户端生成随机字符串num1，用假公钥`pk'`进行非对称加密后为y'，把y'传输给服务器，传输的过程中，黑客又介入，黑客用自己的私钥`sk'`解密y'，成功拿到num1，之后用公钥`pk`加密num1为数据y，把y发送给服务器
4. 服务器收到信息后，返回ok，黑客收到服务器的ok，也告诉客户端ok，从此用num1进行对称加密通信
5. 每次客户端和服务器的通信过程都会被黑客监听、窃取，这个过程一直持续。。。

看完这整个通信过程可以发现，每次客户端和服务器通信，都必须经过黑客，黑客在一开始就把真正的公钥`pk`掉包了，自己拿到了服务器真正的公钥`pk`，而返回给客户端假的公钥`pk'`，可怕的是客户端和服务器都无法察觉。从此开始，客户端发送的信息黑客都可以解密、篡改，并将篡改后的数据发送给服务器，这就是**中间人攻击**，很显然，这种通信方式也是非常不安全的。  

这里根据以上的加密方式小结一下：  

- 明文 = 裸奔
- 对称加密，key唯一 = 明文
- 非对称加密，Server -> Client 不安全
- 对称加密 + 非对称加密，中间人攻击

那么有没有一些比较安全的加密策略呢，答案是有的，请继续清下看。  

----

## 对称加密 + 非对称加密 + CA

`CA`是一个第三方的认证机构，是具有公信力的、权威的机构，可以被大家所信任的，那么就可以借助`CA`来协助我们完成安全的通信。先分析一下中间人攻击的原因所在，客户端在向服务器通信的第一步，也就是在索要公钥的时候就出现了问题，公钥被黑客掉包了，那么如何解决这个问题呢，答案就是利用`CA证书`，`CA`机构他也有自己的公钥和私钥，假设`CA`的公钥为`Cpk`，私钥为`Csk`，继续看下面这个例子  

![YCP3Qg.png](https://s1.ax1x.com/2020/05/04/YCP3Qg.png)

客户端首次向服务器通信的时候不再直接请求公钥，而是请求数字证书，数字证书是由`CA`机构用`Csk`对`pk`加密生成的，存放在服务器，只有用`Cpk`解密证书才能获取公钥`pk`，假如客户端去向`CA`机构请求`Cpk`的时候，运气很不好，又遇到黑客劫持掉包，那么干脆就直接把`Cpk`写死在浏览器，就避免了掉包，所以浏览器会存储一些`CA`机构的信息，存储了大量`CA`机构的公钥，这些信息由浏览器自己来维护。上图的通信过程大致如下  

1. 客户端向服务器请求证书`licence`
2. 服务器将证书返回给客户端
3. 客户端拿到证书后用`Cpk`解密获取公钥`pk`
4. 以下通信过程同上，不再叙述。。。

采用这种通信方式的时候，假设中间人介入，当服务器返回证书`licence`给客户端时，中间人截获，它篡改了证书，把篡改后的证书给了客户端，这时候客户端收到证书，但是这时候它并不知道证书的真假，此时它可以用自己本身存储的`CA`机构的公钥`Cpk`去解密证书，如果证书是真的，就可以解密成功，假如证书是假的，解密就会出现问题，这时候浏览器就会提示不安全，如下图  

![YPXDLq.png](https://s1.ax1x.com/2020/05/05/YPXDLq.png)

出现上图的情况可能有以下两种：

1. 黑客篡改了证书
2. 该网站使用的是自签名的证书，或者由不受信任的认证机构提供的签名
3. 网站的主机名与证书的主机名称不匹配
4. 证书过期了

关于自签名证书，其实就是自己制造的一个数字证书，因为我们自己的证书不属于浏览器认可的权威机构的范围内，所以浏览器也会发出安全警告。  

话说回来，上面分析了中间人篡改证书的方式几乎是不可行的，那么中间人在客户端拿到证书之后再进行介入呢。来分析一下，假设现在客户端已经拿到了数字证书，并用`CA`公钥`Cpk`解密拿到了公钥`pk`，开始生成随机数num1，并用`pk`加密成数据y传输给服务器，这个时候就算中间人介入，他最多只能拿到公钥`pk`，所以他并不能解密数据y，所以中间人无法操作。  

所以，使用 `对称加密 + 非对称加密 + CA机构` 这种通信方式是相对较安全的，这也是我们常说的`HTTPS`，但是不排除有更牛逼的黑客~

----

## 数字签名

数字签名顾名思义，就是电子签名，好比你写了一篇文章，然后在上面附上你的电子签名。  

假设有A和B两人，A有两把钥匙，公钥和私钥，A事先把公钥给了B。数字签名具体实现过程如下：  

1. A写了一封信内容是data，他写完后用hash函数生成该信件的摘要`digest`
2. A将摘要`digest`用自己的私钥进行加密，生成数字签名`signature`，A将这个数字签名`signature`附在信件data下面，一起发送给B
3. B收到信息，取下数字签名`signature`，用A的公钥解密得到摘要`digest`，证明这封信是A写的
4. B再对信件本身使用hash函数，将得到的结果和上一步中得到的摘要`digest`进行对比，如果两者一致就证明这份信件未被修改过

数字签名好比现实生活中的签名，数字证书好比我们的身份证。使用数字签名是有可能被中间人截获的，所以建议还是使用数字证书比较安全。

----

## 访问百度的过程

C代表Client客户端，S代表Server服务器

1. C -> S，客户端请求服务器，告诉服务器客户端所支持的SSL版本、非对称算法，并带上随机数num1
2. S -> C，服务器给客户端回信，告诉客户端就使用SSL1.0版本，以及以后通信的对称加密算法，数字证书，并带上随机数num2
3. C验证证书，如验证不通过，浏览器发出警告，证书不安全；验证通过则继续往下
4. C -> S，将前面的num1和num2使用hash算法`hash(num1,num2)`得到数据，假设是xx，同时带上随机数num3
5. S验证xx，S也是知道num1和num2的，拿到数据后先判断`hash(num1,num2)`是否等于xx，不等于则说明出现问题，等于则验证通过，继续往下进行，此时S将前面的num1，num2，num3通过hash算法`hash（num1，num2，num3）`生成秘钥k，存储在自己身上
6. S -> C，S将前面的num1，num2，xx使用hash算法`hash(num1，num2，xx)`得到数据，假设是zz，将zz返回给客户端
7. C收到数据，C也是知道num1，num2，xx的，C也使用hash算法`hash(num1，num2，xx)`，验证是否等于zz，如果不等于说明出现问题，如果等于说明安全，此时C通过hash算法`hash（num1，num2，num3）`生成秘钥k，也存储在自己身上，这个k和第5步生成的k是一样的
8. 如果以上过程都没有出现问题，则客户端和服务器可以正常通信，客户端和服务器都拿到了秘钥k，这个秘钥k都是他们自己生成的，而且都是一样的，避免了在传输过程中秘钥的泄漏，之后的通信都会用到这个秘钥k，这就保证了安全性

这个过程确实是挺复杂的，但是为了安全，这样是值得的而且非常必要的~

----

## 结束语

客户端和浏览器的通信是通过`HTTP`协议进行的，`HTTP`有一个弊端，就是所有传输的信息都是明文的，安全性几乎为0，所以就引出了`HTTPS`进行数据加密，`HTTPS`一般会用到一些加密算法和`CA证书`（常用的证书如SSL，TLS），从而保证数据传输的安全。如果本文中有说的不正确的地方，欢迎大佬鞭策!

**参考资料：**

[简书-面试必备HTTP之TCP三次握手及四次挥手详解](https://www.jianshu.com/p/12790cea57ac)  
[阮一峰-数字签名是什么？](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)  
[b站-HTTPS原理全解析](https://www.bilibili.com/video/BV1w4411m7GL)  