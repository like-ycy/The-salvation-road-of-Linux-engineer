# 数组

## 一、数组的概述与定义

### 1.1、什么是数组？

数组是一种特殊的变量，它能够一次存放一个以上的值。

如果您有一个项目清单（例如，汽车品牌列表），在单个变量中存储汽车品牌应该是这样的：

```javascript
var car1 = "Saab";
var car2 = "Volvo";
var car3 = "BMW"; 
```

不过，假如您希望遍历所有汽车并找到一个特定的值？假如不是三个汽车品牌而是三百个呢？

解决方法就是数组！

数组可以用一个单一的名称存放很多值，并且还可以通过引用索引号来访问这些值。

### 1.2、创建数组

使用数组文本是创建 JavaScript 数组最简单的方法。

#### 语法：

```js
var array-name = [item1, item2, ...];
```

#### 实例

```js
var cars = ["Saab", "Volvo", "BMW"];
```

空格和折行并不重要。声明可横跨多行：

```js
var cars = [
    "Saab",
    "Volvo",
    "BMW"
];
```

> 请不要最后一个元素之后写逗号（比如 "BMW",）。可能存在跨浏览器兼容性问题。

#### 定义二维数组

实例

```javascript
var a1 = [1,2,3];
var a2 = [4,5,6];
var a = [a1, a2];
console.log(a);
console.log(a[0]);
console.log(a[0][2]);
console.log(a.length);
```

## 二、 数组的访问

### 2.1、访问数组元素

我们通过引用**索引号（下标号）**来引用某个数组元素。

+ 下标也称为索引，从0开始，依次递增。数组的最大索引为 arr.length-1
+ 下标可以是变量或表达式
+ 如果取值的索引不存在，返回undefined
+ 给数组添加值的索引不存在，则会按照给定的索引添加值，前面不存在的索引值，以empty占位

#### 实例

```js
var arr = [1, 2, 3];
console.log(arr[0]) // 1

// 获取不存在索引的值
console.log(arr[3]) // undefined

// 获取不存在索引的值
arr[10] = 10  // 给索引为10 设置值为10 前面不存在的值/索引 以empty占位

// 获取最大索引的值
console.log(arr[arr.length-1])
```

### 2.2、改变数组元素

这条语句修改了 cars 中第一个元素的值：

```js
cars[0] = "Opel";
```

#### 实例

```js
var cars = ["Saab", "Volvo", "BMW"];
cars[0] = "Opel";
document.getElementById("demo").innerHTML = cars[0];
```

### 2.3、访问完整数组

通过 JavaScript，可通过引用数组名来访问完整数组：

#### 实例

```js
var cars = ["Saab", "Volvo", "BMW"];
document.getElementById("demo").innerHTML = cars; 
```

## 三、数组的常用方法

**lenght属性**

+ 获取元素的长度

  arr.length

+ 设置长度

  arr.length = 值

  ```javascript
  arr.length = 1;
  arr.length = 10;
  console.log(arr);
  console.log(arr.length)
  ```

**push()**  当前方法可以接收任意参数   把他们逐一添加到数组的尾部  并返回长度

```javascript
arr.push('范冰冰');
console.log(arr.push('易烊千玺', '范冰冰'));
```

**pop()**  移除最后一个元素 并返回

```javascript
console.log(arr.pop())
```

**shift()**  从前部移除一个元素  并返回

```javascript
console.log(arr.shift())
```

**unshift()**  向前面添加一个或多个元素

```javascript
console.log(arr.unshift('张杰'))
console.log(arr.unshift('张杰', '谢娜'))
```

**reverse()**  逆向排序

```javascript
console.log(arr.reverse())
```

**sort()**  排序

```javascript
console.log(arr.sort())
```

**concat()** 追加数据  创建新数组 不改变原数组

```javascript
console.log(arr1.concat(arr2))
```

**slice()**  不修改原数组 提取数组的值(不包含end)

```javascript
console.log(arr.slice(0, 2))
```

**splice()**  截取原数组的值  会更改原数组

+ 删除/截取

  ```javascript
  var arr = [1,2,3,4,5];
  console.log(arr.splice(0,2))
  console.log(arr)
  ```

+ 插入（如果有3个以上的参数 第二个参数为0 则表示插入）

  ```javascript
  var arr = [1,2,3,4,5];
  console.log(arr.splice(0,0,'婷婷'))
  console.log(arr.splice(1,0,'婷婷'))
  console.log(arr)
  ```

+ 替换 如果有三个以上的参数 第二个参数不为0  则表示替换

  ```
  var arr = [1,2,3,4,5];
  console.log(arr.splice(1,3,'婷婷'))  // 1-3的索引区间替换值为 婷婷
  ```

**arr.join()**  数组组成字符串

```javascript
console.log(arr.join('*'))
console.log(arr.join(''))
```