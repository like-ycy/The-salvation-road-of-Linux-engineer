# JavaScript函数

## 一、定义

JavaScript 函数通过 function 关键词进行定义，其后是**函数名**和括号 ()。

函数名可包含字母、数字、下划线和美元符号（规则与变量名相同）。

- 格式

  ```js
  function 函数名 (参数 1, 参数 2, 参数 3) {
      要执行的代码
  }
  ```

+ 定义无参函数

  ```javascript
  function test() {
      console.log('输出')
  }
  ```

  调用

  ```
  test()
  ```

+ 定义形参函数

  ```javascript
  function test(x, y) {
      console.log(x, y)
  }
  ```

  调用

  ```javascript
  test();
  test(1,2);
  test(1, 2, 3, 4);
  ```

+ 定义形参函数并设置默认值

  ```javascript
  function test(x=1, y=2) {
      console.log(x, y)
  }
  ```

  调用

  ```javascript
  test(3)
  test(3,4)
  ```

+ 注意事项
  + 函数调用不区分位置
  + 形参个数可以大于实参个数
  + 设置参数默认值

### 函数表达式

+ 概述

  js的函数可以通过表达式进行定义

  函数存储在变量中

+ 格式

  ```javascript
  var fun = fun([参数]){
      函数体
  }
  ```

+ 实例

  ```javascript
  var fun = function () {
      alert('弹框')
  }
  fun()
  ```

### 自调用函数

+ 概述

  + 函数可以使用自调用 也就是写完之后自己进行调用
  + 如果表达式后面紧跟着() 则为自调用

+ 实例

  ```javascript
  (function () {
      console.log('自调用函数')
  })()
  ```

### 注意

+ 函数调用不区分位置
+ 形参个数可以大于实参个数
+ 设置参数默认值
+ 函数内部可以更改函数外部的全局变量
+ 函数内部使用var定义的变量为局部变量  外部不能获取
+ 函数内部不使用var定义的变量为 全局变量

实例

```javascript
var num = 1;
function test() {
    // num = 2;
    // console.log(num);
    age = 18  // 全局变量age
    var age = 18  // 局部变量age 外部不能获取
    console.log(age)
}
test()
// console.log(num)
console.log(age)
```

## 二、其它

### 1、不定长参数

+ arguments 接收不定长参数

+ 实例

  ```javascript
  function test() {
      // console.log(arguments)
      // console.log(arguments.length)
      for (var obj in arguments) {
          console.log(arguments[obj])
      }
  }
  // test();
  test(1,2,3,4,5);
  ```

### 2、return关键字

+ 实例

  ```javascript
   function test() {
       return 'val'
       console.log('输出') // 和Python一样 不在执行
   }
  console.log(test());
  ```

+ 注意

  + 当函数体执行遇到return 下面代码不在执行
  + return将值 返回给了函数调用处

## 三、闭包

### 1、概述

+ 闭包也就是发生在函数嵌套的时候   将函数体进行返回
+ 函数内部可以修改外部的全局变量

### 2、简单的闭包

```javascript
function test1() {
    function inner() {
        console.log('inner')
    }
    return inner
}
inner = test1()
console.log(inner())
```

### 3、计数器

+ 3.1

  ```javascript
  var count = 1;
  function add() {
      return count++
  }
  count = 10 // 外部可以进行更改
  console.log(add());
  console.log(add());
  ```

+ 3.2

  ```javascript
  function add() {
      var count = 1;
      function inner() {
          return count++
      }
      return inner
  }
  add = add()
  console.log(add());
  console.log(add());
  console.log(add());
  console.log(add());
  ```

+ 3.3

  ```javascript
  var add = function () {
      var count = 1;
      return function () {return count++}
  }();
  console.log(add());
  console.log(add());
  ```