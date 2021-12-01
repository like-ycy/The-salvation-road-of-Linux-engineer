# StringIO、BytesIO

说起IO，很多人首先想到的是磁盘中的文件，将磁盘中的文件读到内存以及内存内容写入文件。但是还有一种内存和内存之间的IO，叫类文件对象，这一篇我们就一起来学习下python中的两个类文件对象：StringIO和BytesIO。

## 内存中的IO

首先必须要搞清一个问题，就是为什么要有内存级别的IO？

之前说的磁盘上的文件，就是将数据持久化到磁盘的一块区域，供后面重复使用。其优点就是持久化稳定不丢失，但是缺点也很明显，就是每次要使用都要从磁盘读入，相对内存而言很缓慢。

如果只是短时间的重复利用，并不希望长期持久化，而且对速度的要求比较高，这时候就可以考虑缓存。说到缓存，很多朋友就想到redis，熟悉python的朋友还会想到装饰器和闭包函数。

不过python已经原生为我们准备好了类文件对象（file-like object），这种对象在内存中创建，可以像文件一样被操作。

下面我们就来学习下两种类文件对象，StringIO和BytesIO。

## StringIO

**作用：**在内存中读写str

要把str写入StringIO，我们需要先创建一个StringIO，然后，像文件一样写入即可：

```python
>>> from io import StringIO
>>> f = StringIO()
>>> f.write('hello')
5
>>> f.write(' ')
1
>>> f.write('world!')
6
>>> print(f.getvalue())
hello world!
# 或者移动指针到开头
>>> f.seek(0)
>>> print(f.read())
```

`getvalue()`方法用于获得写入后的str。

要读取StringIO，可以用一个str初始化StringIO，然后，像读文件一样读取：

```python
>>> from io import StringIO
>>> f = StringIO('Hello!\nHi!\nGoodbye!')
>>> while True:
...     s = f.readline()
...     if s == '':
...         break
...     print(s.strip())
...
Hello!
Hi!
Goodbye!
```



## BytesIO

**作用：**StringIO操作的只能是str，如果要操作二进制数据，就需要使用BytesIO。

BytesIO实现了在内存中读写bytes，我们创建一个BytesIO，然后写入一些bytes：

```
>>> from io import BytesIO
>>> f = BytesIO()
>>> f.write('中文'.encode('utf-8'))
6
>>> print(f.getvalue())
b'\xe4\xb8\xad\xe6\x96\x87'
```

!>请注意，写入的不是str，而是经过UTF-8编码的bytes。

和StringIO类似，可以用一个bytes初始化BytesIO，然后，像读文件一样读取：

```
>>> from io import BytesIO
>>> f = BytesIO(b'\xe4\xb8\xad\xe6\x96\x87')
>>> f.read()
b'\xe4\xb8\xad\xe6\x96\x87'
```