# Javascript对象

## 什么是对象

对象是一种类型  引用类型  而对象的值就是引用类的实例

## 对象的创建

  ```javascript
  var obj = {}
  console.log(obj)
  console.log(typeof obj)
  ```

## 对象使用

+ 创建对象并设置属性

  属性设置引号

  ```javascript
  var obj = {
      'name':'lucky',
      'age':18,
      'run':function () {
          console.log('大伟is running...')
      }
  };
  console.log(obj.name);
  console.log(obj['name']);
  obj.run()
  ```

  不设置引号

  ```javascript
  var obj = {
      name:'lucky',
      age:18
  }
  ```

+ 删除属性

  delete

  ```javascript
  delete obj.name;
  console.log(obj.name);
  ```

## 对象属性

（JavaScript 对象中的）名称:值对被称为**属性**。

```
var person = {firstName:"Bill", lastName:"Gates", age:62, eyeColor:"blue"};
```

| 属性      | 属性值 |
| :-------- | :----- |
| firstName | Bill   |
| lastName  | Gates  |
| age       | 62     |
| eyeColor  | blue   |

## 对象方法

对象也可以有**方法**。

方法是在对象上执行的**动作**。

方法以**函数定义**被存储在属性中。

| 属性      | 属性值                                                    |
| :-------- | :-------------------------------------------------------- |
| firstName | Bill                                                      |
| lastName  | Gates                                                     |
| age       | 62                                                        |
| eyeColor  | blue                                                      |
| fullName  | function() {return this.firstName + " " + this.lastName;} |

方法是作为属性来存储的函数。

**实例**

```
var person = {
  firstName: "Bill",
  lastName : "Gates",
  id       : 678,
  fullName : function() {
    return this.firstName + " " + this.lastName;
  }
};
```

## this 关键词

在函数定义中，this 引用该函数的“拥有者”。

在上面的例子中，this 指的是“拥有” fullName 函数的 *person 对象*。

换言之，this.firstName 的意思是 *this 对象*的 firstName 属性。

## 访问对象属性

您能够以两种方式访问属性：

```
objectName.propertyName
```

或者

```
objectName["propertyName"]
```

**例子** 1

```
person.lastName;
```

**例子** 2

```
person["lastName"];
```

## 访问对象方法

您能够通过如下语法访问对象方法：

```
objectName.methodName()
```

**实例**

```
name = person.fullName();
```

如果您**不使用 ()** 访问 fullName 方法，则将返回**函数定义**：

**实例**

```
name = person.fullName;
```

方法实际上是以属性值的形式存储的函数定义。

!> 请不要把字符串、数值和布尔值声明为对象！

如果通过关键词 "new" 来声明 JavaScript 变量，则该变量会被创建为对象：

```
var x = new String();        // 把 x 声明为 String 对象
var y = new Number();        // 把 y 声明为 Number 对象
var z = new Boolean();       //	把 z 声明为 Boolean 对象
```

请避免字符串、数值或逻辑对象。他们会增加代码的复杂性并降低执行速度。

