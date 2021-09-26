# JavaScript 修改CSS样式

### 1、style

+ 格式

  节点.style.属性 = '样式'

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style>
          div{
              background-color: green;
          }
      </style>
  </head>
  <body>
  <div style="border:1px solid red;">div标签</div>
  </body>
  </html>
  <script>
      var div = document.getElementsByTagName('div')[0];
      // console.log(div.style);
      // 设置样式
      div.style.color = 'red';
      //  错误写法  在js里为小驼峰
      // div.style.font-size = 'red';
      div.style.fontSize = '20px';
      // 获取行内样式边框
      console.log(div.style.border);
      // 获取内嵌样式中背景色（不能获取行内样式以外的样式）
      console.log(div.style.backgroundColor)
  </script>
  ```

+ 注意

  + 样式使用小驼峰命名规则   

    backgroundColor

  + 只能获取行内样式

### 2、获取样式

+ 格式

  window.getComputedStyle('元素','伪类')

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style>
          div{
              background-color: green;
          }
      </style>
  </head>
  <body>
  <div style="border:1px solid red;">div标签</div>
  </body>
  </html>
  <script>
      var div = document.getElementsByTagName('div')[0];
      // 获取所有样式
      console.log(window.getComputedStyle(div, null).backgroundColor)
      console.log(window.getComputedStyle(div, null).border)
  </script>
  ```

+ 注意

  和style的区别

  + style只能获取行内样式
  + getComputedStyle获取所有样式



### 3、设置多个css样式

+ cssText   设置多个css样式

  + 格式

    节点.style.cssText = '多个样式'

  + 实例

    ```javascript
    var div = document.getElementById('div');
    // console.log(div);
    div.style.cssText = 'color:green;font-size:20px;'
    ```

+ setAttribute  设置属性

  + 格式

    节点.setAttribute(属性名,属性值)

  + 实例

    ```javascript
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <style>
            .div{
                color:green;font-size:20px;
            }
        </style>
    </head>
    <body>
    <!--<div id="div" class="div">div标签</div>-->
    <div id="div">div标签</div>
    </body>
    </html>
    <script>
        var div = document.getElementById('div');
        // console.log(div);
        // div.setAttribute('style', 'color:green;font-size:20px;')
        div.setAttribute('class', 'div')
    </script>
    ```

  + 设置属性的其他写法

    节点.属性名 = 值

    ```javascript
    div.className = 'div';
    ```

  + classList  获取/添加多个css选择器

    ```javascript
    // 设置多个类名称
    div.classList.add('div1', 'div2')
    // 移除样式
    div.classList.remove('div1')
    div.classList.remove('div1', 'div2')
    ```

  + 设置和更改类样式(添加/移除)

    ```javascript
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Title</title>
        <style>
            .div1{
                color:green;
                font-size:20px;
            }
            .div2{
                border:1px solid red;
                font-size:40px;
            }
        </style>
    </head>
    <body>
    <!--<div id="div" class="div">div标签</div>-->
    <div id="div" class="div1 div2">div标签</div>
    <!--<div id="div">div标签</div>-->
    </body>
    </html>
    <script>
        var div = document.getElementById('div');
        div.setAttribute('class', 'div1 div2');
        div.className = 'div1 div2';
        // 获取类名称
        console.log(div.classList)
        // 设置多个类名称
        div.classList.add('div1', 'div2')
        // 移除样式
        div.classList.remove('div1')
        div.classList.remove('div1', 'div2')
        div.className = 'div1 div2';
        div.className = 'div1';
        div.className = '';
        div.setAttribute('class', 'div1 div2');
        div.setAttribute('class', '');
    </script>
    ```

