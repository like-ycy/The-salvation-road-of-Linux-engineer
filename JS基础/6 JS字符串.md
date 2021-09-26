# JavaScript 字符串

JavaScript 字符串是引号中的零个或多个字符。

### 创建

var str = ''

```javascript
var str = 'abcefg'
var test = "hello world"
console.log(typeof str);
console.log(str.length);
```

> 注意：字符串内也可以使用引号，只要不匹配围绕字符串的引号即可

### 读取

str[索引值]

```js
console.log(str[0]);
```

+ 字符串索引所对应的值只能获取  不能更改

## 字符串方法

- length   获取字符串的长度

```javascript
var str = 'abcefg'
console.log(str.length); 
```

​		对于字符串 length只可以获取长度不能更改长度

+ str.charAt()    获取下标对应字符
+ str.charCodeAt()  获取对应索引转换为ASCII值
+ String.fromCharCode()  将对应的ASCII值转换为字符
+ str.concat()  字符串连接
+ str.indexOf()  查找字符第一次出现的位置  查找失败返回-1
+ str.lastIndexOf()    查找字符出现的最后的位置  查找失败返回-1
+ str.substring(start, end)  字符串截取
+ str.toLowerCase()  转换为小写
+ str.toUpperCase()  换行为大写

+ str.search()  返回第一次出现的位置
+ str.replace(old, new)    替换
+ str.split()   正则拆分