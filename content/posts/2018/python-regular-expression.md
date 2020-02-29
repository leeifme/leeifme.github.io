---
title: Python 之正则表达式
date: 2018-01-20 20:36:25
tags:
- Python 
categories: 
- collect
---

### 1. 什么是正则表达式

正则表达式：也成为规则表达式，英文名称 Regular Expression，我们在程序中经常会缩写为 regex 或者 regexp，专门用于进行文本检索、匹配、替换等操作的一种技术。
*注意：正则表达式是一种独立的技术，并不是某编程语言独有的*

正则表达式，是一种特殊的符号，这样的符号是需要解释才能使用的，也就是需要正则表达式引擎来进行解释，目前正则表达式的引擎主要分三种：DFA，NFA、POSIX NFA，*有兴趣了正则表达式引擎的童鞋，可以自己查看资料*

### 2. 正则表达式语法结构

接下来，我们开始了解这样一个神秘的可以类似人类神经网络一样思考问题的技术的语法结构。
*注意：我们通过 python 程序进行测试，但是正则表达式的语法结构在各种语言环境中都是通用的。*

###### 2.1. 入门案例：了解正则表达式

我们通过一个简单的案例入手：通常情况下，我们会验证用户输入的手机号码是否合法，是否 156/186/188 开头的手机号码，如果按照常规验证手段，就需要对字符串进行拆分处理，然后逐步匹配

> 重要提示：python 中提供了`re`模块，包含了正则表达式的所有功能，专门用于进行正则表达式的处理；

我们首先看一下，常规的手机号码验证过程

```python
userphone = input("请输入手机号码：")

# 验证用户手机号码是否合法的函数
def validatePhone(phone):
    msg = "提示信息：请输入手机号码"
    # 判断输入的字符的长度是否合法
    if len(phone) == 11:
        # 判断是否156/186/188开头
        if phone.startswith("156") or phone.startswith("186") or phone.startswith("188"):
            # 判断每一个字符都是数字
            for num in phone:
                # isdigit()函数用于判断调用者是否数字
                if not num.isdigit():
                    msg = "不能包含非法字符"
                    return msg
            msg = "手机号码合法"
        else:
            msg = "开头数字不合法"
    else:
        msg = "长度不合法"
    return msg

# 开始测试
print(validatePhone(userphone))
```

执行上面的代码，分别输入不同的手机号码，结果如下

> 请输入手机号码：188
> 长度不合法
>
> 请输入手机号码：15568686868
> 开头数字不合法
>
> 请输入手机号码：1566868686a
> 不能包含非法字符
>
> 请输入手机号码：15688888888
> 手机号码合法

我们再次使用正则表达式来改造这段程序
*注意：如果下面的程序中出现了一些语法不是很明白，没关系，后面会详细讲解*

```python
import re

# 接收用户输入
userphone = input("请输入手机号码")

# 定义验证手机号码的函数
def validatePhone(phone):
    # 定义正则表达式，Python中的正则表达式还是一个字符串，是以r开头的字符串
    regexp = r"^(156|186|188)\d{8}$"
    # 开始验证
    if re.match(regexp, phone):
        return "手机号码合法"
    else:
        return "手机号码只能156/186/188开头，并且每一个字符都是数字，请检查"

# 开始验证
print(validatePhone(userphone))
```

执行上面的代码，我们得到正常验证的结果，大家可以自己试一试。
我们从这两套代码中，可以看出来，使用了正则表达式之后的程序变得非常简洁了，那保持好你的冲动和热情，让正则表达式来搞事吧

###### 2.3. python 中的正则表达式模块 re

python 提供的正则表达式处理模块 re，提供了各种正则表达式的处理函数

**2.3.1 字符串查询匹配的函数：**

| 函数                      | 描述                                       |
| ----------------------- | ---------------------------------------- |
| re.match(reg, info)     | 用于在**开始位置**匹配目标字符串 info 中符合正则表达式 reg 的字符，匹配成功会返回一个 match 对象，匹配不成功返回 None |
| re.search(reg, info)    | 扫描**整个字符串 info**，使用正则表达式 reg 进行匹配，匹配成功返回匹配的第一个 match 对象，匹配不成功返回 None |
| re.findall(reg, info)   | 扫描**整个字符串 info**，将符合正则表达式 reg 的字符全部提取出来存放在列表中返回 |
| re.fullmatch(reg, info) | 扫描整个字符串，如果整个字符串都包含在正则表达式表示的范围中，返回整个字符串，否则返回 None |
| re.finditer(reg, info)  | 扫描整个字符串，将匹配到的字符保存在一个可以遍历的列表中             |

