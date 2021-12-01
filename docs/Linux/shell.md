# Shell

### 一、Shell脚本及编程概述

Shell脚本命令的工作方式有两种：***交互式和批处理\***。

-   交互式：Interactive，用户没输入一条命令就立即执行。到目前为止前文采取的都是这种方式。
-   批处理：Bach，事先编写一个完整的Shell脚本，一次性执行脚本中的诸多指令。接下来我们将采用的就是此种方式。

通过SHELL环境变量，可以查看当前系统默认使用的终端解释器，默认采用Bash解释器

```
[root@ubuntu ~]# echo $SHELL
/bin/bash
```

#### 1.3 第一个Shell脚本

在Shell脚本中不仅会用到前文学习过的Linux命令、管道符、数据流重定向等语法规则，还需要把内部功能模块化后通过逻辑语句进行处理，最终形成日常所见的shell脚本。

约定俗成的所有编程语言第一个程序都是Hello World，我们用Shell编写一个简单的脚本

```
[root@ubuntu test]# ls
test1.txt  test2.txt
[root@ubuntu test]# echo "hello world"
hello world
[root@ubuntu test]# vi hello.sh
#!/bin/bash
#author:ubuntu
echo " hello world"
```

保存并退出。Shell脚本文件可以是任意名称，但是为了避免被误认为是普通文件，建议后缀采用.sh，以表示是一个脚本文件。

>   ps：第一行#!/bin/bash是脚本声明，用来告诉系统使用哪种Shell解释器来执行脚本。
>
>   第二行#author:ubuntu是注释信息，对脚本功能进行介绍方便日后维护。
>
>   第三行echo " hello world"是命令语句，可以是简单的各种linux命令，也可以是通过各种逻辑语句和语法规则组合的复杂语句块。

Shell编程格式就这么简单，我称之为Shell编程“三段式”，以后编码就按照这个格式编写即可。

#### 1.4 执行Shell脚本

Linux中执行脚本有两种方式：***用解释器直接执行、通过脚本路径执行\***。

-   用bash解释器直接执行

语法：bash或sh 脚本名称

案例：执行前文创建的hello.sh脚本文件

```
[root@ubuntu test]# bash hello.sh 
 hello world
[root@ubuntu test]# sh hello.sh 
 hello world
```

-   通过脚本路径执行

语法：脚本的绝对路径或相对路径

案例：执行前文创建的hello.sh脚本文件

```
[root@ubuntu test]# /root/test/hello.sh
-bash: /root/test/hello.sh: Permission denied
[root@ubuntu test]# ./hello.sh
-bash: ./hello.sh: Permission denied
```

提示权限不足。***如果通过路径这种方式运行，需要修改文件的执行权限\***，（默认的权限不可以执行，并且在数据脚本路径时按tab键也无补全提示），此处先按照如下命令添加脚本的可执行权限即可。

```
[root@ubuntu test]# ll
total 12
-rw-r--r--. 1 root root  4 Dec  1 09:48 hello.sh
[root@ubuntu test]# chmod 777 hello.sh 
[root@ubuntu test]# ll
total 8
-rwxrwxrwx. 1 root root 53 Dec  1 09:22 hello.sh
```

再次执行，此时输入路径在按tab键也会提示自动补全了

```
[root@ubuntu test]# ./hello.sh 
 hello world
[root@ubuntu test]# /root/test/hello.sh 
 hello world
```

### 二、Shell编程语法

掌握Shell脚本运行方式后，我们正式开始进入Shell编程语法学习。

任何一门语言的学习都没有速成方法，都离不开大量的练习，只有多敲才能更加熟练，才能理解的更加深刻。

#### 2.1 变量

Linux Shell（此处为默认bash）中的变量分为系统变量和用户自定义变量，系统变量包括`$HOME`、`$PWD`、`$SHELL`、`$USER`等，用户自定义变量则为用户根据实际需要自定义的变量。

可以通过set命令查看Shell中所有变量。

```bash
[root@ubuntu test]# set
BASH=/bin/bash
... # 省略部分内容
```

下文主要演示自定义变量

##### 2.1.1 定义变量

语法：

 ***变量名=值\***

