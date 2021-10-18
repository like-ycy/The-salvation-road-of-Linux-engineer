# HTML标签

标签分为：单标记和双标记

如：

`<b>加粗</b>`

`<br />`

## 主体结构标签

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

</body>
</html>
```

+ 结构说明

  + H5的头 声明文档类型为HTML

    ```html
    <!DOCTYPE html>
    ```

  + html  最外层的标签结构  所有的HTML都要写在这个标签内

  + head 头部分 网页总体信息

    + title  页面的标题
    + meta 网页主体信息
    + link   引入外部文件
    + script  引入js文件/书写js代码
    + style   书写css样式

  + body  身体部分 网页内容

### 1、文本标签

+ `<p></p>`  段落标签 自成一段 会将上下的文字和它保持一定的距离

+ `<br />`  换行标签  单标签 自闭和

+ `<i></i><em></em>`  斜体/强调斜体

+ `<b></b><strong></strong>`  加粗/强调加粗

+ `<h1></h6>`  标题标签 字体加粗 自占一行   建议一个页面最多一个h1标签

+ `<sub></sub><sup></sup>`  下标和上标

+ `<hr />`  水平分割线

+ `<u></u>` 下划线

+ `<del></del>` 删除线

+ `<font size="1-7" color="颜色" face="设置字体"></font>`  设置字体 大小 颜色 字体样式

+ `<sapn></span>`  无意义标签  行级标签

  行级：可以在一行进行显示 不能设置宽高

  块级：自占一行 可以设置宽高

  行级的块标签：可以多个标签在一行展示 并且可以设置宽高

### 2、图片标签

+ 主体结构

  `<img />`

+ 属性

  + src  图片地址(必须)
  + alt  图片加载失败时的显示信息
  + title  文字提示信息
  + width 宽度
  + height 高度
  + border 边框

+ 实例

  ```html
  <img src="./meinv2.jpg"  alt="美女走了？" title="图片标签" width="500" height="300" border="10">
  ```

+ 注意

  当给定宽或者高  则图片为等比缩放



### 3、路径

+ 相对路径

  ./

  ../

  文件名

+ 绝对路径

  一个固定的链接地址

  从磁盘跟下一直到文件的路径也为绝对路径



### 4、超连接

+ 主体结构

  `<a href="跳转的地址" title="提示信息" targer="打开方式">显示内容</a>`

+ target属性值

  + _blank  新窗口的形式打开
  + _self  本窗口打开 默认
  + _parent  父窗口打开
  + _top 顶级窗口打开
  + framename  分帧窗口打开

+ 实例

  ```
  <a href="1图片标签.html" title="提示信息" target="_blank">打开文件</a>
  ```

+ 锚点

  + 定义锚点

    `<a id=""></a>`

  + 使用锚点

    `<a href="#锚点名称"></a>`



### 5、列表

+ 无序列表*

  ```html
  <ul>
      <li>内容</li>
      ...
  </ul>
  ```

  type属性（了解）

  + square 方形
  + disc 默认圆点
  + circle  空心圆

+ 有序列表

  ```html
  <ol>
      <li>内容</li>
      ...
  </ol>
  ```

  type属性

  + i
  + a
  + A
  + 1

+ 自定义列表

  ```html
  <dl>
  	<dt>列表头</dt>
      <dd>列表内容</dd>
  </dl>
  ```



### 6、注释

+ 结构

  ```html
  <!--注释内容-->
  ```

  多行注释

+ 作用

  代码调试

  解释说明

### 7、HTML 中有用的字符实体

**注释：**实体名称对大小写敏感！

| 显示结果 | 描述              | 实体名称          | 实体编号 |
| :------- | :---------------- | :---------------- | :------- |
|          | 空格              | &nbsp;            | &#160;   |
| <        | 小于号            | &lt;              | &#60;    |
| >        | 大于号            | &gt;              | &#62;    |
| &        | 和号              | &amp;             | &#38;    |
| "        | 引号              | &quot;            | &#34;    |
| '        | 撇号              | &apos; (IE不支持) | &#39;    |
| ￠       | 分（cent）        | &cent;            | &#162;   |
| £        | 镑（pound）       | &pound;           | &#163;   |
| ¥        | 元（yen）         | &yen;             | &#165;   |
| €        | 欧元（euro）      | &euro;            | &#8364;  |
| §        | 小节              | &sect;            | &#167;   |
| ©        | 版权（copyright） | &copy;            | &#169;   |
| ®        | 注册商标          | &reg;             | &#174;   |
| ™        | 商标              | &trade;           | &#8482;  |
| ×        | 乘号              | &times;           | &#215;   |
| ÷        | 除号              | &divide;          | &#247;   |

如需完整的实体符号参考，请访问我们的 [HTML 实体符号参考手册](https://www.w3school.com.cn/tags/html_ref_entities.html)。

- [HTML5 代码约定](https://www.w3school.com.cn/html/html5_syntax.asp)
- [HTML 符号](https://www.w3school.com.cn/html/html_symbols.asp)



## table表格

+ 主体结构

  ```html
  <table>
      <caption>表格标题</caption>
      <tr>
          <th></th>
      </tr>
      <tr>
          <td></td>
      </tr>
  </table>
  ```

+ 属性

  + width 宽度
  + height 高度
  + border 边框
  + cellpadding/cellspacing 内边距/外边距
  + background背景图片
  + bgcolor 背景色
  + align  位置  left/center/right
  + rowspan  合并行*
  + colspan 合并列*

## form表单

+ 标签

  `<form></form>`

### 1、form属性

+ action  表单提交地址

+ method  提交的方式 

  + get   默认不写为get  url地址栏可见

    长度受限制  IE2k  火狐8K

  + post

    url地址栏不可见

    相对安全

+ enctype  提交类型

  只有在文件上传的时候需要更改其值  multipart/form-data

### 2、select标签

+ 属性

  + *name  定义名称 存储值
  + size  定义可见数目
  + disabled 禁用
  + multiple  多选

+ 内嵌标签

  + `<option></option>`  存储下拉值

  + 属性

    *value 存储下拉值

    selected  默认选中

### 3、textarea  多行文本域

+ 属性

  + *name 存储值
  + cols   设定宽度
  + rows 设定高度
  + disabled  禁用（数据作废 不会传递给后台）
  + readonly  只读
  + placeholder  提示信息

+ 注意

  默认值放在俩个标签中

### 4、button 按钮

+ 标签

  `<button></button>`

+ type

  + submit  默认为submit 提交
  + button   点击(无效果)  配合js来使用
  + reset  重置  恢复到第一次打开时候的状态

### 5、optgroup分组

+ 作用

  给下拉列表进行分组

+ 标签

  `<optgroup></optgroup>`  分组

+ 实例

  ```html
  <select name="address">
      <optgroup label="直辖市">
          <option value="1001">北京</option>
          <option value="1001">天津</option>
          <option value="1001">上海</option>
      </optgroup>
      <optgroup label="省份">
          <option value="1003" selected>河南</option>
          <option value="1002">黑龙江</option>
          <option value="1004">河北</option>
      </optgroup>
  </select>
  ```

### 6、input标签

+ 结构

  `<input type="" />`

+ type属性  表示表单项的类型

  + text  单行文本域
  + password 密码输入框
  + checkbox   复选框
  + radio  单选框
  + file  文件上传
  + button  普通按钮
  + submit  提交按钮
  + reset  重置按钮
  + image 图片提交按钮（了解）
  + hidden  隐藏域

+ 属性

  name属性 input名称 作为值的存储

  value  值

  size  宽度（了解）

  maxlength  最大输入内容的长度

  minlength   输入的最小长度

  readonly   只读

  disabled  禁用

  checked  默认选中

  placeholder  提示信息

  required  必填属性

  

### 7、HTML5新属性(了解)

+ type属性

  + time 时间
  + date  日期
  + range  区间
  + email 邮箱
  + url  url地址
  + color  颜色
  + number  数值
  + search  搜索

  

  

## HTML5多媒体标签（了解）

### 1、音频

属性

| 属性                                                         | 值       | 描述                                                         |
| :----------------------------------------------------------- | :------- | :----------------------------------------------------------- |
| [autoplay](https://www.w3school.com.cn/html5/att_audio_autoplay.asp) | autoplay | 如果出现该属性，则音频在就绪后马上播放。                     |
| [controls](https://www.w3school.com.cn/html5/att_audio_controls.asp) | controls | 如果出现该属性，则向用户显示控件，比如播放按钮。             |
| [loop](https://www.w3school.com.cn/html5/att_audio_loop.asp) | loop     | 如果出现该属性，则每当音频结束时重新开始播放。               |
| [preload](https://www.w3school.com.cn/html5/att_audio_preload.asp) | preload  | 如果出现该属性，则音频在页面加载时进行加载，并预备播放。如果使用 "autoplay"，则忽略该属性。 |
| [src](https://www.w3school.com.cn/html5/att_audio_src.asp)   | *url*    | 要播放的音频的 URL。                                         |

```html
<audio src="_WStyle - 副本.mp3" controls="controls"></audio>
<audio  controls="controls">
    <source src="_WStyle - 副本.mp3">
