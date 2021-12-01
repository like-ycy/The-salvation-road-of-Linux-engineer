# JavaScript数据类型

### 1、类型

+ Boolean  布尔类型   true/false
+ Number    整数（int）/小数 (float)
+ String   字符串
+ Array    数组
+ Object    对象
+ 特殊类型
  + null    对象类型
  + undefined    undefined   派生自null

### 2、typeof 操作符

+ 概述

  用来检测变量的类型

+ 类型

  ```javascript
  console.log(typeof true)  //boolean
  console.log(typeof 1)  //number
  console.log(typeof 1.2)  //number
  console.log(typeof '1.2')  //string
  console.log(typeof [1,2,3])  //object
  console.log(typeof {})  //object
  console.log(typeof undefined)  //undefined
  console.log(typeof null)  //object
  ```

+ 注意

  在给`name`变量赋值的时候，无论什么值类型都为`string`，原因是 `name`为`window.name`属性 

### 3、undefined类型

+ 概述

  声明一个变量但未赋值时，该变量的默认值是undefined。函数无明确的返回值时，返回的也是undefined。

+ 例如

  ```javascript
  var a;
  console.log(typeof a);  //undefined
  ```

  所以我们在定义变量的时候，最好定义并赋值

### 4、null类型

+ 概述

  null类型是只有一个值的数据类型，即特殊的值 null。它表示的是空对象使用typeof操作符会返回object

+ 实例

  ```javascript
  var a = null;
  console.log(typeof a);  // object
  console.log(null == undefined)
  ```

### 5、Boolean类型

+ 类型转换

  + String  非空字符串为真  空字符串为假
  + Boolean   false本身问false  true为true
  + Object 对象不为null则为真 否则为假
  + Number  有值为真（不为0）  为0则假
  + Undefined    为假

+ 实例

  ```javascript
  console.log('undefined',Boolean(undefined));// false
  console.log('null',Boolean(null));// false
  console.log('{}',Boolean({}));// true
  console.log('[]',Boolean([]));// true
  console.log('',Boolean(''));// false
  console.log('1',Boolean(1));// true
  console.log('0',Boolean(0));// false
  console.log('1.1',Boolean(1.1));// true
  console.log('true',Boolean(true));// true
  console.log('false',Boolean(false));// false
  ```



### 6、Number类型

+ 分类

  + 整数  int
  + 小数 float

+ 整数

  ```javascript
  var num = 1;
  console.log(typeof num); // number
  ```

+ 浮点数

  数值中包含一个小数点，小数点后面必须要有一个数字

  ```javascript
  var f1 = 1.1;
  var f1 = 0.1;
  var f1 = .1;  // 可以 不建议
  console.log(f1, typeof f1);
  ```

+ 注意事项

  ```javascript
  var f1 = 12.0;   // 12
  var f1 = 8.;     // 8
  console.log(f1, typeof f1);  
  ```

  + 由于在js中，存储浮点数的所占内存空间是整数的俩倍，所以会将整数的浮点数值转为整形

+ NaN 非数值 Not  A Number（特殊的值）

  + 概述  

    这个数值，用于表示一个本来要返回数值的操作，但是并没有返回数值。这样就会抛出错误，则会影响程序的执行，所以在js中返回NaN特殊的值，防止报错以影响程序的执行

  + 实例

    ```javascript
    console.log(0/0);  // NaN
    console.log(12/0);  // Infinity  浮点数值超出了范围  则显示正无穷
    console.log(12/0*0); // NaN
    ```

  + isNaN   

    用来判断这个值，是否为NaN  

    ```javascript
    console.log(isNaN(10));  // false
    console.log(isNaN(1.2)); // flase
    console.log(isNaN('abc')); // true
    console.log(isNaN(NaN));  // true
    ```

+ 常用方法

  + 向上取整

    Math.ceil()

  + 四舍五入

    Math.round()

  + 向下取整

    Math.floor()

  + 保留两位小数

    number.toFixed(2)

    ```javascript
    console.log(1.23456.toFixed(2))
    ```

+ 类型转换

  + parseInt  转换为整形

  + parseFloat   转换为浮点型

    ```javascript
    console.log(parseInt('12.3'), typeof parseInt('12.3'));
    console.log(parseFloat('12.3'), typeof parseFloat('12.3'));
    ```

