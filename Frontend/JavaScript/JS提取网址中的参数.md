#### 方法一：采用正则表达式获取地址栏参数 (代码简洁，重点正则）

```js
function getQueryString(name) {
    let reg = new RegExp("(^|&)" + name + "=([^&]*)(&|$)", "i");
    let r = window.location.search.substr(1).match(reg);
    if (r != null) {
        return decodeURIComponent(r[2]);
    };
    return null;
 }
 ```
-   调用方法：

> let 参数1 = GetQueryString("参数名1"));

用这个方法，如果有参数名，但是没有参数值，结果是一个空字符串，而不是null或者空对象。

#### 方法二：split拆分法 (代码较复杂，较易理解)

```jsx
function GetRequest() {
   const url = location.search; //获取url中"?"符后的字串
   let theRequest = new Object();
   if (url.indexOf("?") != -1) {
      let str = url.substr(1);
      strs = str.split("&");
      for(let i = 0; i < strs.length; i ++) {
         theRequest[strs[i].split("=")[0]]=unescape(strs[i].split("=")[1]);
      }
   }
   return theRequest;
}
```

-   调用方法：

> let Request = new Object();  
> Request = GetRequest();  
> var 参数1,参数2 ...;  
> 参数1 = Request['参数1'];  
> 参数2 = Request['参数2'];  
> 参数... = Request['参数...'];


#### 方法三：split拆分法(易于理解，代码中规)

```jsx
function getQueryVariable(variable){
       let query = window.location.search.substring(1);
       let vars = query.split("&");
       for (let i=0;i<vars.length;i++) {
               let pair = vars[i].split("=");
               if(pair[0] == variable){return pair[1];}
       }
       return(false);
}
```

-   调用方法：

> let 参数1 = getQueryVariable("参数名1");

#### _补充URL知识：_

> 示例url = [http://www.jianshu.com/search?q=js&page=1&type=note](https://www.jianshu.com/search?q=js&page=1&type=note)

-   1、window.location.href(设置或获取整个 URL 为字符串)  
    console.log(window.location.href)  
    打印结果：`http://www.jianshu.com/search?q=123&page=1&type=note`
-   2、window.location.protocol(设置或获取 URL 的协议部分)  
    console.log(window.location.protocol）  
    打印结果：`http:`
-   3、window.location.host(设置或获取 URL 的主机部分)  
    console.log(window.location.host）  
    打印结果：`www.jianshu.com`
-   4、window.location.port(设置或获取与 URL 关联的端口号码)  
    console.log(window.location.port）  
    打印结果：`空字符(如果采用默认的80端口(update:即使添加了:80)，那么返回值并不是默认的80而是空字符)`
-   5、window.location.pathname(设置或获取与 URL 的路径部分（就是文件地址）)  
    console.log(window.location.pathname）  
    打印结果：`/search`
-   6、window.location.search(设置或获取 href 属性中跟在问号后面的部分)  
    console.log(window.location.search）  
    打印结果：`?q=123&page=1&type=note`  
    **PS：获得查询（参数）部分，除了给动态语言赋值以外，我们同样可以给静态页面，并使用javascript来获得相信应的参数值。**
-   7、window.location.hash(设置或获取 href 属性中在井号“#”后面的分段)  
    console.log(window.location.hash）  
    打印结果：`空字符(因为url中没有)`

  
  
作者：大小伍  
链接：https://www.jianshu.com/p/708c915fb905  
来源：简书  
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。