# JavaScript Window - 浏览器对象模型

**浏览器对象模型（Browser Object Model (BOM)）允许 JavaScript 与浏览器对话。**

## 浏览器对象模型（Browser Object Model (BOM)）

不存在浏览器对象模型（BOM）的官方标准。

现代的浏览器已经（几乎）实现了 JavaScript 交互相同的方法和属性，因此它经常作为 BOM 的方法和属性被提到。

## Window 对象

所有浏览器都支持 **window** 对象。它代表浏览器的窗口。

所有全局 JavaScript 对象，函数和变量自动成为 window 对象的成员。

全局变量是 window 对象的属性。

全局函数是 window 对象的方法。

甚至（HTML DOM 的）document 对象也是 window 对象属性：

```js
window.document.getElementById("header");
```

等同于：

```js
document.getElementById("header");
```

## 窗口尺寸

两个属性可用用于确定浏览器窗口的尺寸。

这两个属性都以像素返回尺寸：

- window.innerHeight - 浏览器窗口的内高度（以像素计）
- window.innerWidth - 浏览器窗口的内宽度（以像素计）

浏览器窗口（浏览器视口）不包括工具栏和滚动条。

对于 Internet Explorer 8, 7, 6, 5：

- document.documentElement.clientHeight
- document.documentElement.clientWidth

或

- document.body.clientHeight
- document.body.clientWidth

一个实用的 JavaScript 解决方案（包括所有浏览器）：

**实例**

```js
var w = window.innerWidth
|| document.documentElement.clientWidth
|| document.body.clientWidth;

var h = window.innerHeight
|| document.documentElement.clientHeight
|| document.body.clientHeight; 
```

该例显示浏览器窗口的高度和宽度：（不包括工具栏和滚动条）

## window其它方法

+ window.open()  打开新窗口
+ window.close()  关闭窗口
+ window.moveTo() -移动当前窗口
+ window.resizeTo() -重新调整当前窗口

## window.location

+ window.location.hostname   获取主机名
+ window.location.href      获取地址
+ window.location.href = url  地址跳转
+ window.location.port   获取端口
+ window.location.assign   加载新文档

### 前进/后退

+ history.back()  后退 与浏览器中后退按钮一致
+ history.forward()    前进 与浏览器中前进按钮一致
+ history.go(+1)  前进
+ history.go(-1)  后退

前进.HTML

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<a href="./5后退.html">前进</a>
<p><button onclick="history.go(+1)">前进</button></p>
<p><button onclick="history.forward()">前进</button></p>
</body>
</html>
```

后退.html

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<p><button onclick="history.go(-1)">后退</button></p>
<p><button onclick="history.back()">后退</button></p>
</body>
</html>
```

## js弹窗

+ 警告框

  window.alert(消息)

+ 确认框

  用户接收消息的确认

  window.confirm(消息)

+ 提示框

  window.prompt(消息)   [prɑːmpt]

+ ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <script>
          window.alert('警\n告框')
          window.confirm('确认框')
          if(window.confirm('你要去百度吗？')){
              window.location.href = 'http://www.baidu.com'
          }else{
              console.log('取消')
          }
          console.log(window.prompt('消息'))
      </script>
  </head>
  <body>
  
  </body>
  </html>
  ```

