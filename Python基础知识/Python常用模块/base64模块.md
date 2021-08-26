# base64模块

+ 概述

  是网络上最常见的用于传输8Bit字节码的编码方式之一，*Base64*就是一种基于64个可打印字符来表示二进制数据的方法。

+ 作用

  适用于小段内容的编码  比如 cookie、django数据库中存储的session值

+ 使用

  import base64

  ```python
  import base64
  s1 = b'lucky is a good man'
  encod_s1 = base64.b64encode(s1)
  # 解码
  print(base64.b64decode(encod_s1))
  # base64 只能进行编码  而不是加密
  # 如果需要加密 的 可以使用md5, hash加密
  ```