参考官方 re.py 源代码如下：

```python
def match(pattern, string, flags=0):
    """Try to apply the pattern at the start of the string, returning
    a match object, or None if no match was found."""
    return _compile(pattern, flags).match(string)

def fullmatch(pattern, string, flags=0):
    """Try to apply the pattern to all of the string, returning
    a match object, or None if no match was found."""
    return _compile(pattern, flags).fullmatch(string)

def search(pattern, string, flags=0):
    """Scan through string looking for a match to the pattern, returning
    a match object, or None if no match was found."""
    return _compile(pattern, flags).search(string)

def findall(pattern, string, flags=0):
    """Return a list of all non-overlapping matches in the string.

    If one or more capturing groups are present in the pattern, return
    a list of groups; this will be a list of tuples if the pattern
    has more than one group.

    Empty matches are included in the result."""
    return _compile(pattern, flags).findall(string)

def finditer(pattern, string, flags=0):
    """Return an iterator over all non-overlapping matches in the
    string.  For each match, the iterator returns a match object.

    Empty matches are included in the result."""
    return _compile(pattern, flags).finditer(string)
```

**2.3.2 字符串拆分替换的函数：**

| 函数                        | 描述                                       |
| ------------------------- | ---------------------------------------- |
| re.split(reg, string)     | 使用指定的正则表达式 reg 匹配的字符，将字符串 string 拆分成一个字符串列表，如：re.split(r"\s+", info)，表示使用一个或者多个空白字符对**字符串 info** 进行拆分，并返回一个拆分后的字符串列表 |
| re.sub(reg, repl, string) | 使用**指定的字符串 repl** 来**替换目标字符串 string** 中**匹配正则表达式 reg** 的字符 |

参考官方源代码如下：

```python
def split(pattern, string, maxsplit=0, flags=0):
    """Split the source string by the occurrences of the pattern,
    returning a list containing the resulting substrings.  If
    capturing parentheses are used in pattern, then the text of all
    groups in the pattern are also returned as part of the resulting
    list.  If maxsplit is nonzero, at most maxsplit splits occur,
    and the remainder of the string is returned as the final element
    of the list."""
    return _compile(pattern, flags).split(string, maxsplit)

def sub(pattern, repl, string, count=0, flags=0):
    """Return the string obtained by replacing the leftmost
    non-overlapping occurrences of the pattern in string by the
    replacement repl.  repl can be either a string or a callable;
    if a string, backslash escapes in it are processed.  If it is
    a callable, it's passed the match object and must return
    a replacement string to be used."""
    return _compile(pattern, flags).sub(repl, string, count)
```

接下来，我们进入正则表达式干货部分

###### 2.4. 正则表达式中的元字符

在使用正则表达式的过程中，一些包含特殊含义的字符，用于表示字符串中一些特殊的位置，非常重要，我们先简单了解一下一些常用的元字符

| 元字符  | 描述                       |
| ---- | ------------------------ |
| ^    | 表示匹配字符串的开头位置的字符          |
| $    | 表示匹配字符串的结束位置的字符          |
| .    | 表示匹配任意一个字符               |
| \d   | 匹配一个数字字符                 |
| \D   | 匹配一个非数字字符                |
| \s   | 匹配一个空白字符                 |
| \S   | 匹配一个非空白字符                |
| \w   | 匹配一个数字 / 字母 / 下划线中任意一个字符 |
| \W   | 匹配一个非数字字母下划线的任意一个字符      |
| \b   | 匹配一个单词的边界                |
| \B   | 匹配不是单词的开头或者结束位置          |

上干货：代码案例

