## BeautifulSoup

一个灵活又方便的网页解析库，处理高效，支持多种解析器，利用它就不用编写正则表达式也能方便的实现网页信息的抓取

官方文档：[官方文档](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id13)

### 快速使用

```python
from bs4 import BeautifulSoup

html = '''
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title"><b>The Dormouse's story</b></p>

<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
'''
soup = BeautifulSoup(html,'lxml')
print(soup.prettify())
print(soup.title)
print(soup.title.name)
print(soup.title.string)
print(soup.title.parent.name)
print(soup.p)
print(soup.p["class"])
print(soup.a)
print(soup.find_all('a'))
print(soup.find(id='link3'))
```

### 如何使用

将一段文档传入BeautifulSoup 的构造方法,就能得到一个文档的对象, 可以传入一段字符串或一个文件句柄.

```python
from bs4 import BeautifulSoup

soup = BeautifulSoup(open("index.html"))

soup = BeautifulSoup("<html>data</html>")
```

首先,文档被转换成Unicode,并且HTML的实例都被转换成Unicode编码

```python
BeautifulSoup("Sacr&eacute; bleu!")
<html><head></head><body>Sacré bleu!</body></html>
```

然后,Beautiful Soup选择最合适的解析器来解析这段文档,如果手动指定解析器那么Beautiful Soup会选择指定的解析器来解析文档.

### 对象的种类

Beautiful Soup将复杂HTML文档转换成一个复杂的树形结构,每个节点都是Python对象,所有对象可以归纳为4种: `Tag` , `NavigableString` , `BeautifulSoup` , `Comment` 

#### Tag

`Tag` 对象与XML或HTML原生文档中的tag相同:

```python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b
type(tag)
# <class 'bs4.element.Tag'>
```

tag中最重要的属性: name和attributes

##### Name

每个tag都有自己的名字,通过 `.name` 来获取:

```python
tag.name
# u'b'
```

如果改变了tag的name,那将影响所有通过当前Beautiful Soup对象生成的HTML文档:

```python
tag.name = "blockquote"
tag
# <blockquote class="boldest">Extremely bold</blockquote>
```

#### Attributes

一个tag可能有很多个属性. tag `<b class="boldest">` 有一个 “class” 的属性,值为 “boldest” . tag的属性的操作方法与字典相同:

```python
tag['class']
# u'boldest'
```

也可以直接”点”取属性, 比如: `.attrs` :

```python
tag.attrs
# {u'class': u'boldest'}
```

tag的属性可以被添加,删除或修改. 再说一次, tag的属性操作方法与字典一样

```python
tag['class'] = 'verybold'
tag['id'] = 1
tag
# <blockquote class="verybold" id="1">Extremely bold</blockquote>

del tag['class']
del tag['id']
tag
# <blockquote>Extremely bold</blockquote>

tag['class']
# KeyError: 'class'
print(tag.get('class'))
# None
```

##### 多值属性

HTML 4定义了一系列可以包含多个值的属性.在HTML5中移除了一些,却增加更多.最常见的多值的属性是 class (一个tag可以有多个CSS的class). 还有一些属性 `rel` , `rev` , `accept-charset` , `headers` , `accesskey` . 在Beautiful Soup中多值属性的返回类型是list:

```python
css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.p['class']
# ["body", "strikeout"]

css_soup = BeautifulSoup('<p class="body"></p>')
css_soup.p['class']
# ["body"]
```

如果某个属性看起来好像有多个值,但在任何版本的HTML定义中都没有被定义为多值属性,那么Beautiful Soup会将这个属性作为字符串返回

```python
id_soup = BeautifulSoup('<p id="my id"></p>')
id_soup.p['id']
# 'my id

```

将tag转换成字符串时,多值属性会合并为一个值

```python
rel_soup = BeautifulSoup('<p>Back to the <a rel="index">homepage</a></p>')
rel_soup.a['rel']
# ['index']
rel_soup.a['rel'] = ['index', 'contents']
print(rel_soup.p)
# <p>Back to the <a rel="index contents">homepage</a></p>

```

如果转换的文档是XML格式,那么tag中不包含多值属性

```python
xml_soup = BeautifulSoup('<p class="body strikeout"></p>', 'xml')
xml_soup.p['class']
# u'body strikeout'

```

如果转换的文档是XML格式,那么tag中不包含多值属性

```python
xml_soup = BeautifulSoup('<p class="body strikeout"></p>', 'xml')
xml_soup.p['class']
# u'body strikeout'

```

##### 可以遍历的字符串

字符串常被包含在tag内.Beautiful Soup用 `NavigableString` 类来包装tag中的字符串:

```python
tag.string
# u'Extremely bold'
type(tag.string)
# <class 'bs4.element.NavigableString'>

```

