### _分类_ [编程技术](https://www.runoob.com/w3cnote_genre/code "编程技术")

JavaScript 实现页面跳转重定向可以使用以下两种方法：

### window.location.replace("url")

**类似 HTTP 重定向**

将地址替换成新 url，该方法通过指定 URL 替换当前缓存在历史里（客户端）的项目，因此当使用 replace 方法之后，你不能通过"前进"和"后退"来访问已经被替换的URL，这个特点对于做一些过渡页面非常有用！

### window.location.href="url"

**类似点击 a 标签的链接**。

跳转到指定的 url。

### 实例

## 实例

// 类似 HTTP 重定向到菜鸟教程  
window.location.replace("https://www.runoob.com");  
  
// 类似点击菜鸟教程的链接（a 标签）  
window.location.href = "https://www.runoob.com";