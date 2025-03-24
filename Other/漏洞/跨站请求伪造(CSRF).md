**跨站请求伪造**（英语：Cross-site request forgery），也被称为 **one-click attack** 或者 **session riding**，通常缩写为 **CSRF** 或者 **XSRF**， 是一种挟制用户在当前已登录的Web应用程序上执行非本意的操作的攻击方法。[[1]](https://zh.wikipedia.org/zh-cn/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0#cite_note-Ristic-1) 跟[跨网站脚本](https://zh.wikipedia.org/wiki/%E8%B7%A8%E7%B6%B2%E7%AB%99%E6%8C%87%E4%BB%A4%E7%A2%BC "跨网站脚本")（XSS）相比，**XSS** 利用的是用户对指定网站的信任，CSRF 利用的是网站对用户网页浏览器的信任。

## 目录

-   [1攻击的细节](https://zh.wikipedia.org/zh-cn/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0#%E6%94%BB%E6%93%8A%E7%9A%84%E7%B4%B0%E7%AF%80)
    -   [1.1例子](https://zh.wikipedia.org/zh-cn/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0#%E4%BE%8B%E5%AD%90)
-   [2防御措施](https://zh.wikipedia.org/zh-cn/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0#%E9%98%B2%E7%A6%A6%E6%8E%AA%E6%96%BD)
    -   [2.1令牌同步模式](https://zh.wikipedia.org/zh-cn/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0#%E4%BB%A4%E7%89%8C%E5%90%8C%E6%AD%A5%E6%A8%A1%E5%BC%8F)
    -   [2.2检查Referer字段](https://zh.wikipedia.org/zh-cn/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0#%E6%AA%A2%E6%9F%A5Referer%E5%AD%97%E6%AE%B5)
    -   [2.3添加校验token](https://zh.wikipedia.org/zh-cn/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0#%E6%B7%BB%E5%8A%A0%E6%A0%A1%E9%A9%97token)
-   [3参考资料](https://zh.wikipedia.org/zh-cn/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0#%E5%8F%82%E8%80%83%E8%B5%84%E6%96%99)

## 攻击的细节[[编辑](https://zh.wikipedia.org/w/index.php?title=%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0&action=edit&section=1 "编辑章节：攻击的细节")]

跨站请求攻击，简单地说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并执行一些操作（如发邮件，发消息，甚至财产操作如转账和购买商品）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去执行。这利用了web中用户身份验证的一个漏洞：**简单的身份验证只能保证请求是发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的**。

### 例子[[编辑](https://zh.wikipedia.org/w/index.php?title=%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0&action=edit&section=2 "编辑章节：例子")]

假如一家银行用以执行转账操作的URL地址如下： `https://bank.example.com/withdraw?account=AccoutName&amount=1000&for=PayeeName`

那么，一个恶意攻击者可以在另一个网站上放置如下代码： `<img src="https://bank.example.com/withdraw?account=Alice&amount=1000&for=Badman" />`

如果有账户名为Alice的用户访问了恶意站点，而她之前刚访问过银行不久，登录信息尚未过期，那么她就会损失1000资金。

这种恶意的网址可以有很多种形式，藏身于网页中的许多地方。此外，攻击者也不需要控制放置恶意网址的网站。例如他可以将这种地址藏在论坛，博客等任何[用户生成内容](https://zh.wikipedia.org/wiki/%E7%94%A8%E6%88%B6%E7%94%9F%E6%88%90%E5%85%A7%E5%AE%B9 "用户生成内容")的网站中。这意味着**如果服务端没有合适的防御措施的话，用户即使访问熟悉的可信网站也有受攻击的危险**。

透过例子能够看出，攻击者并不能通过CSRF攻击来直接获取用户的账户控制权，也不能直接窃取用户的任何信息。他们能做到的，是**欺骗用户的浏览器，让其以用户的名义执行操作**。

## 防御措施[[编辑](https://zh.wikipedia.org/w/index.php?title=%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0&action=edit&section=3 "编辑章节：防御措施")]

### 令牌同步模式[[编辑](https://zh.wikipedia.org/w/index.php?title=%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0&action=edit&section=4 "编辑章节：令牌同步模式")]

令牌同步模式（英语：Synchronizer token pattern，简称STP）。原理是：当用户发送请求时，服务器端应用将令牌（英语：token，一个保密且唯一的值）嵌入HTML表格，并发送给客户端。客户端提交HTML表格时候，会将令牌发送到服务端，令牌的验证是由服务端实行的。令牌可以通过任何方式生成，只要确保随机性和唯一性（如：使用随机种子【英语：random seed】的[哈希链](https://zh.wikipedia.org/wiki/%E5%93%88%E5%B8%8C%E9%93%BE "哈希链") ）。这样确保攻击者发送请求时候，由于没有该令牌而无法通过验证。

[Django](https://zh.wikipedia.org/wiki/Django "Django")框架默认带有STP功能：[[2]](https://zh.wikipedia.org/zh-cn/%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0#cite_note-2)

<form method="post">
    {% csrf_token %}
</form>

渲染后的效果如下：

<form method="post">
    <input type="hidden" name="csrfmiddlewaretoken" value="KbyUmhTLMpYj7CD2di7JKP1P3qmLlkPt" />
</form>

STP能在HTML下运作顺利，但会导致服务端的复杂度升高，复杂度源于令牌的生成和验证。因为令牌是唯一且随机，如果每个表格都使用一个唯一的令牌，那么当页面过多时，服务器由于生产令牌而导致的负担也会增加。而使用[会话](https://zh.wikipedia.org/wiki/%E4%BC%9A%E8%AF%9D_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6) "会话 (计算机科学)")（英语：session）等级的令牌代替的话，服务器的负担将没有那么重。

### 检查Referer字段[[编辑](https://zh.wikipedia.org/w/index.php?title=%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0&action=edit&section=5 "编辑章节：检查Referer字段")]

HTTP头中有一个[Referer字段](https://zh.wikipedia.org/wiki/HTTP%E5%8F%83%E7%85%A7%E4%BD%8D%E5%9D%80 "HTTP引用地址")，这个字段用以标明请求来源于哪个地址。在处理敏感数据请求时，通常来说，Referer字段应和请求的地址位于同一域名下。以上文银行操作为例，Referer字段地址通常应该是转账按钮所在的网页地址，应该也位于bank.example.com之下。而如果是CSRF攻击传来的请求，Referer字段会是包含恶意网址的地址，不会位于bank.example.com之下，这时候服务器就能识别出恶意的访问。

这种办法简单易行，工作量低，仅需要在关键访问处增加一步校验。但这种办法也有其局限性，因其完全依赖浏览器发送正确的Referer字段。虽然http协议对此字段的内容有明确的规定，但并无法保证来访的浏览器的具体实现，亦无法保证浏览器没有安全漏洞影响到此字段。并且也存在攻击者攻击某些浏览器，篡改其Referer字段的可能。

### 添加校验token[[编辑](https://zh.wikipedia.org/w/index.php?title=%E8%B7%A8%E7%AB%99%E8%AF%B7%E6%B1%82%E4%BC%AA%E9%80%A0&action=edit&section=6 "编辑章节：添加校验token")]

由于CSRF的本质在于攻击者欺骗用户去访问自己设置的地址，所以如果要求在访问敏感数据请求时，要求用户浏览器提供不保存在[cookie](https://zh.wikipedia.org/wiki/Cookie "Cookie")中，并且攻击者无法伪造的数据作为校验，那么攻击者就无法再执行CSRF攻击。这种数据通常是窗体中的一个数据项。服务器将其生成并附加在窗体中，其内容是一个伪随机数。当客户端通过窗体提交请求时，这个伪随机数也一并提交上去以供校验。正常的访问时，客户端浏览器能够正确得到并传回这个伪随机数，而通过CSRF传来的欺骗性攻击中，攻击者无从事先得知这个伪随机数的值，服务端就会因为校验token的值为空或者错误，拒绝这个可疑请求。