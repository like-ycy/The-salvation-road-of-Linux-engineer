# OS模块



作用：包含了操作系统的基本功能 提供了非常丰富的用来处理文件和目录的函数或属性



### 1、属性（了解）

| 函数名 | 函数说明                          |
| ------ | --------------------------------- |
| name   | 获取操作系统的类型                |
| uname  | 获取操作系统的信息（用在Linux下） |
| curdir | 返回当前的目录                    |

### 2、方法

| 函数名                 | 函数说明                                       |
| ---------------------- | ---------------------------------------------- |
| getcwd()               | 返回当前工作目录的绝对路径                     |
| listdir(path)          | 以列表的形式返回当前工作目录下的所有文件和目录 |
| mkdir(目录名)          | 创建指定的目录 如果指定的目录已存在 则报错     |
| rmdir(目录名)          | 删除指定的目录 如果指定的目录不存在 则报错     |
| rename(文件名, 新名字) | 重命名                                         |
| remove(文件名)         | 删除文件                                       |
| system(命令)           | 运行命令   shutdow -s -f                       |

### 3、os.path常用方法

| 函数名    | 函数说明                      |
| --------- | ----------------------------- |
| abspath() | 返回指定路径的绝对路径        |
| join()    | 拼接路径 无论是否存在         |
| split()   | 拆分路径  无论是否存在        |
| splitext  | 获取文件的扩展名 无论是否存在 |
| getsize() | 获取文件大小 返回字节         |
| isdir()   | 判断是否为目录                |
| isfile()  | 判断是否为文件                |
| exists()  | 判断文件或目录是否存在        |

### 4、案例

##### 1、递归遍历目录

```python
import os

def demo(path):
    # 获取第一层里面的所有文件和目录
    ListDir = os.listdir(path)
    # print(ListDir)
    for f in ListDir:
        # 拼凑完整路径
        newPath = os.path.join(path, f)
        # print(newPath)
        # 把所有的文件都输出
        if os.path.isdir(newPath):
            demo(newPath)

        if os.path.isfile(newPath):
            print(f)

path = r'C:\Users\xlg\PycharmProjects\online5'
demo(path)
```

##### 2、递归统计大小

```python
import os

def getSize(path):
    Sum = 0
    ListDir = os.listdir(path)
    for f in ListDir:
        newPath = os.path.join(path, f)
        if os.path.isdir(newPath):
            Sum += getSize(newPath)

        if os.path.isfile(newPath):
            Sum += os.path.getsize(newPath)

    return Sum
path = r'C:\Users\xlg\PycharmProjects\online5'

print(getSize(path))
```

##### 3、检索文件小程序

```python
import os, time

def getFileSuffix(path, fileList, suffix):
    # 获取目录下的所有文件和目录的列表
    ListDir = os.listdir(path)
    # 循环迭代获取每一个文件或目录
    for f in ListDir:
        # 拼凑新路径
        newPath = os.path.join(path, f)
        # 判断是否为目录
        if os.path.isdir(newPath):
            getFileSuffix(newPath, fileList, suffix)
        # 判断是否为文件
        if os.path.isfile(newPath):
            # 获取文件后缀
            fileSuffix = newPath.split('.')[-1]
            # 判断是否为要检索得文件
            if fileSuffix.upper() == suffix.upper():
                # 是则扔进列表中
                fileList.append(f)

# 将获取到的文件进行展示的函数
def showFile(path, suffix='py'):
    fileList = []
    # 判断路径是否存在
    getFileSuffix(path, fileList, suffix)
    if not fileList:
        print('您要检索后缀{} 不存在'.format(suffix))
        return
    print('您要检索后缀为：{}的文件个数为：{} 分别是：'.format(suffix, len(fileList)))
    time.sleep(2)
    for f in fileList:
        time.sleep(0.5)
        print(f)


while True:
    path = input('请输入要检索的路径：')
    suffix = input('请输入要检索的后缀（默认为py）：')
    if os.path.exists(path):
        if suffix:
            showFile(path, suffix)
        else:
            showFile(path)
    else:
        print('请输入正确的路径')
```

