# socket

## 一、什么是Socket

socket:在应用层和传输层之间的一个抽象层，它把TCP/IP层复杂的操作抽象为几个简单的接口供应用层调用已实现进程在网络中通信。在设计模式中，Socket其实就是一个门面模式，它把复杂的TCP/IP协议族隐藏在Socket接口后面，对用户来说，一组简单的接口就是全部，让Socket去组织数据，以符合指定的协议。

所以，我们无需深入理解tcp/udp协议，socket已经为我们封装好了，我们只需要遵循socket的规定去编程，写出的程序自然就是遵循tcp/udp标准的。

## 二、套接字发展史及分类

套接字起源于 20 世纪 70 年代加利福尼亚大学伯克利分校版本的 Unix,即人们所说的 BSD Unix。 因此,有时人们也把套接字称为“伯克利套接字”或“BSD 套接字”。一开始,套接字被设计用在同 一台主机上多个应用程序之间的通讯。这也被称进程间通讯,或 IPC。套接字有两种（或者称为有两个种族）,分别是基于文件型的和基于网络型的。

### 2.1 基于文件类型的套接字家族

套接字家族的名字：AF_UNIX

unix一切皆文件，基于文件的套接字调用的就是底层的文件系统来取数据，两个套接字进程运行在同一机器，可以通过访问同一个文件系统间接完成通信

### 2.2 基于网络类型的套接字家族

套接字家族的名字：AF_INET

(还有AF_INET6被用于ipv6，还有一些其他的地址家族，不过，他们要么是只用于某个平台，要么就是已经被废弃，或者是很少被使用，或者是根本没有实现，所有地址家族中，AF_INET是使用最广泛的一个，python支持很多种地址家族，但是由于我们只关心网络编程，所以大部分时候我么只使用AF_INET)

## 三、套接字工作流程

个生活中的场景。你要打电话给一个朋友，先拨号，朋友听到电话铃声后提起电话，这时你和你的朋友就建立起了连接，就可以讲话了。等交流结束，挂断电话结束此次交谈。 生活中的场景就解释了这工作原理。

