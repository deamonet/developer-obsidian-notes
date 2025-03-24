Jsessionid子的就是sessionid，Tomcat中生成的就是叫做jsessionid。

浏览器第一次访问服务器会在服务器端生成一个session，这个session保存的是浏览器的相关信息。有一个sessionid和这个session对应，tomcat的StandardManager类将session存储在内存中，也可以持久化到文件。 

客户端只保存sessionid到cookie上，而不保存session，session销毁只能通过invalidate或超时。关掉浏览器并不会关闭session。

可以概括出，sessionid是服务器产生的一个会话key，用来维持这个会话，但是浏览器可以保持这个会话。当浏览器关闭，或者超过会话时间，sessionid就会丢失，session保存的数据也就对之前的客户端来说--消失了，找不到了。

session在访问tomcat服务器HttpServletRequest的getSession(true)的时候创建，tomcat的ManagerBase类提供创建sessionid的方法：随机数+时间+jvmid；

存储在服务器的内存中，tomcat的StandardManager类将session存储在内存中，也可以持久化到file，数据库，memcache，redis等。客户端只保存sessionid到cookie中，而不会保存session，session销毁只能通过invalidate或超时，关掉浏览器并不会关闭session。

> 那么Session在何时创建呢？当然还是在服务器端程序运行的过程中创建的，不同语言实现的应用程序有不同创建Session的方法，而在Java中是通过调用HttpServletRequest的getSession方法（使用true作为参数）创建的。在创建了Session的同时，服务器会为该Session生成唯一的Session id，而这个Session id在随后的请求中会被用来重新获得已经创建的Session；在Session被创建之后，就可以调用Session相关的方法往Session中增加内容了，而这些内容只会保存在服务器中，发到客户端的只有Session id；当客户端再次发送请求的时候，会将这个Session id带上，服务器接受到请求之后就会依据Session id找到相应的Session，从而再次使用之。

创建：sessionid第一次产生是在直到某server端程序调用 HttpServletRequest.getSession(true)这样的语句时才被创建。

删除：超时；程序调用HttpSession.invalidate()；程序关闭；

session存放在哪里：服务器端的内存中。不过session可以通过特殊的方式做持久化管理（memcache，redis）。

session的id是从哪里来的，sessionID是如何使用的：当客户端第一次请求session对象时候，服务器会为客户端创建一个session，并将通过特殊算法算出一个session的ID，用来标识该session对象

session会因为浏览器的关闭而删除吗？

不会，session只会通过上面提到的方式去关闭。

下面是tomcat中session的创建：

> ManagerBase是所有session管理工具类的基类，它是一个抽象类，所有具体实现session管理功能的类都要继承这个类，该类有一个受保护的方法，该方法就是创建sessionId值的方法：
> 
> （ tomcat的session的id值生成的机制是一个随机数加时间加上jvm的id值，jvm的id值会根据服务器的硬件信息计算得来，因此不同jvm的id值都是唯一的），
> 
> StandardManager类是tomcat容器里默认的session管理实现类，
> 
> 它会将session的信息存储到web容器所在服务器的内存里。
> 
> PersistentManagerBase也是继承ManagerBase类，它是所有持久化存储session信息的基类，PersistentManager继承了PersistentManagerBase，但是这个类只是多了一个静态变量和一个getName方法，目前看来意义不大， 对于持久化存储session，tomcat还提供了StoreBase的抽象类，它是所有持久化存储session的基类，另外tomcat还给出了文件存储FileStore和数据存储JDBCStore两个实现。

所以会出现以下三种情况：

1、server没有关闭，并在session对象销毁时间内，当客户端再次来请求serve端的servlet或jsp时，将会把将第一次请求该serve时生成的sessionid带到请求头上向server端发送，server端收到sessionid后根据此sessionid会去搜索server对应的session对象并直接返回这个session对象，此时不会重新创建session对象。

