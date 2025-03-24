
[12月 22 2014](https://dreamer-yzy.github.io/2014/12/22/Cookie-的-HttpOnly-和-Secure-属性作用/)

[HTTP](https://dreamer-yzy.github.io/categories/HTTP/)►[Web](https://dreamer-yzy.github.io/categories/HTTP/Web/)

今天和总监、同事又讨论起关于Session共享的解决方案问题，讨论到因为Tomcat自带的Session机制在集群时难以做到真正的集群。因为使用Tomcat自带的Session机制，难以做到在集群中节点共享，一般是通过Nginx反向代理使用Hash、固定IP等解决方案，并不能避免单节点崩溃时不能继续提供服务的问题。虽然这种可以解决压力的问题，但是当一直为某IP或通过Hash来分配服务的某台服务器挂了，则它负责服务的客户就都访问不了了（Session失效，只能重新调度分配到其他服务器，这时要重新生成会话）。讨论到的可用的解决方案是Cookie + Redis，然后又讨论了Cookie的安全性问题。然后同事问了下`HttpOnly`这个在浏览器里打勾的作用，然后自己按以前了解到的资料来回答了一下，大概是说：不能通过Javascript来修改带有HttpOnly属性的Cookie，只能通过服务器来修改。但是看到总监却可以通过JS来修改带有HttpOnly属性的Cookie，这让我产生了怀疑自己的正确性。

不过还好，事后向总监确认了一下，原来他是通过删除旧的带有HttpOnly属性的Cookie，然后才用JS添加一个同名同值没有HttpOnly属性来测试。所以，我之前说的大概是对的，但是不够系统，所以再次查了下资料来系统整理一下，与君分享。

下面两个属性都属于Cookie安全方面考虑的。这要视浏览器或服务端有没有支持。

## Secure

Cookie的`Secure`属性，意味着保持Cookie通信只限于加密传输，指示浏览器仅仅在通过`安全/加密`连接才能使用该Cookie。如果一个Web服务器从一个非安全连接里设置了一个带有`secure`属性的Cookie，当Cookie被发送到客户端时，它仍然能通过`中间人攻击`来拦截。

## HttpOnly

Cookie的`HttpOnly`属性，指示浏览器不要在除HTTP（和 HTTPS)请求之外暴露Cookie。一个有`HttpOnly`属性的Cookie，不能通过非HTTP方式来访问，例如通过调用JavaScript(例如，引用 `document.cookie`），因此，不可能通过跨域脚本（一种非常普通的攻击技术）来偷走这种Cookie。尤其是Facebook 和 Google 正在广泛地使用`HttpOnly`属性。

Share

-   [Cookie](https://dreamer-yzy.github.io/tags/Cookie/)
-   [HTTP](https://dreamer-yzy.github.io/tags/HTTP/)