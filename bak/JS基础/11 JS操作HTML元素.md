# JavaScript Document 对象操作HTML元素

#### 查找 HTML 元素

| 方法                                    | 描述                   |
| :-------------------------------------- | :--------------------- |
| document.getElementById(*id*)           | 通过元素 id 来查找元素 |
| document.getElementsByTagName(*name*)   | 通过标签名来查找元素   |
| document.getElementsByClassName(*name*) | 通过类名来查找元素     |

#### 改变 HTML 元素

| 方法                                       | 描述                   |
| :----------------------------------------- | :--------------------- |
| element.innerHTML = *new html content*     | 改变元素的 inner HTML  |
| element.attribute = *new value*            | 改变 HTML 元素的属性值 |
| element.setAttribute(*attribute*, *value*) | 改变 HTML 元素的属性值 |
| element.style.property = *new style*       | 改变 HTML 元素的样式   |

#### 添加和删除元素

| 方法                              | 描述             |
| :-------------------------------- | :--------------- |
| document.createElement(*element*) | 创建 HTML 元素   |
| document.removeChild(*element*)   | 删除 HTML 元素   |
| document.appendChild(*element*)   | 添加 HTML 元素   |
| document.replaceChild(*element*)  | 替换 HTML 元素   |
| document.write(*text*)            | 写入 HTML 输出流 |

## 一、查找HTML元素

通过js，进行HTML的操作

我们常用的查找标签(节点) 常用方式

+ 通过id名称进行查找
+ 通过class名称进行查找(数组)
+ 通过标签名称进行查找(数组)

### 1、通过id进行查找

在dom中，查找HTML最简单的方式就是通过id

+ 结构

  document.getElementById('id名称')

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  <div id="div">div标签</div>
  </body>
  </html>
  <script>
      var div = document.getElementById('div');
      console.log(div)
  </script>
  ```

### 2、通过class进行查找

+ 通过class选择器进行查找 需要注意的时候，返回为**数组** ，因为class选择器可以多次使用

+ 格式

  document.getElementsByClassName('class名称')

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  <div class="div">div1</div>
  <div class="div">div2</div>
  </body>
  </html>
  <script>
      var div = document.getElementsByClassName('div');
      console.log(div[0])
      console.log(div[1])
  </script>
  ```

### 3、通过标签名称

+ 通过标签名称进行查找，返回**数组**

+ 格式

  document.getElementsByTagName('标签名称')

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  <div>div1</div>
  <div>div2</div>
  </body>
  </html>
  <script>
      var div = document.getElementsByTagName('div');
      console.log(div[0])
      console.log(div[1])
      console.log(div[2])
  </script>
  ```

### 4、通过name名称

+ 通过文本域的name名称获取，返回**数组**

+ 格式

  document.getElementsByName('username');

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  <input type="text" name="username" value="1">
  </body>
  </html>
  <script>
      var username = document.getElementsByName('username');
      console.log(username[0])
  </script>
  ```

  

## 二、改变HTML

### 1、改变标签的内容

+ 属性

  可以读取/设置

  + innerHTML
  + innerText

+ 格式

  + 设置值

    节点.innerHTML = '值'

    节点.innerText = '值'

  + 读取

    节点.innerHTML

    节点.innerText

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  <div><b>div标签</b></div>
  </body>
  </html>
  <script>
      var div = document.getElementsByTagName('div')[0];
      console.log(div);
      // 读取
      console.log(div.innerHTML);
      console.log(div.innerText);
      // 设置
      div.innerHTML = '<i>js设置</i>';
      div.innerText = 'js设置';
  </script>
  ```

  

### 2、改变属性

+ 格式

  节点.属性(attribute) = '值'

+ 实例

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
  </head>
  <body>
  <img src="https://img0.baidu.com/it/u=2976014774,2591229928&fm=26&fmt=auto" alt="" onclick="this.src='https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fn.sinaimg.cn%2Fsinacn11%2F607%2Fw870h537%2F20180814%2F6f7c-hhtfwqq7160443.png&refer=http%3A%2F%2Fn.sinaimg.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1635210996&t=29f4ea7d2a590cc3653c05d61ce2f470'">
  </body>
  </html>
  <script>
      var img = document.getElementsByTagName('img')[0];
      // 读取src属性
      console.log(img.src);
      img.src = 'https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fn.sinaimg.cn%2Fsinacn11%2F607%2Fw870h537%2F20180814%2F6f7c-hhtfwqq7160443.png&refer=http%3A%2F%2Fn.sinaimg.cn&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1635210996&t=29f4ea7d2a590cc3653c05d61ce2f470';
  </script>
  ```

+ 密码的显示与隐藏

  ```javascript
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style>
          span{
              cursor: pointer;
          }
      </style>
  </head>
  <body>
  <input type="password" name="userpass"><span onclick="m_change(this)">显示</span>
  </body>
  </html>
  <script>
      function m_change(o) {
          var userpass = document.getElementsByName('userpass')[0];               
          var text = o.innerHTML;
          if(text == '显示'){
              userpass.type = 'text';
              o.innerHTML = '隐藏';
          }else{
              userpass.type = 'password';
              o.innerHTML = '显示';
          }
      }
  </script>
  ```