2、当server关闭（之前产生的session对象也就消亡了），或者session对象过了销毁时间，浏览器窗口没有关闭，并在本窗口继续请求server端的servlet或者jsp时，此时同样会将sessionid 发送到 服务端，server拿着id去找对应的session对象；但是此时session对象已经不存在了。所以会重新生成一个session和对应的sessionid ，将这个新的id以响应报文的形式发到浏览器的内核中，重新更新cookie。

3、当server没有关闭，并且session对象在其销毁时间内，当请求一个jsp页面返回客户端后，关闭此浏览器窗口，此时其内存中的sessionid也就随之销毁。在重新去请求server端的servlet或者jsp时，会重新生成一个sessionid给客户端浏览器，并且存在浏览器内存中。

**cookie的保存方式有两种：**

**如果没有设置cookie的失效时间，这个cookie就存在与浏览器进程；**

**设置了cookie的失效时间，那么这个cookie就存在于硬盘。**

![复制代码](media/复制代码.gif)

        //Cookie的一些基本设置
        Cookie cookie = new Cookie("Admin-Token", token);

        Cookie[] cookie2 = request.getCookies();
        //request.getContextPath()   mdrwebrest
        cookie.setPath("/");        //设置cookies有效路径
        //设置cookie有效时间  正数：存到硬盘，负数存到浏览器，0立刻销毁
        cookie.setMaxAge();      
        cookie.setDomain(loginToMDRConfig.getIP()); //跨域
        response.addCookie(cookie);

![复制代码](media/复制代码.gif)

**当将sessionid保存在硬盘时，可以在打开浏览器时指定先携带jsessionID去找服务端的session，可以看下边的一个案例。**

![](media/1548492-20220331163509335-813514932.png)

下边是看到一个网上的案例，记一下他的解决思路

最近碰到一个问题，统计在线人数，初步想法能将session管理起来，即可达成目的，但是发现直接关闭浏览器，session竟找不到登录用户了，难道关闭浏览器，session就销毁了？不是这样的

**session的运行机制**

-   当一个Session开始时，Servlet容器会创建一个HttpSession对象，那么在HttpSession对象中，可以存放用户状态的信息
    
-   Servlet容器为HttpSession对象分配一个唯一标识符即Sessionid，Servlet容器把Sessionid作为一种Cookie保存在客户端的 _*_浏览器* 中
    
-   用户每次发出Http请求时，Servlet容器会从HttpServletRequest对象中取出Sessionid,然后根据这个Sessionid找到相应的HttpSession对象，从而获取用户的状态信息

**为什么当我们关闭浏览器后，就再也访问不到之前的session了呢？**

其实之前的Session一直都在服务器端，而当我们关闭浏览器时，此时的Cookie是存在

于浏览器的进程中的，当浏览器关闭时，Cookie也就不存在了。

其实Cookie有两种:

-   一种是存在于浏览器的进程中;
-   一种是存在于硬盘上

而session的Cookie是存在于浏览器的进程中，那么这种Cookie我们称为会话Cookie，

当我们重新打开浏览器窗口时，之前的Cookie中存放的Sessionid已经不存在了，此时

服务器从HttpServletRequest对象中没有检查到sessionid，服务器会再发送一个新的存

有Sessionid的Cookie到客户端的浏览器中，此时对应的是一个新的会话，而服务器上

原先的session等到它的默认时间到之后，便会自动销毁。

**以上既是为何找不到session的原因，如何解决？**

可以在登录的时候，手动把JSESSIONID保存到COOKIE中，并令其存在的生命周期更长（可以和session的生命周期保持一致），这样即使浏览器关闭，cookie也不会消失，JSESSIONID还在，服务器即可再找到该session，具体代码如下：

HttpSession session=request.getSession(); //获取session

String sessionid=session.getId();  //获取sessionid

Cookie cookie=new Cookie("JSESSIONID",sessionid); //手动设置一个硬盘存储COOKIE，这个cooike时存在硬盘的，不是存在浏览器线程的

cookie.setMaxAge(30*60);

response.addCookie(cookie); //将COOKIE设置到响应上

如有差错，请各位指正

分类: [JavaEE](https://www.cnblogs.com/Timeouting-Study/category/1679001.html)