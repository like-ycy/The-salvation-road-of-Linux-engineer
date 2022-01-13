# Typescript

英文官网：https://www.typescriptlang.org/

中文官网：https://www.tslang.cn/

## 介绍

TypeScript 是一种**强类型**的编程语言，它起源于使用JavaScript开发的大型项目，由于JavaScript本身属于**弱类型**语言的局限性，难以胜任和维护大型项目的开发工作。因此微软于2012年推出了TypeScript ，使得其能够胜任开发大型项目。

TypeScript通过在JavaScript的基础上添加静态类型定义构建而成，可以编译为 JavaScript代码来执行。它最大特点是强大的类型系统和对ES6规范的支持，TypeScript托管于GitHub上面。

```
ES6：ECMA2015/2016/2017的简称，刚好是属于ECMA的第6个版本。
javascript：ECMA语法，BOM浏览器对象模型和DOM文档对象模型。js是网景公司推出的浏览器端脚本语言。
欧洲计算机制造协会:简称ECMA
```

### 特点

-   类型系统：类型注解、编译时类型检查、类型推断和类型擦除
-   接口
-   枚举
-   Mixin
-   泛型编程
-   命名空间

从 ECMA 6规范中移植而来的：

-   类
-   模块
-   lambda 函数的箭头语法
-   可选参数以及默认参数
-   元组【事实上就是js里面的数组来的，只是typescript提供了不同的玩法而已，和python里面的元祖不是一回事】
-   await / async



#### JavaScript 与 TypeScript 的关系和区别

TypeScript属于Javascript 的**超集**，扩展了Javascript的语法，现有的Javascript代码可以不经任何改动的情况下在TypeScript环境下运行。同时TypeScript代码，可以通过typescript的编译器转换为纯正的 JavaScript代码，且编译出来的 JavaScript代码能够运行在任何浏览器上。TypeScript 的编译工具也可以运行在任何服务器和任何系统上。

typeScript文件的后缀为.ts。

### 优势

+ 现有大部分的流行的代码编辑器IDE工具都支持typescript，在编写typescript代码时，比原生javascript提示更加友好。

  typescript的类型系统相当于最好的文档对于陌生函数或者类的使用更加透明，易懂。

+ typescript提供的类型系统增强了前端代码的可读性和可维护性，在编译时即可提前发现大部分的错误，不需要项目运行即可提前锁定大部分类型相关错误。

+ 完全支持 es6 规范，编译过后生成的javascript代码可以在任何浏览器上运行，解决了各个前端浏览器对于es6规范在不同程度上的兼容问题。

+ 有活跃的社区，大多数的第三方库都可提供给 ts 的类型定义文件



### 缺点

+   具有一定的学习成本，需要理解接口、泛型编程、枚举类型等一系列概念
+   短期增加开发成本，在原生javascript基础上，多写一些类型的定义.
+   集成到项目构建流程需要一些工作量
+   和一些现有的javascript第三方库或框架的结合存在bug



## 安装

有两种主要的方式来获取TypeScript工具：

-   通过npm（Node.js包管理器）
-   安装Visual Studio的TypeScript插件

在此，我们通过npm来安装/更新typescript

```
# 安装
npm install -g typescript
# 更新到最新版本
npm update -g typescript
# 查看版本
tsc -v
```



## 快速入门

### 创建typescript文件 

main.ts，代码：

```typescript
function main(person) {
    return "Hello, " + person;
}

var user = "Jane User";

document.body.textContent = main(user);
```

### 编译代码

上面代码中，虽然我们创建脚本文件是ts，但是里面的代码却是实实在在的js代码。不过因为基于typescript和javascript的关系，我们可以直接通过typescript编译器进行编译。

main.ts --->编译(tsc)--> main.js

终端执行：

```bash
tsc main.ts
# tsc --out main.js main.ts  # --out 可以指定编译完成以后的js文件名
```

命令执行以后的输出结果为一个main.js文件，它包含了和输入文件中相同的JavsScript代码。

虽然，在上面过程中，我们并没有接触到typescript代码，但是我们掌握了typescript编译器的用法。有没有？

最后，编译完成以后得到的js文件就可以直接通过script标签被html使用，变相地，等同于typescript被使用了。

index.html，代码：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <!-- async是javascript在ES6新出的延时加载关键字 -->
    <script async src="main.js"></script>
</head>
<body>
	
</body>
</html>
```



接下来让我们看看TypeScript工具带来的高级功能。

### 类型注解

main.ts，代码中给`main`函数的参数person添加`: string`类型注解，如下：

```typescript
function main(person: string) {
    return "Hello, " + person;
}

let user = "Jane User";

document.body.textContent = main(user);
```

没错，有没有点熟悉的感觉？没错，python3.5以后的版本中也新增了类型注解功能，在新版本的很多python框架里面都存在着类似的代码。

终端执行：

```bash
tsc main.ts
```

尝试把`person`参数的实参改成传入一个数组：

```typescript
function main(person: string) {
    return "Hello, " + person;
}

let user = [0, 1, 2];

