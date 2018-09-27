---
layout:     post
title:      Beautiful Soup教程
date:       2018-07-26
author:     qwt
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - JS
---
# Beautiful Soup教程
参考自https://www.crummy.com/software/BeautifulSoup/bs4/doc/index.zh.html#
## 简介
Beautiful Soup是一个可以从HTML或XML中提取数据的Python库，能通过转换器实现文档的查找、修改。
## 快速上手
先构造一个HTML文档，用于之后的使用。
```python
html_doc = """
<html><head><title>The Dormouse's story</title></head><body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>

<p class="story">...</p>
"""
```
可以看到，上面的Html文档的格式不规范，且缺少标签（如&lt;/body>,&lt;/html>）。不过这些都不是大问题。

可以使用Beautiful Soup解析该文档，得到一个BeautifulSoup对象：
```python
from bs4 import BeautifulSoup
soup = BeautifulSoup(html_doc)

# 按照标准格式打印出html_doc
print(soup.prettify())
'''
<html>
 <head>
  <title>
   The Dormouse's story
  </title>
 </head>
 <body>
  <p class="title">
   <b>
    The Dormouse's story
   </b>
  </p>
  <p class="story">
   Once upon a time there were three little sisters; and their names were
   <a class="sister" href="http://example.com/elsie" id="link1">
    Elsie
   </a>
   ,
   <a class="sister" href="http://example.com/lacie" id="link2">
    Lacie
   </a>
   and
   <a class="sister" href="http://example.com/tillie" id="link3">
    Tillie
   </a>
   ;
and they lived at the bottom of a well.
  </p>
  <p class="story">
   ...
  </p>
 </body>
</html>
'''
```
通过BeautifulSoup解析得到的BeautifulSoup对象，其以标签的名字作为属性，故可以通过它来访问Html元素。
```python
# 直接访问标签：访问title
print(soup.title, '\n', soup.title.name, '\n', soup.title.string)
'''
<title>The Dormouse's story</title> 
 title 
 The Dormouse's story
'''
# 如果标签对应多个元素，则返回第一个元素
print(soup.p,'\n', soup.p.string)
'''
<p class="title"><b>The Dormouse's story</b></p> 
 The Dormouse's story
'''
# 找到标签对应的所有元素
all_p = soup.find_all('p')
#获得元素的属性值
print(soup.p['class'])      # ['title']
print(soup.p.get('class'))  # ['title']
# 获取文档中所有的文字内容
content = soup.text # 或者
content = soup.get_text()
```
## 解析器
在使用BeautifulSoup解析文档时，需要传入一个解析器，若不传入，则会使用默认的解析器并抛出一个警告，因为不同的机器的默认解析器不一样，而如果文档不正确的话，使用不同的解析器会得到不同的结果。

|解析器|使用方法|优势|劣势|
|---|---|---|---|
|Python标准库|BeautifulSoup(doc, "html.parser")|Python的内置标准库；<br>执行速度适中；<br>文档容错能力强|某些Python版本文档容错能力差|
|lxml HTML解析器|BeautifulSoup(doc, "lxml")|速度快；文档容错能力强|需要安装C语言库|
|lxml XML解析器|BeautifulSoup(doc, "xml")|速度快；唯一支持XML的的解析器|需要安装C语言库|
|html5lib|BeautifulSoup(doc, "html5lib")|最好的容错性；<br>以浏览器的方式解析文档；<br>生成html5格式的文档|速度慢；不依赖外部扩展|

推荐使用lxml作为解析器。

## 对象的种类
BS把HTML文档转换成一个树形结构，每个节点都是Python对象，共有四种：
- Tag
- NavigableString
- BeautifulSoup
- Comment
### Tag
Tag，即标签，与XML或HTML中的标签含义相同。向前面的soup.title即为Tag类型：`type(soup.title) # bs4.element.Tag`。
Tag的属性（name，string，以及原生的HTML属性如class等）可以被访问和修改，全部的属性可以通过attrs来访问。
### NavigableString
标签中的内容，如：`type(soup.title.string) # bs4.element.NavigableString`。
### BeautifulSoup
表示一个文档的所有内容，大部分时候，可以当作Tag对象。虽然BeautifulSoup不是真正的HTML中的Tag，但却具有name和attrs属性，值为`'[document]'`和`{}`。
### Comment
即注释，Comment是一个特殊类型的NavigableString对象，添加了一些额外的方法：
```python
doc = '<b><!--Hey Buddy, want to buy a used parser?--></b>'
soup1 = BeautifulSoup(doc)
type(soup1.b.string) # bs4.element.Comment
```
## 遍历文档树
### 子节点
我们已经见到了一种获得子节点的方式，即通过子节点的name来获得，比如通过`soup.head`来获得文档的head元素，通过`soup.p`来获得第一个p元素，通过`soup.p.a`来获得第一个p元素的中第一个a元素。若想获得所有具有给定名字的子节点，可以对BeautifulSoup或Tag对象使用find_all方法，如`soup.p.find_all('b')`用于获得第一个p元素的所有b元素。

