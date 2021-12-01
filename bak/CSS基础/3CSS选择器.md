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

  

## CSS3选择器

### 1、关系选择器

+ div > p 选择div的子元素p

+ div+p   选择紧贴在div后面的p元素

+ div ~ p 选择div后面的所有同级元素p

+ div p   div里面所有的p元素

+ 实例

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style>
          ul li{
          ul>li{
          ul+li{
          ul~li{
              border-bottom: 1px solid red;
          }
  
      </style>
  </head>
  <body>
      <ul>
          <li>我是li标签</li>
          <li>我是li标签</li>
          <li>我是li标签</li>
          <ol>
              <li>ol里面的li</li>
              <li>ol里面的li</li>
              <li>ol里面的li</li>
          </ol>
      </ul>
      <li>外部的li1</li>
      <li>外部的li2</li>
      <li>外部的li3</li>
  </body>
  </html>
  ```

### 2、属性选择器

+ [attr]  选择具有attr属性的元素

+ [attr=value] 选择attr属性等于value的元素

+ [attr^=value] 选择attr属性以value作为开头的元素

+ [attr$=value] 选择attr属性以value作为结尾的元素

+ [attr*=value] 选择attr属性包含value的元素

+ 实例

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style>
          /*选择input标签具有name属性的元素*/
          /*input[name]{*/
          /*input[name=username]{*/
          /*input[name^=u]{*/
          /*input[name$=d]{*/
          input[name*=s]{
              border:10px dotted green;
              color:red;
          }
      </style>
  </head>
  <body>
  <p><input type="text" name="username"></p>
  <p><input type="password" name="password"></p>
  <p><input type="text"></p>
  <p><input type="text" value="123456"></p>
  </body>
  </html>
  ```

### 3、结构性伪类选择器

+ :first-letter    设置对象内第一个字符的样式

+ :first-line   设置对象内第一行的样式

+ :before  设置在对象前发生的内容

+ :after  设置在对象后发生的内容

  实例

  ```css
  span:before{
  span:after{
      content: 'before';
      display: block;
      border:1px solid red;
      width: 300px;
      height: 300px;
  }
  <span>span标签</span>
  ```

+ :first-of-type  匹配同类型中的第一个同级兄弟元素

+ :last-of-type  匹配同类型中的最后一个同级兄弟元素

+ :first-child  匹配父元素的第一个子元素

+ :last-child  匹配父元素的最后一个子元素

+ :nth-child(n)  匹配父元素的第n个子元素

  nth-child(2n)  代表偶数行

  nth-child(2n+1)  代表奇数行

+ :nth-last-child(n)  匹配父元素的倒数第n个子元素

+ 区别：

  type匹配的第一个/最后一个元素 无论前面是否有其他同级兄弟元素  都不会受影响

  child匹配的第一个/最后一个元素   那么第一个/最后一个必须为匹配的元素

+ :not(e)  匹配当前不包含e选择器的元素

  ```css
  ul li:not(:last-child){
      color:red;
  }
  ```

### 4、状态伪类选择器

+ :link  设置超链接未被访问之前的样式

+ :visited 设置超链接访问后的样式

+ :hover  设置鼠标悬停时候的样式

+ :active  设置激活时候的样式

+ :focus  设置获取焦点时候的样式

+ :enabled  匹配可用状态的元素

+ :disabled 匹配处于禁用状态的元素

+ :checked 匹配处于选中状态的元素

  ```
  input:checked+span:after{
      content: '你被选择了';
  }
  <p><input type="radio" name="sex" value="1">radio<span></span></p>
  ```

+ :selection  设置对象被选择时的样式

### 5、important 应用优先权

+ 实例

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style>
          p{
              color:red !important;
          }
          p{
              color:green;
          }
      </style>
  </head>
  <body>
  <p>样式</p>
  </body>
  </html>
  ```