document.body.textContent = main(user);
```

重新编译，你会看到产生了一个错误，当然，类似pycharm这样的IDE工具，要已经有错误提示了。

```
error TS2345: Argument of type 'number[]' is not assignable to parameter of type 'string'.
```

TypeScript告诉你，使用了非期望个数的参数调用了这个函数。在编译过程中，TypeScript提供了静态的代码分析，它可以分析代码结构和提供的类型注解。当然，尽管编译出现了错误，`main.js`文件还是被创建了。 就算你的代码里有错误，你仍然可以使用TypeScript。



## 数据类型

| 数据类型   | 关键字    | 描述                                                         |
| :--------- | :-------- | :----------------------------------------------------------- |
| 任意类型   | any       | 如果不声明类型，默认则声明为 any 的变量可以赋予任意类型的值。 |
| 数值类型   | number    | 等同于JavaScript的number类型，在TypeScript中所有的数字都是浮点数，都是number类型。<br>let num1: number = 0b1010; // 二进制<br>let num2: number = 0o744;    // 八进制<br>let num3: number = 6;    // 十进制 <br>let num4: number = 0xf00d;    // 十六进制<br>**注意：**TypeScript 和 JavaScript 都是没有整型的。 |
| 字符串类型 | string    | 一个字符系列，使用单引号（**'**）或双引号（**"**）来表示字符串类型。反引号来定义多行文本和内嵌表达式。<br/>let name: string = "xiaoming"; <br/>let qq_number: string= '50000000'; <br/>let text: string = \`您好，我叫 \${ name } ，我的QQ号码是${qq_number} `; |
| 布尔类型   | boolean   | 只有2个值：true 和 false。<br/>let sex: boolean = true;      |
| 数组类型   | 无        | 声明变量为数组。<br/>let arr: number[] = [1, 2];            // 在元素类型后面加上[]<br>let arr: Array\<number> = [1, 2]; // 数组泛型 |
| 元组       | 无        | 元组类型用来表示已知元素数量和类型的数组，各元素的类型不必相同，对应位置的类型需要相同。元祖在原生js中本身是支持的。<br>let x: [string, number]; <br>x = ['xiaoming', 16];    // 运行正常 <br/>x = [16, 'xiaoming'];    // 报错 <br/>console.log(x[0]);        // 输出 xiaoming |
| 枚举       | enum      | 枚举类型用于定义数值集合。<br/>enum Color {Red, Green, Blue}; <br/>let c: Color = Color.Blue; <br/>console.log(c);    // 输出 2 |
| void       | void      | 用于标识方法返回值的类型，表示该方法没有返回值。<br/>function hello(): void {    <br/>        alert("Hello xiaoming"); <br/>} |
| null       | null      | 空。                                                         |
| undefined  | undefined | 未定义                                                       |
| never      | never     | never 是其它类型（包括 null 和 undefined）的子类型，代表从不会出现的值。 |

了解了typescript支持的数据类型以后，其实我们可以更加清晰一点就是javascript里面有的，能使用的，typescript就一定有，并也能使用。所以接下来的学习中，针对javascript和typescript存在差异的地方我们进行讲解，而相同的则默认和javascript是一样的。

例如，接下来关于语法中，变量的命名规范，运算符，流程控制语句等等，这些我们就不会提及了。



## 变量

### 变量的声明

```typescript
// 1.声明变量时，直接指定变量的数据类型
var 变量名:类型 = 值;
// 2.不指定类型，默认类型为any
var 变量名 = 值; // var 变量名:any = 值;
// 3.预设变量，指定类型
var 变量名:类型;
// 4.不指定类型和值，默认类型为any, 默认值为 undefined：
var 变量名;
```

在开发中上面4种格式，最常见的是第3种，代码：

```typescript
var username:string = "hello world";
username = "xiaoming";
username = 123;  // error TS2322: Type 'number' is not assignable to type 'string'.
console.log(username);
```



### 变量作用域

变量作用域指代：根据变量定义的位置来决定变量的使用范围和生命周期。

TypeScript提供了三种不同的作用域：

- **全局作用域**

  全局变量定义在程序结构的外部，它可以在你代码的任何位置使用。

- **类作用域**

  这个变量也可以称为 **属性**或者**字段**。

  类变量基本声明在类里的头部位置，不但可以在类的方法里面进行调用，也可以在类方法外面，该变量可以通过类名来访问。

  类变量也可以是静态变量，静态变量可以通过类名直接访问。

- **局部作用域**

  局部变量，局部变量只能在声明它的一个代码块（如：方法）中使用。

```typescript
var global_num = 12          // 全局变量
class Numbers { 
   num_val = 13;             // 实例变量
   static sval = 10;         // 静态变量
   
   storeNum():void { 
      var local_num = 14;    // 局部变量
   } 
} 
console.log("全局变量为: "+global_num)  
console.log(Numbers.sval)   // 静态变量
var obj = new Numbers(); 
console.log("实例变量: "+obj.num_val)
```



### 联合类型

```typescript
// 语法
var 变量 = 类型1|类型2|类型3|...;

// 注意：
// 联合类型（Union Types）可以通过管道(|)将变量设置多种类型，赋值时可以根据设置的类型来赋值。只能赋值指定的类型，如果赋值其它类型就会报错。
// 4.2 新增语法，类型定义
type ctype = string|number|any[];
function sprint(data:ctype){
    console.log(data);
}
```

