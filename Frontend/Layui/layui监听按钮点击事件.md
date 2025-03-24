Layui 是一个前端 UI 框架，可以用于构建网页界面。要在 Layui 中监听按钮点击事件，可以使用 layui.event 模块中的 bind 方法。

具体步骤如下：

1.  在 HTML 中定义一个按钮，并为其添加一个 id：

```html
<button id="myButton">点击我</button>
复制代码
```

2.  在 JavaScript 代码中使用 layui.event.bind 方法绑定点击事件：

```javascript
layui.use(['layer', 'jquery'], function() {
  var layer = layui.layer;
  var $ = layui.jquery;

  $('#myButton').click(function() {
    layer.msg('按钮被点击了！');
  });
});
复制代码
```

以上代码绑定了一个点击事件处理函数，当按钮被点击时，会弹出一个提示框。

希望对你有所帮助。

https://juejin.cn/s/layui%E7%9B%91%E5%90%AC%E6%8C%89%E9%92%AE%E7%82%B9%E5%87%BB%E4%BA%8B%E4%BB%B6