```python
# 导入正则表达式模块
import re

# 定义测试文本字符串，我们后续在这段文本中查询数据
msg1 = """Python is an easy to learn, powerful programming language.
It has efficient high-level data structures and a simple but effective approach to object-oriented programming.
Python’s elegant syntax and dynamic typing, together with its interpreted nature, 
make it an ideal language for scripting and rapid application development in many areas on most platforms.
"""
msg2 = "hello"
msg3 = "hello%"

# 定义正则表达式，匹配字符串开头是否为python
regStart = r"efficient"

# 从字符串开始位置匹配，是否包含符合正则表达式的内容，返回匹配到的字符串的Match对象
print(re.match(regStart, msg1))
# 扫描整个字符串，是否包含符合正则表达式的内容，返回匹配到的第一个字符串的Match对象
print(re.search(regStart, msg1))
# 扫描整个字符串，是否包含符合正则表达式的内容，返回匹配到的所有字符串列表
print(re.findall(regStart, msg1))
# 扫描整个字符串，是否包含符合正则表达式的内容，返回匹配到的字符串的迭代对象
for r in re.finditer(regStart, msg1):
    print("->"+ r.group())
# 扫描整个字符串，是否包含在正则表达式匹配的内容中，是则返回整个字符串，否则返回None
print(re.fullmatch(r"\w*", msg2))
print(re.fullmatch(r"\w*", msg3))
```

上述代码执行结果如下：

> ~ None
> ~<sre.SRE_Match object; span=(66, 75), match='efficient'>
> ~['efficient']
> ~->efficient
> ~<sre.SRE_Match object; span=(0, 5), match='hello'>
> ~None

###### 2.5. 正则表达式中的量词

正则表达式中的量词，是用于限定数量的特殊字符

| 量词     | 描述                                |
| ------ | --------------------------------- |
| x*     | 用于匹配符号 * 前面的字符出现 0 次或者多次          |
| x+     | 用于匹配符号 + 前面的字符出现 1 次或者多次          |
| x？     | 用于匹配符号？前面的字符出现 0 次或者 1 次          |
| x{n}   | 用于匹配符号 {n} 前面的字符出现 n 次            |
| x{m,n} | 用于匹配符号 {m,n} 前面的字符出现至少 m 次，最多 n 次 |
| x{n,}  | 用于匹配符号 {n,} 前面的字符出现至少 n 次         |

接上代码干货：

```python
# 导入正则表达式模块
import re

# 定义测试文本字符串，我们后续在这段文本中查询数据
msg1 = """goodgoodstudy!,dooodooooup"""

# 匹配一段字符串中出现单词o字符0次或者多次的情况
print(re.findall(r"o*", msg1))
# 匹配一段字符串中出现单词o字符1次或者多次的情况
print(re.findall(r"o+", msg1))
# 匹配一段字符串中出现单词o字符0次或者1次的情况
print(re.findall(r"o?", msg1))
# 匹配字符串中连续出现2次字符o的情况
print(re.findall(r"o{2}", msg1))
# 匹配字符串中连续出现2次以上字符o的情况
print(re.findall(r"o{2,}", msg1))
# 匹配字符串中连续出现2次以上3次以内字符o的情况
print(re.findall(r"o{2,3}", msg1))
```

上述代码大家可以自行尝试并分析结果。执行结果如下：

###### 2.6. 正则表达式中的范围匹配

在正则表达式中，针对字符的匹配，除了快捷的元字符的匹配，还有另一种使用方括号进行的范围匹配方式，具体如下：

| 范围            | 描述                               |
| ------------- | -------------------------------- |
| [0-9]         | 用于匹配一个 0~9 之间的数字，等价于 \ d         |
| [^0-9]        | 用于匹配一个非数字字符，等价于 \ D              |
| [3-6]         | 用于匹配一个 3~6 之间的数字                 |
| [a-z]         | 用于匹配一个 a~z 之间的字母                 |
| [A-Z]         | 用于匹配一个 A~Z 之间的字母                 |
| [a-f]         | 用于匹配一个 a~f 之间的字母                 |
| [a-zA-Z]      | 用于匹配一个 a~z 或者 A-Z 之间的字母，匹配任意一个字母 |
| [a-zA-Z0-9]   | 用于匹配一个字母或者数字                     |
| [a-zA-Z0-9_]  | 用于匹配一个字母或者数字或者下划线，等价于 \ w        |
| [^a-zA-Z0-9_] | 用于匹配一个非字母或者数字或者下划线，等价于 \ W       |