代码：

```typescript
// 允许变量在使用过程中,值可以是给出的多个类型之一
var age:string|number; // 联合类型的定义

age = 20;
age = "20";

function sprint(data:string|number|any[]){
    console.log(data);
}

function abc(){
    return "hello!";
}

var num:number = 100;
sprint(num);

var uname:string = "xiaoming";
sprint(uname);

var arr:any[] = [100,"B","C"];
sprint(arr);
```



## 函数

typescript和javascript在函数的声明以及使用中，除了类型注解以外，并没有其他区别。所以接下来的例子中，仅仅以普通函数作为例子进行讲解，至于匿名函数，箭头函数（lambda函数），闭包函数，都一样，所以这里就不会提到了。

### 函数的声明

```typescript
// 普通函数
function func_name( param1 [:type], param2 [?:type],param3[?]....) [:return_type]{
    
}

// lambda函数
var func = (param1:type) => expression;

/*
 type表示形参的数据类型，可以指定类型，也可以不指定类型
 return_type 表示函数执行以后的返回值的数据类型，可以指定，也可以不指定
 形参后面跟着?，表示当前参数是可选参数，可填可不填
 */
```

代码：

```typescript
// 普通函数的定义
// 参数有3种:
// 必填参数[可以限定类型]
// 可选参数 ?  [可以限定类型]
// 默认参数,提供了默认值,[可以限定类型,即便不限定类型,typescript也会通过默认值进行类型判定]
function func1(arg1,arg2:number,arg3?,arg4?:string,arg5:string="xioaming"):void{
    console.log(`arg1=${arg1},arg2=${arg2},arg3=${arg3},arg4=${arg4},arg5=${arg5}`);
}
// javasctipt/typescript在读取代码的时候,分2遍的
// 从上到下,进行词法检测,识别关键字,分配空间[]
// 从上到下,进行代码执行
func1(100,200);

// 此处arg5导致报错,是因为typescript内置的类型系统包含了类型的判断,
// 在函数声明时的参数列表中,已经对arg5进行默认值的分配,因为这个默认值的原因,
// 所以typescript根据arg5的默认值进行了类型判断,识别到了是string,
// 因此,在调用函数时传递的参数是number就肯定报错了
// func1(100,200,300,"400",500);


// 类型判定
// var data = "数据";
// data = 200;

// 箭头函数
// var func = (num1:number,num2:number):number => {return num1+num2}; // 原生script的写法
var func = (num1:number,num2:number):number => num1+num2; // typescript允许出现表达式
console.log( func(100,200) );
```



### 函数重载

错误写法：

```typescript
function func1(str1:string):void{
    console.log(str1);
}

function func1(num1:any,str1?:any):void {
    console.log(num1);
    console.log(str1);
}
func1("hello");
func1(16,"hello");
```

正确写法：

```typescript
function func1(str1:string):void;
function func1(num1:number,str1:string):void;

function func1(num1:string|number, str1?:string):void {
    console.log(`num1=${num1}`);
    console.log(`str1=${str1}`);
}

func1("hello");
func1(16,"hello");
```



## 类

### 类的声明使用和继承

```typescript
class Humen {
    age:number = 12; // 实例属性
    constructor(age?:number) { // 初始化方法,类似python的__init__
      if(typeof age == "number"){
          this.age = age;
      }
    }
    desc():string {
       return `我今年${this.age}岁`;
    }
}

// var xiaoming:Humen = new Humen();
// console.log(xiaoming.age);
// // xiaoming.age = 100;
// console.log(xiaoming.desc());

class Person extends Humen{
    uname:string;
    desc():string{
     return `我叫${this.uname},`+super.desc();
    }
}

var xm = new Person(18);
console.log(xm.age);
xm.uname = "小明";
console.log(xm.desc());
```

在继承中，可以重写父类方法，包括构造函数，代码：

```typescript
// 公有属性: 允许任何一个地方调用
// 私有属性: 仅允许当前类内部进行调用
// 保护属性: 仅允许当前类或直接间接继承了当前类的子类内部进行调用
class Proto{
    private name:string;
    public constructor(name:string){
        this.name=name;
    }
}

class Humen extends Proto{
    private age:number;
    public constructor(name:string,age?:number){
        super(name);
        this.age = age;
    }
}

class People extends Humen{

}

var p1 = new People("xiaoming",13);
console.log(p1);
```



### 静态属性和方法

```typescript
class Static {
   // 静态属性
   static num:number;
   // 静态方法
   static desc():void {
      console.log("num 值为 "+ Static.num)
   }
}
// 不需要实例化对象,就可以通过类进行调用

Static.num = 12;     // 初始化静态属性/变量
Static.desc();       // 调用静态方法
```



### 访问控制符

面向对象中的所谓封装，本质上就是把一系列相关的数据(属性/变量/字段)和操作数据的功能方法(方法/函数)集中到一个数据结构中(类)，达到隐藏数据和操作数据的方法，对外暴露有限的操作数据方法。

TypeScript 中，可以使用访问控制符来保护对类、变量、方法和构造方法的访问。TypeScript 支持 3 种不同的访问权限。

