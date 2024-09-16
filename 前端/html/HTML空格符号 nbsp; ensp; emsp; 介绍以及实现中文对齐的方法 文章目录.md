文章目录

-   [一：不同空格符合的区别](https://www.cnblogs.com/moqiutao/p/6828263.html#articleHeader0)
-   [二：使用场景](https://www.cnblogs.com/moqiutao/p/6828263.html#articleHeader1)
-   [三：空格新成员](https://www.cnblogs.com/moqiutao/p/6828263.html#articleHeader2)　

## 一：不同空格符合的区别

-   &nbsp; 半角的不断行的空白格（推荐使用）
-   &ensp;  半角的空格 
-   &emsp;  全角的空格

详细的含义：

&nbsp;：这是我们使用最多的空格，也就是按下space键产生的空格。在HTML中，如果你用空格键产生此空格，空格是不会累加的（只算1个）。要使用html实体表示才可累加。该空格占据宽度受字体影响明显而强烈。在inline-block布局中会搞些小破坏，在两端对齐布局中又是不可少的元素。

&ensp;：此空格有个相当稳健的特性，就是其**占据的宽度正好是1/2个中文宽度**，而且基本上不受字体影响。

&emsp; ：此空格也有个相当稳健的特性，就是**其占据的宽度正好是1个中文宽度**，而且基本上不受字体影响。

## 二：使用场景

对于**&ensp;**和**&emsp;**在一些中文排版对齐方面可以使用，如下html代码：

<ul>
    <li class="li">姓&emsp;&emsp;名：<input type="text" /></li>
    <li class="li">手&ensp;机&ensp;号：<input type="text" /></li>
    <li class="li">电子邮箱：<input type="text" /></li>
</ul>

实现的效果如图所示：

![](media/595142-20170508213546066-2094539815.png)

值得注意的是：上面的空白字符中文对齐方法在IE6下不能完全兼容。（现在谁还在兼容IE6呢，所以还是非常有用的。）

## 三：空格新成员&#x3000

大多数编辑器中空格是透明滴，很容易就被删掉；另外，HTML压缩时候，空格也会被删除掉，所以需要转换书写形式。

在web页面上，一般有3种书写：

-   直接，例如搜狗输入法输入“版权” – `©`.
-   web字符，`&copy;`
-   charCode表示：`&#xa9;`

而上面的`&ensp;`, `&emsp;`就是具有特定名称的web字符。但是，恕我寡闻，我并不清楚全角空格是否有对应`& + 关键字`示意，所以，就使用工具转成了charCode字符表示，也就是这里的`&#x3000;`

-   `&ensp;` → `&#x2002;`
-   `&emsp;` → `&#x2003;`

字符使用技巧：

1. HTML中字符输出使用`&#x`配上charCode值；  
2. 在JavaScript文件中为防止乱码转义，则是`\u`配上charCode值；  
3. 而在CSS文件中，如CSS伪元素的`content`属性，直接使用`\`配上charCode值。

因此，想在HTML/JS/CSS中转义“我”这个汉字，分别是：

-   `&#x6211;`
-   `\u6211`, 如`console.log('\u6211');`
-   `\6211`, 如`.xxx:before { content: '\6211'; }`

考虑到直接`&#x3000;`这种形式暴露在HTML中，可能会让屏幕阅读器等辅助设备读取，从而影响正常阅读流，因此，我们可以进一步优化下，使用标签，利用伪元素，例如：

.half:before { content: '\2002'; speak: none; }
.full:before { content: '\2003'; speak: none; }

html代码：

<ul>
    <li class="li">姓<span class="full"></span><span class="full"></span>名：<input type="text" /></li>
    <li class="li">手<span class="half"></span>机<span class="half"></span>号：<input type="text" /></li>
    <li class="li">电子邮箱：<input type="text" /></li>
</ul>

css代码：

![复制代码](media/复制代码.gif)

.half {
    *zoom: expression( this.runtimeStyle['zoom'] = '1', this.innerHTML = '&#x2002;');
}
.full {
    *zoom: expression( this.runtimeStyle['zoom'] = '1', this.innerHTML = '&#x2003;');
}
.half:before { content: '\2002'; speak: none; }
.full:before { content: '\2003'; speak: none; }

![复制代码](media/复制代码.gif)

上面用到了runtimeStyle这个对象属性，这个是IE专属的。

下面简单介绍下style、 currentStyle、 runtimeStyle以及getComputedStyle的区别，在IE下测试如下。

html代码：

<div id="tt" style="color:blue;">这里是来检测style,currentStyle,runtimeStyle的区别</div> 

js代码：

![复制代码](media/复制代码.gif)

var myDiv = document.getElementById("tt");
myDiv.runtimeStyle.color="black"; 
console.log(myDiv.currentStyle.color);  //black
console.log(myDiv.runtimeStyle.color);  //black
console.log(document.defaultView.getComputedStyle(myDiv, null).color); //rgb(0, 0, 0)
console.log(myDiv.style.color);         //blue

![复制代码](media/复制代码.gif)

**说明一下：**

obj.style：这个方法只能JS只能获取写在html标签中的写在style属性中的值（style=”…”），而无法获取定义在<style type="text/css">里面的属性。

IE中使用的是obj.currentStyle方法，而FF是用的是getComputedStyle 方法 。

“DOM2级样式”增强了document.defaultView，提供了getComputedStyle()方法。这个方法接受两个参数：要取得计算样式的元素和一个伪元素字符串（例如“:after”）。如果不需要伪元素信息，第二个参数可以是null。getComputerStyle()方法返回一个CSSStyleDeclaration对象，其中包含当前元素的所有计算的样式。

其语法为：**document.defaultView.getComputedStyle('元素', '伪类')**；IE9及以上支持该写法，IE8以及以下不支持。

**总结一下：**

通过**document.defaultView.getComputedStyle()**得到背景色，不同浏览器得到的不一样，可能会返回将所有颜色转换成RGB格式，也可能是颜色值。

IE通过**currentStyle**方法得到的颜色值没有将颜色转化成RGB格式。

详细了解：[《js中使用document.defaultView.getComputedStyle()、currentStyle()方法获取CSS属性值》](http://www.cnblogs.com/moqiutao/p/6276826.html)

参考地址：

[小tips: 使用&#x3000;等空格实现最小成本中文对齐](http://www.zhangxinxu.com/wordpress/2015/01/tips-blank-character-chinese-align/)

[CSS 听觉参考手册](http://www.runoob.com/cssref/css-ref-aural.html)

[web页面相关的一些常见可用字符介绍](http://www.zhangxinxu.com/wordpress/2011/05/web%E9%A1%B5%E9%9D%A2%E7%9B%B8%E5%85%B3%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B8%B8%E8%A7%81%E5%8F%AF%E7%94%A8%E5%AD%97%E7%AC%A6%E4%BB%8B%E7%BB%8D/)

作者：[风雨后见彩虹](http://www.cnblogs.com/moqiutao/)

如果觉得文章写得不错或对您有用，请随意打赏。点击文章右下角“喜欢”二字，您的支持是我最大的鼓励

打赏支持

分类: [HTML/CSS](https://www.cnblogs.com/moqiutao/category/572637.html)

标签: [css](https://www.cnblogs.com/moqiutao/tag/css/), [nbsp](https://www.cnblogs.com/moqiutao/tag/nbsp/), [ensp](https://www.cnblogs.com/moqiutao/tag/ensp/), [emsp](https://www.cnblogs.com/moqiutao/tag/emsp/), [中文对齐](https://www.cnblogs.com/moqiutao/tag/%E4%B8%AD%E6%96%87%E5%AF%B9%E9%BD%90/)
