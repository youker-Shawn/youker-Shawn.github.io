---
layout: post
title:  "编解码指南"
date:   2022-12-23
categories: coding Python
description:
---



因为`Python2.x`和`Python3.x`在字符串编码解码存在差异比较大，这里特别说明：文中的代码以及交互过程，**没有说明时默认指**`Python3`**下。**

# 基本认知

## 字符、字符串

字符，每个独立的符号图像。

比如 中文的每个方块字、英文的每个字母、每个数字、每个符号......都是字符。

连在一起的字符文本，就是字符串。



## 字符集、编解码规则

「字符集」即特定一堆字符的集合。

同时，每个**字符**对应有各自的**编号**。这一套**映射机制**，也可以叫这个字符集的「编解码规则」。

因为计算机历史发展和各国语言区别等原因，形成了多套字符集，如 ASCII 字符集、GBK 字符集、UTF-8 字符集。



## 字节、字节串

机器，只认得「01」这样的二进制信息。

每个「0」或「1」就是「1 个比特位」，即`1 bit`。

计算机中，常将「8个bit」作为一组，称为「1个字节」，即`1 Byte`。

多个字节，就是字节串了。

每个字节，会利用「4个bit位一组用十六进制来表示」的方式，来简化表示。

看起来是什么样呢？

比如十进制的数字255，用二进制表示：`11111111`。那这个字节可写作`\xff`、`0xff`，或者`FF`。



## 比特流、字节串、字符串

- 字符串：给人看的文本信息。
- 字节串：用于节省信息占用的空间。特别是在「硬盘存储信息、网络传输信息」的场景下，是以字节串存在的。
- 比特流：给机器看的。



## 编解码的乱码问题

1个「字符」，在字符集中的编号，可能由几个「字节」来表示，取决于是哪种字符集。



编解码，就是在「字符串」与「字节串」之间转变的过程。

- 编码：字符串 -> 字节串
- 解码：字节串 -> 字符串

「每个字符」编码为「几个什么字节」，或者，「哪几个字节一组」解码为「1个字符」，这些转变时的规则，由指定的「字符集」定好了。



乱码的根本原因：

**对同一段信息，「解码时使用的字符集」与「编码时使用的字符集」不一致。**导致解不出「常人能理解的」有效信息。



# 字符集的来龙去脉

起初，

只有`ascii`字符集：**一个字节对应一个字符。**

字符集中只有**128**个字符：普通符号、英文字母、数字。

所以只用到了一个字节中的**0～127。**还剩了一半的高位那128个编号没用上。



后来，

出现了`latin-1`、`gbk`、`Shift-JIS`、`Euc-kr`...各种字符集：**单字节、多字节的映射。**

因为计算机传到了世界各地，各国有各自的语言系统，文字符号需要对应的编号，就开始各自自由发挥了。

- 单字节映射字符集：拉丁文编码`latin-1`字符集，同样是一个字节对应一个字符。但在兼容`ascii`基础上，使用了剩余的「128～255」编号。
- 多字节映射字符集：许多国家的语言字符，多到一个字节不够用。可能要两三个字节才能代表一个字符。

混乱就是这么开始的！

同一个字符，在不同国家的编号（字节串）往往是不一样的！如下，`latin-1`甚至无法表示汉字的「中」字符。

```python
>>> "中".encode("utf-8")
b'\xe4\xb8\xad'
>>> "中".encode("gbk")
b'\xd6\xd0'
>>> "中".encode("Euc-kr")
b'\xf1\xe9'
>>> "中".encode("latin-1")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'latin-1' codec can't encode character '\u4e2d' in position 0: ordinal not in range(256)
```

同理，超过`ascii`范围的同一个编号，在不同字符集下，解码出来的字符往往不同！如下，只有用对应的字符集解码才能看到正确的字符：

```python
>>> a = "中".encode("gbk")
>>> a
b'\xd6\xd0'
>>> b = a.decode("Euc-kr")
>>> b
'櫓'
>>> c = a.decode("gbk")
>>> c
'中'
```