-   **public（默认）** : 公有的，可以在任何地方被访问。
-   **protected** : 受保护，可以被其自身以及其子类和父类访问。
-   **private** : 私有，只能被其定义所在的类访问。

代码：

```typescript
// 公有属性: 允许任何一个地方调用
// 私有属性: 仅允许当前类内部进行调用
// 保护属性: 仅允许当前类或直接间接继承了当前类的子类内部进行调用
class Proto{
   public desc(){                   // 公有方法
      return `我住在树上`;
   }
}

class Humen extends Proto{
   public address:string = "北京市"; // 公有属性
   public desc(){                   // 公有方法
      return `我住在${this.address}`;
   }

   private money:number = 10000;    // 私有属性
   private calc_money(){
      return this.money*0.1;  // 类的内部才可以调用私有属性,私有方法
   }
   // 如果允许私有属性提供给外界查看, 往往通过公有方法来进行暴露
   public show_money(){
      return this.calc_money();
   }

   protected phone:string = "13300000000";  // 保护属性
   protected get_phone(){                   // 保护方法
      return `我的手机号码:${this.phone}`; // 类的内部或者子类才可以调用保护属性/方法
   }
   // 如果允许保护属性提供给外界查看,往往通过公有方法来进行暴露
   public show_phone(key?){
      if(key == "123456"){
         return this.get_phone();
      }
   }
}

class People extends Humen{
   public show_father_data(){
      // return this.phone;    // 调用了父类的保护属性
      // return this.get_phone(); // 调用了父类的保护方法

      return this.show_money();       // 子类无法调用父类的私有属性或方法

      // return this.desc();  // 调用继承到的父类方法或者属性,如果当前类重载了则出现覆盖
      // return super.desc();
   }

   public desc(){
      return `您好, 我住在${this.address}`;
   }

}

var xiaoming = new People();
// console.log(xiaoming.phone); // 类的外部无法调用私有或受保护的属性
// console.log(xiaoming.address);
// console.log(xiaoming.desc());
// console.log(xiaoming.show_money());
// console.log(xiaoming.show_phone());
// console.log(xiaoming.show_phone(123456));
console.log(xiaoming.show_father_data());
```

### 存取器

```typescript
class Humen{
    private mymoney: number; // 私有属性
    constructor(money:number) {
        this.mymoney = money;
    }

    public set money(money:number) { // 存储器
        this.mymoney = money;
    }

    public get money(): number { // 读取器
        return this.mymoney;
    }
}

var xm = new Humen(10000);
console.log(xm.money);
xm.money = xm.money - 300; // 这段代码会出发存储器的执行,把等号右边的值作为参数传递给set存储器
console.log(xm.money);
```

编译命令：

```bash
tsc --target ES2020 main.ts
```

#### python中的存取器

```python
class Money(object):
    _money = 0
    def __init__(self,money):
        self._money = money

    # 使用装饰器对money进行装饰，那么会自动添加一个叫money的属性，当调用获取money的值时，调用装饰的方法
    @property
    def money(self):
        return self._money

    # 使用装饰器对money进行装饰，当对money设置值时，调用装饰的方法
    @money.setter
    def money(self, value):
        if isinstance(value, int):
            self._money = value
        else:
            print("error:不是整型数字")

if __name__ == '__main__':
    a = Money(100)
    print(a.money)
    a.money = a.money - 30
    print(a.money)
```



## 接口

接口(interface)是一系列抽象属性和方法的集合声明，这些方法都应该是抽象的，需要由具体的类去实现，然后外界就可以通过这组抽象方法调用，让具体的类执行具体的方法。

**接口的作用在开发中针对的是数据对象和类的结构进行描述和规范化。**说白了，就是你老大叫你声明一个类/对象，但是这个类/对象长什么样？他会以接口的格式先定义好，然后你照着这个接口定义好的格式进行编写一个类/对象出来，免得你弄乱结构，以后没法复用代码。

一般只有在中大型项目，或者框架/大型模块中为了更好的组织代码结构才会出现抽象类/接口

### 定义接口

```typescript
interface PayTool {
    SERVER_URL:string;
    pay: ()=>string
}
```

### 接口的声明和实现

```typescript
interface PayTool {
    SERVER_URL:string;
    pay: ()=>string;
}

class Alipay implements PayTool{
    SERVER_URL:string;
    constructor(server_url:string){
        this.SERVER_URL = server_url
    }
    pay(){
        return "ok"
    }
}

class WechatPay implements PayTool{
    SERVER_URL:string;
    constructor(server_url:string){
        this.SERVER_URL = server_url;
    }
    pay(){
        return "ok"
    }
}

var alipay = new Alipay("http://api.alipay.com");
console.log(alipay);
console.log( alipay.pay() );
```

>   凡是实现(implements)了接口/抽象类的类, 都要和接口/抽象类保持拥有一样的属性和方法

#### 接口的简写实现

typescript允许直通过json对象来直接实现接口,跳过了类的实现过程

```typescript
interface Person {
    username: string;
    age: number;
    desc():string
}

function main(person: Person) {
    return "Hello, 我叫" + person.username + "，我今年" + person.age+"岁.";
}

// typescript允许直通过对象来直接实现接口,跳过了类的实现过程
// var 对象名 = <接口名称>{
//    属性;
//    方法;
// }
let user = <Person>{
   username: "小白",
   age: 16,
   desc(){
      return "hello"
   }
};
console.log(main(user));
// js就是披着面向对象外壳的函数式编程语言
```