变量命名规则：

 （1）变量名可以由字母、数字和下划线注册，但不能以数字开头

 （2）等号两侧不能有空格

 （3）变量名一般大写

案例：

（1）定义变量并使用

```
[root@ubuntu test]# SRT="wellcome"
[root@ubuntu test]# echo $SRT
wellcome
[root@ubuntu test]# set |grep SRT
SRT=wellcome
[root@ubuntu test]# env |grep SRT
[root@ubuntu test]# 
```

可以看到自定义变量SRT，可以通过set命令查询出来。

（2）变量提升为全局环境变量

```
[root@ubuntu test]# export SRT
[root@ubuntu test]# env |grep SRT
SRT=wellcome
```

上例中自定义的变量SRT通过env命令查看，并未在环境变量中查询出来，通过export将自定义变量SRT提升为环境变量后，就可以通过env查询出来

（3）撤销变量

```
[root@ubuntu test]# unset SRT
[root@ubuntu test]# echo $SRT

[root@ubuntu test]# set |grep SRT
[root@ubuntu test]# env |grep SRT
```

撤销变量使用unset命令，撤销之后变量将不存在

##### 2.2 变量赋值

除了直接赋值，还可以将命令执行的结果赋值给变量

语法：

 ***变量=`命令`\*** 或 ***变量=$(命令)\***

说明：

 命令用反引号或$()包含起来，先执行命令然后将命令执行结果赋值给变量。

案例：

```
[root@ubuntu test]# ls
hello.sh  test1.txt  test2.txt
[root@ubuntu test]# RESULT=`ls`
[root@ubuntu test]# echo $RESULT
hello.sh test1.txt test2.txt
```

##### 2.3 位置参数变量

位置参数变量主要用于取脚本的参数值，语法如下

| 变量名称                                                     | 功能                                       |
| :----------------------------------------------------------- | :----------------------------------------- |
| $n       | n为数字，$0表示命令本身，$1-9表示第一到第九个参数，十以上的参数需要用大括号包含，如第十个参数为${10} |                                            |
| $*                                                           | 表示命令行中所有的参数，把参数看成一个整体 |
| $@                                                           | 表示命令行中所有的参数，把每个参数区分对待 |
| $#                                                           | 表示命令行中所有参数的个数                 |

案例：

输入2个参数，计算俩个数的和并打印输出

```
[root@ubuntu test]# vi sum.sh
#sum
#!/bin/bash
#分别接收2个参数
num1=$1
num2=$2
#求和
sum=$(($num1+$num2))
#打印
echo $sum
```

保存并退出，执行脚本输入2个数查看执行结果

```
[root@ubuntu test]# bash sum.sh 1 2
3
```

##### 2.4 预定义变量

预定义变量都有特殊的作用，参看下表

| 变量名 | 功能                                                         |
| :----- | :----------------------------------------------------------- |
| $?     | 表示***当前进程中\***最后一条命令直接的返回状态。0：执行成功；非0：执行失败 |
| $$     | 当前进程号（PID）                                            |
| $!     | ***后台运行\***的最后一个进程的进程号（PID）                 |

案例：

（1）查看当前进程和后台运行的最后一个进程

```
[root@ubuntu test]# vi mypid.sh 
#!/bin/bash
#输出当前进程PID，也就是当前脚本运行时生成的PID
echo "当前进程PID=$$"
echo "最后一个后台进程PID=$!"
```

保存退出，执行脚本

```
[root@ubuntu test]# bash mypid.sh 
当前进程PID=7810
最后一个后台进程PID=
```

可以看到$$和 $!的区别，$$表示当前进程PID，而$!表示是后台最有运行的一个进程的PID。

（2）输出当前进程PID，并查看上一次命令执行结果

```
[root@ubuntu test]# vi pid.sh
#!/bin/bash
#输出当前进程PID，也就是当前脚本运行时生成的PID
echo "当前进程PID=$$"
#通过ls命令，查找不存在的文件，&表示让命令后台执行
ls -l XXX.txt&
echo "最后一个进程PID=$!"
echo "最后一条命令执行结果：$?"
```

保存退出，执行脚本

