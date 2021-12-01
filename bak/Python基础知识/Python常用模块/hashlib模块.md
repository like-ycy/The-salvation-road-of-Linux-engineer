# hashlib模块

+ 概述

  提供了一些常见的算法

+ MD5

  是最常见的加密算法 速度比较快  生产32位的十六进制的加密

+ 使用

  ```python
  import hashlib
  s1 = b'lucky is a good man'
  m1 = hashlib.md5()
  # print(m1)
  m1.update(s1)
  print(m1.hexdigest())
  ```