怎么解决混乱呢？

只要大家都用同一种字符集，同样的映射规则，就不会乱码了呀。

这种字符集，需要将所有国家所有语言的文字符号都涵盖进来。

这就是`unicode`字符集。（兼容了`ascii`）

表示时，在字节串前加上前缀`U+`，如汉字「中」的 unicode 编号为`U+4E2D`。



还有个需要解决的问题，

随着不断纳入各国的各种字符，包括现在流行的emoji表情字符，使得`unicode`字符集，从最初的「2个字节代表1个字符」到现在要多字节才能表示新字符。

这种固定长度表示一个字符的方式，会导致资源浪费的问题。

比如，对于原先使用`ascii`字符集的国家，一个字节表示一个「A」已经足够，为什么还要2倍甚至更多倍的空间来表示呢？

所以，诞生了「可变长的字符集」。

最流行的就是`utf-8`字符集，1～4字节表示1个字符，不同国家的字符使用不同的长度编解码。



至此，全世界的计算机使用者，**开始使用**`**unicode**`**作为中间字符集来表示字符串，并流行用**`**utf-8**`**字符集作为编解码方式。**



# 该编码还是该解码？

这就需要知道，在什么情形下存放的是`unicode`字符集表示的「字符串」，哪些情形下是某种字符集编码过的「字节串」。

首先要明白：无论哪种，本质**都是二进制格式的数据**！

总体来说，分为2种情况：

- 加载在内存中：`unicode`字符集
- 网络传输、磁盘存取：某种字符集编码过的字节串



原因是，

编码过的字节串，往往比`unicode`字符串要节省空间。这在网络传输、硬盘存取时节省流量，也省硬盘空间。

而在内存中，需要的是高效的操作，浪费一些空间，用统一的长度来避免转换的耗时。



比如在IDE中编写代码，这些数据都是加载在内存中的，所以环境是`unicode`字符串。

网络传输时是字节串，磁盘上存放的也是字节串。

在代码中将数据交给`socket`发送前，要先将数据进行「字符串->字节串」的编码过程，一般指定`utf-8`进行编码。

打开文件，写入数据保存关闭文件时，也是从内存到磁盘，「字符串->字节串」的编码过程。



那么，反过来的场景，就是解码过程了，使用的字符集，只要对应，就不会出现乱码。



# 编解码时遇到特殊情况如何兼容？

编解码器可以通过接受`errors`字符串参数，来实现不同的错误处理方案:

```python
>>> 'German ß, ♬'.encode(encoding='ascii', errors='backslashreplace')
b'German \\xdf, \\u266c'
>>> 'German ß, ♬'.encode(encoding='ascii', errors='xmlcharrefreplace')
b'German &#223;, &#9836;'
```

常用的有这些：

| 值               | 含义                                                         |
| ---------------- | ------------------------------------------------------------ |
| strict           | 引发 UnicodeError (或其子类)，这是默认的方案。 在 strict_errors() 中实现。 |
| ignore           | 忽略错误格式的数据并且不加进一步通知就继续执行。 在 ignore_errors() 中实现。 |
| replace          | 用一个替代标记来替换。 在编码时，使用 ? (ASCII 字符)。 在解码时，使用 � (U+FFFD，官方的 REPLACEMENT CHARACTER)。 在 replace_errors() 中实现。 |
| backslashreplace | 用反斜杠转义序列来替换。 在编码时，使用格式为 \xhh \uxxxx \Uxxxxxxxx 的 Unicode 码位十六进制表示形式。 在解码时，使用格式为 \xhh 的字节值十六进制表示形式。 在 backslashreplace_errors() 中实现。 |
| surrogateescape  | 在解码时，将字节替换为 U+DC80 至 U+DCFF 范围内的单个代理代码。 当在编码数据时使用 'surrogateescape' 错误处理方案时，此代理将被转换回相同的字节。 （请参阅 PEP 383 了解详情。） |



# 乱码时常见的「�」与「锟斤拷」