```
[root@ubuntu test]# bash pid.sh 
当前进程PID=7395
最后一个进程PID=7396
最后一条命令执行结果：0
[root@ubuntu test]# ls: cannot access XXX.txt: No such file or directory

[root@ubuntu test]# 
```

可以看到命令执行的进程和当前脚本运行的进程不是同一个，并且虽然xxx.txt文件不存在，但结果仍然返回为0。如果改为查找一个已经存在的文件，毋庸置疑，返回结果肯定仍然为0。也就是说上边脚本不管命令执行是否成功，都将返回0，原因是通过&让命令后台执行，实际上是新开了一个进程，而$?只能获取到当前进程的最后一次执行命令结果，因此在做判断命令是否执行成功是特别要小心。

#### 2.2 运算符和表达式

语法：

（1）***$((运算式)) 或 $[运算式]\***

（2）expr m + n ***注意expr运算符间要有空格\***

案例：

（1）采用$(())实现两数相加

```
[root@ubuntu test]# S=$((2+3))
[root@ubuntu test]# echo $S
5
```

（2）采用$[]实现两数相加

```
[root@ubuntu test]# SUM=$[2+3]
[root@ubuntu test]# echo $SUM 
5
```

（3）采用expr命令实现两数相加

```
[root@ubuntu test]# S1=`expr 2 + 3` 
[root@ubuntu test]# echo $S1
5
```

注意expr命令时，操作符之间一定要有空格，否则执行的不是计算，而是字符串连接，如下示例演示了有空格和无空格的区别，另外乘法符号*由于与通配符冲突，因此需要用\转义

```
[root@ubuntu test]# expr 2 + 3
5
[root@ubuntu test]# expr 2+3
2+3
[root@ubuntu test]# expr 2\*3
2*3
[root@ubuntu test]# expr 2 \* 3
6
[root@ubuntu test]# expr 2 * 3 
expr: syntax error
```

（4）采用expr命令计算“2加3的和乘以5”

```
[root@ubuntu test]# S2=`expr 2 + 3`
[root@ubuntu test]# echo $S2
5
[root@ubuntu test]# expr $S2 \* 5
25
```

注意操作符之间的空格，以上为分步计算，也可以直接一步计算，多层嵌套是注意需要转义反引号，如下：

