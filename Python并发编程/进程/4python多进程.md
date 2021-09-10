# python程序中的进程操作

运行起来的python程序也是一个进程，那么我们也可以在程序中再创建进程。多个进程可以实现并发效果，也就是说，当我们的程序中存在多个进程的时候，在某些时候，就会让程序的执行速度变快。以我们之前所学的知识，并不能实现创建进程这个功能，所以我们就需要借助python中强大的模块。

## 1、multiprocess模块

 仔细说来，multiprocess不是一个模块而是python中一个操作、管理进程的包。 之所以叫multi是取自multiple的多功能的意思，在这个包中几乎包含了和进程有关的所有子模块。

## 2、Process模块

```python
p = Process([group [, target [, name [, args [, kwargs]]]]])
# 由该类实例化得到的对象，表示一个子进程中的任务（尚未启动）
```

> 强调：s
>
> 1、需要使用关键字的方式来指定参数 
>
> 2、args指定的为传给target函数的位置参数，是一个元组形式，必须有逗号

参数介绍：

- group参数未使用，值始终为None
- target表示调用对象，即子进程要执行的任务，即函数
- args表示调用对象的位置参数元组，`args=(1,2,'egon',)`
- kwargs表示调用对象的字典，`kwargs={'name':'egon','age':18}`
- name为子进程的名称

### 2.1、方法

- `p.start()`：启动进程，并调用该子进程中的p.run()
- `p.run()`：进程启动时运行的方法，正是它去调用target指定的函数，我们自定义类的类中一定要实现该方法
- `p.terminate()`：强制终止进程p，不会进行任何清理操作，如果p创建了子进程，该子进程就成了僵尸进程，使用该方法需要特别小心这种情况。如果p还保存了一个锁那么也将不会被释放，进而导致死锁
- `p.is_alive()`：如果p仍然运行，返回True
- `p.join([timeout])`：主线程等待p终止（强调：是主线程处于等的状态，而p是处于运行的状态）。timeout是可选的超时时间，需要强调的是，p.join只能join住start开启的进程，而不能join住run开启的进程

### 2.2 属性介绍

- `p.daemon`：默认值为False，如果设为True，代表p为后台运行的守护进程，当p的父进程终止时，p也随之终止，并且设定为True后，p不能创建自己的新进程，必须在`p.start()`之前设置
- `p.name`：进程的名称
- `p.pid`：进程的pid
- `p.exitcode`：进程在运行时为None、如果为–N，表示被信号N结束(了解即可)
- `p.authkey`：进程的身份验证键,默认是由`os.urandom()`随机生成的32字符的字符串。这个键的用途是为涉及网络连接的底层进程间通信提供安全性，这类连接只有在具有相同的身份验证键时才能成功（了解即可）

### 2.3 在windows中使用Process类的注意事项

在Windows操作系统中由于没有fork(linux操作系统中创建进程的机制)，在创建子进程的时候会自动 import 启动它的这个文件，而在 import 的时候又执行了整个文件。因此如果将process()直接写在文件中就会无限递归创建子进程报错。所以必须把创建子进程的部分使用`if __name__ =='__main__'` 判断保护起来，import 的时候，就不会递归运行了。



## 3、Process的使用

### 3.1、启动一个多进程任务

- 方式一：通过函数

```python
from multiprocessing import Process
import time


def a():
    for i in range(4):
        print(os.getpid(), os.getppid(), multiprocessing.current_process().name)
        print('today is a good day')
        time.sleep(1)

def b():
    print('lucky is a good man')


if __name__ == '__main__':
    # p = Process(target=干的活（函数名）, args=参数, name='进程名')
    p = Process(target=a)  #创建进程对象  执行的任务为a
    p.start()  # 启动多进程
    # a()
    b()  # 在主进程中执行方法b
```

- 方式二：通过类

```python
from multiprocessing import Process
import time

'''
开启子进程的两种方式:
1. 通过函数
2. 通过类。继承Process
'''

class myProcess(Process):
    def __init__(self,name):
        # self.name = name #错误 ### 这样没有给对象添加属性name，而是在修改父类的进程名(name)

        # 父类Process的进程名也是name
        super().__init__()  # 调用父类super().init方法完成创建进程初始化，重新给name属性赋值了。
        self.name = name    ## 在父类的init方法后设置name，才是为自己对象添加属性

        # super().__init__(name=name)  # 调用父类super().init,并设置进程名(name)


    def run(self):  # 创建进程会默认调用run方法
        print("进程%s start" % (self.name))
        time.sleep(2)
        print(f"进程{self.name} end")
        
# 必须加main
if __name__ == '__main__':
    p = myProcess("xc")
    p.start()
    print('主进程/父进程')
    print('主进程/父进程')
    p1 = myProcess("cyx")
    p1.start()  # 告诉操作系统启动子进程，但一定是父进程先执行，多个子进程的执行顺序是根据操作系统调度决定的
    print('主进程/父进程')
    print('主进程/父进程')
    print('主进程/父进程')
    print('主进程/父进程')
```

### 3.2、进程常用方法/属性

+ os.getpid()  获取当前进程id号
+ os.getppid()  获取进程的父进程id号
+ multiprocessing.current_process().name   获取当前进程名称

### 3.3、父子进程的先后顺序

默认：主进程的结束不会影响子进程

### 3.4、join 方法

+ 阻塞当前进程  直到调用join方法的那个进程执行完毕  才会向下执行（主进程等待子进程）

  ```python
  	...
  if __name__ == '__main__':
      # p = Process(target=干的活（函数名）, args=参数, name='进程名')
      p = Process(target=a, name='进程1')
      p.start()  # 启动多进程
      p.join()  # 进程阻塞 等待子进程执行完毕 在执行主进程
      b()  # 主进程调用b
  ```

+ 主进程一般负责任务的调度

  自己不负责处理业务逻辑

  ```python
  from multiprocessing import Process
  import time
  import os
  import multiprocessing
  
  def a():
      for i in range(4):
          print(os.getpid(), os.getppid(), multiprocessing.current_process().name)
          print('today is a good day')
          time.sleep(1)
  
  def b():
      print(os.getpid(), os.getppid(), multiprocessing.current_process().name)
      print('lucky is a good man')
  
  
  if __name__ == '__main__':
      # p = Process(target=干的活（函数名）, args=参数, name='进程名')
      p1 = Process(target=a, name='进程1')
      p2 = Process(target=b, name='进程2')
      p1.start()  # 启动多进程
      p2.start()
      p1.join()  # 进程阻塞 等待子进程执行完毕 在执行主进程
      p2.join()
      print('over')
  ```

### 3.5、全部参数的完整实例

  ```python
  import multiprocessing
  from multiprocessing import Process
  import time
  
  
  def a(count):
      for i in range(count):
          print(multiprocessing.current_process().name)
          time.sleep(1)
  
  def b(count):
      for i in range(count):
          print(multiprocessing.current_process().name)
          time.sleep(1)
  
  
  if __name__ == '__main__':
      t1 = time.time()
      p1 = Process(target=a, args=(5, ), name='process_1')
      p2 = Process(target=b, args=(5, ), name='process_2')
      p1.start()
      p2.start()
      p1.join()
      p2.join()
      # a()
      # b()
      t2 = time.time()
      print(t2 - t1)
      print('over')
  ```

