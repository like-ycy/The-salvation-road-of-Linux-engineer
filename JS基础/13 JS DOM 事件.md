# JavaScript DOM 事件

#### 常见的 HTML 事件

下面是一些常见的 HTML 事件：

| 事件        | 描述                         |
| :---------- | :--------------------------- |
| onchange    | HTML 元素已被改变            |
| onclick     | 用户点击了 HTML 元素         |
| onmouseover | 用户把鼠标移动到 HTML 元素上 |
| onmouseout  | 用户把鼠标移开 HTML 元素     |
| onkeydown   | 用户按下键盘按键             |
| onload      | 浏览器已经完成页面加载       |

#### 事件使用场景

- 当用户点击鼠标时
- 当网页加载后
- 当图像加载后
- 当鼠标移至元素上时
- 当输入字段被改变时
- 当 HTML 表单被提交时
- 当用户敲击按键时

### 1、点击事件onclick

+ 标签中onclick属性

  onclick = 事件

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <script>
          // function test() {
          function test(obj) {
              alert('点击');
              console.log(obj);
          }
      </script>
  </head>
  <body>
  <!--<p onclick="test()">点击</p>-->
  <!--<p onclick="test(this)">点击</p>-->
  <p onclick="this.innerHTML='new'">点击</p>
  </body>
  </html>
  ```

### 2、DOM分配事件

+ 格式

  节点.事件名称 = function（）{}

```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

</head>
<body>
<button>点击</button>
</body>
</html>
<script>
    var but = document.getElementsByTagName('button')[0];
    // console.log(but);
    but.onclick = function () {
        console.log('点击')
    }
</script>
```

### 3、onload

+ 概述

  onload在页面加载完后立即发生

  作为入口

+ 使用

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <script>
          onload = function (ev) {
              var but = document.getElementsByTagName('button')[0];
              but.onclick = function () {
                  console.log('点击')
              }
          }
      </script>
  </head>
  <body>
  <button>点击</button>
  </body>
  </html>
  ```

### 4、onchange 事件

+ 使用场景

  input/select/textarea 

+ 说明

  当内容发生改变进行触发

+ 使用

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <script>
          onload = function (ev) {
              var input = document.getElementsByName('test_input')[0];
              // console.log(input)
              input.onchange = function (ev2) {
                  console.log(ev2);
              }
  
          }
          function test(doc) {
              console.log(doc.value)
          }
      </script>
  </head>
  <body>
  <p><input type="text" value="123" name="test_input"></p>
  <select name="p" id="" onchange="test(this)">
      <option value=""></option>
      <option value="1001">北京</option>
      <option value="1002">上海</option>
      <option value="1003">深圳</option>
      <option value="1004">广州</option>
  </select>
  </body>
  </html>
  ```

### 5、onsubmit 点击事件

+ 概述

  当表单提交以后 进行触发

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <script>
          function check_form() {
              var username = document.getElementsByName('username')[0].value;
              if(username){
                  console.log(username)
              }
              return false;
  
          }
      </script>
  </head>
  <body>
  <form action="2.html" onsubmit="return check_form();">
      <p>用户名：<input type="text" name="username"><span>请输入正确的手机号码</span></p>
      <p><input type="submit" value="提交"></p>
  </form>
  </body>
  </html>
  ```

### 6、onfocus()/onblur()  获取焦点失去焦点

+ 说明

  onfocus 获取焦点时触发

  onblur  失去焦点时触发

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <script>
          onload = function (ev) {
              var username = document.getElementsByName('username')[0];
              username.onfocus = function () {
                  console.log('获取焦点');
              }
              username.onblur = function () {
                  console.log('失去焦点')
              }
          }
      </script>
  </head>
  <body>
  <input type="text" name="username">
  <!--<input type="text" onfocus="alert('focus')" onblur="alert('blur')">-->
  <input type="text" onfocus="console.log('focus')" onblur="console.log('onblur')">
  </body>
  </html>
  ```

### 7、onmouseover()/onmouseout()  鼠标移入移出

+ 鼠标移入

  onmouseover()  当鼠标移入进行触发

+ 鼠标移出进行触发

  onmouseout()  当鼠标移出进行触发

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style>
          div{
              width:100px;height: 100px;border:1px solid red;
          }
      </style>
  </head>
  <body>
  <div onmouseover="console.log('over')" onmouseout="console.log('out')"></div>
  </body>
  </html>
  ```

### 8、onmouseenter()/ onmouseleave()

+ 鼠标移入

  onmouseenter()  当鼠标移入进行触发

+ 鼠标移出进行触发

  onmouseleave()  当鼠标移出进行触发

+ 注意

  单独使用的时候都一样的没有区别

  当标签发生嵌套  有区别的

### 9、onmousemove()  鼠标移动

+ 说明

  当鼠标移动的时候 进行触发

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style>
          div{
              width:200px;height: 200px;border:1px solid red;
          }
      </style>
  </head>
  <body>
  <div></div>
  </body>
  </html>
  <script>
      var div = document.getElementsByTagName('div')[0];
      // console.log(div);
      var i = 1;
      div.onmousemove = function () {
          console.log(i++);
      }
  </script>
  ```

### 10、onmousedow/onmouseup/onclick

+ 鼠标单击

  onmousedow

+ 鼠标抬起

  onmouseup

+ 点击 

  onclick

+ 实例

  ```javascript
  <p onmousedown="console.log('down')" onmouseup="console.log('up')" onclick="console.log('click')">点击</p>
  ```

  

### 11、事件对象event(隐藏参数)

+ 鼠标事件
  + clientX  浏览器可视区域X坐标
  + clientY   浏览器可视区域Y坐标
  + pageX  参照浏览器内容区域左上角 不会随着滚动条而变动
  + pageY  参照浏览器内容区域y坐标
  + screenX  显示器屏幕的x坐标
  + screenY  显示器屏幕的y坐标
  + offsetX   鼠标点击元素距离左侧的坐标
  + offsetY     鼠标点击元素距离右侧的坐标



```javascript
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<style>
    body{
        height: 1000px;
    }
</style>
<body>

</body>
</html>
<script>
    document.onmousedown = function (ev) {
        var e = ev;
        console.log(e.clientX,e.clientY);
        console.log(e.screenX,e.screenY);
        console.log(e.pageX,e.pageY);
    }
</script>
```



### 12、键盘事件

+ onkeydown  按住键盘不放会一直触发 任意键

+ onkeyup  释放键盘进行触发

+ onkeypress   按住键盘不放会一直触发 字符键

+ 属性

  keycode

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  
  </body>
  </html>
  <script>
      // document.onkeydown = function (ev) {
      document.onkeypress = function (ev) {
          var e = ev;
          // console.log(e.keyCode);
          console.log(String.fromCharCode(e.charCode));
      }
  </script>
  ```