```
[root@ubuntu test]# expr `expr 2 + 3` \* 5
25
[root@ubuntu test]# echo `expr \`expr 2 + 3\` \* 5`
25
```

#### 2.3 条件判断语句

##### 2.3.1 条件判断基本使用

语法：[ 条件 ]

说明：条件前后必须要有空格，非空返回true，可以使用$?验证（0：true，非0：false）

案例：

（1）分别判断存在和不存在的变量，检验返回值

```
[root@ubuntu test]# [ $HOME ]  
[root@ubuntu test]# echo $?    
0
[root@ubuntu test]# [ $TEST ]      
[root@ubuntu test]# echo $?    
1
```

由于$HOME是环境变量肯定存在，因此返回0，表示条件满足，变量不为空；而TEST变量由于没定义，所以不存在，返回非0。

（2）条件判断语句用于逻辑判断

```
[root@ubuntu test]# [ $HOME ]&& echo ok || echo notok
ok
[root@ubuntu test]# [ $TEST ]&& echo ok || echo notok     
notok
```

类似于其他语言中的三元运算符，条件满足则执行紧随其后的语句，否则不执行。

##### 2.3.2 常用判断条件

-   两个整数之间比较

| 符号 | 含义       |
| :--- | :--------- |
| =    | 字符串比较 |
| -lt  | 小于       |
| -le  | 小于等于   |
| -eq  | 等于       |
| -gt  | 大于       |
| -ge  | 大于等于   |
| -ne  | 不等于     |

-   按文件权限判断

| 符号 | 含义         |
| :--- | :----------- |
| -r   | 有读的权限   |
| -w   | 有写的权限   |
| -x   | 有执行的权限 |

-   按文件类型判断

| 符号 | 含义                     |
| :--- | :----------------------- |
| -f   | 文件存在且是一个常规文件 |
| -d   | 文件存在并且是一个目录   |
| -e   | 文件存在                 |

-   案例

（1）整数比较

```
[root@ubuntu test]# [ 1 -gt 2 ]
[root@ubuntu test]# echo $?    
1
```

1不大于2，所以输出1，表示false

（2）文件类型判断

```
[root@ubuntu test]# [ -f test1.txt ]
[root@ubuntu test]# echo $?
0
[root@ubuntu test]# [ -f xxxx.txt ]     
[root@ubuntu test]# echo $?        
1
```

test1.txt文件存在，输出0表示true；而xxxx.txt文件不存在，所以输出1表示false

（3）文件权限判断

```
[root@ubuntu test]# ll
-rw-r--r--. 1 root root   9 Nov 30 20:43 test1.txt
[root@ubuntu test]# [ -x test1.txt ]
[root@ubuntu test]# echo $?
1
```

由于test1.txt 无执行权限，因此返回1，表示false

#### 2.4 流程控制语句

流程控制结构分为：顺序结构、分支结构、循环结构。顺序结构根据语句依次顺序执行，分支结构根据条件判断运行不同的分支，循环结构则根据条件判断是否循环执行。

##### 2.4.1 分支语句

分支语句分为：if判断语句、case语句

-   if判断

语法：

------

if [ 判断条件 ]

then

 程序

fi

------

或者then写到判断条件之后，用逗号隔开,等效于上边语句

if [ 判断条件 ];then

 程序

fi

------

if判断还可以多层嵌套，形如：

if [ 判断条件1 ]

then

 程序1

elif [ 判断条件2 ]

 then

 程序2

else

 程序3

fi

------

说明：条件判断语句中，注意***前中括号与if之间必须有空格，中括号前后也必须都有空格\***

案例：

从键盘输入年龄，根据年龄返回不同的形容语言

```
[root@ubuntu test]# vim if.sh
#!/bin/bash
read -p "请输入您的年龄:" AGE
if [ $AGE -le 18 ];then
      echo "未成年"
  elif [ $AGE -le 30 ];then
      echo "年轻气盛"
  else
      echo "糟老头子"
fi
```

使用vim编辑器编辑内容（此处使用vim是因为vim比vi更适合在编程环境使用，有错误提示等功能，建议编写脚本都采用vim），保存退出并执行脚本

```
[root@ubuntu test]# bash if.sh 
请输入您的年龄:16
未成年
[root@ubuntu test]# bash if.sh 
请输入您的年龄:40
糟老头子
```

此处使用了read命令读取键盘输入，-p参数表示显示提示符内容。

-   case语句

语法：

------

case $变量 in

 " 值1")

 语句1

 ;;

 " 值2")

 语句2

 ;;

 *)

 语句

 ;;

esac

------

案例：

根据传入脚本的参数分别输出1、2、3对应的英文

```
[root@ubuntu test]#vim case.sh
#!/bin/bash
case $1 in
  1)
    echo one
    ;;
  2)
    echo two
    ;;
  3)
    echo three
    ;;
  *)
    echo "error"
    ;;
esac
```

保存退出并执行

```
[root@ubuntu test]# bash case.sh 1
one
[root@ubuntu test]# bash case.sh 2
two
[root@ubuntu test]# bash case.sh 3
three
[root@ubuntu test]# bash case.sh 8
error
```

##### 2.4.2 循环语句

循环语句分为：for循环、while循环

-   for循环语句

for循环有两种语法格式，分别如下：

语法1：

------

for 变量 in 值1 值2 值3...

 do

 程序

 done

------

语法2：

------

for ((初始值；循环控制条件；变量变化))

do

 程序

 done

------

案例：

（1）一天三次问好

```
[root@ubuntu test]#vim greeting.sh 
#!/bin/bash
for time in morning afternoon evening
  do
     echo "good $time"
  done
```

保存并退出，执行问候脚本，输出问候语句

```
[root@ubuntu test]# bash greeting.sh 
good morning
good afternoon
good evening
```

（2）用for循环求1累加到5的和

```
[root@ubuntu test]# vim getsum.sh
#!/bin/bash
sum=0
for ((i=0;i<=5;i++))
   do
     sum=$(($i+$sum))
   done
