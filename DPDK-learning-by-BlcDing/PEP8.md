# PEP 8 Style Guide for Python Code

> **date**: 2021.08.15
>
> **auther**: BlcDing
>
> **tag**: `python`

PEP的全称是 Python Enhancement Proposals，通常被翻译为Python增强提案或Python改进建议书。

Python核心开发者主要通过邮件列表讨论问题、提议、计划等，PEP通常是汇总了多方信息，经过了部分核心开发者review和认可，最终形成的正式文档。

PEP主要分为3类：

- I - Informational PEP

- P - Process PEP

- S - Standards Track PEP

信息类：这类PEP就是提供信息，有告知类信息，也有指导类信息等等。例如PEP 20（The Zen of Python，即著名的Python之禅）、PEP 404 (Python 2.8 Un-release Schedule，即宣告不会有Python2.8版本)。

流程类：这类PEP主要是Python本身之外的周边信息。例如PEP 1（PEP Purpose and Guidelines，即关于PEP的指南）、PEP 347（Migrating the Python CVS to Subversion，即关于迁移Python代码仓）。

标准类：这类PEP主要描述了Python的新功能和新实践（implementation），是数量最多的提案。例如PEP 498（Literal String Interpolation，字面字符串插值）。

## 1.PEP 8介绍

| PEP:          | 8                                            |
| :------------ | -------------------------------------------- |
| Title:        | Style Guide for Python Code                  |
| Author:       | Guido van Rossum, Barry Warsaw, Nick Coghlan |
| Status:       | Active                                       |
| Type:         | Process                                      |
| Created:      | 05-Jul-2001                                  |
| Post-History: | 05-Jul-2001, 01-Aug-2013                     |

PEP 8给出了Python代码的编码约定，旨在提高代码的可读性，并使其在各种Python代码中保持一致。

## 2.代码布局

### 2.1.缩进

每个缩进级别使用4个空格。

#### 2.1.1.续行缩进的两种方案

- 隐式续行 (implicit line)，垂直对齐与圆括号，方括号或花括号
- 悬挂缩进（hanging indent），首行没有参数，后续行应该多缩进一级以便和正常的缩进区分

```python
# Correct:
# 垂直对齐（与打开定界符对齐）
foo = long_function_name(var_one, var_two,
                         var_three, var_four)

# 添加4个空格（额外的缩进级别）将参数与其他行区分
def long_function_name(
    	var_one, var_two, var_three,
    	var_four):
    print(var_one)

# 悬挂的缩进应添加一个级别
foo = long_function_name(
    var_one, var_two,
    var_three, var_four)

# Wrong:
# 不使用垂直对齐时，第一行的参数将被禁止
foo = long_function_name(var_one, var_two,
    var_three, var_four)

# 由于缩进不可区分，应使用额外的缩进级别
def long_function_name(
    var_one, var_two, var_three,
    var_four):
    print(var_one)
```

四空格的规则对于续行是可选的。

```python
# 悬挂缩进可以缩进到4个空格以外的其他位置
foo = long_function_name(
  var_one, var_two,
  var_three, var_four)
```

#### 2.1.2.if语句的特殊性

如果 if 语句的条件部分足够长，可以要求将其写成多行。

值得注意的是，两个字符的关键字（即if），一个空格和一个左括号的组合会产生一个自然的多行条件的后续行的4个空格缩进。 这可能与嵌套在if语句中的缩进代码集产生视觉冲突，该缩进代码集自然也将缩进4个空格。

对于如何从if语句内的嵌套套件中进一步从视觉上区分此类条件行，此PEP不采取明确立场。 

在这种情况下，可接受的选项包括但不限于：

```python
# 没有额外的缩进
if (this_is_one_thing and
    that_is_another_thing):
    do_something()

# 增加一个注释，在能提供语法高亮的编辑器中可以有一些区分
if (this_is_one_thing and
    that_is_another_thing):
    # Since both conditions are true, we can frobnicate.
    do_something()

# 在条件判断的语句添加额外的缩进
if (this_is_one_thing
        and that_is_another_thing):
    do_something()
```