先明白一点，这两个都是在**解码时出现的所谓「乱码字符」。**



� 这个字符，是`**unicode**`**字符集中的一个特殊的字符**`0xFFFD(65533)`。

表示一个「占位符」，用来表达**使用某个字符集解码时，无法解码的不认识的字符。**

比如当某个字节串用`UTF-8` 解码为`unicode` 字符串时，发现有不认识的，就会用� 来占位。

所以，出现这个字符，说明当前的文本，至少已经是`unicode`字符串了！



「锟斤拷」的出现原因和「�」有关系：

一个「�」用`UTF-8` 编码，是字节串`0xEFBFBD`，3个字节。

当`unicode`字符串有连续两个�时，就像这样「��」，对应`UTF-8`字节串就是`0xEFBFBDEFBFBD`。

（这个若字节数组表示，就是`[-17, -65, -67, -17, -65, -67]`）



此时，拿到这个字节串的人，却用了中文常用的`GBK`来解码！

因为`GBK` 采用「双字节编码」方案，两个字节解释为一个字符！

所以上面的字节串**被理解为了3个字符：**`**0xEFBF, 0xBDEF, 0xBFBD**`，对应`GBK`的映射规则，解码出来就是**「锟（0xEFBF），斤（0xBDEF），拷（0xBFBD）」！！！**



总结这个出现的过程就是：

起初字节串解码异常，导致`unicdoe`字符串带有「��」的2个字符 --> 没有解决问题，就用`utf-8`编码传输 --> 又错用`GBK`解码，**最终「��」变成了「锟斤拷」**



# 使用chardet探测字节串的编码字符集 

安装`chardet`第三方库：`pip install chardet`

```shell
>>> import chardet
>>> 
>>> str_bytes = b'\xc0\xeb\xc0\xeb\xd4\xad\xc9\xcf\xb2\xdd\xa3\xac\xd2\xbb\xcb\xea\xd2\xbb\xbf\xdd\xc8\xd9'
>>> chardet.detect(str_bytes)
{'encoding': 'GB2312', 'confidence': 0.7407407407407407, 'language': 'Chinese'}
>>> 
>>> str_bytes.decode('GB2312')
'离离原上草，一岁一枯荣'
>>> str_bytes.decode('gb2312')
'离离原上草，一岁一枯荣'
>>> str_bytes.decode('gbk')
'离离原上草，一岁一枯荣'
```

给出的字节串文本内容越多，猜测出的编码字符集越准确。



# 网络字节序

使用`socket`进行传输二进制数据前，数据映射到「**字节串」**，涉及到**「网络字节序」**的问题：

比如数字，十进制时的左侧为高位，但在转为二进制的字节串之后可能是几个字节，高位的字节到底是存放在**内存地址低的一侧还是高的一侧？**

- **大端：高位**放在前面（内存低地址）
- **小端：低位**放在前面（内存低地址）

可以使用Python的`hex()`来看存储的方式。

```python
>>> hex(4253)
'0x109d'
```

`\x10`就是4253的高位字节，这种存放方式表示大端方式。



存储为字节序时，究竟大端还是小端，这**取决于处理器的架构。**

不过传输时使用哪种字节序，可以手动指定进行转换。**推荐使用**`**struct**`**模块来进行转换。**例子：

```python
>>> import struct
>>> struct.pack('<i', 4253)
b'\x9d\x10\x00\x00'
>>> struct.pack('>i', 4253)
b'\x00\x00\x10\x9d'
```

`i`表示用4字节存储一个整数。`<`表示小端，`>`表示大端。

使用网络传输二进制数据时，两边的转换一致即可，可以使用`pack()`和`unpack()`测试下。





# 2.x 与 3.x 的「字符串、字节串」差异

代码文件中，**Python3定义的是「字符串」，而Python2定义的其实是「字节串」，只是都显示类型**`**str**`**，但其实类型不同！**