#### 鸭子类型

在传递实例参数时，不管当前传入的实例参数是否是限定的类/接口的实例对象，只要有同样的属性/方法，那么我们就认为当前实例参数就是这个限定类/接口的实例对象。这就是所谓的**鸭子类型**。

```typescript
// 鸭子类型:
// 鸭子类型规定了: 一个对象有效的语义，不是由继承自特定的类或实现特定的接口来决定的，
// 而是由"当前对象的方法和属性的集合"决定

interface Person {
    username: string;
    age: number;
}

function main(person: Person) {
    return "Hello, 我叫" + person.username + "，我今年" + person.age+"岁.";
}

var xm = {username:"小明",age:20}; // 问题来了,这里明明没有实现Person接口,为什么也能调用
console.log(main(xm));
```



##### python中的鸭子类型

```python
class Person(object):
    def __init__(self,username,age):
        self.username = username
        self.age = age

class Humen(object):
    def __init__(self,username,age):
        self.username = username
        self.age = age

def main(obj:Person):
    return "我叫%s,我今年%s岁了" % (obj.username,obj.age)

if __name__ == '__main__':
    p1 = Person("小明", 15)
    p2 = Humen("小白", 15)
    print( main(p2) )
```



### 接口继承

#### 单继承

```typescript
interface Person {
   age:number
}

interface Humen extends Person {
   username:string
   desc(user:string):string
}

class People implements Humen{
   age:number;
   username:string;
   constructor(username,age){
      this.age = age;
      this.username=username;
   }
   desc(user:string):string{
      return `${user},您好!我叫${this.username},我今年${this.age}岁.`
   }
}

var xm = new People("小明",15);
console.log( xm.desc("小红") );
```

#### 多继承

```typescript
interface Person {
   age:number
}

interface Humen{
   username:string
   desc(user:string):string
}

// 可以同时实现多个接口
class People implements Person, Humen{
   age:number;
   username:string;
   constructor(username,age){
      this.age = age;
      this.username=username;
   }
   desc(user:string):string{
      return `${user},您好!我叫${this.username},我今年${this.age}岁.`
   }
}

var xm = new People("小明",15);
console.log( xm.desc("小红") );
```



## 抽象类

抽象类（abstract class）做为其它派生类的基类使用。 它们一般不会直接被实例化。 不同于接口，抽象类可以包含成员的实现细节。 `abstract`关键字是用于定义抽象类和在抽象类内部定义抽象方法。

代码：

```typescript
// 抽象父类
abstract class Animal{
   abstract makeSound(): void;        // 抽象方法，没有函数体
   desc(): void {   // 抽象类中也可以定义子类的公共方法或公共属性
      console.log('roaming the earch...');
   }
}

// 抽象父类
abstract class Dog extends Animal{
   abstract nickname:string;
   abstract move(): string;
}


// 具象类/具体类
class ChineseGardenDog extends Dog{
   public nickname:string;
   constructor(nickname:string){
      super(); // 继承了抽象类的子类,必须对父类进行初始化
      this.nickname = nickname;
   }
   makeSound(){
        return "汪汪汪~"
   }
   move(): string {
      return "奔跑中....";
   }
}

var dog = new ChineseGardenDog("来福");
console.log(dog.nickname);
console.log(dog.makeSound());
```



#### python中的抽象类

在python的基本语法中是没有抽象类和接口概念的。所以，要实现类似接口或者抽象类的场景，可以借助python内置的标准库，abc模块来实现。

```python
import abc  # 利用abc模块实现抽象类

class File(metaclass=abc.ABCMeta):  # abc.ABCMeta是实现抽象类的一个基础类
    @abc.abstractmethod  # 定义抽象方法，无需实现功能
    def read(self):
        pass


class Txt(File):  # 子类继承抽象类，但是必须定义read方法将抽象类中的read方法覆盖
    def read(self):
        print('文本数据的读取方法')


if __name__ == '__main__':
    txt1 = Txt()
    txt1.read()
    # 抽象类是无法直接实例化的
    tet2 = File()
    tet2.read()
```



## 装饰器

