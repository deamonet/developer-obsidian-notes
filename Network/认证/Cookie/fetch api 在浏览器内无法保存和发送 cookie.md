 

  [FaiChou](https://www.v2ex.com/member/FaiChou) · 2018-12-24 14:26:07 +08:00 · 1735 次点击

这是一个创建于 1573 天前的主题，其中的信息可能已经有所发展或是发生改变。

应该是使用姿势有问题, 特来请教.

网上的答案有:

1.  需要设置 credentials: 'include' ✅
2.  跨域需要设置 Access-Control-Allow-Credentials: true ✅
3.  跨域需要设置 Access-Control-Allow-Origin: http://localhost:3000 ✅

这三个都设置的正常:

```
fetch(COR_API, {
  credentials: 'include',
  method: 'POST',
  body: ...
  headers: {'Accept': 'application/json'}
})
```

![chrome](https://i.imgur.com/hlssuPv.png)

在 chrome-devtool 中没有 `Set-Cookie` 字段.

在 Safari 和 Firefox 中会有 `Set-Cookie` 字段, 但是不会保存 cookie.

使用 postman 测试, 发现是正常的, 第二次请求会带上第一次的 cookies.

![postman](https://i.imgur.com/tdYcsHO.png)

[SO 上的提问](https://stackoverflow.com/questions/53909632/browser-cannot-read-and-send-cookies-with-fetch-api-even-set-credentials-to-incl)

[

include'](https://www.v2ex.com/tag/include')[

Postman](https://www.v2ex.com/tag/Postman)[

set-cookie](https://www.v2ex.com/tag/set-cookie)[

Fetch](https://www.v2ex.com/tag/Fetch)

4 条回复  **•**  2018-12-24 20:35:49 +08:00

![FaiChou](https://cdn.v2ex.com/avatar/e7d0/b1a7/254353_normal.png?m=1680507813)

    1

**[FaiChou](https://www.v2ex.com/member/FaiChou)**  

OP

   2018-12-24 14:30:17 +08:00

附上 Safari 的 response  
  
[![](https://i.imgur.com/e8AuAMZ.png)](https://i.imgur.com/e8AuAMZ.png)

![FaiChou](https://cdn.v2ex.com/avatar/e7d0/b1a7/254353_normal.png?m=1680507813)

    2

**[FaiChou](https://www.v2ex.com/member/FaiChou)**  

OP

   2018-12-24 15:08:48 +08:00

使用 axios 也是一样的结果..

![FaiChou](https://cdn.v2ex.com/avatar/e7d0/b1a7/254353_normal.png?m=1680507813)

    3

**[FaiChou](https://www.v2ex.com/member/FaiChou)**  

OP

   2018-12-24 16:02:51 +08:00

哈哈哈哈 😂  
  
结果是 server 把 cookie 的过期时间设置错了, 服务器时间和本地时间差 8 小时.. i18n  
  
No one can save my day..

![FaiChou](https://cdn.v2ex.com/avatar/e7d0/b1a7/254353_normal.png?m=1680507813)

    4

**[FaiChou](https://www.v2ex.com/member/FaiChou)**  

OP

   2018-12-24 20:35:49 +08:00

哦.. 不是三楼的答案.. 而是 因为浏览器禁止了第三方 cookie, 需要手动打开. chrome 和 Safari 是这样的.  
  
而且在手机上也是禁止第三方 cookie..  
  
没啥好办法么? 那 h5 页面咋办呢? 只能改成 token 验证吗?