[![img](4socket.assets/1549538-20190905200538392-2098549890.jpg)

先从服务器端说起。服务器端先初始化Socket，然后与端口绑定(bind)，对端口进行监听(listen)，调用accept阻塞，等待客户端连接。在这时如果有个客户端初始化一个Socket，然后连接服务器(connect)，如果连接成功，这时客户端与服务器端的连接就建立了。客户端发送数据请求，服务器端接收请求并处理请求，然后把回应数据发送给客户端，客户端读取数据，最后关闭连接，一次交互结束，使用以下Python代码实现：

```python
import socket

# socket_family 可以是 AF_UNIX 或 AF_INET。socket_type 可以是 SOCK_STREAM 或 SOCK_DGRAM。protocol 一般不填，默认值为 0
socket.socket(socket_family, socket_type, protocal=0)

# 获取tcp/ip套接字
tcpSock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 获取udp/ip套接字
udpSock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
```

### 3.1 服务端套接字函数

|    方法    |                     用途                     |
| :--------: | :------------------------------------------: |
|  s.bind()  |          绑定(主机,端口号)到套接字           |
| s.listen() |                 开始TCP监听                  |
| s.accept() | 被动接受TCP客户的连接,(阻塞式)等待连接的到来 |

### 3.2 客户端套接字函数

|      方法      |                          用途                           |
| :------------: | :-----------------------------------------------------: |
|  s.connect()   |                 主动初始化TCP服务器连接                 |
| s.connect_ex() | connect()函数的扩展版本,出错时返回出错码,而不是抛出异常 |

### 3.3 公共用途的套接字函数

|      方法       |                             用途                             |
| :-------------: | :----------------------------------------------------------: |
|    s.recv()     |                         接收TCP数据                          |
|    s.send()     | 发送TCP数据(send在待发送数据量大于己端缓存区剩余空间时,数据丢失,不会发完) |
|   s.sendall()   | 发送完整的TCP数据(本质就是循环调用send,sendall在待发送数据量大于己端缓存区剩余空间时,数据不丢失,循环调用send直到发完) |
|  s.recvfrom()   |                         接收UDP数据                          |
|   s.sendto()    |                         发送UDP数据                          |
| s.getpeername() |                 连接到当前套接字的远端的地址                 |
| s.getsockname() |                       当前套接字的地址                       |
| s.getsockopt()  |                     返回指定套接字的参数                     |
| s.setsockopt()  |                     设置指定套接字的参数                     |
|    s.close()    |                          关闭套接字                          |

### 3.4 面向锁的套接字方法

|      方法       |             用途             |
| :-------------: | :--------------------------: |
| s.setblocking() | 设置套接字的阻塞与非阻塞模式 |
| s.settimeout()  | 设置阻塞套接字操作的超时时间 |
| s.gettimeout()  | 得到阻塞套接字操作的超时时间 |

### 3.5 面向文件的套接字的函数

|     方法     |             用途             |
| :----------: | :--------------------------: |
|  s.fileno()  |      套接字的文件描述符      |
| s.makefile() | 创建一个与该套接字相关的文件 |

## 四、基于套接字编写一个简单的c/s架构的程序

C：client 客户机程序

```python
HOST = "192.168.11.237"

import socket

# 1.创建一个socket对象
soc=socket.socket()

# 2.连接服务端
soc.connect((HOST,8080))
print(f"连接服务器{HOST}成功")

# 3.通讯
while 1:
    flag = input("请输入数据：")
    if flag == "q":
        break

    try:
        soc.send(flag.encode("utf8"))
        print(f"发送数据成功！\n")

        print("等待接收数据。。。")
        data = soc.recv(1024)
        print('来自服务器的数据:  ', data.decode("utf"))
    except Exception as a:
        print("出错了:",a)
        break

# 4.关闭连接
soc.close()
```

S: Service 服务器端

```python
HOST = "192.168.11.237"

import socket

# 1.创建socket对象 :socket.SOCK_STREAM 表示建立tcp连接 ,udp连接socket.SOCK_DGRAM
soc = socket.socket(socket.AF_INET,socket.SOCK_STREAM)

# 2.绑定ip地址：传元组:(ip,端口号)
soc.bind((HOST,8080))  #如果写本机ip，局域网外部可以访问
# soc.bind(('127.0.0.1',8080))    #如果写127.0.0.1，只能自己访问

# 3.监听:等待连接 这个5 是半连接池的大小
soc.listen(5)

# 4.等待连接
print('等待连接中。。。')
conn,addr=soc.accept()
print(f'我被IP:{addr}连接了')

# 5.进行通讯

while 1:
    try:
        # 接收
        print("准备接收。。。")
        rec_data = conn.recv(1024)  # 最大字节，# 会阻塞
        print("来自客户端的数据:  ",rec_data.decode("utf8").upper(),"\n")
        # print('来自客户端的数据:', rec_data.decode("utf"))

        send_data = input("请输入数据：")
        if send_data == "q":
            break

        # 发送
        conn.send(send_data.encode("utf8"))

    except Exception as e:
        print("出错了:",e)
        break

# 6.释放连接
conn.close()

# 7.关闭socket通讯
soc.close()
```

有客户客户机连不上服务器，可能是因为防火墙的原因

## 五、基于UDP写一个聊天室

- 服务端

  ```python
  import socket
  
  addr = ('127.0.0.1', 8000)
  server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
  server.bind(addr)
  
  userdic = {}  # {addr:name}
  
  while True:
      # 接受客户端消息
      data, addr = server.recvfrom(1024)
      # 判断用户是否在字典中
      if addr not in userdic:
          # 循环拿到地址，然后群发欢迎消息，然后开始聊天
          for address in userdic:
              server.sendto(data + ': 加入了群聊'.encode('utf-8'), address)
          userdic[addr] = data.decode('utf-8')
          continue
  
      # 退出的处理
      if data.decode('utf-8') == 'q':
          userdic.pop(addr)
          for address in userdic:
              name = userdic[addr]
              server.sendto(name + '：离开了聊天室', address)
      else:
          # 正常的聊天内容
          print(data.decode('utf-8'))
  
          # !广播给所有人，排除自己，自己不用再看一遍自己发的信息了
          for address in userdic:
              if address != addr:
                  server.sendto(data, address)
  
  ```

- 客户端

  ```python
  import socket
  import threading
  
  # 接受消息
  def recv(client, addr):
      # !先一步发送名字给server，以便server创建用户字典
      client.sendto(name.encode('utf-8'), addr)
      while True:
          data = client.recv(1024)
          print(data.decode('utf-8'))
  
  
  # 发送消息
  def send(client, addr):
      while True:
          msg = input('请输入你要发送的信息：')
          # 拼接名字和数据
          message = name + ':' + msg
          data = message.encode('utf-8')
          client.sendto(data, addr)
          # 判断输入的是否为q，退出
          if msg == 'q':
              break
  
  
  # 生成两个线程，接受消息和发送消息互不影响
  def main():
      addr = ('127.0.0.1', 8000)
      client = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
      # !接受消息不需要阻塞，所以添加daemon=True设置
      t1 = threading.Thread(target=recv, args=(client, addr), daemon=True)
      t2 = threading.Thread(target=send, args=(client, addr))
      t1.start()
      t2.start()
      t2.join()
      client.close()
  
  
  if __name__ == '__main__':
      name = input('请输入你的名字：')
      main()
  ```

  