一个 `NavigableString` 字符串与Python中的Unicode字符串相同,并且还支持包含在 [遍历文档树](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id18) 和 [搜索文档树](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id27) 中的一些特性. 通过 `unicode()` 方法可以直接将 `NavigableString` 对象转换成Unicode字符串:

```python
unicode_string = unicode(tag.string)
unicode_string
# u'Extremely bold'
type(unicode_string)
# <type 'unicode'>
```

tag中包含的字符串不能编辑,但是可以被替换成其它的字符串,用 [replace_with()](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#replace-with) 方法:

```python
tag.string.replace_with("No longer bold")
tag
# <blockquote>No longer bold</blockquote>
```

`NavigableString` 对象支持 [遍历文档树](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id18) 和 [搜索文档树](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id27) 中定义的大部分属性, 并非全部.尤其是,一个字符串不能包含其它内容(tag能够包含字符串或是其它tag),字符串不支持 `.contents` 或 `.string` 属性或 `find()` 方法.

如果想在Beautiful Soup之外使用 `NavigableString` 对象,需要调用 `unicode()` 方法,将该对象转换成普通的Unicode字符串,否则就算Beautiful Soup已方法已经执行结束,该对象的输出也会带有对象的引用地址.这样会浪费内存.

### 遍历文档树

#### 子节点

一个Tag可能包含多个字符串或其它的Tag,这些都是这个Tag的子节点.Beautiful Soup提供了许多操作和遍历子节点的属性.

注意: Beautiful Soup中字符串节点不支持这些属性,因为字符串没有子节点

#### tag的名字

操作文档树最简单的方法就是告诉它你想获取的tag的name.如果想获取 <head> 标签,只要用 `soup.head` :

```python
soup.head
# <head><title>The Dormouse's story</title></head>

soup.title
# <title>The Dormouse's story</title>
```

这是个获取tag的小窍门,可以在文档树的tag中多次调用这个方法.下面的代码可以获取<body>标签中的第一个<b>标签:

```python
soup.body.b
# <b>The Dormouse's story</b>
```

通过点取属性的方式只能获得当前名字的第一个tag:

```python
soup.a
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
```

如果想要得到所有的<a>标签,或是通过名字得到比一个tag更多的内容的时候,就需要用到 Searching the tree 中描述的方法,比如: find_all()

```python
soup.find_all('a')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```

#### contents 和 .children

tag的 `.contents` 属性可以将tag的子节点以列表的方式输出:

```python
head_tag = soup.head
head_tag
# <head><title>The Dormouse's story</title></head>

head_tag.contents
[<title>The Dormouse's story</title>]

title_tag = head_tag.contents[0]
title_tag
# <title>The Dormouse's story</title>
title_tag.contents
# [u'The Dormouse's story']

```

`BeautifulSoup` 对象本身一定会包含子节点,也就是说<html>标签也是 `BeautifulSoup` 对象的子节点:

```python
len(soup.contents)
# 1
soup.contents[0].name
# u'html'
```

字符串没有 `.contents` 属性,因为字符串没有子节点:

```python
text = title_tag.contents[0]
text.contents
# AttributeError: 'NavigableString' object has no attribute 'contents'
```

通过tag的 `.children` 生成器,可以对tag的子节点进行循环:

```python
for child in title_tag.children:
    print(child)
    # The Dormouse's story
```

#### .descendants

`.contents` 和 `.children` 属性仅包含tag的直接子节点.例如,<head>标签只有一个直接子节点<title>

```python
head_tag.contents
# [<title>The Dormouse's story</title>]
```

但是<title>标签也包含一个子节点:字符串 “The Dormouse’s story”,这种情况下字符串 “The Dormouse’s story”也属于<head>标签的子孙节点. `.descendants` 属性可以对所有tag的子孙节点进行递归循环 [[5\]](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id92) :

```python
for child in head_tag.descendants:
    print(child)
    # <title>The Dormouse's story</title>
    # The Dormouse's story
```

上面的例子中, <head>标签只有一个子节点,但是有2个子孙节点:<head>节点和<head>的子节点, `BeautifulSoup` 有一个直接子节点(<html>节点),却有很多子孙节点:

```python
len(list(soup.children))
# 1
len(list(soup.descendants))
# 25
```

#### .string

如果tag只有一个 `NavigableString` 类型子节点,那么这个tag可以使用 `.string` 得到子节点:

```python
title_tag.string
# u'The Dormouse's story'
```

如果一个tag仅有一个子节点,那么这个tag也可以使用 `.string` 方法,输出结果与当前唯一子节点的 `.string` 结果相同:

```python
head_tag.contents
# [<title>The Dormouse's story</title>]

head_tag.string
# u'The Dormouse's story'
```

如果tag包含了多个子节点,tag就无法确定 `.string` 方法应该调用哪个子节点的内容, `.string` 的输出结果是 `None` :

```python
print(soup.html.string)
# None
```

#### .strings 和 stripped_strings

如果tag中包含多个字符串 [[2\]](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id89) ,可以使用 `.strings` 来循环获取:

```python
for string in soup.strings:
    print(repr(string))
    # u"The Dormouse's story"
    # u'\n\n'
    # u"The Dormouse's story"
    # u'\n\n'
    # u'Once upon a time there were three little sisters; and their names were\n'
    # u'Elsie'
    # u',\n'
    # u'Lacie'
    # u' and\n'
    # u'Tillie'
    # u';\nand they lived at the bottom of a well.'
    # u'\n\n'
    # u'...'
    # u'\n'
```

输出的字符串中可能包含了很多空格或空行,使用 `.stripped_strings` 可以去除多余空白内容:

```python
for string in soup.stripped_strings:
    print(repr(string))
    # u"The Dormouse's story"
    # u"The Dormouse's story"
    # u'Once upon a time there were three little sisters; and their names were'
    # u'Elsie'
    # u','
    # u'Lacie'
    # u'and'
    # u'Tillie'
    # u';\nand they lived at the bottom of a well.'
    # u'...'
```

全部是空格的行会被忽略掉,段首和段末的空白会被删除

### 父节点

继续分析文档树,每个tag或字符串都有父节点:被包含在某个tag中

#### .parent

通过 `.parent` 属性来获取某个元素的父节点.在例子“爱丽丝”的文档中,<head>标签是<title>标签的父节点:

```python
title_tag = soup.title
title_tag
# <title>The Dormouse's story</title>
title_tag.parent
# <head><title>The Dormouse's story</title></head>
```

文档title的字符串也有父节点:<title>标签

```python
title_tag.string.parent
# <title>The Dormouse's story</title>
```

文档的顶层节点比如<html>的父节点是 `BeautifulSoup` 对象:

```python
html_tag = soup.html
type(html_tag.parent)
# <class 'bs4.BeautifulSoup'>
```

`BeautifulSoup` 对象的 `.parent` 是None:

```python
print(soup.parent)
# None
```

#### .parents

通过元素的 `.parents` 属性可以递归得到元素的所有父辈节点,下面的例子使用了 `.parents` 方法遍历了<a>标签到根节点的所有节点.

```python
link = soup.a
link
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
for parent in link.parents:
    if parent is None:
        print(parent)
    else:
        print(parent.name)
# p
# body
# html
# [document]
# None
```

### 兄弟节点

看一段简单的例子:

```python
sibling_soup = BeautifulSoup("<a><b>text1</b><c>text2</c></b></a>")
print(sibling_soup.prettify())
# <html>
#  <body>
#   <a>
#    <b>
#     text1
#    </b>
#    <c>
#     text2
#    </c>
#   </a>
#  </body>
# </html>
```

因为<b>标签和<c>标签是同一层:他们是同一个元素的子节点,所以<b>和<c>可以被称为兄弟节点.一段文档以标准格式输出时,兄弟节点有相同的缩进级别.在代码中也可以使用这种关系.

#### .next_sibling 和 .previous_sibling

在文档树中,使用 `.next_sibling` 和 `.previous_sibling` 属性来查询兄弟节点:

```python
sibling_soup.b.next_sibling
# <c>text2</c>

sibling_soup.c.previous_sibling
# <b>text1</b>
```

<b>标签有 `.next_sibling` 属性,但是没有 `.previous_sibling` 属性,因为<b>标签在同级节点中是第一个.同理,<c>标签有 `.previous_sibling` 属性,却没有 `.next_sibling` 属性:

```python
print(sibling_soup.b.previous_sibling)
# None
print(sibling_soup.c.next_sibling)
# None

```

例子中的字符串“text1”和“text2”不是兄弟节点,因为它们的父节点不同:

```python
sibling_soup.b.string
# u'text1'

print(sibling_soup.b.string.next_sibling)
# None

```

实际文档中的tag的 `.next_sibling` 和 `.previous_sibling` 属性通常是字符串或空白. 看看“爱丽丝”文档:

```python
<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a>
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>

```

如果以为第一个<a>标签的 `.next_sibling` 结果是第二个<a>标签,那就错了,真实结果是第一个<a>标签和第二个<a>标签之间的顿号和换行符:

```python
link = soup.a
link
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

link.next_sibling
# u',\n'

```

第二个<a>标签是顿号的 `.next_sibling` 属性:

```python
link.next_sibling.next_sibling
# <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>

```

#### .next_siblings 和 .previous_siblings

通过 `.next_siblings` 和 `.previous_siblings` 属性可以对当前节点的兄弟节点迭代输出:

```python
for sibling in soup.a.next_siblings:
    print(repr(sibling))
    # u',\n'
    # <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
    # u' and\n'
    # <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>
    # u'; and they lived at the bottom of a well.'
    # None

for sibling in soup.find(id="link3").previous_siblings:
    print(repr(sibling))
    # ' and\n'
    # <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>
    # u',\n'
    # <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>
    # u'Once upon a time there were three little sisters; and their names were\n'
    # None
```

### 回退和前进

看一下“爱丽丝” 文档:

```html
<html><head><title>The Dormouse's story</title></head>
<p class="title"><b>The Dormouse's story</b></p>
```

HTML解析器把这段字符串转换成一连串的事件: “打开<html>标签”,”打开一个<head>标签”,”打开一个<title>标签”,”添加一段字符串”,”关闭<title>标签”,”打开<p>标签”,等等.Beautiful Soup提供了重现解析器初始化过程的方法.

#### .next_element 和 .previous_element

`.next_element` 属性指向解析过程中下一个被解析的对象(字符串或tag),结果可能与 `.next_sibling` 相同,但通常是不一样的.

这是“爱丽丝”文档中最后一个<a>标签,它的 `.next_sibling` 结果是一个字符串,因为当前的解析过程 [[2\]](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id89) 因为当前的解析过程因为遇到了<a>标签而中断了:

```python
last_a_tag = soup.find("a", id="link3")
last_a_tag
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>

last_a_tag.next_sibling
# '; and they lived at the bottom of a well.'

```

但这个<a>标签的 `.next_element` 属性结果是在<a>标签被解析之后的解析内容,不是<a>标签后的句子部分,应该是字符串”Tillie”:

```python
last_a_tag.next_element
# u'Tillie'

```

这是因为在原始文档中,字符串“Tillie” 在分号前出现,解析器先进入<a>标签,然后是字符串“Tillie”,然后关闭</a>标签,然后是分号和剩余部分.分号与<a>标签在同一层级,但是字符串“Tillie”会被先解析.

`.previous_element` 属性刚好与 `.next_element` 相反,它指向当前被解析的对象的前一个解析对象:

```python
last_a_tag.previous_element
# u' and\n'
last_a_tag.previous_element.next_element
# <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>

```

#### .next_elements 和 .previous_elements

通过 `.next_elements` 和 `.previous_elements` 的迭代器就可以向前或向后访问文档的解析内容,就好像文档正在被解析一样:

```python
for element in last_a_tag.next_elements:
    print(repr(element))
# u'Tillie'
# u';\nand they lived at the bottom of a well.'
# u'\n\n'
# <p class="story">...</p>
# u'...'
# u'\n'
# None

```

### 搜索文档树

Beautiful Soup定义了很多搜索方法,这里着重介绍2个: `find()` 和 `find_all()` .其它方法的参数和用法类似

#### 过滤器

介绍 `find_all()` 方法前,先介绍一下过滤器的类型 [[3\]](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id90) ,这些过滤器贯穿整个搜索的API.过滤器可以被用在tag的name中,节点的属性中,字符串中或他们的混合中.

#### 字符串

最简单的过滤器是字符串.在搜索方法中传入一个字符串参数,Beautiful Soup会查找与字符串完整匹配的内容,下面的例子用于查找文档中所有的<b>标签:

```python
soup.find_all('b')
# [<b>The Dormouse's story</b>]

```

如果传入字节码参数,Beautiful Soup会当作UTF-8编码,可以传入一段Unicode 编码来避免Beautiful Soup解析编码出错

#### 正则表达式

如果传入正则表达式作为参数,Beautiful Soup会通过正则表达式的 `match()` 来匹配内容.下面例子中找出所有以b开头的标签,这表示<body>和<b>标签都应该被找到:

```python
import re
for tag in soup.find_all(re.compile("^b")):
    print(tag.name)
# body
# b

```

下面代码找出所有名字中包含”t”的标签:

```python
for tag in soup.find_all(re.compile("t")):
    print(tag.name)
# html
# title

```

#### 列表

如果传入列表参数,Beautiful Soup会将与列表中任一元素匹配的内容返回.下面代码找到文档中所有<a>标签和<b>标签:

```python
soup.find_all(["a", "b"])
# [<b>The Dormouse's story</b>,
#  <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```

#### True

`True` 可以匹配任何值,下面代码查找到所有的tag,但是不会返回字符串节点

```python
for tag in soup.find_all(True):
    print(tag.name)
# html
# head
# title
# body
# p
# b
# p
# a
# a
# a
# p

```

#### 方法

如果没有合适过滤器,那么还可以定义一个方法,方法只接受一个元素参数 [[4\]](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id91) ,如果这个方法返回 `True` 表示当前元素匹配并且被找到,如果不是则反回 `False`

下面方法校验了当前元素,如果包含 `class` 属性却不包含 `id` 属性,那么将返回 `True`:

```python
def has_class_but_no_id(tag):
    return tag.has_attr('class') and not tag.has_attr('id')

```

将这个方法作为参数传入 `find_all()` 方法,将得到所有<p>标签:

```python
soup.find_all(has_class_but_no_id)
# [<p class="title"><b>The Dormouse's story</b></p>,
#  <p class="story">Once upon a time there were...</p>,
#  <p class="story">...</p>]

```

返回结果中只有<p>标签没有<a>标签,因为<a>标签还定义了”id”,没有返回<html>和<head>,因为<html>和<head>中没有定义”class”属性.

通过一个方法来过滤一类标签属性的时候, 这个方法的参数是要被过滤的属性的值, 而不是这个标签. 下面的例子是找出 `href` 属性不符合指定正则的 `a` 标签.

```python
def not_lacie(href):
        return href and not re.compile("lacie").search(href)
soup.find_all(href=not_lacie)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```

标签过滤方法可以使用复杂方法. 下面的例子可以过滤出前后都有文字的标签.

```python
from bs4 import NavigableString
def surrounded_by_strings(tag):
    return (isinstance(tag.next_element, NavigableString)
            and isinstance(tag.previous_element, NavigableString))

for tag in soup.find_all(surrounded_by_strings):
    print tag.name
# p
# a
# a
# a
# p

```

#### find_all()

find_all( [name](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id35) , [attrs](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#css) , [recursive](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#recursive) , [string](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id36) , [**kwargs](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#keyword) )

`find_all()` 方法搜索当前tag的所有tag子节点,并判断是否符合过滤器的条件.这里有几个例子:

```python
soup.find_all("title")
# [<title>The Dormouse's story</title>]

soup.find_all("p", "title")
# [<p class="title"><b>The Dormouse's story</b></p>]

soup.find_all("a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.find_all(id="link2")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

import re
soup.find(string=re.compile("sisters"))
# u'Once upon a time there were three little sisters; and their names were\n'

```

有几个方法很相似,还有几个方法是新的,参数中的 `string` 和 `id` 是什么含义? 为什么 `find_all("p", "title")` 返回的是CSS Class为”title”的<p>标签? 我们来仔细看一下 `find_all()` 的参数

#### name 参数

`name` 参数可以查找所有名字为 `name` 的tag,字符串对象会被自动忽略掉.

简单的用法如下:

```python
soup.find_all("title")
# [<title>The Dormouse's story</title>]

```

重申: 搜索 `name` 参数的值可以使任一类型的 [过滤器](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id28) ,字符窜,正则表达式,列表,方法或是 `True` .

#### keyword 参数

如果一个指定名字的参数不是搜索内置的参数名,搜索时会把该参数当作指定名字tag的属性来搜索,如果包含一个名字为 `id` 的参数,Beautiful Soup会搜索每个tag的”id”属性.

```python
soup.find_all(id='link2')
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

如果传入 `href` 参数,Beautiful Soup会搜索每个tag的”href”属性:

```python
soup.find_all(href=re.compile("elsie"))
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```

搜索指定名字的属性时可以使用的参数值包括 [字符串](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id30) , [正则表达式](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id31) , [列表](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id32), [True](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#true) .

下面的例子在文档树中查找所有包含 `id` 属性的tag,无论 `id` 的值是什么:

```python
soup.find_all(id=True)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

使用多个指定名字的参数可以同时过滤tag的多个属性:

```python
soup.find_all(href=re.compile("elsie"), id='link1')
# [<a class="sister" href="http://example.com/elsie" id="link1">three</a>]
```

有些tag属性在搜索不能使用,比如HTML5中的 data-* 属性:

```python
data_soup = BeautifulSoup('<div data-foo="value">foo!</div>')
data_soup.find_all(data-foo="value")
# SyntaxError: keyword can't be an expression
```

但是可以通过 `find_all()` 方法的 `attrs` 参数定义一个字典参数来搜索包含特殊属性的tag:

```python
data_soup.find_all(attrs={"data-foo": "value"})
# [<div data-foo="value">foo!</div>]

```

#### 按CSS搜索

按照CSS类名搜索tag的功能非常实用,但标识CSS类名的关键字 `class` 在Python中是保留字,使用 `class` 做参数会导致语法错误.从Beautiful Soup的4.1.1版本开始,可以通过 `class_` 参数搜索有指定CSS类名的tag:

```python
soup.find_all("a", class_="sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```

`class_` 参数同样接受不同类型的 `过滤器` ,字符串,正则表达式,方法或 `True` :

```python
soup.find_all(class_=re.compile("itl"))
# [<p class="title"><b>The Dormouse's story</b></p>]

def has_six_characters(css_class):
    return css_class is not None and len(css_class) == 6

soup.find_all(class_=has_six_characters)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```

tag的 `class` 属性是 [多值属性](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id15) .按照CSS类名搜索tag时,可以分别搜索tag中的每个CSS类名:

```python
css_soup = BeautifulSoup('<p class="body strikeout"></p>')
css_soup.find_all("p", class_="strikeout")
# [<p class="body strikeout"></p>]

css_soup.find_all("p", class_="body")
# [<p class="body strikeout"></p>]

```

搜索 `class` 属性时也可以通过CSS值完全匹配:

```python
css_soup.find_all("p", class_="body strikeout")
# [<p class="body strikeout"></p>]

```

完全匹配 `class` 的值时,如果CSS类名的顺序与实际不符,将搜索不到结果:

```python
soup.find_all("a", attrs={"class": "sister"})
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```

#### `string` 参数

通过 `string` 参数可以搜搜文档中的字符串内容.与 `name` 参数的可选值一样, `string` 参数接受 [字符串](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id30) , [正则表达式](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id31) , [列表](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id32), [True](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#true) . 看例子:

```python
soup.find_all(string="Elsie")
# [u'Elsie']

soup.find_all(string=["Tillie", "Elsie", "Lacie"])
# [u'Elsie', u'Lacie', u'Tillie']

soup.find_all(string=re.compile("Dormouse"))
[u"The Dormouse's story", u"The Dormouse's story"]

def is_the_only_string_within_a_tag(s):
    ""Return True if this string is the only child of its parent tag.""
    return (s == s.parent.string)

soup.find_all(string=is_the_only_string_within_a_tag)
# [u"The Dormouse's story", u"The Dormouse's story", u'Elsie', u'Lacie', u'Tillie', u'...']

```

虽然 `string` 参数用于搜索字符串,还可以与其它参数混合使用来过滤tag.Beautiful Soup会找到 `.string` 方法与 `string` 参数值相符的tag.下面代码用来搜索内容里面包含“Elsie”的<a>标签:

```python
soup.find_all("a", string="Elsie")
# [<a href="http://example.com/elsie" class="sister" id="link1">Elsie</a>]

```

#### `limit` 参数

`find_all()` 方法返回全部的搜索结构,如果文档树很大那么搜索会很慢.如果我们不需要全部结果,可以使用 `limit` 参数限制返回结果的数量.效果与SQL中的limit关键字类似,当搜索到的结果数量达到 `limit` 的限制时,就停止搜索返回结果.

文档树中有3个tag符合搜索条件,但结果只返回了2个,因为我们限制了返回数量:

```python
soup.find_all("a", limit=2)
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

```

#### `recursive` 参数

调用tag的 `find_all()` 方法时,Beautiful Soup会检索当前tag的所有子孙节点,如果只想搜索tag的直接子节点,可以使用参数 `recursive=False` .

一段简单的文档:

```html
<html>
 <head>
  <title>
   The Dormouse's story
  </title>
 </head>
...

```

是否使用 `recursive` 参数的搜索结果:

```python
soup.html.find_all("title")
# [<title>The Dormouse's story</title>]

soup.html.find_all("title", recursive=False)
# []

```

这是文档片段

```html
<html>
        <head>
        <title>
        The Dormouse's story
    </title>
        </head>
        ...

```

<title>标签在 <html> 标签下, 但并不是直接子节点, <head> 标签才是直接子节点. 在允许查询所有后代节点时 Beautiful Soup 能够查找到 <title> 标签. 但是使用了 `recursive=False` 参数之后,只能查找直接子节点,这样就查不到 <title> 标签了.

Beautiful Soup 提供了多种DOM树搜索方法. 这些方法都使用了类似的参数定义. 比如这些方法: `find_all()`: `name`, `attrs`, `text`, `limit`. 但是只有 `find_all()` 和 `find()` 支持 `recursive` 参数.

#### 像调用 `find_all()` 一样调用tag

`find_all()` 几乎是Beautiful Soup中最常用的搜索方法,所以我们定义了它的简写方法. `BeautifulSoup` 对象和 `tag` 对象可以被当作一个方法来使用,这个方法的执行结果与调用这个对象的 `find_all()` 方法相同,下面两行代码是等价的:

```python
soup.find_all("a")
soup("a")

```

这两行代码也是等价的:

```python
soup.title.find_all(string=True)
soup.title(string=True)

```

#### find()

find( [name](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id35) , [attrs](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#css) , [recursive](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#recursive) , [string](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id36) , [**kwargs](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#keyword) )

`find_all()` 方法将返回文档中符合条件的所有tag,尽管有时候我们只想得到一个结果.比如文档中只有一个<body>标签,那么使用 `find_all()` 方法来查找<body>标签就不太合适, 使用 `find_all` 方法并设置 `limit=1` 参数不如直接使用 `find()` 方法.下面两行代码是等价的:

```python
soup.find_all('title', limit=1)
# [<title>The Dormouse's story</title>]

soup.find('title')
# <title>The Dormouse's story</title>

```

唯一的区别是 `find_all()` 方法的返回结果是值包含一个元素的列表,而 `find()` 方法直接返回结果.

`find_all()` 方法没有找到目标是返回空列表, `find()` 方法找不到目标时,返回 `None` .

```python
print(soup.find("nosuchtag"))
# None

```

`soup.head.title` 是 [tag的名字](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id20) 方法的简写.这个简写的原理就是多次调用当前tag的 `find()` 方法:

```python
soup.head.title
# <title>The Dormouse's story</title>

soup.find("head").find("title")
# <title>The Dormouse's story</title>

```

#### CSS选择器

Beautiful Soup支持大部分的CSS选择器 http://www.w3.org/TR/CSS2/selector.html [[6\]](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id93) , 在 `Tag` 或 `BeautifulSoup` 对象的 `.select()` 方法中传入字符串参数, 即可使用CSS选择器的语法找到tag:

```python
soup.select("title")
# [<title>The Dormouse's story</title>]

soup.select("p nth-of-type(3)")
# [<p class="story">...</p>]

```

通过tag标签逐层查找:

```python
soup.select("body a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("html head title")
# [<title>The Dormouse's story</title>]

```

找到某个tag标签下的直接子标签 [[6\]](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id93) :

```python
soup.select("head > title")
# [<title>The Dormouse's story</title>]

soup.select("p > a")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie"  id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("p > a:nth-of-type(2)")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

soup.select("p > #link1")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select("body > a")
# []

```

找到兄弟节点标签:

```python
soup.select("#link1 ~ .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie"  id="link3">Tillie</a>]

soup.select("#link1 + .sister")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]

```

通过CSS的类名查找:

```python
soup.select(".sister")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select("[class~=sister]")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

```

通过tag的id查找:

```python
soup.select("#link1")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select("a#link2")
# [<a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

同时用多种CSS选择器查询元素:

```
soup.select("#link1,#link2")
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>]
```

通过是否存在某个属性来查找:

```python
soup.select('a[href]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]
```

通过属性的值来查找:

```python
soup.select('a[href="http://example.com/elsie"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]

soup.select('a[href^="http://example.com/"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>,
#  <a class="sister" href="http://example.com/lacie" id="link2">Lacie</a>,
#  <a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href$="tillie"]')
# [<a class="sister" href="http://example.com/tillie" id="link3">Tillie</a>]

soup.select('a[href*=".com/el"]')
# [<a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>]
```

通过语言设置来查找:

```python
multilingual_markup = """
 <p lang="en">Hello</p>
 <p lang="en-us">Howdy, y'all</p>
 <p lang="en-gb">Pip-pip, old fruit</p>
 <p lang="fr">Bonjour mes amis</p>
"""
multilingual_soup = BeautifulSoup(multilingual_markup)
multilingual_soup.select('p[lang|=en]')
# [<p lang="en">Hello</p>,
#  <p lang="en-us">Howdy, y'all</p>,
#  <p lang="en-gb">Pip-pip, old fruit</p>]

```

返回查找到的元素的第一个

```python
soup.select_one(".sister")
# <a class="sister" href="http://example.com/elsie" id="link1">Elsie</a>

```

对于熟悉CSS选择器语法的人来说这是个非常方便的方法.Beautiful Soup也支持CSS选择器API, 如果你仅仅需要CSS选择器的功能,那么直接使用 `lxml` 也可以, 而且速度更快,支持更多的CSS选择器语法,但Beautiful Soup整合了CSS选择器的语法和自身方便使用API.

### 修改文档树

#### 修改tag的名称和属性

在 [Attributes](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#attributes) 的章节中已经介绍过这个功能,但是再看一遍也无妨. 重命名一个tag,改变属性的值,添加或删除属性:

```python
soup = BeautifulSoup('<b class="boldest">Extremely bold</b>')
tag = soup.b

tag.name = "blockquote"
tag['class'] = 'verybold'
tag['id'] = 1
tag
# <blockquote class="verybold" id="1">Extremely bold</blockquote>

del tag['class']
del tag['id']
tag
# <blockquote>Extremely bold</blockquote>

```

#### 修改 .string

给tag的 `.string` 属性赋值,就相当于用当前的内容替代了原来的内容:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)

tag = soup.a
tag.string = "New link text."
tag
# <a href="http://example.com/">New link text.</a>
```

注意: 如果当前的tag包含了其它tag,那么给它的 `.string` 属性赋值会覆盖掉原有的所有内容包括子tag

#### append()

`Tag.append()` 方法想tag中添加内容,就好像Python的列表的 `.append()` 方法:

```python
soup = BeautifulSoup("<a>Foo</a>")
soup.a.append("Bar")

soup
# <html><head></head><body><a>FooBar</a></body></html>
soup.a.contents
# [u'Foo', u'Bar']
```

### NavigableString() 和 .new_tag()

如果想添加一段文本内容到文档中也没问题,可以调用Python的 `append()` 方法 或调用 `NavigableString` 的构造方法:

```python
soup = BeautifulSoup("<b></b>")
tag = soup.b
tag.append("Hello")
new_string = NavigableString(" there")
tag.append(new_string)
tag
# <b>Hello there.</b>
tag.contents
# [u'Hello', u' there']
```

如果想要创建一段注释,或 `NavigableString` 的任何子类, 只要调用 NavigableString 的构造方法:

```python
from bs4 import Comment
new_comment = soup.new_string("Nice to see you.", Comment)
tag.append(new_comment)
tag
# <b>Hello there<!--Nice to see you.--></b>
tag.contents
# [u'Hello', u' there', u'Nice to see you.']

```

\# 这是Beautiful Soup 4.2.1 中新增的方法

创建一个tag最好的方法是调用工厂方法 `BeautifulSoup.new_tag()` :

```python
soup = BeautifulSoup("<b></b>")
original_tag = soup.b

new_tag = soup.new_tag("a", href="http://www.example.com")
original_tag.append(new_tag)
original_tag
# <b><a href="http://www.example.com"></a></b>

new_tag.string = "Link text."
original_tag
# <b><a href="http://www.example.com">Link text.</a></b>

```

第一个参数作为tag的name,是必填,其它参数选填

### insert()

`Tag.insert()` 方法与 `Tag.append()` 方法类似,区别是不会把新元素添加到父节点 `.contents` 属性的最后,而是把元素插入到指定的位置.与Python列表总的 `.insert()` 方法的用法下同:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
tag = soup.a

tag.insert(1, "but did not endorse ")
tag
# <a href="http://example.com/">I linked to but did not endorse <i>example.com</i></a>
tag.contents
# [u'I linked to ', u'but did not endorse', <i>example.com</i>]
```

### insert_before() 和 insert_after()

`insert_before()` 方法在当前tag或文本节点前插入内容:

```python
soup = BeautifulSoup("<b>stop</b>")
tag = soup.new_tag("i")
tag.string = "Don't"
soup.b.string.insert_before(tag)
soup.b
# <b><i>Don't</i>stop</b>
```

`insert_after()` 方法在当前tag或文本节点后插入内容:

```python
soup.b.i.insert_after(soup.new_string(" ever "))
soup.b
# <b><i>Don't</i> ever stop</b>
soup.b.contents
# [<i>Don't</i>, u' ever ', u'stop']
```

### clear()

`Tag.clear()` 方法移除当前tag的内容:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
tag = soup.a

tag.clear()
tag
# <a href="http://example.com/"></a>
```

### extract()

`PageElement.extract()` 方法将当前tag移除文档树,并作为方法结果返回:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

i_tag = soup.i.extract()

a_tag
# <a href="http://example.com/">I linked to</a>

i_tag
# <i>example.com</i>

print(i_tag.parent)
None
```

这个方法实际上产生了2个文档树: 一个是用来解析原始文档的 `BeautifulSoup` 对象,另一个是被移除并且返回的tag.被移除并返回的tag可以继续调用 `extract` 方法:

```python
my_string = i_tag.string.extract()
my_string
# u'example.com'

print(my_string.parent)
# None
i_tag
# <i></i>
```

### decompose()

`Tag.decompose()` 方法将当前节点移除文档树并完全销毁:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

soup.i.decompose()

a_tag
# <a href="http://example.com/">I linked to</a>
```

### replace_with()

`PageElement.replace_with()` 方法移除文档树中的某段内容,并用新tag或文本节点替代它:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

new_tag = soup.new_tag("b")
new_tag.string = "example.net"
a_tag.i.replace_with(new_tag)

a_tag
# <a href="http://example.com/">I linked to <b>example.net</b></a>
```

`replace_with()` 方法返回被替代的tag或文本节点,可以用来浏览或添加到文档树其它地方

### wrap()

`PageElement.wrap()` 方法可以对指定的tag元素进行包装 [[8\]](https://beautifulsoup.readthedocs.io/zh_CN/v4.4.0/#id95) ,并返回包装后的结果:

```python
soup = BeautifulSoup("<p>I wish I was bold.</p>")
soup.p.string.wrap(soup.new_tag("b"))
# <b>I wish I was bold.</b>

soup.p.wrap(soup.new_tag("div"))
# <div><p><b>I wish I was bold.</b></p></div>
```

该方法在 Beautiful Soup 4.0.5 中添加

### unwrap()

`Tag.unwrap()` 方法与 `wrap()` 方法相反.将移除tag内的所有tag标签,该方法常被用来进行标记的解包:

```python
markup = '<a href="http://example.com/">I linked to <i>example.com</i></a>'
soup = BeautifulSoup(markup)
a_tag = soup.a

a_tag.i.unwrap()
a_tag
# <a href="http://example.com/">I linked to example.com</a>
```

与 `replace_with()` 方法相同, `unwrap()` 方法返回被移除的tag