*注意：不要使用 [0-120] 来表示 0~120 之间的数字，这是错误的*

整理测试代码如下：

```python
# 引入正则表达式模块
import re

msg = "Hello, The count of Today is 800"
# 匹配字符串msg中所有的数字
print(re.findall(r"[0-9]+", msg))
# 匹配字符串msg中所有的小写字母
print(re.findall(r"[a-z]+", msg))
# 匹配字符串msg中所有的大写字母
print(re.findall(r"[A-Z]+", msg))
# 匹配字符串msg中所有的字母
print(re.findall(r"[A-Za-z]+", msg))
```

上述代码执行结果如下：

> ['800']
> ['ello', 'he', 'count', 'of', 'oday', 'is']
> ['H', 'T', 'T']
> ['Hello', 'The', 'count', 'of', 'Today', 'is']

###### 2.7. 正则表达式中的分组

正则表达式主要是用于进行字符串检索匹配操作的利器
在一次完整的匹配过程中，可以将匹配到的结果进行分组，这样就更加的细化了我们对匹配结果的操作
正则表达式通过圆括号 () 进行分组，以提取匹配结果的部分结果

常用的两种分组：

| 分组                   | 描述                                       |
| -------------------- | ---------------------------------------- |
| (expression)         | 使用圆括号直接分组；正则表达式本身匹配的结果就是一个组，可以通过 group() 或者 group(0) 获取；然后正则表达式中包含的圆括号就是按照顺序从 1 开始编号的小组 |
| (?P<name>expression) | 使用圆括号分组，然后给当前的圆括号表示的小组命名为 name，可以通过 group(name) 进行数据的获取 |

废话少说，上干货：

```python
# 引入正则表达式模块
import re

# 用户输入座机号码，如"010-6688465"
phone = input("请输入座机号码：")
# 1.进行正则匹配,得到Match对象，对象中就包含了分组信息
res1 = re.search(r"^(\d{3,4})-(\d{4,8})$", phone)
# 查看匹配结果
print(res1)
# 匹配结果为默认的组，可以通过group()或者group(0)获取
print(res1.group())
# 获取结果中第一个括号对应的组数据：处理区号
print(res1.group(1))
# 获取结果中第二个括号对应的组数据：处理号码
print(res1.group(2))

# 2.进行正则匹配,得到Match对象，对象中就包含了命名分组信息
res2 = re.search(r"^(?P<nstart>\d{3,4})-(?P<nend>\d{4,8})$", phone)
# 查看匹配结果
print(res2)
# 匹配结果为默认的组，可以通过group()或者group(0)获取
print(res2.group(0))
# 通过名称获取指定的分组信息：处理区号
print(res2.group("nstart"))
# 通过名称获取指定分组的信息：处理号码
print(res2.group("nend"))
```

上述代码就是从原始字符串中，通过正则表达式匹配得到一个结果，但是使用了分组之后，就可以将结果数据通过分组进行细化处理，执行结果如下：

> 请输入座机号码：021-6565789
> <_sre.SRE_Match object; span=(0, 11), match='021-6565789'>
> 021-6565789
> 021
> 6565789
> <_sre.SRE_Match object; span=(0, 11), match='021-6565789'>
> 021-6565789
> 021
> 6565789

###### 2.8. 正则表达式中的特殊用法

使用分组的同时，会有一些特殊的使用方式如下：

| 表达式             | 描述                                       |
| --------------- | ---------------------------------------- |
| (?:expression)  | 作为正则表达式的一部分，但是匹配结果丢弃                     |
| (?=expression)  | 匹配 expression 表达式前面的字符，如 "How are you doing" , 正则 "(?<txt>.+(?=ing))" 这里取 ing 前所有的字符，并定义了一个捕获分组名字为 "txt" 而 "txt" 这个组里的值为 "How are you do" |
| (?<=expression) | 匹配 expression 表达式后面的字符，如 "How are you doing" 正则 "(?<txt>(?<=How).+)"这里取"How"之后所有的字符，并定义了一个捕获分组名字为"txt"而"txt"这个组里的值为" are you doing"; |
| (?!expression)  | 匹配字符串后面不是 expression 表达式字符，如 "123abc" 正则 "\d{3}(?!\d)" 匹配 3 位数字后非数字的结果 |
| (?<!expression) | 匹配字符串前面不是 expression 表达式字符，如 "abc123" 正则 "(?<![0-9])123"匹配"123"前面是非数字的结果也可写成"(?!<\d)123" |

