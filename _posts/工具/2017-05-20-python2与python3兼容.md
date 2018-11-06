---
layout: post
title: "python2与python3兼容：__future__"
date: 2017-05-20
author: silly
desc: "__future__模块提供某些将要引入的特性"
categories: ["工具"]
tags: ["python"]
permalink: "/tool/python-future.html"
--- 

python3写的代码如何也能在pyhon2上跑？请无论如何加上这一句，python3没有啥影响

```
from __future__ import absolute_import, unicode_literals, division, print_function
```

`__future_`_模块提供某些将要引入的特性,python 2.7.5的`__future__`基本上是python3中的特性


# 开始

使用我们的jupyter交互式工具进行探讨，以下皆为python2

有以下内容

```
In [1]: import __future__

In [2]: __future__.
__future__.CO_FUTURE_ABSOLUTE_IMPORT  __future__.all_feature_names
__future__.CO_FUTURE_DIVISION         __future__.division
__future__.CO_FUTURE_PRINT_FUNCTION   __future__.generators
__future__.CO_FUTURE_UNICODE_LITERALS __future__.nested_scopes
__future__.CO_FUTURE_WITH_STATEMENT   __future__.print_function
__future__.CO_GENERATOR_ALLOWED       __future__.unicode_literals
__future__.CO_NESTED                  __future__.with_statement
__future__.absolute_import

In [2]: __future__.
```

可导入的功能有哪些？

```
In [3]: import __future__

In [4]: __future__.all_feature_names
Out[4]:
['nested_scopes',
 'generators',
 'division',
 'absolute_import',
 'with_statement',
 'print_function',
 'unicode_literals']
```

对应功能如下

## division 

division 新的除法特性，本来的除号`/`对于分子分母是整数的情况会取整，但新特性中在此情况下的除法不会取整，取整的使用`//`。如下可见，只有分子分母都是整数时结果不同。

```
In [1]: 3 / 5
Out[1]: 0

In [2]: 3 // 5
Out[2]: 0

In [3]: 3.0 / 5.0
Out[3]: 0.6

In [4]: 3.0 // 5.0
Out[4]: 0.0

In [5]: from __future__ import division

In [6]: 3 / 5
Out[6]: 0.6

In [7]: 3 // 5
Out[7]: 0

In [8]: 3.0 / 5.0
Out[8]: 0.6

In [9]: 3.0 // 5.0
Out[9]: 0.0
```

## print_function

print_function 新的print是一个函数，如果导入此特性，之前的print语句就不能用了。

```
In [1]: print 'test __future__'
test __future__

In [2]: from __future__ import print_function

In [3]: print('test')
test

In [4]: print 'test'
  File "<ipython-input-4-ed4b06bfff9f>", line 1
    print 'test'
               ^
SyntaxError: invalid syntax
```

## unicode_literals

unicode_literals 这个是对字符串使用unicode字符

```
In [1]: print '目录'
鐩綍

In [2]: from __future__ import unicode_literals

In [3]: print '目录'
目录
```

在`python 2.x`中, 对于汉字字符串, 默认还不是采用`unicode`编码的, 除非在字符串前加上前缀`u`. 比如:

```
x='中国' 
x 
'\xd6\xd0\xb9\xfa'   这不是unicode编码
print(x) 
中国 
x=u'中国' 
u'\u4e2d\u56fd' 
print(x) 
中国
```

在`python3`中默认的编码采用了`unicode`, 并取消了前缀`u`. 如果代码要兼容`python2/3`, 就很麻烦了. 下面的两个选择都不方便: 

1. 字符串前面不加`u`. 这种处理方式多数情况下没有问题, 比如print输出, 但因为汉字在py2和py3的编码方式不一样, 如果进行编码转换就麻烦了. 
2. 加`python`版本判断, `if sys.version < '3'`, 字符串不加前缀`u`, 如果是`py2`, 加上前缀`u`. 这样代码显得很拖沓. 
3. 现在有第3种, 比较好的方法是引入`unicode_literals, from __future__ import unicode_literals` , 这样在`py2`下, `'中国'`这样的字符串不用家前缀`u`, 也是unicode编码.

## absolute_import

字面理解好像是仅仅允许绝对引用, 其实不然, 真实意思是禁用隐式相对引用：implicit relative import, 但并不会禁掉显式相对引用：explicit relative import.

举个例子, 目录结构如下, 

```
-cake
|- __init__.py
|- icing.py
|- sponge.py
-drink
|- __init__.py
|- water.py
```

在 `sponge.py` 引用 `icing` , 有多种方法: 

1. `import icing`隐式相对引用, py2已强烈不推荐使用, py3已经不可用了
2. `from . import icing`显式相对引用, python.org 官方虽不推荐, 但这却是事实标准
3. `from cake import icing`绝对引用 , python 官方推荐.

使用 `__future__ absolute_import`之后, 常遇到的一个问题

```
PackageA
|- module1.py
|- module2.py
|- __init__.py 

在module1.py中, 
from __future__ import absolute_impact
from . import module2 #引入同包下的另一个module

if __name__=="__main__":
print("module2 was imported in module1.")
```

运行会报错, `ValueError: Attempted relative import in non-package. `

原因分析: `from . import module2` 这样的写法是显式相对引用, 这种引用方式只能用于package中, 而不能用于主模块中. 
因为主`module`的`name`总是为`main`, 并没有层次结构, 也就无从谈起相对引用了. 
换句话, `if __name__=="__main__":` 和相对引用是不能并存的. 

解决方法: 

1. 在`module1`中使用绝对引用, 这个最简单了, 但相对引用的好处也没了.
2. 使用`python -m`来启动你的`module1.py`, 这个也不推荐. 
3. (推荐，我觉得还是和第一个差不多，只不过测试换在了另外的地方)在`module1`中, 加个`main()`函数, 然后再新建一个`PackageA/entry.py`做为主程序, 在`entry.py`中使用绝对引用来引用`module1`, 并调用`module1.main()`, 这一办法虽不完美, 但我觉得是最好的方法了.

## nested_scopes 

这个是修改嵌套函数或lambda函数中变量的搜索顺序，从`当前函数命名空间->模块命名空间`的顺序更改为了`当前函数命名空间->父函数命名空间->模块命名空间`,python2.7.5中默认使用

## generators 

生成器，对应yield的语法，python2.7.5中默认使用

## with_statement 

使用with关键字，python2.7.5是默认使用

# 运用

首先是可以做个性化的用法，比如你喜欢用`print（）`而不是`print`

更重要的是基本用以下几句就可以让python2和python3有良好的兼容性了

```
from __future__ import print_function
from __future__ import unicode_literals
from __future__ import division
from __future__ import absolute_import
```