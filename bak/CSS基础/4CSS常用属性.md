# CSS常用属性

### 1、颜色属性

+ 属性名 color

+ 格式

  {color:值;}

+ 值

  + 英文单词

    red green

  + 十六进制   前两位代表红  中间俩位绿  最后俩位为蓝

    \#0-F    #00FF00

  + rgb(0-255,0-255,0-255)  分别代表红  绿 蓝

  + rgba(0-255,0-255,0-255,0-1)  最后一个代表透明度

+ opacity  设置图片透明度（0-1）

  ```css
  img{
      opacity: 0.5;
  }
  ```



### 2、font 字体

属性：font

分开属性：

+ font-size  字体大小
+ font-family  字体
+ font-style normal默认  italic 倾斜
+ font-weight  bold加粗

集体书写：

```css
font:italic bold 20px 宋体,楷体;
font:italic bold 20px/2 宋体,楷体;  /*设置字体并设置行高*/
```

注意：如果只使用font属性  则font-size 和font-family俩个属性必须有值

### 3、文本属性

+ text-indent   首行缩进  2em

+ text-overflow   文本的溢出是否显示省略标记

  注意：需要配合超出隐藏  阻止换行 设定宽度 最后搭配text-overflow

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Title</title>
      <style>
          li{
              list-style: none;
              border-bottom: 1px solid #ccc;
              height: 30px;
              line-height: 30px;
              width: 350px;
              overflow: hidden;
              text-overflow: ellipsis;
              white-space:nowrap;
          }
      </style>
  </head>
  <body>
  <div>
  </div>
  <ul>
      <li>今天晨报周日周一天气有雨今天晨报周日周一天气有雨今天晨报周日周一天气有雨</li>
      <li>今天晨报周日周一天气有雨今天晨报周日周一天气有雨</li>
      <li>今天晨报周日周一天气有雨今天晨报周日周一天气有雨</li>
      <li>今天晨报周日周一天气有雨今天晨报周日周一天气有雨</li>
      <li>今天晨报周日周一天气有雨今天晨报周日周一天气有雨</li>
  </ul>
  </body>
  </html>
  ```

+ text-align  文本位置 left/center/right

+ text-decoration  字体划线   none无  underline下划线  line-through 贯穿线

+ text-shadow  文本的文字是否有阴影以及模糊效果

  + 参数1  h-shadow  水平阴影的位置 可以为负值
  + 参数2  v-shadow  垂直阴影的位置  可以为负值
  + 参数3  blur    模糊的距离
  + 参数4 color   阴影的颜色颜色

  ```css
  text-shadow:-3px 3px 3px red;
  ```

+ white-space:nowrap;  使内容在一行显示  不让换行

+ letter-spacing   文字或字母的间距

+ word-spacing  单词间距

+ line-height   行高   当前行的上下距离  如果行高和高度为同一值 则内容为垂直居中

+ word-brek:break-all;  强制换行

### 4、border边框

总体设置

border:line-width  line-style  color;  

+ line-width 代表边框宽度

+ line-style 代表边框样式

  none | hidden | dotted |  dashed | solid | double | groove | ridge | inset | outset

单边设置

border-left/top/right/bottom:line-width  line-style  color;  

单边的三个属性分别设置

border-left/top/right/bottom-color

border-left/top/right/bottom-style

border-left/top/right/bottom-width

### 5、background背景

+ background-color  设置背景色

+ background-image  设置背景图片

+ background-repeat    是否平铺 no-repeat   repeat-x  repeat-y

+ background-position   背景定位

  + center  居中
  + left   左侧
  + right  右侧
  + top 顶部
  + bottom  底部

+ background-size   背景图片大小

+ background-attachment  是否固定背景

  + scroll  默认值   随着滚动条滚动
  + fixed  固定

+ 总体书写：

  ```css
  background: aqua url("https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fww4.sinaimg.cn%2Fmw690%2F004j3Ynnly1guipld160wj61jk223e8102.jpg&refer=http%3A%2F%2Fwww.sina.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1634529647&t=fc73f5ecf27be9e117bdcb59db54cce1") no-repeat scroll center center/100px 100px;
  ```

  + 注意：背景色或者背景图片需要存在

+ 多重背景图

  ```css
  background: url("./game/map_11.gif") repeat-x left 100px top 300px,url("./game/map_03.gif");
  ```

  + 注意：前面的背景图 会覆盖掉后面的背景图



### 6、内补白(内补丁)

+ padding

+ 单边设置

  + padding-top    设置顶部的内边距
  + padding-bottom  设置底部的内边距
  + padding-left   设置左侧的内边距
  + padding-right  设置右侧的内边距

+ 单个参数

  + 设置四边

+ 俩个参数

  + 上下  左右

+ 三个参数

  + 上 左右 下

+ 四个参数

  + 上 右 下 左

+ 注意

  会增加原本的宽度和高度

### 7、margin  外补白（外补丁）

+ margin

+ 单边设置

  + margin-top    设置顶部的外边距
  + margin-bottom  设置底部的外边距
  + margin-left   设置左侧的外边距
  + margin-right  设置右侧的外边距

+ 单个参数

  + 设置四边

+ 俩个参数

  + 上下  左右

+ 三个参数

  + 上 左右 下

+ 四个参数

  + 上 右 下 左

+ 注意

  会增加原本的宽度和高度



### 8、position  定位

+ position

  + absolute  绝对定位
  + fixed  固定定位
  + relative  相对定位（用于样式微调）

+ 值

  + top   上

  + bottom  下

  + left   左

  + right  右

  + 注意

    绝对定位

    + 如果标签嵌套/不嵌套 内部标签为绝对定位  则参照body体
    + 如果嵌套标签都有定位  则参照外层的定位  否则参照body体
    + 如果当前存在浮动(当前摆放位置)  如果设置了绝对定位 则不会保留物理空间

    固定定位

    + 和absolute一样   区别在于不会随着滚动条的滚动而滚动(不动)
    + 如果当前存在浮动(当前摆放位置)  如果设置了固定定位 则不会保留物理空间

    相对定位

    + 不会更改整个文本流  只会更改当前的代码块

+ z-index  层叠顺序 值大的在上方

+ div居中展示

  ```css
  width: 400px;
  height: 180px;
  background-color: #000;
  position: absolute;
  left: 0;
  right: 0;
  top:0;
  bottom: 0;
  margin: auto;
  ```



### 9、布局

+ display  是否及如何显示

  + none 隐藏
  + block 转换为块级标签
  + inline-block  行级块标签
  + inline   转换为行级标签

+ float  浮动

  + *left   左浮动
  + right    右浮动

  注意：整体漂浮起来了 

+ clear  清除浮动

  + left  清除左浮动
  + right  清除右浮动
  + none  不清楚
  + *both  清除左右浮动

+ *overflow  超出隐藏

  + hidden  清除浮动与超出隐藏的作用(当有标签嵌套的时候  给外层标签清除浮动)

  + overflow-x

  + overflow-y

  + 属性值

    - visible： 

      对溢出内容不做处理，内容可能会超出容器。 

    - hidden： 

      隐藏溢出容器的内容且不出现滚动条。 

    - scroll： 

      隐藏溢出容器的内容，溢出的内容将以卷动滚动条的方式呈现。 

    - auto： 

      当内容没有溢出容器时不出现滚动条，当内容溢出容器时出现滚动条，按需出现滚动条。此为body对象和textarea的默认值。

+ visibility   隐藏

  ```css
  visibility: hidden; /隐藏 保留物理空间/
  ```

+ 盒子模型

  + 概述

    css定义所有的元素 都可以像拥有盒子一样的外形和平面空间 即都包含了 边框(border)  外边距(margin)  内边距(padding) 内容区(content)

  + 计算公式

    width = margin-left+padding-left+content+padding-right+margin-right+border-left+border-right

    height = margin-top+padding-top+content+padding-bottom+margin-bottom+border-top+border-bottom

  

+ 怪异盒子模型

  + 计算公式

    宽 = 从边框左边到边框右边的距离则为宽度距离

    高 =  从边框上边到边框下边边的距离则为宽度距离

  + box-sizing:border-box;



### 10、其它属性

+ cursor  光标形状   poninter
+ border-collapse    collapse  合并表格  [kəˈlæps]
+ transition  过渡效果
+ list-style:none;  去掉自带li 的样式
  + list-style-image   设置li的点的图片样式
  + list-style-position  设置inside  outside
  + list-style-type   设置左侧圆点的类型