随着TypeScript和ES6里引入了类，在一些场景下我们需要额外的特性来支持标注或修改类及其成员。 装饰器（Decorators）为我们在类的声明及成员上通过元编程语法添加标注提供了一种方式。装饰器是一种特殊类型的声明，它能够被附加到[类声明](https://www.tslang.cn/docs/handbook/decorators.html#class-decorators)，[方法](https://www.tslang.cn/docs/handbook/decorators.html#method-decorators)， [访问符](https://www.tslang.cn/docs/handbook/decorators.html#accessor-decorators)，[属性](https://www.tslang.cn/docs/handbook/decorators.html#property-decorators)或[参数](https://www.tslang.cn/docs/handbook/decorators.html#parameter-decorators)上。

代码：

```typescript
function derator1() {
    console.log(`derator1()`);
    function wrapper(target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log(`derator1()`);
    }
    return wrapper
}

function derator2(key:string) {
    console.log(`derator2()`);
    return function(target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log(`derator2()`);
    }
}

class Demo {
    @derator1()
    @derator2("abc")
    show(username:string,password:string) {
        console.log("show()执行了");
    }
}

var d = new Demo();
d.show("xioaming","123456");
```

因为装饰器是ES7中的内容，所以默认情况下， 所以tsc在编译过程中默认会不支持，所以，我们需要把编译代码的es版本转换成es5并开启代码编译时支持装饰器的功能。

官方文档说明：https://typescript.bootcss.com/decorators.html

终端运行如下：

```bash
tsc --target ES2020 --experimentalDecorators --emitDecoratorMetadata main.ts
```



## 命名空间

当项目大了以后，需要创建和声明的函数，类就多了，自然人也就多了，人多就坏事。想想全国有几个张三？

命名空间（namespace）一个最明确的目的就是解决标记符重名问题。

命名空间定义了标识符的可见范围，一个标识符可在多个名字空间中定义，它在不同名字空间中的含义是互不相干的。这样，在一个新的名字空间中可定义任何标识符，它们不会与任何已有的标识符发生冲突，因为已有的定义都处于其他名字空间中。

### 定义命名空间

App.ts，代码：

```typescript
// 命名空间的名称采用驼峰式写法
namespace App{ 
    // 需要在命名空间外部可以调用 当前命名空间的类,函数和接口等，则需要在左边添加 export 关键字。
    // 变量
    export var username:string="App空间的变量";
    // 常量,一旦定义以后,不能修改值
    export const NAME = "App命名空间的常量";
    // 函数
    export function func(){
        return "App命名空间里面的func"
    }
    // 类
    export class Humen{

    }
    // 当然，在当前命名空间下也是可以执行代码的    
}

```

### 命名空间的导入和使用

main.ts，代码：

导入其他命名空间的格式：`/// <reference path = "文件名" />`，可以导入多个命名空间，一行一个。

```typescript
/// <reference path="App.ts" />
console.log(App.func());  // 调用其他命名空间的内容，必须以"命名空间的名称.xxxx"格式进行调用
console.log(App.NAME);
console.log(App.username);
console.log(new App.Humen());
```

使用了命名空间以后，编译命令需要稍微调整如下：

```bash
tsc --out main.js main.ts  # 必须指定--out参数才能正常编译
```



## 模块

TypeScript 模块的设计理念是可以更换的组织代码。模块是在其自身的作用域里执行，并不是在全局作用域，这意味着定义在模块里面的变量、函数和类等在模块外部是不可见的。

typescript提供了两种模块：**内部模块**和**外部模块**，因为外部模块需要依赖第三方框架才可以使用，例如：commonjs，requirejs等。所以在此，我们只简单介绍内部模块的声明和使用。

### 内部模块

#### 声明模块

app.ts，代码：

```typescript
module App{
    export var username:string = "App模块下的变量";
    export const WEB_URL = 'http://www.oldboyredu.com';
    export class Humen {
        desc(){
            console.log("hello");
        }
    }

    export function func(){
        console.log("hello, func");
    }

}
```

#### 调用模块

main.ts，代码：

```typescript
/// <reference path="app.ts" />
console.log(new App.Humen());
console.log(App.username);
console.log(App.WEB_URL);
```

使用了内部模块以后，编译命令和命名空间一样：

```
tsc --out main.js main.ts
```



### 外部模块

App.ts，代码：

```typescript
export class Humen{
    uname:string;
    constructor(uname){
        this.uname = uname;
    }
    desc() {
        return `您好,我叫 ${this.uname}`;
    }
}
export { Humen };
export { Humen as People };
```

导入模块，main.ts，代码：

```typescript
import { Humen, People } from "./Out";

let obj1 = new Humen("小白");
let obj2 = new People("小黑");
obj1.desc();
obj2.desc();
```

编译命令：

```bash
tsc --module es6 main.ts   # --module 表示代码中编写模块的规范和标准
```



## 编译配置文件

基于typescript开发的项目根目录，一般都会存在一个文件，叫**tsconfig**。这是typescript的编译配置文件。

配置选项：https://www.tslang.cn/docs/handbook/compiler-options.html

tsconfig.json，常用配置项说明，代码：

```json
// 当前配置文件名必须固定是: tsconfig.json
// 同时,json文件中不能出现注释的,所以此处的注释仅仅是为了学习,开发中决不能有
{
    "compilerOptions": {
        "module": "system",     // 项目中编写模块的规范标准
        "noImplicitAny": true,  // 表达式或声明上有隐含的 any类型时报错
        "removeComments": true, // 删除所有注释，除了以 /!*开头的版权信息。
        "preserveConstEnums": true,  // 保留const和Enums声明
        "outDir": "script",  // 编译结果保存目录
        // "outFile": "../../built/local/tsc.js",  // 编译以后输出的文件,一般用不上
        "sourceMap": true,  // 生成相应的 .map文件
        "experimentalDecorators": true,  // 启用实验性的ES装饰器
        "lib": [ // 编译过程中需要引入的库文件的列表
            "es5",
            "dom",
            "es2015.promise"
        ]
    },
    "files": [   // 指定要编译的文件列表, 与include和exclude冲突,开发中,一般使用exclude
      "main.ts"
    ]
  //    "include": [ // 指定要编译的文件所在目录
////        "src/**/*",
//        "./"
//    ],
//    "exclude": [ // 指定在编译时排除的文件目录
//        "node_modules",
//        "**/*.spec.ts"
//    ]
}
```



## 析构表达式

```typescript
var book = {
    title: "重构",
    price: 100
};

// 直接析构,类似python的解包
// var {title,price} = book;
// console.log(title);
// console.log(price);

// 别名析构,参数使用别名进行替换
// var {title:tname,price} = book;
// console.log(tname);
// console.log(price);

var user = {
    username: "阿明",
    book:{title:"重构",price:100},
};
// 嵌套析构
// var {username,book:{title,price}} = user;
// console.log(username);
// console.log(book);
// console.log(title);
// console.log(price);

// 列表析构
var user_list = ["小明","小兰","小黑","小红"];
var [xm,xh,...o] = user_list; // 中间留空,表示不要析构中间的成员
// console.log(xm);
// console.log(xh);
// console.log(o); // 接受剩余成员


// 列表嵌套析构
var arr_list = [
    ["小梁","小明","小黄","小白"],
    ["小刘","小黑","小李","小龙"],
];

var [l1,[ls1,ls2]] = arr_list;
console.log(l1);
console.log(ls1,ls2);
```



## 泛型

泛型的作用主要是让数据类型参数化，保证使用数据过程中的一致性，让我们编写的代码复用性更强，也更加灵活。

泛型函数

```typescript
function func<T>(arg:T):T{
    // 变量T因为在7行调用时候,系统自动判断为string,所以返回值因此必须是string
    console.log(arg);
    console.log(typeof arg);
    return arg;
    // return 100; // 报错
}

func("username");

function func<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);
    arr[0] = 100; // 因为传入进来的Array第1个成员是300,所以可以赋值成功,因为T就是number
    arr[1] = "xiaoming"; // 因为传入进来的Array第2个成员是"xiaohei",所以可以赋值成功,因为T就是string
    return arg;
}

var arr = [300,"xiaohei"];
func(arr);
```

泛型变量

```typescript
function func<T>(arg:T):T{
    console.log(typeof arg);
    return arg;
}

var data = "100";
let output1=func<string>(data); // 手动指定泛型类型
let output2=func(data); // 根据参数的类型,系统会自动判断
//
let output3:number=func<number>(100);
let output4:number=func(200);
```

泛型类型

```typescript
function func<T>(arg: T): T {
    return arg;
}

// ret是一个函数,是func的别名函数,此时可以泛型类型指定函数的输入和输出类型
let ret: {<T>(arg: T): T} = func;

console.log( ret("100") );
```

泛型接口

```typescript
interface Inter {
    <T>(arg: T): T;
}

function func<T>(arg: T): T {
    return arg;
}

let myfunc: Inter = func;

console.log(myfunc(100));

```

泛型类

```typescript
class Calc<T> {
    num: T;
    add: (x: T, y: T) => T;
    sub: (x: T, y: T) => T;
}

let calc = new Calc<number>();
calc.num = 100;
calc.add = (x, y)=>{ return x + y; };
calc.sub = (x, y)=>{ return x - y; };

console.log(calc.add(2,3));
console.log(calc.sub(2,3));
```

泛型约束

```typescript
interface Arr {
    length: number;
}

function func<T extends Arr>(arg: T): T {
    console.log(arg.length);
    return arg;
}

// func(1000); // 报错,因为数值number没有实现Arr接口类型要求的length属性
// func("abc"); // 正确,因为字符串string虽然没有实现Arr借口类型,但是string拥有length属性,所以因为鸭子类型的原因,所以也算实现了Arr接口类型


class Data implements Arr{
    length: number;
}
var data = new Data();
data.length = 10;
// func(data);


// 数组的类型约束
function func2<K extends keyof T, T> (items: K[], obj: T): T[K][] { // T[K][] 为K类型的数组，且满足K为T的key
    return items.map(item => obj[item]);
}

// console.log( func2([1,2],["xiaoming","xiaohong","xiaohei"] ) );
// 输出结果: ["xiaohong", "xiaohei"]

// console.log( func2([1,2,"A"],["xiaoming","xiaohong","xiaohei"] ) ); // 报错,typescript中的数组根本没有A属性
// console.log( func2([1,2,"length"],["xiaoming","xiaohong","xiaohei"] ) ); // 正确,typescript中的数组有length属性

console.log( func2(["username","age"],{"username":"xiaoming","password":"123456","age":21} ) );
// 输出结果: ["xiaoming", 21]
```



## 生成器

### 迭代器(iterator)

迭代器是一种特殊的可迭代对象，他实现了[迭代器协议](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Iteration_protocols)。

#### 可迭代对象

在javascript/Typescript中，要实现一个**可迭代对象**， 该对象必须实现 `Symbol.iterator` 方法（也可以说是遵循可迭代协议），当一个对象实现了`Symbol.iterator`方法时，我们就认为它是可迭代的。在javascript/Typescript中，内置类型：`Array`，`Map`，`Set`，`String`都是实现了Symbol.iterator方法的。对象上的 `Symbol.iterator`函数负责返回供迭代的值。

```typescript
var arr = [1,2,3];
console.log( arr[Symbol.iterator] );
console.log( arr[Symbol.iterator]() ); // 返回值就是一个迭代器
// tsc --target ES2020 main.ts
```

#### Symbol类型

https://www.tslang.cn/docs/handbook/symbols.html#symboliterator

自ECMAScript 2015起，`symbol`成为了一种新的原生类型，就像`number`和`string`一样。可以通过`Symbol()` 函数返回 `symbol` 类型的值。每个 `symbol` 值都是**不可改变并且唯一**的，所以开发中，Symbol非常适合做唯一key。

```typescript
const s1: symbol = Symbol();
const s2: symbol = Symbol('foo');
const s3: symbol = Symbol('foo');

console.log(s1);
console.log(s2);
console.log(s3);
console.log(s2 === s3); // false, 唯一,所以不相等
s3 = 123; // 不能再次赋值, 编译报错
// console.log(s3);
```



#### 自定义可迭代对象

```typescript
var myIterable = {
    *[Symbol.iterator](){
        yield 1;
        yield 2;
        yield 3;
    }
};

console.log(myIterable);
for( let item of myIterable){
    console.log(item);
}
```



#### 迭代器

在javascript中，一个对象只有实现了指定格式的next()方法才能成为迭代器。next必须返回值类型是 `{ value: any, done: boolean }`。

迭代器一旦创建，迭代器对象就可以通过重复调用 `next()` 显式地迭代。

自定义迭代器

```typescript
function MyIterator(array) {
    let nextIndex = 0;
    return {
       next() {
           if(nextIndex >= array.length){
               return {
                   done: true
               }
           }

           return {
               value: array[nextIndex++],
               done: false
           }
       }
    };
}

let it = MyIterator(['A', 'B', "C", "D"]);

console.log(it.next().value);  // 'A'
console.log(it.next().value);  // 'B'
console.log(it.next().value);  // 'C'
console.log(it.next().value);  // 'D'
console.log(it.next().done);   // True
// 注意：此处的MyIterator没有实现可迭代协议，所以并非可迭代对象。
```



### 生成器(Generator)

[生成器](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Generator)对象既是迭代器，也是可迭代对象。`Generator`其实是ES6对协程的一种实现。由于JavaScript是单线程语言，本身就是一个不停循环的执行器，所以它的协程是比较简单的，js中的线程和协程关系是 1:N，一个线程里同一时间只能有一个协程在执行。

```typescript
function* generatorFunction() {
  console.log('开始执行');
  yield 'Hello, ';

  console.log('暂停后再次执行');
  yield 'World!';
}

let gen = generatorFunction();

// console.log( gen.next() ); // {value: "Hello, ", done: false}
// console.log( gen.next() ); // {value: "World!", done: false}
// console.log( gen.next() ); // {value: undefined, done: true}

for(let item of gen){
    console.log(item); // 这里返回的结果经过 for-of处理只会得到value的值
}
```

通过 next() 参数向生成器传值

```typescript
function* gen() {
  console.log('开始执行');
  let res1 = yield 1;
  console.log('中断后继续执行');
  console.log(res1);

  let res2 = yield 2;
  console.log(res2);

  let res3 = yield 3;
  console.log(res3);

  console.log('执行结束');
  return 4
}

let gen1 = gen();
console.log( gen1.next() ); // 激活生成器
console.log(gen1.next('A'));
console.log(gen1.next('B'));
console.log(gen1.next('C'));
```



### 异步回调对象（Promise）

`Promise`是ES6开始支持的一个可以获取异步结果，并封装了一些异步操作的对象。有三个状态：
`pending`: 进行中， 初始状态。
`resolved`: 成功
`rejected`: 失败

并且这三个状态只有两种转换：`pending`->`resolved`、`pending`->`rejected`，不是成功就是失败，并没有多余的状态转换。这两种转换都是由异步返回的结果给定的，成功取回数据就是`resolved`，取数据出异常就是`rejected`。异步调用过程中，`Promise`不能在任务进行中取消，只能等结果返回。

```typescript
function aiohttp(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open("GET", url);
    xhr.onload = () => resolve(JSON.parse(xhr.responseText));
    xhr.onerror = () => reject(xhr.statusText);
    xhr.send();
  });
}

let ip = "123.112.18.111";

aiohttp(`http://ip-api.com/json/${ip}?lang=zh-CN`).then((response)=>{
    console.log(response);
}).catch((error)=>{
    console.log(error);
});
console.log("hello!");
```



### 基于协程实现异步编程

async和await是ES7的议案，TypeScript也支持。

`async`和`await`其实是`Generator`的语法糖。除了`*`换成`async`， `yield`换成`await`之外，最主要是`async await`内置了执行器，不用像`Generator`用那样`next()`一直往下执行。

```typescript
function delay(): Promise<void>{
    return new Promise<void>((resolve, reject)=>{
        setTimeout(()=>resolve(), 2000)
    });
}

async function run(){
    console.info('start');
    await delay(); // 这里的await就是yield,所以会交出程序的执行权
    console.info('finish');
}

run();
console.info('run');

/*
start
run
finish
 */
```

