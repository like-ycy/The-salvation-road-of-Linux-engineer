# python程序中的进程操作

运行起来的python程序也是一个进程，那么我们也可以在程序中再创建进程。多个进程可以实现并发效果，也就是说，当我们的程序中存在多个进程的时候，在某些时候，就会让程序的执行速度变快。以我们之前所学的知识，并不能实现创建进程这个功能，所以我们就需要借助python中强大的模块。

## multiprocess模块

 仔细说来，multiprocess不是一个模块而是python中一个操作、管理进程的包。 之所以叫multi是取自multiple的多功能的意思，在这个包中几乎包含了和进程有关的所有子模块。

## Process模块

#### 1、作用

由该类实例化得到的对象，表示一个子进程中的任务（尚未启动）

#### 2、导入

```python
from multiprocess import Process
```

#### 3、参数

```python
p = Process(target=干的活（函数名）, args=参数, name='进程名')
```

- target  函数名
- args   给任务传递的参数
- name  给当前进程起名

!> 强调： 

1. 需要使用关键字的方式来指定参数 
2. args指定的为传给target函数的位置参数，是一个元组形式，必须有逗号

#### 4、启动一个多进程任务

p.start()

**实例**

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

!> 强调：在Windows操作系统中由于没有fork(linux操作系统中创建进程的机制)，在创建子进程的时候会自动import 启动它的这个文件，而在 import 的时候又执行了整个文件。因此如果将process()直接写在文件中就会无限递归创建子进程报错。所以必须把创建子进程的部分使用if _*name*_ ==‘__main__’ 判断保护起来，import 的时候 ，就不会递归运行了。

#### 5、进程常用方法/属性

+ os.getpid()  获取当前进程id号
+ os.getppid()  获取进程的父进程id号
+ multiprocessing.current_process().name   获取当前进程名称

#### 6、父子进程的先后顺序

默认：主进程的结束不会影响子进程

#### 7、join 方法

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

#### 8、全部参数的完整实例

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