（另请参见下面有关[是否在二进制运算符之前或之后中断](#2.4.换行符是在二进制运算符之前还是之后？)的讨论。）

#### 2.1.3.小括号，中括号以及大括号的续行方式

- 列表最后一行的第一个非空白字符下对齐

```python
my_list = [
    1, 2, 3,
    4, 5, 6,
    ]
result = some_function_that_takes_arguments(
    'a', 'b', 'c',
    'd', 'e', 'f',
    )
```

- 将其排列在开始多行构造的行的第一个字符下

```python
my_list = [
    1, 2, 3,
    4, 5, 6,
]
result = some_function_that_takes_arguments(
    'a', 'b', 'c',
    'd', 'e', 'f',
)
```

### 2.2.Tabs or Spaces?

**制表符应仅用于与已经用制表符缩进的代码保持一致。**

Python3不允许混合使用制表符和空格进行缩进。

> 针对pycharm，在如下位置可以设置是否使用制表符，默认是tab键作为4个空格的缩进
>
> File => Settings => Editor => Code Style => Python => Tabs and Indents

### 2.3.最大行长

行长限制为79个字符（文档字符串/注释限制为72个）。

- 较长的代码行应该优先使用[小括号，中括号以及大括号的续行方式](#2.1.3.小括号，中括号以及大括号的续行方式)，代替反斜杠换行。

- 在某些不能使用隐式续行方式的情况下，可以使用反斜杠换行。
  - 例如，长的多个with语句不能使用隐式续行：

```python
with open('/path/to/some/file/you/want/to/read') as file_1, \
     open('/path/to/some/file/being/written', 'w') as file_2:
    file_2.write(file_1.read())
```

### 2.4.换行符是在二进制运算符之前还是之后？

```python
# Correct:
# 易于将运算符与操作数进行匹配
income = (gross_wages
          + taxable_interest
          + (dividends - qualified_dividends)
          - ira_deduction
          - student_loan_interest)
```

### 2.5.空白行

- 顶级函数和类定义，前后用两个空白行隔开
- 类内部的方法定义，用一个空白行隔开
- 相关的功能组可以用额外的空行（谨慎使用）隔开
- 一堆相关的单行代码之间的空白行可以省略

### 2.6.源文件编码 

- Python源代码建议为UTF-8(在python 2也可以设置为ASCII)

- Python 2中使用ASCII，在Python3中使用UTF-8编码，都不应该添加编码声明
- 在标准库中，非默认编码仅应用于测试目的，或者在注释或文档字符串需要提及包含非ASCII字符的作者姓名时使用；否则，使用转义是在字符串文字中包含非ASCII数据的首选方法

对于Python 3.0及更高版本，标准库规定了以下策略：

- Python标准库中的所有标识符务必使用纯ASCII标识符，并且在可行的情况下应使用英文单词
- 字符串文字和注释也必须使用ASCII
- 例外是
  - 测试非ASCII功能的测试用例
  - 作者的姓名

### 2.7.imports 导入

#### 2.7.1.导入位置

导入总是位于文件的顶部，在模块注释和文档字符串之后，在模块的全局变量与常量之前。

#### 2.7.2.导入分组

导入应该按照以下顺序分组：

- 标准库导入
- 相关第三方库导入
- 本地应用/库特定导入

应该在每一组导入之间加入空行

#### 2.7.3.其他规则

- 导入通常应放在单独的行上

```python
# Correct:
import os
import sys

# Wrong:
import sys, os
```

- 从单个模块导入多个部分可以写在一行

```python
# Correct:
from subprocess import Popen, PIPE
```

- 推荐使用绝对导入

```python
import mypkg.sibling
from mypkg import sibling
from mypkg.sibling import example
```

除此以外，显式相对导入也可以接受，特别是在处理复杂的包结构，而绝对导入显得冗余的情况下

```python
from . import sibling
from .sibling import example
```

标准库代码应避免复杂的程序包布局，并始终使用绝对导入

#### 2.7.4.导入类

- 从包含类的模块中导入类时，通常可以这样拼写：

```python
from myclass import MyClass
from foo.bar.yourclass import YourClass
```

如果此拼写引起本地名称冲突，则应明确拼写它们：

```python
import myclass
import foo.bar.yourclass
```

并使用 `myclass.MyClass` 和 `foo.bar.yourclass.YourClass` 。 

#### 2.7.5.通配符

- 通配符导入应该避免，类似于 [原因](#ps: 不提倡通配符的原因)

```python
from module import *
```

### 2.8.模块级双下划线变量

模块级别双下划线变量比如 `__all__` ， `__author__` 应该放在模块文档字符串之后，但是需要放在 `import` 语句之前。但是存在一个例外，`from __future__ import` 应该放在双下划线变量之前。代码如下所示：

```python
"""This is the example module.

This module does stuff.
"""

from __future__ import barry_as_FLUFL

__all__ = ['a', 'b', 'c']
__version__ = '0.1'
__author__ = 'Cardinal Biggles'

import os
import sys
```

## 3.字符串引号

- 在Python中，单引号字符串和双引号字符串是相同的，选择一条规则并坚持下去。 
- 当字符串包含单引号或双引号字符时，请使用另一个以避免在字符串中使用反斜杠。

```python
a = 'a"b"c'
a = "a\"b\"c"
```

- 对于三引号字符串，请始终使用双引号字符以与PEP 257中的docstring约定一致。（说明如下）

### 3.1.文档字符串

应为所有公共模块，函数，类和方法编写文档字符串。 

对于非公共方法，文档字符串不是必需的，但是您应该有一条注释，描述该方法的作用。 

该注释应出现在 `def` 行之后。

- docstring是一个字符串文字，它作为模块，函数，类或方法定义中的第一条语句出现。这样的文档字符串成为该对象的 `__doc__` 特殊属性。
- 为了保持一致性，请始终在文档字符串前后使用 `"""` 三重双引号
- 如果在文档字符串中使用反斜杠，请使用 `r"""` 原始三重双引号
- 对于Unicode文档字符串，请使用 `u"""`  Unicode三引号字符串
- 文档字符串有两种形式：单行文档和多行文档字符串。

```python
"""Return a foobang

Optional plotz says to frobnicate the bizbaz first.
"""
```

#### 3.1.1.单行文档字符串

单行文档字符串，应写在一行内

```python
def kos_root():
    """Return the pathname of the KOS root directory.""" 
    global _kos_root
    if _kos_root: return _kos_root
    ...
```

- 说明：
- 文档字符串前后没有空行
- 尽量不要对参数和返回值进行说明，而是对函数功能进行说明

#### 3.1.2.多行文档字符串

- 结束多行文档字符串的 `"""` 应单独位于一行上

```python
def func():
	"""一个摘要行
	一个空行
	函数详细描述
	
	参数
	返回
	
	异常说明
	"""
```

- 模块的文档字符串应列出该模块导出的类，异常和函数（以及任何其他对象），并以每行的一行摘要显示
- 程序包的文档字符串（即程序包的 `__init__.py` 模块的文档字符串）也应列出该程序包导出的模块和子程序包
- 函数或方法的文档字符串应总结其行为并记录其参数，返回值，副作用，引发的异常以及何时可以调用它的限制。应该指出可选参数。应该记录关键字参数是否是接口的一部分。
- 类的文档字符串应总结其行为，并列出公共方法和实例变量。如果该类打算被子类化，并且具有用于子类的附加接口，则该接口应单独列出（在文档字符串中）。类构造函数的 `__init__` 方法应记录在文档字符串中。各个方法应由自己的文档字符串记录。
- 如果一个类继承了另一个类，并且其行为主要是从该类继承的，则其文档字符串应提及这一点并总结差异。使用动词 `override` 来表示子类方法代替了超类方法，并且不调用超类方法；使用动词 `extend` 表示子类方法（除了其自身的行为之外）还会调用超类方法。

### 3.2.处理文档字符串缩进

Docstring处理工具将从docstring的第二行和其他行中去除均匀的缩进量，等于第一行之后所有非空白行的最小缩进量。文档字符串第一行中的任何缩进（即，直到第一行）都可以忽略不计。保留文档字符串中后面几行的相对缩进。应从文档字符串的开头和结尾删除空白行。

由于代码比单词要精确得多，因此以下是该算法的实现：

```python
def trim(docstring):
    if not docstring:
        return ''
    # Convert tabs to spaces (following the normal Python rules)
    # and split into a list of lines:
    lines = docstring.expandtabs().splitlines()
    # Determine minimum indentation (first line doesn't count):
    indent = sys.maxint
    for line in lines[1:]:
        stripped = line.lstrip()
        if stripped:
            indent = min(indent, len(line) - len(stripped))
    # Remove indentation (first line is special):
    trimmed = [lines[0].strip()]
    if indent < sys.maxint:
        for line in lines[1:]:
            trimmed.append(line[indent:].rstrip())
    # Strip off trailing and leading blank lines:
    while trimmed and not trimmed[-1]:
        trimmed.pop()
    while trimmed and not trimmed[0]:
        trimmed.pop(0)
    # Return a single string:
    return '\n'.join(trimmed)
```

### 3.3. `__doc__` 属性

此示例中的文档字符串包含两个换行符，因此为3行。 第一行和最后一行是空白的：

```python
def foo():
    """
    This is the second line of the docstring.
    """
```

`__doc__` 显示：

```python
>>> print repr(foo.__doc__)
'\n    This is the second line of the docstring.\n    '
>>> foo.__doc__.splitlines()
['', '    This is the second line of the docstring.', '    ']
>>> trim(foo.__doc__)
'This is the second line of the docstring.'
```

修剪后，这些文档字符串是等效的：

```python
def foo():
    """A multi-line
    docstring.
    """

def bar():
    """
    A multi-line
    docstring.
    """
```

## 4.表达式和语句中的空格

### 4.1.不推荐的

在以下情况下，请避免使用多余的空格

- 在括号，中括号或大括号内

```python
# Correct:
spam(ham[1], {eggs: 2})

# Wrong:
spam( ham[ 1 ], { eggs: 2 } )
```

- 在尾随逗号和后面的右括号之间

```python
# Correct:
foo = (0,)

# Wrong:
bar = (0, )
```

- 在逗号，分号或冒号之前

```python
# Correct:
if x == 4: print x, y; x, y = y, x

# Wrong:
if x == 4 : print x , y ; x , y = y , x
```

- 在切片操作中，冒号在切片中就像二元运算符，在两边应该有相同数量的空格（当做优先级最低的操作符)
- 在拓展的切片操作中，所有冒号应该有相同数量的空格

```python
# Correct:
ham[1:9], ham[1:9:3], ham[:9:3], ham[1::3], ham[1:9:]
ham[lower:upper], ham[lower:upper:], ham[lower::step]
ham[lower+offset : upper+offset]
ham[: upper_fn(x) : step_fn(x)], ham[:: step_fn(x)]
ham[lower + offset : upper + offset]

# Wrong:
ham[lower + offset:upper + offset]
ham[1: 9], ham[1 :9], ham[1:9 :3]
ham[lower : : upper]
ham[ : upper]
```

- 紧贴在函数参数的左括号之前

```python
# Correct:
spam(1)

# Wrong:
spam (1)
```

- 紧贴索引或者切片的左括号之前

```python
# Correct:
dct['key'] = lst[index]

# Wrong:
dct ['key'] = lst [index]
```

- 在赋值操作符前使用多个空格去对齐其他语句

```python
# Correct:
x = 1
y = 2
long_variable = 3

# Wrong:
x             = 1
y             = 2
long_variable = 3
```

### 4.2.其他建议

- 避免在尾部添加空格
- 始终在二元运算符两边加一个空格：赋值（`=`），扩展赋值（ `+=` ，`-=` 等），比较（`==`，`<`，`>`，`!=`，`<>`，`<=`，`>=` ，`in`，`not in`，`is`，`is not`），布尔值（`and`，`or`，`not`）
- 如果使用具有不同优先级的运算符，请考虑在具有**最低优先级的运算符周围添加空格**（不要使用一个以上的空格，并且在二元运算符的两边使用相同数量的空格）

```python
# Correct:
i = i + 1
submitted += 1
x = x*2 - 1
hypot2 = x*x + y*y
c = (a+b) * (a-b)

# Wrong:
i=i+1
submitted +=1
x = x * 2 - 1
hypot2 = x * x + y * y
c = (a + b) * (a - b)
```

- 函数注解应使用冒号的常规规则，并且在使用 `->` 的时候要在两边加空格

```python
# Correct:
def munge(input: AnyStr): ...
def munge() -> PosInt: ...

# Wrong:
def munge(input:AnyStr): ...
def munge()->PosInt: ...
```

- 当用于指示关键字参数或默认参数值时，请不要在 `=` 号前后使用空格

```python
# Correct:
def complex(real, imag=0.0):
    return magic(r=real, i=imag)

# Wrong:
def complex(real, imag = 0.0):
    return magic(r = real, i = imag)
```

- 当将参数注释与默认值组合时，请在=符号周围使用空格（但仅适用于同时具有注释和默认值的参数）

```python
# Correct:
def munge(sep: AnyStr = None): ...
def munge(input: AnyStr, sep: AnyStr = None, limit=1000): ...

# Wrong:
def munge(input: AnyStr=None): ...
def munge(input: AnyStr, limit = 1000): ...
```

以下是一个针对函数参数注释的例子说明

```python
def test(a: int, b: int = 5, c=6) -> int:
	"""一个测试函数
	
	为了描述函数参数注释和默认值同时出现时，空格处理
	
	:param a: 参数1
	:param b: 默认值为5
	:param c: 默认值为6
	:return: 返回了一个什么
	"""
    
    if a == 1:
        return a+b+1
```

- 通常不建议使用复合语句（同一行上的多个语句）

```python
# Correct:
if foo == 'blah':
    do_blah_thing()
do_one()
do_two()
do_three()

Rather not:
# Wrong:
if foo == 'blah': do_blah_thing()
do_one(); do_two(); do_three()
```

- 虽然有时可以将 `if / for / while` 的小主体放在同一行上是可以的，但对于多子句的语句则永远不要这样做

```python
Rather not:
# Wrong:
if foo == 'blah': do_blah_thing()
for x in lst: total += x
while t < 10: t = delay()

Definitely not:
# Wrong:
if foo == 'blah': do_blah_thing()
else: do_non_blah_thing()

try: something()
finally: cleanup()

do_one(); do_two(); do_three(long, argument,
                             list, like, this)

if foo == 'blah': one(); two(); three()
```

## 5.何时使用行尾逗号

行尾逗号一般是可选的，除了表示单个元素的元组。为了清晰起见，建议使用括号将内容包起来

```python
# Correct:
FILES = ('setup.cfg',)

# Wrong:
FILES = 'setup.cfg',
```

当使用版本控制系统时，如果列表，参数或引入的项随着时间会逐渐增长，使用行尾逗号是很有用的。这种模式就是每行一个值，同时加上行尾的逗号，在内容的最后一行的下一行加上结束括号。但是如果如果内容都在同一行就没有必要加上行尾逗号了。

```python
# Correct:
FILES = [
    'setup.cfg',
    'tox.ini',
]
initialize(FILES,
           error=True,
           )
           
# Wrong:
FILES = ['setup.cfg', 'tox.ini',]
initialize(FILES, error=True,)
```

## 6.注释

- 和代码违背的注释还不如没有，始终保证注释随着代码更新而更新

- 注释应为完整的句子，第一个单词应大写，除非它是一个以小写字母开头的标识符

- 整体注释通常由一个或多个完整句子组成的段落组成，每个句子都以句点结尾

- 在多句注释中，除了最后一句之外，在一个句子结束后应该使用两个空格

- 来自非英语国家的 Python 程序员：请用英语写你的注释，除非你有 120%的把握知道代码不会被不懂你语言的人读到。


### 6.1.块注释

块注释一般情况是应用到注释后的代码上的，应该与代码保持相同层次的缩进。每一行的块注释应该以`#` 和一个空格开始（除非它是在注释内缩进对齐的）

块注释内的段落应该使用仅包含 `#` 的行隔开

```python
# class YourClass:
#     def __init__(self):
#         self.a = a
# 
#     def function_1(self):
#         print(self.a)
```

### 6.2.行内注释

- 行内注释是与语句在同一行上的注释，谨慎使用行内注释

- 行内注释应与该语句至少分隔两个空格
- 它们应以 `＃` 和单个空格开头

```python
# 不必要的写法
x = x + 1                 # Increment x

# 有时有用
x = x + 1                 # Compensate for border
```

## 7.命名规范

Python库的命名规范有一点糟糕，我们一直都没能保持一致。

下面是当前推荐的命名规范，新的模块和库（包括第三方框架）应该按照这些标准写。

但是已经存在的库如果之前采用其他风格，更推荐保持代码内部的一致性。

### 7.1.首要原则

作为公开API被用户可见的命名应该遵循**命名表达用途而不是实现**的原则。

### 7.2.命名风格

有很多不同的命名样式。 能够独立于它们的用途来识别正在使用的命名方式。

通常区分以下命名样式：

- `b` （单个小写字母）
- `B` （单个大写字母）
- `lowercase` （全小写）
- `lower_case_with_underscores` （下划线分割的小写）
- `UPPERCASE` （全大写）
- `UPPER_CASE_WITH_UNDERSCORES` （下划线分割的大写）
- `CapitalizedWords` （首字母大写的驼峰命名）
- `mixedCase` （驼峰命名）
- `Capitalized_Words_With_Underscores` （驼峰命名混合下划线分割）（不推荐！）

还有一种使用短的唯一前缀将相关名称组合在一起的样式，这在Python中使用不多，但是为了完整起见提到它。 

例如， `os.stat()` 函数返回一个元组，其元组传统上具有诸如 `st_mode`，`st_size`，`st_mtime` 等名称。 （这样做是为了强调与POSIX系统调用结构的字段的对应关系，这有助于程序员熟悉该结构）

同时，要注意以下划线开头和结尾的特殊形式（这种风格可以与其他规范联合使用）：

- `_single_leading_underscore` 单下划线开头， 弱的内部引用标志，比如 `from M import *` 不会导入下划线开头的对象
- `single_trailing_underscore_` 单下划线结束，使用这种方式避免和Python关键字冲突，比如：

```python
Tkinter.Toplevel(master, class_='ClassName')
```

- `__double_leading_underscore` 双下划线开头，此风格命名类属性会触发命名修饰（在类FooBar中，`__boo` 属性会修饰为`_FooBar__boo`）
- `__double_leading_and_trailing_underscore__` 双下划线开头和结束，在用户命名空间内的”魔术“对象或属性，比如 `__init__`， `__import__`， `__file__`。不要发明这样的名字，仅按照文档使用

### 7.3.命名约定

#### 7.3.1.避免使用的名称

切勿将字符 `l`，`O` 或 `I` 用作单个字符变量名称。

在某些字体中，这些字符与数字1和0没有区别。 当尝试使用 `l` 时，请改用 `L `。

#### 7.3.2.ASCII兼容性

在标准库中使用的标识符一定要是ASCII兼容的。

#### 7.3.3.软件包和模块名称

模块应使用简短的全小写名称。

如果可以提高模块的可读性，则可以在模块名称中使用下划线。 不鼓励使用下划线，但Python软件包也应使用短的全小写名称。

当用 C 或 C++ 编写的扩展模块具有随附的 Python 模块提供更高级别的接口（例如，面向对象的接口）时，C / C++模块应该有前导下划线（例如 `_socket` ）。

#### 7.3.4.类名

类名通常应使用首字母大写的驼峰命名。

使用 `_` 单下划线开头的类名为内部使用，默认不被导入的情况 `_InnerClass`

```python
from module_name import *
```

#### 7.3.5.类型变量名

在PEP 484中引入的类型变量的名称通常应使用简短的驼峰命名：T，AnyStr，Num。 建议将后缀 `_co` 或 `_contra `分别添加到用于声明协变或反变行为的变量中：

```python
from typing import TypeVar

VT_co = TypeVar('VT_co', covariant=True)
KT_contra = TypeVar('KT_contra', contravariant=True)
```

> [typing 模块详细说明](https://docs.python.org/zh-cn/3.9/library/typing.html)

#### 7.3.6.异常名称

因为异常是类，所以在这里适用类命名约定。

但是，应该在异常名称上使用后缀“ Error”（如果异常实际上是一个错误）。

#### 7.3.7.全局变量名

`__all__` 可以在模块级别暴露接口，形式如下：

```
__all__ = ["foo", "bar"]
```

Python 没有原生的可见性控制，其可见性的维护是靠一套需要大家自觉遵守的”约定“，比如，下划线开头的变量对外部不可见。`__all__` 是针对模块公开接口的一种约定，以提供了”白名单“的形式暴露接口。如果定义了 `__all__` ，其他文件中使用 `from xxx import *` 导入该文件时，只会导入 `__all__` 列出的成员，可以其他成员都被排除在外。

```python
# test1.py
__all__ = ['func']
def func():
	pass

# test2.py
import test1
__all__ = ['func2', 'test1']
def func2():
	pass
def func22():
	pass

# test3.py
import test2
from test2 import *
func2() # 能正常引用
test1.func() # 能正常引用
func22() # 不能正常引用
```

##### ps: 不提倡通配符的原因

python不提倡用 `from xxx import *` 这种写法。

如果一个模块 `xxx` 没有定义 `__all__`，执行 `from xxx import *` 时会将 `xxx` 中所有非下划线开头的成员（包括该模块import的其他模块成员）都会导入当前命名空间，这样就可能弄脏当前的命名空间。

显式声明了 `__all__`，`import *` 就只会导入 `__all__` 列出的成员，如果 `__all__ `定义有误，还会明确地抛出异常，方便检查错误。

- 按照 PEP8 建议的风格，`__all__` 应该写在所有 `import` 语句下面，函数、常量等成员定义的上面。
- 如果一个模块需要暴露的接口改动频繁，`__all__` 可以这样定义：

```python
__all__ = [
	"foo",
	"bar",
	"egg",
]
```

#### 7.3.8.函数和变量名

函数名称应小写，用下划线分隔单词，以提高可读性。

变量名与函数名遵循相同的约定。

混合大小写的情况，仅仅用于当前代码已经是混合大小写的情况下（比如, threading.py），为了维持后续兼容性才使用。

#### 7.3.9.函数和方法参数

始终将self作为实例方法的第一个参数。

始终对类方法的第一个参数使用cls。

如果函数参数的名称与保留关键字发生冲突，通常最好在末尾附加一个下划线，而不要使用缩写或拼写错误。 

因此，`class_` 优于 `clss` 。

使用同义词是一种更好地解决冲突的方法。

#### 7.3.10.方法名称和实例变量

使用函数命名规则：小写，必要时用下划线分隔单词，以提高可读性。

使用前导下划线：表示非公开的方法和实例变量。

##### ps: 下划线总结

|                      | 例        | 说明                                                         |
| -------------------- | --------- | ------------------------------------------------------------ |
| 单前导下划线         | `_var`    | 表示名称的命名约定仅供内部使用，通常不由Python解释器强制执行（通配符导入除外），并且仅作为对程序员的提示[说明1] |
| 单后缀下划线         | `var_`    | 按照惯例使用，以避免与Python关键字命名冲突                   |
| 双前导下划线         | `__var`   | 在类上下文中使用时触发名称修改，由Python解释器实施[说明2]    |
| 双重前导和后缀下划线 | `__var__` | 表示由Python语言定义的特殊方法，避免为自己的属性使用此命名方案 |
| 单下划线             | `_`       | 有时用作临时变量或无关紧要变量的名称（“无关紧要”）[说明3]    |

> - 说明1：
>
> ```python
> # This is my_module.py:
> def external_func():
>     return 23
> 
> def _internal_func():
>     return 42
> 
> # another.py
> >>> from my_module import * # 替换为 import my_module，则可以正常执行
> >>> external_func()
> 23
> >>> _internal_func()
> NameError: "name '_internal_func' is not defined"
> ```
>
> - 说明2：
>
> 如果类Foo有一个叫做`__a` 的属性，不能通过`Foo.__a` 访问到（事实上，用户依旧可以通过 `Foo._Foo__a` 访问到）。通常情况下，双下划线仅仅用于避免子类中的命名冲突。
>
> - 说明3:
>
> ```python
> n = 0
> for _ in a: # 通常用于需要迭代，但不需要明确迭代的每一个对象是什么的情况
>     n += 1 # 比如计数a中有多少成员
> ```

##### ps: 类变量与实例变量的区别

```python
class Student:
    grade = "二年级" # 类变量
    
    def __init__(self, name):
        self.name = name # 实例变量
        self.age = 8 # 实例变量
```

区别在于：

- 如果修改grade，那么所有实例化的对象的grade属性，全部更改
- 如果修改实例变量，那么只有当前实例化的对象的属性发生改变

#### 7.3.11.常量

常量通常在模块级别定义，并以所有大写字母书写，并用下划线分隔单词。 

```python
import os

MAX_OVERFLOW = 233
```

#### 7.3.12.继承设计

- 始终确定类的方法和实例变量（统称为“属性”）应该是公共的还是非公共的。 如有疑问，请选择非公开； 稍后将其公开比将公共属性设为不公开要容易得多。

- 公开属性是你期待和你定义的类无关的客户使用的，并且确保不会出现向后不兼容的问题。非公开属性是不打算给第三方使用的，你不能保证非公开属性不会改变甚至是删除。

- 在这里我们没有使用‘’私有“，因为Python中没有真正私有的属性（这样设计省去大量不必要的工作）

- 另一类属性是属于“子类API”（在其他语言中通常称为“protected”）的那些属性。 有些类被设计为用于继承，取决于类的行为进行拓展或修改。当设计这样的类时，需要谨慎地明确哪些属性时公开的，哪些是子类的一部分，以及哪些仅仅用在基类中。

考虑到以上，以下是Pythonic代码风格指南：

- 公共属性不应有前导下划线
- 如果您的公共属性名称与保留关键字冲突，请在属性名称后附加一个下划线。 这比缩写和简写更好。（但是，和这条规则冲突的是，`cls` 对于类中变量和参数来说是一个更好的变量名，特别是对于变量名的第一个参数）
- 对于单一的共有属性数据，最好直接对外暴露它的变量名，而不是复杂的访问/修改方法。请记住，如果需要把属性装饰为功能，python提供了 `@property` 装饰器（尽量避免）

- 如果你的类用于被子类继承，而且你有不想被子类使用的属性，考虑使用双下划线开头而且没有尾部下划线的方式 `__var` 命名。这样会触发Python的命名修饰算法，将类名修饰到属性名中。这样可以避免属性名冲突，以免子类中包含相同的属性名。

### 7.4.公共和内部接口

任何向后兼容性保证都仅适用于公共接口。 因此，重要的是用户能够清楚地区分公共接口和内部接口。

除非文档明确声明它们是临时接口或内部接口不受通常的向后兼容性保证，否则已说明文件的接口被视为公共接口。 所有未记录的接口都应假定为内部接口。

为了更好地支持自省，模块应使用 `__all__` 属性在其公共API中显式声明名称。 将 `__all__` 设置为空列表表示该模块没有公共API。

即使适当地设置了 `__all__` ，内部接口（包，模块，类，函数，属性或其他名称）仍应以单个下划线作为前缀。

如果任何包含名称空间（包，模块或类）的内部接口都被认为是内部接口，则该接口也被认为是内部接口。

导入的名称应始终被视为实现细节。除非其他模块是包含模块的API中明确记录的一部分，否则其他模块不得依赖对此类导入名称的间接访问，例如 `os.path` 或从子模块公开功能的软件包的 `__init__` 模块。

## 8.编程建议

- Python应该以不影响其他Python的实现（PyPy, Jython, IronPython, CPython, Psyco等）编写

比如，不要依赖CPython的高效实现 `a += b` 和 `a = a + b` 语句的原地字符串拼接。

在性能敏感的库中，应该使用`''.join()` 进行替代，这种方式保证拼接在多种实现下都是线性时间的。

```python
>>> a = "1234"
>>> b = "qwer"
>>> "".join((a, b)) # (a, b)作为一个tuple
'1234qwer'
```

- 与类似None的单例进行比较应该使用 `is` 或 `is not` ，不要使用 `==` `!=` 操作符

另外，如果你在写 `if x` 的时候，请注意你是否表达的意思是 `if x is not None` 。

举个例子，当测试一个默认值为 `None` 的变量或者参数是否被设置为其他值的时候。

这个其他值可能在上下文中能被认为是 `bool` 类型的 `False` 。

- 使用 `is not` 操作符而不是 `not ... is ` 。两种操作符是功能上等价的，前者可读性更好，是首选

```python
# Correct:
if foo is not None:

# Wrong:
if not foo is None:
```

- 当使用富比较方法实现排序操作时，最好是实现所有的六种操作，而不是依赖其他代码仅仅实现特定的操作

| `__eq__` | `__ne__` | `__lt__` | `__le__` | `__gt__` | `__ge__` |
| -------- | -------- | -------- | -------- | -------- | -------- |
| ==       | !=       | <        | <=       | >        | >=       |

`functools.total_ordering()` 装饰器就是用来简化这个处理的。

使用它来装饰一个类，你只需定义一个 `__eq__()` 方法， 外加其他方法(`__lt__`, `__le__`, `__gt__`, or `__ge__`)中的一个即可。 然后装饰器会自动为你填充其它比较方法。

- 始终使用 `def` 语句而不是绑定了 `lambda` 语句的赋值语句

```python
# Correct:
def f(x): return 2*x

# Wrong:
f = lambda x: 2*x
```

第一种形式表示结果函数对象的名称专门为 `f`，而不是通用的 `lambda` 。

通常，这对于回溯和字符串表示形式更为有用。

- 从 `Exception` 而不是 `BaseException` 派生异常。从 `BaseException` 的直接继承保留给捕获异常几乎总是做错事情的异常

设计异常层级应该基于代码可能需要捕获异常的不同之处，而不是基于异常的位置。

目的是程序性地回答“出了什么问题”，而不是仅仅声明“一个问题出现了”

- 合适地适用异常链。在Python 3中，`raise X from Y` 应该被用于表面显示的替代，而不会失去原始的追溯

慎重地替代一个内部异常（在Python3.3+中使用 `rasie X from None`）时，保证相关的详情被转移到新的异常中（比如转换 `KeyError` 到 `AttributeError` 时保留属性名，或者在新的异常消息中嵌入原始异常的文本）

- 引发异常时，请使用 `raise ValueError('message')`

使用括号的形式还意味着，当异常参数很长或包含字符串格式时，由于包含括号，不需要使用续行字符。

- 当捕获异常时，在可能的时候提到特定的异常而不是使用空的 `except: ` 语句

```Python
try:
    import platform_specific_module
except ImportError:
    platform_specific_module = None
```

空的 `except: ` 语句会捕获系统退出异常和键盘中断异常，这样让使用control-C终止程序变得更难，而且会掩饰其他问题。

如果你想捕获所有的程序异常，使用`except Exception:` （空的 `except: `等价于`except BaseException:`）

一个好的经验法则是将 `except` 子句的使用限制为两种情况：

1. 如果异常处理会打印或记录回溯信息；至少用户将会意识到错误发生了。
2. 如通过代码需要进行一些清理工作，但是使用 `with.try...finally` 让异常向上传播应该是处理这种情况更好的方法。

- 在将捕获的异常绑定到名称时，最好使用显式名称绑定语法：

```python
try:
    process_data()
except Exception as exc:
    raise DataProcessingFailedError(str(exc))
```

- 捕获操作系统错误时，最好使用Python 3.3中引入的显式异常层次结构，而不是对errno值的自省

- 此外，对于所有 `try / except` 子句，请将 `try` 子句限制为所需的绝对最小数量的代码，这避免了掩盖错误

```python
# Correct:
try:
    value = collection[key]
except KeyError:
    return key_not_found(key)
else:
    return handle_value(value)
   
# Wrong:
try:
    # 太宽泛
    return handle_value(collection[key])
except KeyError:
    # 还将捕获 handle_value() 引发的 KeyError
    return key_not_found(key)
```

- 当资源位于特定代码段的本地时，请使用with语句以确保在使用后迅速\可靠地对其进行清理。 `try / finally` 语句也是可以接受的
- 每当他们执行除获取和释放资源以外的其他操作时，都应通过单独的函数或方法来调用上下文管理器

```python
# Correct:
with conn.begin_transaction():
    do_stuff_in_transaction(conn)
    
# Wrong:
with conn:
    do_stuff_in_transaction(conn)
```

后面这个例子没有提供信息表明 `__enter__` 和 `__exit__` 方法在事务处理之后在做关闭连接之外的事情。在这种情况下明确很重要。

- 在返回语句中保持一致。 函数中的所有 `return` 语句应该返回表达式，或者都不返回。 如果任何 `return` 语句返回表达式，则不返回任何值的任何 `return `语句应将其显式声明为 `return None` ，并且在函数末尾（如果可访问）应存在一个显式 `return` 语句：

```python
# Correct:
def foo(x):
    if x >= 0:
        return math.sqrt(x)
    else:
        return None

def bar(x):
    if x < 0:
        return None
    return math.sqrt(x)

# Wrong:
def foo(x):
    if x >= 0:
        return math.sqrt(x)

def bar(x):
    if x < 0:
        return
    return math.sqrt(x)
```

- 使用字符串方法代替字符串模块，字符串方法总是更快

- 使用 `''.startswith()` 和 `''.endswith()` 而不是字符串切片来检查前缀或后缀

`startWith()` 和 `endsWith()` 方法更清晰，更不容易出错。

```python
# Correct:
if foo.startswith('bar'):

# Wrong:
if foo[:3] == 'bar':
```

- 对象类型比较应始终使用 `isinstance()` 而不是直接比较类型

```python
# Correct:
if isinstance(obj, int):

# Wrong:
if type(obj) is type(1):
```

- 对于序列 (strings, lists, tuples)，请使用空序列为 `False`

```python
# Correct:
if not seq:
if seq:

# Wrong:
if len(seq):
if not len(seq):
```

- 不要写出依赖显式尾部空格的字符串，这种尾部的空格在视觉上是无法区分的

- 不要使用 `==` 将布尔值与 `True` 或 `False` 进行比较

```python
# Correct:
if greeting:

# Wrong:
if greeting == True:
    
# Worse:
if greeting is True:
```

- 不鼓励在 `try ... finally` 套件中使用流控制语句 `return`/`break`/`continue`。最后，在该方法中，流控制语句将跳到 `finally` 套件之外，这样的语句将隐式取消通过 `finally` 套件传播的任何活动异常

```python
# Wrong:
def foo():
    try:
        1 / 0
    finally:
        return 42
```

### 8.1.功能注释

注释样例

```python
"""这是一个什么模块。

该模块是干什么的。
"""

from __future__ import barry_as_FLUFL

__all__ = ['a', 'b', 'c']
__version__ = '0.1'
__author__ = 'iie'

import os
import sys

import requests

from example_moudle import example

MAX_OVERFLOW = 233


class ExampleClass:
    example_var: int

    def __init__(self, example_var1):
        self.example_var1 = example_var1


def function_1(val_a: int, val_b: str = "test") -> str:
    """函数说明摘要

    函数具体说明

    :param val_a: 参数a是干什么的
    :param val_b: 参数b是干什么的
    :return: void
    """
    val_c = val_a * val_b
    return val_c


def a():
    pass


def b():
    pass


def c():
    pass


if __name__ == 'main':
    function_1(1, "test")
```

- 对于想要不同使用功能注释的代码，建议添加以下形式的注释：

```python
# type: ignore # 这告诉类型检查器忽略所有注释
```

### 8.2.变量注释

PEP 526引入了变量注释。 针对它们的样式建议与上述函数注释中的样式建议类似：

- 模块级变量，类和实例变量以及局部变量的注释应在冒号后面有一个空格

- 冒号前不应有空格

- 赋值的等号在两边应该各有一个空格

```python
# Correct:
code: int
    
class Point:
    coords: Tuple[int, int]
    label: str = '<unknown>'

# Wrong:
code:int  # No space after colon
code : int  # Space before colon
    
class Test:
    result: int=0  # No spaces around equality sign
```

## Reference

- [PEP 0 -- Index of Python Enhancement Proposals (PEPs)](https://www.python.org/dev/peps/)
- [PEP 8 -- Style Guide for Python Code](https://www.python.org/dev/peps/pep-0008/)
- [PEP 257 -- Docstring Conventions](https://www.python.org/dev/peps/pep-0257/)
- [PEP 484 -- Type Hints](https://www.python.org/dev/peps/pep-0484/)
- [PEP 526 -- Syntax for Variable Annotations](https://www.python.org/dev/peps/pep-0526/)

