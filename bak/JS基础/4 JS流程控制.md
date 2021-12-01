# 流程控制

### 1、基本结构

+ 顺序： 从上之下而执行
+ 分支(选择) ： 根据不同的情况，执行对应的代码
+ 循环：重复做一件事



### 2、分支结构

+ if单一分支

  + 结构

    ```
    if(表达式){
    	代码
    }
    ```

    当条件表达式为真，则执行里面的代码段

    if后面的括号是不可省略的

  + 简写

    ```javascript
    if(表达式)
        console.log('代码段')
    if(表达式) console.log('代码段')
    ```

+ if双向分支

  + 结构

    ```javascript
    if(表达式){
        代码段1
    }else{
        代码段2
    }
    ```

  + 简写

    ```javascript
    if(表达式)
        代码段1
    else
        代码段2
    ```

  + 使用

    ```javascript
    if(1>2){
        console.log('大于')
    }else{
        console.log('小于')
    }
    
    if(1>2)
        console.log('大于');
    else
        console.log('小于');
    ```

+ if多向分支

  + 格式

    ```javascript
    if(表达式){
        代码段1
    }else if(表达式){
        代码段2
    }else if(表达式){
        代码段3
    }else{
        代码段4
    }
    ```

    从上至下而执行，如果遇到表达式为真的则执行对应的代码段，如果都没有为真，则执行最后的else

    ```javascript
    var grade = 83;
    if(grade>=90 && grade<=100){
        console.log('优秀')
    }else if(grade>80 && grade<=89){
        console.log('良')
    }else{
        console.log('不及格')
    }
    ```

+ if嵌套（巢状分支结构）

  + 格式

    ```javascript
    if(表达式){
        if(表达式){
        	代码段1
    	}else{
        	代码段2
    }else{
        if(表达式){
        	代码段1
    	}else{
        	代码段2
    	}
    }
    ```

### 2、switch语句

+ 格式

  ```javascript
  switch(表达式){
      case 常量1:
          语句
          break;
      case 常量2:
          语句
          break;
      default:
          语句
          break;
  }
  ```

+ 使用

  ```javascript
  var num = 1;
  switch (num){
      case 1:
          document.write('星期一')
          break;
      case 2:
          document.write('星期二')
          break;
      case 3:
          document.write('星期三')
          break;
      default:
          document.write('默认值')
          break;
  }
  ```

+ 注意

  break从语法来说不是必须的

  但是从使用角度是必须存在的 

  如果有匹配的值，会执行对应的语句，遇到break则跳出，否则执行完所有的代码语句

### 3、循环结构

+ 概述

  执行重复同一操作，直到条件不满足，则跳出循环

+ 循环分类

  + while
  + do while
  + for

  + for  in
  + forEach

+ while循环

  + 格式

    ```javascript
    while(条件表达式){
        循环体
    }
    ```

  + 注意：变量的自增自减  

  + 使用

    ```javascript
    var num = 1;
    while(num<=9){
        document.write(num+'<br/>');
        num ++;
    }
    ```

    左侧正三角形

    ```javascript
    var i = 1;
    //    左侧正三角形
    while(i<=9){
        var j = 1;
        while(j<=i){
            document.write(j+' ');
            j++;
        }
        document.write('<br/>')
        i++;
    }
    ```

    自己完成另外的3个三角形

+ do while  先执行 再判断

  + 格式

    ```javascript
    do{
        代码段
    }while(表达式);
    ```

  + 注意 无论条件是否满足，则都会执行一次

  + 使用

    ```javascript
    var i = 1;
    do{
        var j = 1;
        do{
            document.write(j+' ');
            j++;
        }while(j<=i);
        document.write('<br>');
        i++;
    }while(i<=9);
    ```

+ for循环

  + 格式

    ```javascript
    for(变量;表达式;自增){
        循环体
    }
    ```

  + 使用

    ```javascript
    for(var i=1;i<=9;i++){
        console.log(i)
    }
    ```

  + 拆分

    ```javascript
    var i=1;
    for(;;){
        if(i<=9){
            console.log(i);
            i++
        }else{
            break;
        }
    }
    ```

+ for in  迭代数组 返回索引

  + 格式

    ```javascript
    for(var i in Array){
        console.log(i)
    }
    ```

  + 使用

    ```javascript
    var arr = [1, 2, 3, 4];
    for(var i in arr){
        // console.log(i)  // 获取索引
        console.log(arr[i])
    }
    ```

+ forEach 循环

  + 格式

    ```javascript
    Array.forEach(function(arg){
        console.log(arg);
    })
    ```

  + 使用

    ```javascript
    var arr = [1, 2, 3, 4];
    arr.forEach(function(arg){
        console.log(arg);  // 获取值
    })
    ```

+ 关键字

  break   跳出循环  （只能跳出当前循环） 应用在循环与switch

  continue  跳过本次循环继续下一次循环   只能用在循环中