```python
# Python3.x
>>> s = "123abc中文"
>>> type(s)
<class 'str'>
>>> s.encode("utf-8")
b'123abc\xe4\xb8\xad\xe6\x96\x87'

# Python2.x
>>> s = "123abc中文"
>>> type(s)
<type 'str'>
>>> s.encode("utf-8")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 6: ordinal not in range(128)
```

只有`unicode`字符串才能`encode`。

Python2隐式地想要先`decode`字节串，但因为默认编码问题，无法解码，所以如上报错了。



- 3的`str` == 2的`unicode`
- 3的`bytes` == 2的`str`

```python
# Python3.x
>>> s = "123abc中文"
>>> s
'123abc中文'
>>> type(s)
<class 'str'>

>>> bytes_str = b'123abc\xe4\xb8\xad\xe6\x96\x87'
>>> type(bytes_str)
<class 'bytes'>

>>> type(s.encode("utf-8"))
<class 'bytes'>

>>> equal_s_str = '123abc\u4e2d\u6587'
>>> equal_s_str
'123abc中文'
>>> type(equal_s_str)
<class 'str'>
>>> equal_s_str == s
True
>>> equal_s_str is s
False

# ========

# Python2.x
>>> s = "123abc中文"  # 这里因为交互环境，没有文件的编码声明，所以用系统的UTF-8编码过了。
>>> s
'123abc\xe4\xb8\xad\xe6\x96\x87
>>> type(s)
<type 'str'>

>>> unicode_str = u'123abc\u4e2d\u6587'
>>> unicode_str
u'123abc\u4e2d\u6587'
>>> type(unicode_str)
<type 'unicode'>

>>> type(s.decode("utf-8"))
<type 'unicode'>

>>> just_bytes_str = '123abc\u4e2d\u6587'
>>> type(just_bytes_str)
<type 'str'>
>>> just_bytes_str
'123abc\\u4e2d\\u6587'
>>> just_bytes_str == s
False
```



另外，**默认的编解码字符集，也有差异**：

- `**Python2**`**使用的是**`**ascii**`
- `**Python3**`**使用的是**`**utf-8**`

查看方法如下：

```python
# Python3.x
>>> import sys
>>> sys.getdefaultencoding()
'utf-8'

# Python2.x
>>> import sys
>>> sys.getdefaultencoding()
'ascii'
```



而以上提到的差异，导致了2和3都在用时，容易出现的各种奇怪问题！见下文。



# 大毒瘤：Python2的默认编码

工作中使用Python这么久以来，每次碰上匪夷所思的乱码问题，或多或少和「使用Python2」有关系！

「Python3」省心多了，下面列举一些「Python2」中才有的现象。



## Python代码文件开头的编码声明

常常看到**Py文件的开头**，注释着：`# -*- coding:utf-8 -*-`或 `# coding=utf-8`。

**Python2环境下，代码文件中，任意位置出现了中文等非ASCII字符时，就必须声明，否则报错！**

而Python3没有这个问题。



原因是，

代码文件也是一堆文本字符串，无论是其中的注释，还是定义变量时的字符串值，文件保存到磁盘时，都要编码为字节串二进制数据才能保存。

那么用什么字符集来编码呢？

Python2用的就是`ascii`，必然无法编码非ASCII字符呀，这就是原因。



通过开头的**编码声明，可以指定：该python代码文件被保存时，以哪种字符集对文件内容进行编码。**

但**也就仅此而已，这个声明并没有替换Python2的默认编码**，其他地方用到了默认编码的，依然是`ascii`，依然会出现奇奇怪怪的问题。



## 当字符串参与拼接时

Python2 中，只要有`**unicode**`**字符串，参与字节串拼接时**，就**会先将「字节串」转为**`**unicode**`**字符串，用的就是Python2默认编码**`**ascii**`**。**

**仅仅都是字节串时不进行解码，字节拼接。**

而Python3默认是`utf-8`，所以只是`unicode`字符串的拼接，并没有字节串。

（Python3也可以用`u`开头表示`unicode`字符串）

（Python3不支持拼接 字符串和字节串，因为认为是不同类型，不作转换！）