其他的方式：
- .contents和.children

.contents属性将子节点以列表的方式输出，.children会返回一个子节点的迭代器，可用`for in`语句遍历。

注意到，这两者获得的是直接子节点，不会获得更深的节点。

再者，子节点可以是标签，也可以是字符串。字符串没有子节点。
- .descendants

.descendants采用先序遍历的方式返回包含所有的后代节点（直接子节点和更深的子节点）的生成器。
- .string

如果只有一个NavigableString类型的子节点，那么可以直接使用.string得到子节点；如果只有一个子节点，那么使用.string方法得到的结果与唯一子节点的.string的结果相同；如果有多个子节点，那么使用.string会返回None，因为无法确定返回哪个子节点的.string。
- .strings和.stripped_strings

如果包含多个字符串，可以使用.strings来获取，其返回一个生成器。.stripped_strings与.strings类似，不过其可以去除多余的空白内容（空行或空格）。
### 父节点
- .parent

获取父节点。
```python
print(soup.title.string.parent) # <title>The Dormouse's story</title>
print(soup.title.parent)        #<head><title>The Dormouse's story</title></head>
print(soup.html.parent == soup) # True
print(soup.parent == None)      # True
```
- .parents
通过递归的方式获得包含所有父节点的生成器。
```python
for p in soup.a.parents:
    print(p.name)
# p body html [document]
```
### 兄弟节点
一个新的、简单的例子：
```python
bro_soup = BeautifulSoup('<a><b>text1</b><c>text2</c></b></a>', 'lxml')
print(bro_soup.prettify())
'''
<html>
 <body>
  <a>
   <b>
    text1
   </b>
   <c>
    text2
   </c>
  </a>
 </body>
</html>
'''
```
- .next_sibling和.previous_sibling
使用节点的next_sibling和previous_sibling属性得到后一个兄弟和前一个兄弟节点：
```python
bro_soup.b.next_sibling     # <c>text2</c>
bro_soup.b.previous_sibling # None
```
- .next_siblings和.previous_siblings
类似于.parents之于.parent，.next_siblings的功能是返回节点所有后面兄弟节点的生成器，.previous_siblings的功能是返回所有前面兄弟节点的生成器。
### 回退与前进
- .next_element和.previous_element

为了了解这二者的功能，就要理解解析文档时的流程。对于文档
`<html><body><a><b>text1</b><c>text2</c></a></body></html>`，其解析的流程为：打开&lt;html>标签，打开&lt;body>标签，打开&lt;a>标签，打开&lt;b>标签，添加字符串text1，关闭&lt;b>标签，打开&lt;c>标签，添加字符串text2，关闭&lt;c>标签，关闭&lt;a>标签，关闭&lt;body>标签，关闭&lt;html>标签。

节点的next_element属性就是下一个解析的节点：
```python
bro_soup.b.next_element # 'text1'
bro_soup.b.next_element.next_element # <c>text2</c>
bro_soup.b.next_element.next_element.next_element # 'text2'
```
同理，previous_element属性就是前一个解析的节点。
- .next_elements和.previous_elements

结合前面的内容，这二者的意思显而易见。
## 搜索文档树
主要介绍`find_all`函数，它作为节点的方法，可以在节点的内部找到所有满足某条件的元素。为了方便地利用它们找到我们想要的元素，有必要了解一下**过滤器**，来设置搜索的条件。
### 过滤器
- 字符串

这个我们之前就看到过，例如：`soup.find_all('p')`会搜索文档中所有的p标签，并以列表的方式返回它们。
- 正则表达式

如果传入正则表达式为参数，BS会通过正则表达式的match()来匹配内容，如在下面的例子中，会找出所有以b为开头的标签，所以，像&lt;body>和&ltb>都会被找到：
```python
import re # 正则表达式模块
for tag in soup.find_all(re.compile('^b')):
    print(tag.name)
# body b
```
再如，找出所有包含t的标签：
```python
for tag in soup.find_all(re.compile('t')):
    print(tag.name)
# html title
```
- 列表
传入的参数是列表的话，BS会将与列表中任一元素匹配的节点返回：
```python
wanted = soup.find_all(['a', re.compile('^b')])
# wanted列表包含5个元素，第一个为body，第二个为b，还有三个a元素
```
- True

True可以匹配任何值，但是不会返回字符串节点。
- 方法，即函数

