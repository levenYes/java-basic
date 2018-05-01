# 深入Web请求过程
如何发起一个HTTP请求和如何建立一个Socket连接区别不大，只不过outputStream.write写的二进制字节数据格式要符合HTTP。浏览器在建立Socket连接之前，必须根据地址栏里输入的URL的域名DNS解析出IP地址，再根据这个IP地址和默认的80端口与远程服务器建立Socket连接，然后浏览器根据这个URL组装成一个get类型的HTTP请求头，通过outputStream.write发送到目标服务器，服务器等待inputStream.read返回数据，最后断开这个连接。

一句话，发起一个HTTP请求的过程就是建立一个Socket通信的过程。

## HTTP解析
要理解HTTP，最重要的就是要熟悉HTTP中的HTTP Header，它控制着互联网成千上万的用户的数据的传输。最关键的是，它控制着用户浏览器的渲染行为和服务器的执行逻辑。

例如，当服务器没有用户请求的数据时就会返回一个404状态码，告诉浏览器没有要请求的数据，通常浏览器就会展示一个非常不愿意看到的该页面不存在的错误信息。

200，客户端请求成功；302，临时跳转，跳转的地址通过Location指定；400，客户端请求有语法错误，不能被服务器识别；403，服务器收到请求，但是拒绝提供服务；404，请求的资源不存在；500，服务器发生不可预期的错误。

## DNS域名解析
第1步，浏览器会检查缓存中有没有和这个域名对应的解析过得IP地址，如果缓存中有，这个解析过程就将结束。

第2步，如果用户的浏览器缓存中没有，浏览器会查找操作系统缓存中是否有这个域名对应的DNS解析结果。

第3步，在我们的网络配置中都会有“DNS服务器地址”这一项，这个地址就用于解决前面所说的如果两个过程无法解析时要怎么办，操作系统会把这个域名发送给这里设置的LDNS，也就是本地区的域名服务器。这个DNS通常都提供给你本地互联网接入的一个DNS解析服务。

第4步，如果LDNS仍然没有命中，就直接到Root Server域名服务器请求解析。

第5步，根域名服务器返回给本地域名服务器一个所查询域的主域名服务器（gTLD Server）地址。gTLD是国际顶级域名服务器，如.com、.cn、.org等。

第6步，本地域名服务器（Local DNS Server）再向上一步返回的gTLD服务器发送请求。

第7步，接受请求的gTLD服务器查找并返回此域名对应的Name Server域名服务器的地址，这个Name Server通常就是你注册的域名服务器，例如你在某个域名服务提供商的域名，那么这个域名解析任务就由这个域名提供商的服务商来完成。

第8步，Name Server域名服务器会查询存储的域名和IP的映射关系表，在正常情况下都根据域名得到目标IP记录，连同一个TTL值返回给DNS Server域名服务器。

第9步，返回给域名对应的IP和TTL值，Local DNS Server会缓存这个域名和IP的对应关系，缓存的时间由TTL值控制。

第10步，把解析的结果返回给用户，用户根据TTL值缓存在本地系统缓存中，域名解析过程结束。

在实际的DNS解析过程中，可能还不止这10个步骤，如Name Server也可能有多级，或者有一个GTM来负载均衡控制，这都有可能会影响域名解析的过程。(可以用一个dig命令来查询DNS的解析过程）。

## TCP状态转化
主动关闭的一方，接收到对方的FIN ACK，进入此状态。由此不能再接收对方的数据，但是能够向对方发送数据。

搞清楚TCP连接的几种状态转换对我们调试网络程序是非常有帮助的。例如，当我们在压测一个网络程序时可能遇到CPU、网卡、带宽等都不是瓶颈，但是性能就是压不上去的情况。你如果观察一下网络连接情况，看看当前的网络连接都处于什么状态，可能就会发现由于网络连接的并发数不够导致连接都处于TIME\_WAIT状态，这时就要做TCP网络参数调优了。

TCP拥塞控制：我们知道TCP传输是一个“停-等-停-等”的协议，传输方和接受方的步调要一致，要达到步调一致就要通过拥塞控制来调节。TCP在传输时会设定一个“窗口（BDP，Bandwidth Delay Product）”，这个窗口的大小是由带宽和RTT（Round-Trip Time，数据在两端的来回时间，也就是响应时间）决定的。计算的公式是带宽（b/s）x RTT（s）。通过这个值可以得出理论上最优的TCP缓冲区的大小。Linux2.4已经可以自动地调整发送端的缓冲区的大小，而到Linux2.6.7时接收端也可以自动调整了。

## 深入理解Cookie
Cookie的作用，通俗地说就是当一个用户通过HTTP访问一个服务器时，这个服务器会将一些Key/Value键值对返回给客户端浏览器，并给这些数据加上一些限制条件，在条件符合时这个用户下次访问这个服务器时，数据又被完整地带回给服务器。

在我们请求某个URL路径时，浏览器会根据这个URL路径将符合条件的Cookie放在Request请求头中传回给服务端，服务端通过request.getCookies()来取得所有Cookie。

Cookie是HTTP头中的一个字段，浏览器有一些限制，一般是50个左右，总大小为4KB。

## 深入理解Session
同一个客户端每次和服务端交互时，不需要每次都传回所有的Cookie值，而是只要传回一个ID，这个ID是客户端第一次访问服务器时生成的，而且每个客户端是唯一的。这样每个客户端就有了一个唯一的ID，客户端只要传回这个ID就行了，这个ID通常是NAME为JSESIONID的Cookie。

三种方式让Session正常工作：（1）基于URL Path Parameter，默认支持。（2）基于Cookie，如果没有修改Context容器的Cookie标识，则默认也是支持的。（3）基于SSL，默认不支持，只有connector.getAttribute(“SSLEnabled”)为TRUE时才支持。

请注意，如果客户端也支持Cookie，则Tomcat仍然会解析Cookie中的Session ID，并会覆盖URL中的Session ID。如果是第三个种情况，则会根据javax.servlet.request.ssl\_session属性值设置Session ID。

有了Session ID，服务端就可以创建HttpSession对象了，第一次触发通过request.getSession()方法。如果当前的Session ID还没有对应的HttpSession对象，那么就创建一个新的，并将这个对象加到org.apache.catalina.Manager的sessions容器中保存。Manager类将管理所有Session的生命周期，Session过期将被回收，服务器关闭，Session将被序列化到磁盘等。只要这个HttpSession对象存在，用户就可以根据Session ID来获取这个对象，也就做到了对状态的保持。

从request.getSession中获取的HttpSession对象实际上是StandardSession对象的门面对象，这与前面的Request和Servlet是一样的原理。

如果不想自动创建Session对象，也可以通过request.getSession(boolean create)方法来判断与该客户端关联的Session对象是否存在。
