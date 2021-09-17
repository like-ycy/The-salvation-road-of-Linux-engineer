# CSS选择器

CSS 选择器用于“查找”（或选取）要设置样式的 HTML 元素。

我们可以将 CSS 选择器分为五类：

- 简单选择器（根据名称、id、类来选取元素）
- [组合器选择器](https://www.w3school.com.cn/css/css_combinators.asp)（根据它们之间的特定关系来选取元素）
- [伪类选择器](https://www.w3school.com.cn/css/css_pseudo_classes.asp)（根据特定状态选取元素）
- [伪元素选择器](https://www.w3school.com.cn/css/css_pseudo_elements.asp)（选取元素的一部分并设置其样式）
- [属性选择器](https://www.w3school.com.cn/css/css_attribute_selectors.asp)（根据属性或属性值来选取元素）

### 1、HTML选择符(HTML选择器)

+ 概述

  就是把HTML作为选择器来使用

+ 实例

  ```css
  div{color:red;...}
  ```

  页面中所有的div标签 都采用此样式

### 2、class选择符（class选择器/类选择器）

+ 定义

  ```css
  .class名称{样式}
  ```

+ 实例

  ```css
    <style type="text/css">
      .my_p{
          font-weight: bold;
          font-size:30px;
      }
  </style>
  </head>
  <body>
  <div class="my_p">class选择器</div>
  <p class="my_p">另一个div标签</p>
  ```

+ 样式限定

  html选择器.class名称{样式}

  限定只有给定的HTML标签并且名称为.class名的才采用此样式

+ 特点

  可以在页面中重复使用

  

### 3、id选择器

+ 定义

  `#id名称{样式}`

+ 使用

  `<标签 id="名称">`

+ 注意

  id选择器只能使用一次

+ 实例

  ```html
  #span{
      /*将行级标签转换为块级标签*/
      display: block;
      width:300px;
      height:300px;
      /*背景色*/
      background-color: green;
  }
  <span id="span">行级标签</span>
  ```

### 4、优先级

+ 当选择器不同时  优先级为

  id 选择器 > class选择器 > html选择器

+ 实例

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="utf-8">
      <style type="text/css">
          #span{
              /*将行级标签转换为块级标签*/
              display: block;
              background-color: green;
          }
          .span{
              width:300px;
              height:300px;
          }
          span{
              /*背景色*/
              background-color: red;
          }
      </style>
  </head>
  <body>
      <span class="span" id="span">行级标签</span>
  </body>
  </html>
  ```



### 5、关联选择符(包含选择器)

+ 格式

  选择器1 选择器2 ...{样式}

+ 比如

  div a{样式}   div里面的所有超链接都采用此样式

+ 实例

  ```html
  <!DOCTYPE html>
  <html lang="en">
      <head>
          <meta charset="utf-8">
          <style type="text/css">
          /*所有 ul里面的li都采用此样式*/
              ul li{
                  /*去掉无序列表的点*/
                  list-style: none;
                  color:red;
              }
          </style>
      </head>
      <body>
          <ul>
              <li>我是li</li>
              <li>我是li</li>
              <li>我是li</li>
          </ul>
          <li>我是li</li>
          <li>我是li</li>
          <li>我是li</li>
      </body>
  </html>
  ```



### 6、组合选择器(选择符组)

+ 格式

  选择器1,选择器2,选择器3...{样式}     选择器1选择器2选择器3都采用此样式

+ 实例

  ```html
  <!DOCTYPE html>
  <html lang="en">
      <head>
          <meta charset="utf-8">
          <style type="text/css">
          /*h1 h2 h3 统一设置此样式*/
              h1,h2,h3{
                  color:red;
              }
          </style>
      </head>
      <body>
      <h1>h1标签</h1>
      <h2>h1标签</h2>
      <h3>h1标签</h3>
      </body>
  </html>
  ```



### 7、*通配符（全局选择器）

+ 说明

  通配符的符号位*  代表的是所有

+ 场景

  用来重置标签样式

+ 实例

  ```html
  <!DOCTYPE html>
  <html lang="en">
      <head>
          <meta charset="utf-8">
          <style type="text/css">
              *{
                  /*去掉自带的点样式*/
                  list-style: none;
                  /*设置颜色*/
                  color:#000;
                  /*去掉划线*/
                  text-decoration: none;
                  /*去掉内边距*/
                  padding: 0;
                  /*去掉外边距*/
                  margin: 0;
              }
          </style>
      </head>
      <body>
      <ul>
          <li>li</li>
          <li>li</li>
          <li>li</li>
          <li>li</li>
      </ul>
      <a href="">超链接</a>
      </body>
  </html>
  ```

### 8、伪类选择器(伪元素选择符)

+ 格式 

  选择器:伪类名{样式}

+ 实例

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="utf-8">
      <style type="text/css">
      /*未访问链接的样式*/
          a:link{
              color:red;
              text-decoration: none;
          }
          /*已访问的样式*/
          a:visited{
              color:green;
          }
          /*鼠标放在连接上*/
          a:hover{
              color:blue;
              font-size: 30px;
          }
          /*激活时候的状态*/
          a:active{
              color:yellow;
              font-size: 40px;
          }
      </style>
  </head>
  <body>
      <a href="http://www.baidu.com" target="_blank">超连接</a>
  </body>
  </html>
  ```

  





