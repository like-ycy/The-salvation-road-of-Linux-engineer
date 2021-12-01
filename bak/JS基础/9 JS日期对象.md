# JavaScript日期

### 1、概述

格林威治1970年1月1日到现在的毫秒数

### 2、创建

+ 不传参

  var date = new Date()  为当前时间

+ 传参

  var date = new Date(2019,5,5) 

  var date = new Date("2019/5/5")

### 3、格式化方法(了解)

| 方法                | 说明                            |
| ------------------- | ------------------------------- |
| toDateString()      | 以特定格式显示星期 月日年       |
| toTimeString()      | 以特定格式显示时 分 秒 还有时区 |
| toLocalDateString() | 以特定地区格式显示年月日        |
| toLocalTimeString() | 以特定地区格式 显示 时分秒      |

实例

```javascript
var date = new Date();
var date = new Date(2019,12,6);
console.log(date.toDateString());
console.log(date.toTimeString());
console.log(date.toLocaleDateString())
console.log(date.toLocaleTimeString())
```

### 4、获取单独的日期

+ setDate()/getDate()  返回一个月中的某一天(1-31)
+ getDay  获取一周的星期几（0-6）
+ setMonth()/getMonth()   设置/获取月份(0-11)
+ setFullYear()/getFullYear()  设置/获取年份
+ setHours()/getHours()   设置/获取小时
+ setMinutes()/getMinutes()  设置/返回分钟数
+ setSeconds()/getSeconds()    设置/返回秒数
+ 注意
  + 月份 0-11
  + 星期 0-6

```javascript
var obj = new Date();
console.log(obj.getDay())
console.log(obj.getDate())
console.log(obj.getMonth())
console.log(obj.getFullYear())
console.log(obj.getHours())
console.log(obj.getMinutes())
console.log(obj.getSeconds())
console.log(obj.getTime())
console.log(obj.getFullYear()+'年'+(obj.getMonth()+1)+'月'+obj.getDate()+'日'+obj.getHours()+':'+obj.getMinutes()+':'+obj.getSeconds())
```

## 定时器与定时执行

### 1、定时执行

+ 函数

  setInterval()

+ 格式

  ```javascript
  setInterval(function(){},毫秒数)
  ```

+ 使用

  ```javascript
  var span = document.getElementById('t');
  var timer = setInterval(function () {
      var obj = new Date();
      span.innerHTML =obj.getFullYear()+'年'+(obj.getMonth()+1)+'月'+obj.getDate()+'日'+obj.getHours()+':'+obj.getMinutes()+':'+obj.getSeconds();
  }, 1000)
  ```

+ 关闭定时器

  clearInterval(名称)

  ```javascript
  clearInterval(timer);
  ```



### 2、执行一次

+ 函数

  setTimeout

+ 格式

  ```javascript
  var timer = setTimeout(function(){},毫秒)
  ```

  ```javascript
  var num = 1;
  var timer = setTimeout(function () {
      console.log(num);
      num ++;
  },100);
  ```

+ 清除执行

  clearTimeout()

