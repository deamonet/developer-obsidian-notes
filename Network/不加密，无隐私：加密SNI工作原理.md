09/24/2018

-   [![Alessandro Ghedini](media/Alessandro_Ghedini.jpg)](https://blog.cloudflare.com/author/alessandro-ghedini/)
    
    [Alessandro Ghedini](https://blog.cloudflare.com/author/alessandro-ghedini)
    

![](media/esni-3@3.5x-1.png)

今天，我们发布了[加密SNI支持](https://blog.cloudflare.com/esni)，[这](https://tools.ietf.org/html/draft-ietf-tls-esni)是[TLS 1.3](https://blog.cloudflare.com/rfc-8446-aka-tls-1-3/)协议[的扩展](https://tools.ietf.org/html/draft-ietf-tls-esni)，它通过防止路径上的观察者（包括互联网服务提供商，咖啡店所有者和防火墙）拦截TLS服务器名称指示（SNI）扩展，并使用它来确定用户正在访问哪些网站，从而提高互联网用户的隐私。

加密SNI，连同Cloudflare已经免费提供的其他互联网安全功能，将使在互联网上（恶意）检查内容和跟踪用户变得更加困难。继续往下读，来了解它是如何工作的吧。

### 为什么需要SNI？

TLS服务器名称指示（SNI）扩展[最初是在2003年标准化的](https://tools.ietf.org/html/rfc3546)，它允许服务器在同一组IP地址上托管多个启用了TLS的站点，要求客户端在初始TLS握手期间指定要连接到哪个站点。例如，如果没有SNI，服务器就不知道要向客户端提供哪个证书，或者要向连接应用什么配置。

客户端将包含其连接到的站点主机名的SNI扩展添加到ClientHello消息。它在TLS握手期间将ClientHello发送到服务器。不幸的是，由于客户端和服务器此时不共享加密密钥，因此ClientHello消息未被加密发送。

![](media/tls13_unencrypted_server_name_indication-2.png)

_TLS 1.3 with Unencrypted SNI_

这意味着该网路上的观察者（例如互联网服务提供商、咖啡店老板或防火墙）可以拦截明文ClientHello消息，并确定客户端要连接到哪个网站。这使得观察者可以跟踪用户正在访问的站点。

但是，使用了SNI加密，即使ClientHello的其余部分以明文形式发送，客户端也会加密SNI。

![](media/tls13_encrypted_server_name_indication-1.png)

_TLS 1.3 with Encrypted SNI_

那么为什么以前无法对原始SNI进行加密，现在却又可以加密了呢？如果客户端和服务器尚未协商，加密密钥又从何而来？

### 如果必须先有鸡后有蛋，那么你的鸡又在哪里呢？

与[许多其他Internet功能](https://datatracker.ietf.org/meeting/101/materials/slides-101-dnsop-sessa-the-dns-camel-01)一样，答案就是“DNS”。

服务器在已知的DNS记录上发布一个公钥，客户端可以在连接之前获取该公钥（A、AAAA和其他记录已经是这样做的了）。然后，客户端将ClientHello中的SNI扩展替换为“加密的SNI”扩展，该扩展不同于原来的SNI扩展，而是使用对称加密密钥（用服务器公共密钥派生的密钥）进行加密的，如下所述。拥有私钥的服务器也可以派生对称加密密钥，然后可以解密扩展并终止连接（或将其转发到后端服务器）。因为只有客户端和它所连接的服务器可以派生加密密钥，所以加密的SNI不能被第三方解密和访问。

请务必注意，这是TLS 1.3及更高版本的扩展，不适用于该协议的早期版本。原因很简单：TLS 1.3引入的一项更改（[存在问题](https://blog.cloudflare.com/you-get-tls-1-3-you-get-tls-1-3-everyone-gets-tls-1-3/)）内容是将服务器发送的证书消息移至TLS握手的加密部分（这在1.3之前是以明文形式发送的）。如果没有对协议进行根本性的更改，攻击者仍然可以通过简单地观察在网络上发送的明文证书来确定服务器的身份。

底层加密机制涉及使用[Diffie-Hellman密钥交换算法](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)，该算法允许客户端和服务器在不受信任的通道上生成共享密钥。因此，已加密的SNI加密密钥是在客户端上计算得出的，通过使用服务器的公共密钥（实际上是Diffie-Hellman半静态密钥共享的公共部分）和由服务器生成的临时Diffie-Hellman共享的私有部分（动态地由客户端本身生成，且在ClientHello发送到服务器后立即被丢弃）。额外的数据（例如客户端作为其ClientHello消息的一部分发送的一些加密参数）也混合到加密过程中，以达到更好的加密效果。

然后，客户端的ESNI扩展将不仅包括实际的加密SNI位，还包括客户端的公钥共享，用于加密的密码套件以及服务器的ESNI DNS记录的摘要。另一方面，服务器使用其自己的私钥共享和客户端共享的公共部分来生成加密密钥并解密扩展。

虽然这看起来可能过于复杂，但这可以确保加密密钥以密码方式绑定到为其生成的特定TLS会话，并且不能跨多个连接重复使用。这样可以防止攻击者观察到客户端发送的加密扩展，从而避免攻击者在单独的会话中捕获它并将其重新放到服务器，暴露用户试图连接到的网站的身份（这被称为“剪切和粘贴”攻击）。

然而，服务器私钥的妥协将危及派生自服务器私钥的所有ESNI对称密钥（这将使观察者可以解密以前收集的加密数据），这就是为什么Cloudflare的SNI加密执行每小时轮转服务器秘钥以提高安全性，但同时也会跟踪几个小时前的秘钥，以便DNS完成缓存和响应延迟。因此，密钥稍微过时的客户端仍然可以毫无问题地使用ESNI（但最终所有密钥都会被丢弃和遗忘）。

### 但先等等，DNS？说真的？

细心的读者可能已经意识到，仅使用DNS（默认情况下为未加密）将使整个加密SNI的想法完全没有意义：路径上的观察者将能够简单地通过观察客户端本身发送的纯文本DNS查询来确定客户端连接到哪个网站，无论连接中是否使用了加密的SNI。

但是随着DNS的一些特性的引入，例如DNS over TLS（DoT）以及DNS over HTTPS（DoH），以及有向用户提供这些特性的公共DNS解析器（例如Cloudflare自己的[1.1.1.1](https://blog.cloudflare.com/announcing-1111/)），DNS查询现在可以被加密并且防止被观察者和追踪者窥视。

然而，尽管在一定程度上（因为仍存在有害的解析器）来自DoT/DoH DNS解析器的请求是可以被信任的，但一个下定决心的攻击者仍然有可能通过拦截解析器与权威DNS服务器的通信并注入恶意数据来毒害解析器的缓存。也就是说，除非权威服务器和解析器都支持[DNSSEC](https://www.cloudflare.com/dns/dnssec/) [1]­，DNS解析器的请求才可以被信任。顺便说一句，Cloudflare的权威DNS服务器可以对返回给解析器的响应进行签名，而1.1.1.1解析器可以验证这些响应。

### 那么IP地址呢？

虽然DNS查询和TLS SNI加密现在都可以得到保护，但路径上的攻击者仍可以通过查看来自用户设备流量上的目标IP地址来确定用户正在访问哪些网站。由于许多Cloudflare域共享着相同的地址集，因此我们的一些客户在一定程度上受到了这种保护，但这还不够，我们需要做更多的工作从而能更大程度地保护最终用户。请继续关注Cloudflare关于这个主题的更多更新。

### 在哪里可以注册呢？

在Cloudflare域上使用我们名称服务器的网站都可以免费启用加密SNI，因此您无需执行任何操作即可在Cloudflare网站上启用它。在浏览器方面，Firefox的朋友告诉我们，他们期望本周可以向[Firefox Nightly](https://www.mozilla.org/firefox/channel/desktop/)添加加密的SNI支持（请注意，加密的SNI规范仍在开发中，因此尚不稳定）。

通过访问[encryptedsni.com](https://encryptedsni.com/)，您可以检查您的浏览体验是否安全。您使用的是安全DNS吗？您的解析器是否正在验证DNSSEC签名？您的浏览器是否支持TLS 1.3？您的浏览器是否加密了SNI？如果对所有这些问题的回答都是“是”，那么您就可以确定您的浏览已受到保护，不会被人窥视，您可以睡个好觉了。

### 结论

加密的SNI，连同TLS 1.3、DNSSEC和DoT/DoH，填补了互联网上仅存的几个能够进行监视和审查的漏洞之一。要实现一个没有窥探的互联网，我们还需要做更多的工作，但我们正在（慢慢地）朝着这个目标迈进。

[1]：值得一提的是，攻击者可以通过劫持在DNS解析器和TLD服务器之间的BGP（边界网关协议）路由来禁用DNSSEC。上周我们[公告](https://blog.cloudflare.com/rpki/)了我们对RPKI（互联网码号资源公钥基础设施）承担的义务，如果DNS解析器和顶级域名也开始运营RPKI，那么这种劫持将会更难发生。

[_订阅博客_](https://blog.cloudflare.com/subscribe/)_，来获取有关我们生日周所有公告的每日更新吧。_

![](media/Cloudflare-Birthday-Week-3.png)

我们保护 [整个企业网络](https://www.cloudflare.com/zh-cn/network-services/), 帮助客户高效构建 [互联网规模应用](https://workers.cloudflare.com/), 加速一切 [网站或互联网应用](https://www.cloudflare.com/zh-cn/performance/accelerate-internet-applications/) , 抵御 [DDoS 攻击](https://www.cloudflare.com/zh-cn/ddos/), 阻止 [黑客](https://www.cloudflare.com/zh-cn/application-security/), 并可帮助您踏上 [Zero Trust 之旅](https://www.cloudflare.com/zh-cn/products/zero-trust/)。

从任何设备访问 [1.1.1.1，](https://1.1.1.1/) 使用我们的免费应用加速和保护您的互联网。

如需进一步了解我们帮助构建更美好互联网的使命，请从 [这里](https://www.cloudflare.com/zh-cn/learning/what-is-cloudflare/) 开始。如果您正在寻找新的职业方向，请查看 [我们的空缺职位](https://cloudflare.com/zh-cn/careers)。