</audio>
```

### 2、视频

**new** : HTML5 中的新属性。

| 属性                                                         | 值       | 描述                                                         |
| :----------------------------------------------------------- | :------- | :----------------------------------------------------------- |
| [autoplay](https://www.w3school.com.cn/tags/att_video_autoplay.asp) | autoplay | 如果出现该属性，则视频在就绪后马上播放。                     |
| [controls](https://www.w3school.com.cn/tags/att_video_controls.asp) | controls | 如果出现该属性，则向用户显示控件，比如播放按钮。             |
| [height](https://www.w3school.com.cn/tags/att_video_height.asp) | *pixels* | 设置视频播放器的高度。                                       |
| [loop](https://www.w3school.com.cn/tags/att_video_loop.asp)  | loop     | 如果出现该属性，则当媒介文件完成播放后再次开始播放。         |
| [muted](https://www.w3school.com.cn/tags/att_video_muted.asp) | muted    | 规定视频的音频输出应该被静音。                               |
| [poster](https://www.w3school.com.cn/tags/att_video_poster.asp) | *URL*    | 规定视频下载时显示的图像，或者在用户点击播放按钮前显示的图像。 |
| [preload](https://www.w3school.com.cn/tags/att_video_preload.asp) | preload  | 如果出现该属性，则视频在页面加载时进行加载，并预备播放。如果使用 "autoplay"，则忽略该属性。 |
| [src](https://www.w3school.com.cn/tags/att_video_src.asp)    | *url*    | 要播放的视频的 URL。                                         |
| [width](https://www.w3school.com.cn/tags/att_video_width.asp) | *pixels* | 设置视频播放器的宽度。                                       |

使用

```html
<video src="fun.mp4" controls="controls" poster="meinv.jpg"></video>
```