### 2.9 正则表达式的贪婪模式和懒惰模式

在某些情况下，我们匹配的字符串出现一些特殊的规律时，就会出现匹配结果不尽如人意的意外情况
如：在下面的字符串中，将 div 标签中的所有内容获取出来

```
<div>内容1</div><p>这本来是不需要的内容</p><div>内容2</div>
```

此时，我们想到的是，使用 <div> 作为关键信息进行正则表达式的定义，如下

```
regexp = r"<div>.*</div>"
```

本意是使用上述代码来完成 div 开始标签和结束标签之间的内容匹配，但是，匹配的结果如下

```
<div> [内容1</div><p>这本来是不需要的内容</p><div>内容2] </div>
```

我们可以看到，上面匹配的结果，是将字符串开头的 <div> 标签和字符串结束的 </div> 当成了匹配元素，对包含在中间的内容直接进行了匹配，也就得到了我们期望之外的结果：

```
内容1</div><p>这本来是不需要的内容</p><div>内容2
```

上述就是我们要说的正则表达式的第一种模式：贪婪模式
**贪婪模式**：正则表达式匹配的一种模式，速度快，但是匹配的内容会从字符串两头向中间搜索匹配（比较贪婪~），一旦匹配选中，就不继续向字符串中间搜索了，过程如下：

```html
开始：<div>内容1</div><p>这本来是不需要的内容</p><div>内容2</div>

第一次匹配：【<div>内容1</div><p>这本来是不需要的内容</p><div>内容2</div>】

第二次匹配<div>【内容1</div><p>这本来是不需要的内容</p><div>内容2】</div>

匹配到正则中需要的结果，不再继续匹配，直接返回匹配结果如下：
内容1</div><p>这本来是不需要的内容</p><div>内容2
```

明显贪婪模式某些情况下，不是我们想要的，所以出现了另一种模式：懒惰模式
**懒惰模式**：正则表达式匹配的另一种模式，会首先搜索匹配正则表达式开始位置的字符，然后逐步向字符串的结束位置查找，一旦找到匹配的就返回，然后接着查找

```
regexp = r"<div>.*?</div>"
```

```html
开始：<div>内容1</div><p>这本来是不需要的内容</p><div>内容2</div>

第一次匹配：【<div>】内容1</div><p>这本来是不需要的内容</p><div>内容2</div>

第二次匹配【<div>内容1</div>】<p>这本来是不需要的内容</p><div>内容2</div>

匹配到正则中需要的结果：内容1

继续向后查找

第三次匹配<div>内容1</div>【<p>这本来是不需要的内容</p>】<div>内容2</div>

第四次匹配<div>内容1</div><p>这本来是不需要的内容</p>【<div>内容2</div>】

匹配到正则中需要的结果：内容2

查找字符串结束！
```

> 正则表达式匹配的两种模式：贪婪模式、懒惰模式
> **贪婪模式**：从目标字符串的两头开始搜索，一次尽可能多的匹配符合条件的字符串，但是有可能会匹配到不需要的内容，正则表达式中的元字符、量词、范围等都模式是贪婪匹配模式，使用的时候一定要注意分析结果，如：`<div>.*</div>`就是一个贪婪模式，用于匹配 <div> 和 </div> 之间所有的字符
> **懒惰模式**：从目标字符串按照顺序从头到位进行检索匹配，尽可能的检索到最小范围的匹配结果，语法结构是在贪婪模式的表达式后面加上一个符号? 即可，如`<div>.*?</div>`就是一个懒惰模式的正则，用于仅仅匹配最小范围的 <div> 和 </div> 之间的内容
>
> 不论贪婪模式还是懒惰模式，都有适合自己使用的地方，大家一定要根据实际需求进行解决方案的确定

 

**转载: https://www.imooc.com/article/22158**