echo "sum=$sum"
```

保存退出，并执行脚本

```
[root@ubuntu test]# bash getsum.sh 
sum=15
```

-   while循环语句

语法：

------

while [ 条件判断式 ]

 do

 程序

 done

------

案例：

（1）用while循环求1累加到5的和

```
[root@ubuntu test]# vim while.sh  
#!/bin/bash
i=1
sum=0
while [ $i -le 5 ]
    do
     sum=$[$i+$sum]
     i=$[$i+1]
    done
echo "sum=$sum" 
```

保存并执行

```
[root@ubuntu test]# bash while.sh 
sum=15
```

>   注意：
>
>   1、是给变量赋值，左边不用加$符号，比如i=$[$i+1]不能写成$i=$[$i+1]，否则报错“command not found”
>
>   2、条件判断语句注意左中括号与关键字之间、括号内部首位、操作符与操作数之间都有空格
>
>   3、i=$[$i+1]，如果错写为i=$i+1则会报错“integer expression expected”。

#### 2.5 函数

函数是对功能的封装，可以提供代码重用性。函数分为系统函数和用户自定义函数，对于系统函数直接拿来使用即可，自定义函数则是根据具体需求编写。

##### 2.5.1 系统函数

使用现有的系统函数可以提高工作效率，由于篇幅所限，只简单介绍两个与文件路径相关的系统函数，具体使用方法也可以通过前面介绍的man命令来查询函数具体的用法。

-   basename

语法：

 basename [文件路径或字符串] [后缀]

功能描述：

 删掉所有的前缀，包括最后一个/,然后打印字符串；指定了后缀则在此基础上再去掉后缀。

案例：

```
[root@ubuntu test]# basename "sdf/sdf/sdf"
sdf
[root@ubuntu test]# basename /root/test/test1.txt 
test1.txt
[root@ubuntu test]# basename /root/test/test1.txt txt
test1.
[root@ubuntu test]# basename /root/test/test1.txt .txt
test1
```

-   dirname

语法：

 dirname 文件绝对路径

功能描述：

 从给定的包含绝对路径的文件名中去除文件名，然后返回生效的路径。简单说就是去掉非目录部分，返回保留目录部分，但不包含最后的/

案例：

```
[root@ubuntu test]# dirname /root/test/test1.txt    
/root/test
```

##### 2.5.2 自定义函数

语法：

------

[function] 函数名[()]

{

 语句；

 [return 返回值;]

}

------

说明：

（1）shell是解释执行而非编译执行，因此语句是逐行执行的，必须在函数调用之前先声明函数。

（2）函数返回值只能通过$?系统变量获取。可以显示添加return语句返回（返回值范围为0-255），否则将以最后一条命令运行结果作为返回值返回。

>   方括号内的部分function、参数、返回值可以是可选的。function show() 和function show和show()这几种形式都是可以的。

案例：

（2）输入1个整数，打印出比输入小的整数

```
[root@ubuntu test]# vim function.sh  
#!/bin/bash
function printNumber()
{
  i=0;
  while [ $i -lt $1 ]
  do
     echo $i;
     i=$[$i+1];
#    i=`expr \`expr $i + 1 \``     
     sleep 1;
  done
  return 0;
}
read -p "请输入一个数：" n;
printNumber $n;
```

保存文件并执行脚本

```
[root@ubuntu test]# bash function.sh 
请输入一个数：5
0
1
2
3
4
```

### 三、编写Shell脚本

通过上边的演示，基本已经会编写简单的shell脚本了，本小节讲解shell脚本接收用户参数以及用户参数的判断。

#### 3.1 接收用户参数

之前提到过对linux命令来说，能否根据需要采用各种参数组合来完成特定的功能才是衡量命令是否掌握的标准。同样，函数或脚本也需要与用户交互，能灵活处理用户参数。

前面2.3中中已经提到几个shell内设的用于接收参数的变量：$1...,$*,$@,$#

下面用案例进行演示

```
[root@ubuntu test]# vim para.sh
#!/bin/bash
echo "当前脚本名称：$0"
echo "总共有$#个参数，分别为：$*"
echo "第一个参数为:$1,第三个参数为：$3"
```

保存脚本，并执行

```
[root@ubuntu test]# bash para.sh 1 2 3
当前脚本名称：para.sh
总共有3个参数，分别为：1 2 3
第一个参数为:1,第三个参数为：3
```

可以看到，脚本名称后直接带参数，参数之间用空格隔开，在脚本内部直接可以获取到各个位置上的参数

#### 3.2 判断用户参数

shell脚本中条件判断语句可以用来判断表达式是否成立，若条件成立返回0表示true，否则返回其他非0随机数表示false。按判断对象的不同，可以分为以下四种判断语句：文件测试语句、逻辑测试语句、整数值比较语句、字符串比较语句。

##### 3.2.1 文件测试语句

文件测试即是用指定的条件判断文件是否存在或权限是否满足，文件测试具体参数如下

| 运算符 | 作用                       |
| :----- | :------------------------- |
| -d     | 测试文件是否为目录类型     |
| -e     | 测试文件是否存在           |
| -f     | 判断是否为一般文件         |
| -r     | 测试当前用户是否有读权限   |
| -w     | 测试当前用户是否有写权限   |
| -x     | 测试当前用户是否有执行权限 |

案例：

判断是否为目录类型的文件

```
[root@ubuntu test]# [ -d /root/test ]
[root@ubuntu test]# echo $?
0
[root@ubuntu test]# [ -d /root/test/test1.txt ]
[root@ubuntu test]# echo $?
1
```

先通过条件测试语句进行判断，再使用shell解释器内置的$?变量查看执行结果。由于/root/test为目录，所以返回0；test1.txt是文件，所以返回1.

##### 3.2.2 逻辑测试语句

逻辑语句用于对测试结果进行逻辑分析，根据测试结果可实现不同的效果。逻辑运算符如下：

| 运算符 | 作用                                                 |
| :----- | :--------------------------------------------------- |
| &&     | 逻辑与，表示当前面的命令执行成功后才会执行后边的命令 |
| \|\|   | 逻辑或，表示当前面的命令执行失败后才会执行后边的命令 |
| ！     | 逻辑非，表示把条件测试中判断结果取反。               |

案例：

判断当前登录用户是否为管理员

```
[root@ubuntu test]# [ ! $USER = root ] && echo "user" || echo "root"
root
```

##### 3.2.3 整数值比较语句

整数比较运算符仅是对数字的操作，不能将数字与字符串、文件等内容一起操作，而且不能想当然地使用日常生活中的等号、大于号、小于号等来进行判断。

>   ps：等号与赋值命令符冲突，大于小于号分别与输出输入重定向命令符冲突。因此一定要按照规范的整数比较运算符来进行操作。

可用的整数比较运算符如下表：

| 运算符 | 作用           |
| :----- | :------------- |
| -eq    | 是否等于       |
| -ne    | 是否不等于     |
| -gt    | 是否大于       |
| -lt    | 是否小于       |
| -le    | 是否小于或等于 |
| -ge    | 是否大于或等于 |

案例：

```
[root@ubuntu test]# [ 1 -lt 2 ]
[root@ubuntu test]# echo $?    
0
[root@ubuntu test]# [ 1 -gt 2 ]  
[root@ubuntu test]# echo $?    
1
```

##### 3.3.4 字符串比较语句

字符串比较语句用于判断测试字符串是否为空或两个字符串是否相等。常用来判断某个变量是否未被定义，即内容为空值。字符串常见运算符如下表：

| 运算符 | 作用                            |
| ------ | ------------------------------- |
| =      | 比较字符串内容是否相同，相同为0 |
| !=     | 比较字符串内容是否不同，不同为0 |
| -z     | 判断字符串内容是否为空，空则为0 |

案例：

分别查看存在和不存在的变量，区别返回值

```
[root@ubuntu test]# echo $USER
root
[root@ubuntu test]# echo $TEST

[root@ubuntu test]# [ -z $USER ]
[root@ubuntu test]# echo $?
1
[root@ubuntu test]# [ -z $TEST ]
[root@ubuntu test]# echo $?     
0
```

至此，shell编程相关知识就介绍完毕，下一篇文章继续学习与用户和文件、权限相关的知识。