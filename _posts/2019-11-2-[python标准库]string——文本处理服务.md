---
layout:     post
title:      [python标准库]string——文本处理服务
date:       2019-11-2
author:     Mr.C
header-img: img/string.jpg
catalog: true
tags:
    - 标准库

---
## [String](https://docs.python.org/zh-cn/3/library/string.html?highlight=string#module-string)
#### string——字符串

~~~python
import string

print(string.ascii_letters)
# 下文所述 ascii_lowercase 和 ascii_uppercase 常量的拼连

print(string.ascii_lowercase)
# 小写字母 'abcdefghijklmnopqrstuvwxyz'

print(string.ascii_uppercase)
# 大写字母 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'

print(string.digits)
# 字符串 '0123456789'

print(string.punctuation)
# 标点符号的 ASCII 字符所组成的字符串: !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~.

print(string.printable)
# 由 digits, ascii_letters, punctuation 和 whitespace 的总和
~~~

#### string——格式字符串语法

~~~python
# 使用特定类型的专属格式化

import datetime


d = datetime.datetime(2019, 11, 2, 12, 15, 58)
print('{:%Y-%m-%d %H:%M:%S}'.format(d))
# 运行结果：2019-11-02 12:15:58


# 表示为百分数
points = 19
total = 22
print('Correct answers: {:.2%}'.format(points / total))
# 运行结果：Correct answers: 86.36%


# 访问参数的项
coord = (3, 5)
print('X: {0[0]};  Y: {0[1]}'.format(coord))
# 运行结果：X: 3;  Y: 5


# 按位置访问参数
print('{0}, {1}, {2}'.format('a', 'b', 'c'))
# 运行结果：'a, b, c'
~~~


**注：原创文章，转载请注明出处**