可定义一个方法，该方法接收一个参数（类型为除字符串外的节点），如果方法返回True，则匹配，否则，不匹配。比如，我们想要找到所有包含class属性却不包含id属性的节点：
```python
def has_class_not_id(tag):
    return tag.has_attr('class') and not tag.has_attr('id')
soup.find_all(has_class_not_id)
```
### 细究`find_all()`
find_all的定义为：
```python
find_all(name, attrs, recursive, text, **kwargs)
```
- name

查找所有名字为name的Tag，name可以为上述的任一过滤器
- attrs

按照css类名搜索Tag，attrs指示类名。在使用的时候，可以同`class_`来作为形参：
```python
soup.find_all('a', 'sister')          # 等价于
soup,find_all('a', class_ = 'sister') # 等价于
soup.find_all('a', attrs = 'sister')
```
当然，attrs的值除了字符串外，也可以是任一过滤器。

由于Tag的class是多值属性，即一个Tag可以对应多个class，故可以分别搜索Tag中的每个类名：
```python
css_soup = BeautifulSoup('<p class="body strikeout"></p>', 'lxml')
css_soup.find_all("p", class_="strikeout")
# 返回值: [<p class="body strikeout"></p>]
css_soup.find_all("p", class_="body strikeout")
# 返回值: [<p class="body strikeout"></p>]
css_soup.find_all("p", class_="strikeout body")
# 返回值：[] 这是因为BS会考虑类名的顺序
```
- recursive

默认的情况下，recursive的值为True，即会搜索当前节点的所有内部节点。如设置为False，则只会在直接子节点进行搜索。
- text

其会搜索文档中的字符串内容，其值可为任一过滤器：
```python
soup.find_all(text=["Tillie", "Elsie"])
# 返回值： ['Elsie', 'Tillie']
soup.find_all(text=re.compile("Dormouse"))
# 返回值：["The Dormouse's story", "The Dormouse's story"]
```
- **kwargs

如果一个指定名字的参数不在name、attrs/class_、recursive、text即内置的参数之中，则搜索时会把该参数当作指定名字的属性来搜索。例如：
```python
soup.find_all(id="link2")
# 返回值：[<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```
其返回了id为link2的Tag元素。属性的值可以为任一过滤器。例如`soup.find_all(id=True)`将返回所有包含具有id属性的Tag，不论其id值是什么。

有些Tag的属性，比如Html5中的形如`data-*`属性，由于不符合变量命名规则，故不能直接作为参数，这时，可以借助attrs参数：
```python
data_soup = BeautifulSoup('<div data-foo="value">foo!</div>', 'lxml')
data_soup.find_all(attrs={"data-foo": "value"})
# 返回值：[<div data-foo="value">foo!</div>]
```
- limit
额外的limit的参数会限制返回列表的的元素个数，默认不受限。

关于find_all的额外的说明：对于Tag和BeautifulSoup对象，可以作为函数被调用，参数的用法与find_all一样，且功能与find_all也一样：
```python
soup.find_all('a') # 等价于
soup('a')
soup.p.find('b')   # 等价于
soup.p('b')
```
方便了不少。
### find方法
find方法等价于limit设为1的find_all，即找到第一个满足条件的元素。其返回值不是列表，而是直接返回结果。若没找到，则返回None。
### 更多的搜索方式
接下来，有10个与搜索相关的方法，其又分为5对，每对中，一个和find_all用法类似，一个与find用法类似。
1. find_parents()与find_parent()
用于找当前节点的父辈节点
2. find_next_siblings()与find_next_sibling()
用于找当前节点的后面的兄弟节点（顺着next_siblings来找）
3. find_previous_siblings()与find_previous_sibling()
用于找当前节点的前面的兄弟节点（顺着previous_siblings来找）
4. find_all_next()与find_next()
用于找当前节点后面的元素（顺着next_elements来找）
5. find_all_previous()与find_previous()
用于找当前节点前面的元素（顺着previous_elements来找）
### CSS选择器
BeautifulSoup和Tag具有select方法，向其中传入字符串参数，即可使用CSS选择器的语法找到满足条件的节点。返回值为列表，列表的每个元素均为满足条件的元素。
- 根据元素名来查找
```python
soup.select('p')
```
会返回包含所有p元素的列表。
- 根据祖先节点来查找
```python
soup.select('p b')
```
返回包含所有以p元素为祖先的b元素。
- 通过类名来查找
```python
soup.select('.sister')
```
返回类名包含sister的元素。
- 根据id来查找
```python
soup.select('#link1')
```
选择id值为link1的元素。
- 根据是否包含某属性来查找
```python
soup.select('[href]')
```
选择包含href属性的元素。还可以结合元素选择器：
```python
soup.select('a[href]')
```
其含义为选择包含href属性的a元素。
- 根据属性值来查找
```python
soup.select('a[href="http://example.com/elsie"]')
```
选择href属性值为给定值的a元素。

BS支持大部分的CSS选择器，熟悉了CSS选择器就可以直接上手select方法了。
