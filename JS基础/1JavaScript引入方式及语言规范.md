## JavaScript引入方式

+ 在header头部内嵌

  `<script type="text/javascript"></script>`

+ 在header头部导入

  `<script type="text/javascript" src="**.js"></script>`

+ 给HTML绑定js事件

  `<button onclick="js代码"></button>`

  ```javascript
  <button onclick="alert('你是不是点我了小老弟')">点击</button>
  ```

注意：

+ 在外部导入的标签内 不要写任何的js代码

```javascript
<!--错误的写法  外部导入内不要写代码-->
<script src="script.js">alert('你是不是点我了小老弟');</script>
```

+ 外部导入标签是双标签，不要写成单标签

## 注释

```javascript
// 这是单行注释 

/* 这是多行
注释*/

```

## 输出方式

+ 控制台输出

  console.log('输出到控制台')

+ 弹框

  alert('内容')

+ 输出到浏览器上(body体内)

  document.write('内容')

## Script标签

script可以出现多次，并且可以出现在HTML文件中的任何地方。但是建议写在head头部之间，在同一个HTML文件中，js和html代码都是自上而下而执行的，谁在前面先执行谁

## 变量

### 命名规范

+ 变量名是以数字、字母、下划线和$组成
+ 第一个字符不能为数字
+ 不能使用关键字作为变量名
+ 变量名区分大小写

声明变量使用 `var 变量名;` 的格式来进行声明

```javascript
// 定义变量不赋值 值为undefined
var age;  
console.log(age)

// 定义变量并赋值
var age = 18;
console.log(age);
console.log(typeof age);

// 定义多个变量
var name='lucky',age=18,sex='man';
console.log(name,age,sex);
```

>+ 注意
>
> + js是弱类型语言，只有在赋值的时候，才能确定其类型
>
> + typeof  查看类型
>
> ```javascript
> console.log(typeof 变量);
> ```
>
> + 定义变量最好使用var关键字，在函数外部定义的var变量为全局变量，在函数内部也可以获取。如果在函数内部定义变量，使用var 则为局部变量，如果内部定义变量没有var，则为全局变量，函数内外都可以，会弄脏变量的命名空间

## 常量

ES6新增const用来声明常量。一旦声明，其值就不能改变。

```javascript
const PI = 3.1415;
```

>一旦被定义不能被修改和销毁
>
>建议大写

## 代码规范

+ 保持代码缩进（提高代码的可读性）
+ 变量名见名知意，遵守命名规范
+ js语句结尾建议加上分号
+ 运算符俩侧存在空格