```python
# Python3.x
>>> type("abc" + "123")
<class 'str'>
>>> type("abc" + "123" + "中文")
<class 'str'>
>>> type(u"abc" + "123")
<class 'str'>
>>> type(u"abc" + u"123")
<class 'str'>

>>> type(u"abc中文" + "abc")
<class 'str'>
>>> type("abc中文" + u"abc")
<class 'str'>
>>> type(u"abc中文" + "abc中文")
<class 'str'>

>>> "abc123中文".encode("utf-8") + "abc123中文"
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: can't concat str to bytes


# Python2.x
>>> type("abc" + "123")
<type 'str'>
>>> type("abc" + "123" + "中文")
<type 'str'>
>>> type(u"abc" + "123")
<type 'unicode'>
>>> type(u"abc" + u"123")
<type 'unicode'>

>>> type(u"abc中文" + "abc")
<type 'unicode'>

>>> type("abc中文" + u"abc")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 3: ordinal not in range(128)

>>> type(u"abc中文" + "abc中文")
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 3: ordinal not in range(128)
>>> 
```

总结个经验：

Python2中，**使用带有中文等非ASCII字符的字符串时，一定要带u开头，默认unicode，避免奇奇怪怪的问题。**



## 使用`str()`强制转换为字节串

将`unicode`转为字节串时，极其容易导致`UnicodeEncodeError`，最好先确认编码。

```shell
>>> a = u'123abc中文'
>>> str(a)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
UnicodeEncodeError: 'ascii' codec can't encode characters in position 6-7: ordinal not in range(128)
```

 

## 修改Python2的默认编码

Python2中，每个编码解码的过程都需要小心默认`ascii`带来的BUG。

所以，常常需要在代码中，手动指定编解码的字符集。这个指定的代码可能散落在项目中四处都是。



还有一种省心的方法：`setdefaultencoding()`：

```python
import sys
# sys.setdefaultencoding() 这时已经被删除
reload(sys)  # 重新加载sys  
sys.setdefaultencoding('utf-8')
```

解释器执行完文件的开头的这段代码后，**采用的默认编码就被替换了！**后续代码的隐式编解码过程都得到了替换。

调用`reload(sys)`，是因为`setdefaultencoding()`函数在`sys`模块导入时被系统调用完删除了。

所以必须`reload`一次`sys`模块，`setdefaultencoding`才可用。



此外，

部分`Python2`**内置库的源码**，也是采用默认的编码方式去编码。

源码写好的代码逻辑自己无法修改，传入的字符串若包含了中文，那么必然报错。

**此时，用**`**setdefaultencoding()**`**修改默认编码就很有用了！**

当然，这也有潜在风险，如果其他某处代码就是需要`ascii`字符集编码，那就可能出问题。（我认为还是很少的，毕竟`utf-8`是兼容的）



# 读写文件

使用`open()`函数打开文件对象时，是内存中的进程与磁盘在交互。

如果「文本模式」打开，会使用「操作系统默认编解码字符集」进行编解码。

而「二进制模式」（即打开模式加一个`b`）打开，则直接传递`bytes`字节流，不做编解码操作！



操作系统的默认编码，可如下查看：

```python
>>> import locale
>>> locale.getdefaultlocale()
('en_US', 'UTF-8')
```

类unix中是`utf-8`，**windows中则默认是**`**gbk**`**！**

**对于**`**utf-8**`**保存的代码文件，在windows下打开，要在**`**open**`**时指定**`**encoding**`**为**`**utf-8**`**，否则看到的就是乱码！！**



# 复杂的print()

`print()` 的内部机制很复杂，做了很多背后工作，需要深入的话就要看CPython源码实现。



# 参考

- https://docs.python.org/zh-cn/3/library/codecs.html?highlight=encode#error-handlers
- 《Python网络编程（第3版）》 by [美]布兰登·罗德（Brandon Rhodes） 诸豪文 (译)
- https://www.cnblogs.com/vipchenwei/p/6993788.html

其他来自网络的文章，链接已丢失，可联系补上。