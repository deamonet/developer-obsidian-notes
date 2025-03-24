# data: image/png； base64 用法详解 ( 作用，语法，优缺点 )
![](media/reprint.png)

[雁 南飞](https://blog.csdn.net/weixin_50339217 "雁 南飞") ![](media/newCurrentTime2.png) 于 2021-01-29 16:48:20 发布 ![](media/articleReadEyes2.png) 39015 ![](media/tobarCollect2.png) 收藏 71

分类专栏： [网页优化](https://blog.csdn.net/weixin_50339217/category_10781247.html) 文章标签： [html](https://so.csdn.net/so/search/s.do?q=html&t=all&o=vip&s=&l=&f=&viparticle=) [css](https://so.csdn.net/so/search/s.do?q=css&t=all&o=vip&s=&l=&f=&viparticle=) [js](https://so.csdn.net/so/search/s.do?q=js&t=all&o=vip&s=&l=&f=&viparticle=)

版权

 ![](media/489fad64a62648818eaaebc28e5c8659.jpg) 华为云开发者联盟 该内容已被华为云开发者联盟社区收录

加入社区

 [![](media/20201014180756930.png) 网页优化 专栏收录该内容](https://blog.csdn.net/weixin_50339217/category_10781247.html "网页优化")

1 篇文章 0 订阅

订阅专栏

大家可能注意到了，网页上有些图片的 src 或 css 背景图片的 url 后面跟了一大串字符，如：

```css

background-image:url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAAkCAYAAABIdFAMAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAHhJREFUeNo8zjsOxCAMBFB/KEAUFFR0Cbng3nQPw68ArZdAlOZppPFIBhH5EAB8b+Tlt9MYQ6i1BuqFaq1CKSVcxZ2Acs6406KUgpt5/LCKuVgz5BDCSb13ZO99ZOdcZGvt4mJjzMVKqcha68iIeP86GAiOv8CDADlIUQBs7MD3wAAAABJRU5ErkJggg%3D%3D）

```

  
data:image/png;base64, 字符串... 这个表示什么意思，又有什么作用呢？  
其实这就是所谓的 Data URI scheme。 直译过来的意思是：URI 数据处理方案...

---

  

## 1. Data URI scheme 是什么 ？？？

  

> **Data URI scheme 是在 RFC2397 中定义的，目的是将一些小的数据，直接嵌入到网页中，从而不用再从外部文件载入。减少对 HTTP 的请求次数。达到优化网页的效果。**

> **base64 后面那一串字符，其实是一张图片，将这些字符串复制粘贴到浏览器的中打开，就能看到图片了**  
>   

---

  

假设你有的图像：A.jpg ，把它在网页上显示出来的标准方法是：

```css
<img src="http://sjolzy.cn/images/A.jpg"/>
```

这种取得数据的方法称为 **http URI scheme** 。  
  

```css

<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAQAAAADCAIAAAA7ljmRAAAAGElEQVQIW2P4DwcMDAxAfBvMAhEQMYgcACEHG8ELxtbPAAAAAElFTkSuQmCC" />

```

这种取得数据的方法称为 **Data URI scheme** 。

---

  

### 2. Data URI scheme 的语法

  

在上面的 Data URI scheme 中：

**data 表示取得数据的协定名称；**

**image/png 是数据类型名称；**

**base64 是数据的编码方法，逗号后面就是这个image/png文件base64编码后的数据。**

目前，Data URI scheme支持的类型有：

> **data: 文本数据  
> data: text/plain, ------- 文本数据  
> data: text/html, -------- HTML代码  
> data: text/html;base64, -------- base64编码的HTML代码  
> data: text/css, ---------- CSS代码  
> data: text/css;base64, ---------- base64编码的CSS代码  
> data: text/javascript, ------------ Javascript代码  
> data: text/javascript;base64, --------- base64编码的Javascript代码  
> data: image/gif;base64, ---------------- base64编码的gif图片数据  
> data: image/png;base64, -------------- base64编码的png图片数据  
> data: image/jpeg;base64, ------------- base64编码的jpeg图片数据  
> data: image/x-icon;base64, ---------- base64编码的icon图片数据**

  

① 在 HTML 中使用 data URL （不建议这样使用）

```html

<img src=“data:image/png;base64,image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAQAAAADCAIAAAA7ljmRAAAAGElEQVQIW2P4DwcMDAxAfBvMAhEQMYgcACEHG8ELxtbPAAAAAElFTkSuQmCC"/>

```

  

② 在 CSS 中使用 data URL

```css

body { 
      background-image: url("data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAQAAAADCAIAAAA7ljmRAAAAGElEQVQIW2P4DwcMDAxA
      fBvMAhEQMYgcACEHG8ELxtbPAAAAAElFTkSuQmCC")
     };
     
```

  

③ 在 script 中使用 data URL

```javascript

   _captchaImage() {
		captchaImage().then(res => {  //请求接口
			if (res.code == 200) {
				this.codeUrl = 'data:image/gif;base64,' + res.img; // 拼接请求回来的数据
				this.formModel.uuid = res.uuid;
			   }
		   });
		},
			
```

---

  

### 3. Data URI scheme 的优缺点

  

优点：

> **减少HTTP请求数**，没有了TCP连接消耗和同一域名下浏览器的并发数限制。对于小文件会降低带宽。虽然编码后数据量会增加，但是却减少了http头，当http头的数据量大于文件编码的增量，那么就会降低带宽。 对于HTTPS站点，HTTPS和HTTP混用会有安全提示，而HTTPS相对于HTTP来讲开销要大更多，所以Data URI在这方面的优势更明显。可以把整个多媒体页面保存为一个文件。

缺点：

> 1.  **无法被重复利用**，同一个文档多次被应用到同一内容中，数据被大量增加，消耗了下载时间。
> 2.  **无法被独自缓存**，其包含文档重新加载时，它也要重新加载。
> 3.  **耗时**，客户端需要重新解码和显示，增加消耗。
> 4.  **不支持数据压缩**，base64编码会增加1/3大小，而urlencode后数据量会增加更多
> 5.  **不安全**，不利于安全软件的过滤，同时也存在一定的安全隐患。

  

##### 在线转译 [base64](https://so.csdn.net/so/search?q=base64&spm=1001.2101.3001.7020) 图片小工具

https://www.css-js.com/tools/base64.html


**注**：Data URI 和 HTML 两者虽然可以完成并解决所有的主流浏览器，它们由于无法被缓存和重复利用的缺陷，所以并不适合直接在页面中使用，但在 CSS 和 JS 文件中对图片适当地使用有非常大的优越性：大大减少请求数，现在大型网站的 CSS 引用了大量的图片资源。CSS 和 JS 都可以被缓存，间接的实现了数据的缓存。利用 CSS 可以解决 Data URI 的重复利用问题。