---
layout: post
title: Python中import的用法
date: 2019-04-20
author: qs
header-img: img/post-bg-hacker.jpg
catalog: true
tags: 
- Python
---

Python用了快两年了吧，其中有些东西一直是稀里糊涂地用，import便是我一直没有明白的东西。曾经有过三次解决它的机会，我都因得过且过、一拖再拖而没能化敌为友。今天下午，它又给了我一次机会，我想我还是从了它的心愿吧。

故事是从这篇台湾同胞的博客（[Python 的 Import 陷阱](https://medium.com/pyladies-taiwan/python-%E7%9A%84-import-%E9%99%B7%E9%98%B1-3538e74f57e3)）开始的，然后又跳到了Python社区的PEP 328提案（[PEP 328 -- Imports: Multi-Line and Absolute/Relative](https://www.python.org/dev/peps/pep-0328/#id1)），再结合过去的经验以及一些测试，我想我大概懂了吧。下面是我的总结，希望内容能够言简意赅、易于理解。


import语句有什么用？import语句用来导入其他python文件（称为模块module），使用该模块里定义了类、方法、变量，从而达到代码复用的目的。为了方便说明，我们用实例来说明import的用法，读者朋友可以跟着尝试（尝试时建议使用python3，python2和python3在import的表现有差异，之后会提到）。

首先，先建立一个工作目录Tree，然后在其中建立两个文件m1.py和m2.py，在m1.py写入如下代码：
```python
import os
import m2
m2.printSelf()
```
在m2.py写入如下代码：
```python
def printSelf():
	print('In m2')
```
打开命令行，进入到Tree目录下，敲下`python m1.py`运行，发现没有报错，且打印出`In m2`，说明这样使用import没有问题。由此我们总结出import语句的第一种用法。
- `import module_name`。即import后直接接模块名。在这种情况下，Python会在两个地方寻找这个模块，第一是sys.path（通过运行代码`import sys; print(sys.path)`查看），os这个模块所在的目录就在列表sys.path中，一般安装的Python库的目录都可以在sys.path中找到（前提是要将Python的安装目录添加到电脑的环境变量），所以对于安装好的库，我们直接import即可。第二个地方就是运行文件（这里是m1.py）所在的目录，因为m2.py和运行文件在同一目录下，所以上述写法没有问题。

用上述方法导入原有的sys.path中的库没有问题。但是，最好不要用上述方法导入同目录下的文件！因为这可能会出错。演示这个错误需要用到import语句的第二种写法，所以先来学一学import的第二种写法。在Tree目录下新建一个目录Branch，在Branch中新建文件m3.py，m3.py的内容如下：
```python
def printSelf():
	print('In m3')
```
如何在m1中导入m3.py呢，请看更改后的m1.py：
```python
from Branch import m3
m3.printSelf()
```
总结import语句的第二种用法：
- `from package_name import module_name`。一般把模块组成的集合称为包（package）。与第一种写法类似，Python会在sys.path和运行文件目录这两个地方寻找包，然后导入包中名为module_name的模块。

现在我们来说明为什么不要用import的第一种写法来导入同目录下的文件。在Branch目录下新建m4.py文件，m4.py的内容如下：
```python
def printSelf():
	print('In m4')
```
然后我们在m3.py中直接导入m4，m3.py变为：
```python
import m4
def printSelf():
	print('In m3')
```
这时候运行m1.py就会报错了，说没法导入m4模块。为什么呢？我们来看一下导入流程：m1使用`from Branch import m3`导入m3，然后在m3.py中用`import m4`导入m4。看出问题了吗？m4.py和m1.py不在同一目录，怎么能直接使用`import m4`导入m4呢。（读者可以试试直接在Tree目录下新建另一个m4.py文件，你会发现再运行m1.py就不会出错了，只不过导入的是第二个m4.py了）

面对上面的错误，使用python2运行m1.py就不会报错，因为在python2中，上面提到的import的两种写法都属于相对导入，而在python3中，却属于绝对导入。话说到了这里，就要牵扯到import中最关键的部分了——相对导入和绝对导入。

我们还是谈论python3的import用法。上面提到的两种写法属于绝对导入，即用于导入sys.path中的包和运行文件所在目录下的包。对于sys.path中的包，这种写法毫无问题；导入自己写的文件，如果是非运行入口文件（上面的m1.py是运行入口文件，可以使用绝对导入），则需要相对导入。

比如对于非运行入口文件m3.py，其导入m4.py需要使用相对导入：
```python
from . import m4
def printSelf():
	print('In m3')
```
这时候再运行m1.py就ok了。列举一下相对导入的写法：
- `from . import module_name`。导入和自己同目录下的模块。
- `from .package_name import module_name`。导入和自己同目录的包的模块。
- `from .. import module_name`。导入上级目录的模块。
- `from ..package_name import module_name`。导入位于上级目录下的包的模块。
- 当然还可以有更多的'.'，每多一个点就多往上一层目录。

不知道你有没有留神上面的一句话——“上面的m1.py是运行入口文件，可以使用绝对导入”，这句话是没问题的，也和我平时的做法一致。那么，运行入口文件可不可以使用相对导入呢？比如m1.py内容改成：
```python
from .Branch import m3
m3.printSelf()
```
答案是可以，但不能用`python m1.py`命令，而是需要使用`python -m m1`来运行。为什么？关于前者，PEP 328提案中的一段文字好像给出了原因：
> Relative imports use a module's __name__ attribute to determine that module's position in the package hierarchy. If the module's name does not contain any package information (e.g. it is set to '__main__') then relative imports are resolved as if the module were a top level module, regardless of where the module is actually located on the file system.
我不太懂，但是又有一点明白。我们应该见过下面一段代码：
```python
if __name__ == '__main__':
	main()
```
意思是如果运行了当前文件，则__name__变量会置为__main__，然后会执行main函数，如果当前文件是被其他文件作为模块导入的话，则__name__为模块名，不等于__main__，就不会执行main函数。比如执行`python m1.py`命令后，会报如下错误：
>Traceback (most recent call last):
>  File "m1.py", line 1, in <module>
>    from .Branch import m3
>ModuleNotFoundError: No module named '__main__.Branch'; '__main__' is not a package
据此我猜测执行`python m1.py`命令后，当前目录所代表的包'.'变成了__main__。

那为什么`python -m m1`就可以呢？那位台湾老师给出了解释：
> 执行指令中的-m是为了让Python预先import你要的package或module给你，然后再执行script。
即不把m1.py当作运行入口文件，而是也把它当作被导入的模块，这就和非运行入口文件有一样的表现了。

那反过来，如果m1.py使用绝对导入（`from Branch import m3`），能使用`python -m m1`运行吗？我试了一下，如果当前目录是Tree就可以。如果在其他目录下运行，比如在Tree所在的目录，使用`python -m Tree.m1`运行，就不可以。这可能还是与绝对导入相关。

（之前看到了一个大型项目，其运行入口文件有一大堆的相对导入，我还傻乎乎地用python直接运行它。之后看到他给的样例运行命令是带了-m参数的。现在才恍然大悟。）

理解import的难点差不多就这样了。下面说一说import的其他简单但实用的用法。
- `import moudle_name as alias`。有些module_name比较长，之后写它时较为麻烦，或者module_name会出现名字冲突，可以用as来给它改名，如`import numpy as np`。
- `from module_name import function_name, variable_name, class_name`。上面导入的都是整个模块，有时候我们只想使用模块中的某些函数、某些变量、某些类，用这种写法就可以了。使用逗号可以导入模块中的多个元素。
- 有时候导入的元素很多，可以使用反斜杠来换行，官方推荐使用括号。
```python
from Tkinter import Tk, Frame, Button, Entry, Canvas, Text, \
    LEFT, DISABLED, NORMAL, RIDGE, END	# 反斜杠换行
from Tkinter import (Tk, Frame, Button, Entry, Canvas, Text,
    LEFT, DISABLED, NORMAL, RIDGE, END)	# 括号换行（推荐）
```

说到这感觉import的核心已经说完了。再跟着上面的博客说一说使用import可能碰到的问题吧。

问题1描述：ValueError: attempted relative import beyond top-level package。直面问题的第一步是去了解熟悉它，最好是能复现它，让它躺在两跨之间任我们去践踏蹂躏。仍然是上面四个文件，稍作修改，四个文件如下：
```python
# m1.py
from Branch import m3
m3.printSelf()
# m2.py
def printSelf():
	print('module2')
# m3.py
from .. import m2 # 复现的关键在这 #
print(__name__)
def printSelf():
	print('In m3')
# m4.py
def printSelf():
	print('In m4')
```
运行`python m1.py`，就会出现该问题。问题何在？我猜测，运行m1.py后，m1代表的模块就是顶层模块（参见上面PEP 328的引用），而m3.py中尝试导入的m2模块所在的包（即Tree目录代表的包）比m1的层级更高，所以会报出这样的错误。怎么解决呢？将m1.py的所有导入改为相对导入，然后进入m1.py的上层目录，运行`python -m Tree.m1`即可。

对于使用import出现的其他问题，碰到了再接着更新。





