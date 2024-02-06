# Python 内存优化

<show-structure depth="2"/>

当项目变得越来越大时，有效地管理计算资源是一个不可避免的需求。Python 与 C 或 C++ 等低级语言相比，似乎不够节省内存。但是其实有许多方法可以显著优化 Python 程序的内存使用，这些方法可能在实际应用中并没有人注意，所以本文将重点介绍 Python 的内置机制，掌握它们将大大提高 Python 编程技能。

## 1. 查看对象占用的内存 {collapsible="true" default-state="expanded"}

有几种方法可以在 Python 中获取对象的大小。可以使用 `sys.getsizeof()` 来获取对象的确切大小，使用 `objgraph.show_refs()` 来可视化对象的结构，或者使用 `psutil.Process().memory_info().rss` 获取当前分配的所有内存，这样我们才能根据对象的内存占用来查看实际的优化结果。

```Python
import sys
import objgraph
import numpy as np
import pandas as pd
from sklearn.datasets import load_boston


# 使用 `sys.getsizeof` 函数获取对象占用内存大小
ob = np.ones((1024, 1024, 1024, 3), dtype=np.uint8)
print("size: ", sys.getsizeof(ob) / (1024 * 1024))
# size: 3072.0001373291016

print("size: "， psutil.Process().memory_info().rss / (1024 * 1024))
# size: 3234.19140625

# 可视化对象结构
objgraph.show_refs([ob], filename='sample-graph.png')

# Pandas DataFrame 自带内存查看
data = load_boston()
df = pd.DataFrame(data['data'])
print(df.info(verbose=False, memory_usage='deep'))
# <class 'pandas.core.frame.DataFrame'>
# RangeIndex: 506 entries, 0 to 505
# Columns: 13 entries, 0 to 12
# dtypes: float64(13)
# memory usage: 51.5 KB

print(df[0].memory_usage(deep=True))
# parts that construct the pd.Series
#  4176
```

## 2. `__slots__` {collapsible="true" default-state="expanded"}

Python 作为一种动态类型语言，在面向对象方面具有更大的灵活性。在运行时可以向 Python 类添加额外属性和方法的能力。例如，下面的代码定义了一个名为 `Author` 的类。最初它有两个属性 `name` 和 `age`。但是我们以后可以很容易地添加一个额外的 `job`:

```Python
 class Author:
    def __init__(self, name, age):
        self.name = name
        self.age = age
 
 
 me = Author('Yang Zhou', 30)
 me.job = 'Software Engineer'
 print(me.job)
```

但是这种灵活性在底层浪费了更多内存，因为Python中每个类的实例都维护一个特殊的字典(`__dict__`) 来存储实例变量，字典的底层基于哈希表的实现所以消耗了大量的内存。 在大多数情况下，我们不需要在运行时更改实例的变量或方法，并且 `__dict__` 不会（也不应该）在类定义后更改。所以 Python 为此提供了一个属性: `__slots__`，它通过指定类的所有有效属性的名称来作为白名单。


```Python
class Author:
    __slots__ = ('name', 'age')
 
    def __init__(self, name, age):
        self.name = name
        self.age = age
 
 
 me = Author('Yang Zhou', 30)
 me.job = 'Software Engineer'
 print(me.job)
 # AttributeError: 'Author' object has no attribute 'job'
```

> 白名单只定义了两个有效的属性 `name` 和 age`，由于属性是固定的，Python 不需要为它维护字典，只为 `__slots__` 中定义的属性分配必要的内存空间。

下面我们做一个简单的比较：

<tabs>
<tab title="未使用slots">
<code-block lang="python">
<![CDATA[
import sys

class Author:
    def __init__(self, name, age):
        self.name = name
        self.age = age


me = Author('Yang', 30)
memory_me = sys.getsizeof(me) + sys.getsizeof(me.__dict__)
print(memory_me)  # 152
]]>
</code-block>
</tab>
<tab title="使用slots">
<code-block lang="python">
<![CDATA[
import sys

class Author:
    __slots__ = ('name', 'age')

    def __init__(self, name, age):
        self.name = name
        self.age = age

me = Author('Yang', 30)
# 使用 `__slots__` 之后，就不再有 `__dict__` 属性了
memory_me = sys.getsizeof(me)
print(memory_me)  # 48
]]>
</code-block>
</tab>
</tabs>

## 3. 生成器(Generators) {collapsible="true" default-state="expanded"}

生成器是 Python 中列表的惰性求值版本，每当调用 `next()` 方法时生成一个项，而不是一次计算所有项，所以它们在处理大型数据集时非常节省内存。

```Python
def number_generator():
    for i in range(100):
        yield i

numbers = number_generator()
print(numbers)
# <generator object number_generator at 0x104a57e40>

print(next(numbers))  # 0
print(next(numbers))  # 1
```

上面的代码显示了一个编写和使用生成器的基本示例。关键字 `yield` 是生成器定义的核心，应用它意味着只有在调用 `next()` 方法时才会产生项。

接下来让我们比较一个生成器和一个列表，看看哪个更节省内存:

<tabs>
<tab title="列表">
<code-block lang="python">
<![CDATA[
import sys

# 如果使用 list(range(100)) 可能占用的内存更小点
# 因为减少了中间变量 i 的产生
numbers = [i for i in range(100)]
print(sys.getsizeof(numbers))  # 920
]]>
</code-block>
</tab>
<tab title="生成器">
<code-block lang="python">
<![CDATA[
import sys

def number_generator():
    for i in range(100):
        yield i

numbers = number_generator()
print(sys.getsizeof(numbers))  # 112
]]>
</code-block>
</tab>
</tabs>

可以看到使用生成器可以显著节省内存使用。如果我们将列表推导式的方括号转换成圆括号，它将转换为**生成器表达式**。这是在 Python 中定义生成器的更简单的方法:

```Python
import sys
 
numbers = [i for i in range(100)]
numbers_generator = (i for i in range(100))
 
print(sys.getsizeof(numbers_generator))  # 112
print(sys.getsizeof(numbers))  # 920
```

## 4. 利用内存映射文件支持大文件处理 {collapsible="true" default-state="expanded"}

MMAP
: 内存映射文件 I/O，简称 “mmap”，是一种操作系统级优化。简单地说，当使用 mmap 技术对文件进行内存映射时，它直接在当前进程的虚拟内存空间中创建文件的映射，而不是将整个文件加载到内存中，这节省了大量内存。Python 已经提供了用于使用此技术的内置模块，因此我们可以轻松地利用它，而无需考虑操作系统级别的实现。

以下是如何在 Python 中使用 mmap 进行文件处理:

```Python
import mmap
 
 
with open('test.txt', "rb") as f:
    # memory-map 读取打开, 0 表示整个文件
    with mmap.mmap(f.fileno(), 0) as mm:
        # 我们可以使用常规的方法读取文件内容
        print(mm.read())
        # 通过切片方式读取内容
        snippet = mm[0:10]
        print(snippet.decode('utf-8'))
```

Python 使内存映射文件 I/O 技术的使用变得方便，我们所需要做的只是应用 `mmap.mmap()` 方法，然后使用标准文件方法甚至切片方式处理打开的对象。

## 5. 选择适当的数据类型 {collapsible="true" default-state="expanded"}

开发人员应仔细而精确地选择数据类型。因为在某些情况下，使用一种数据类型比使用另一种数据类型更节省内存。

### 5.1 元组比列表更节省内存

元组是不可变的(在创建后不能更改)，它允许 Python 在内存分配方面进行优化。列表是可变的，因此需要额外的空间来容纳潜在的修改。

```Python
import sys
 
my_tuple = (1, 2, 3, 4, 5)
my_list = [1, 2, 3, 4, 5]
 
print(sys.getsizeof(my_tuple))  # 80
print(sys.getsizeof(my_list))  # 120
```

元组比列表使用更少的内存，如果创建后不需要更改数据，我们应该选择元组而不是列表。

### 5.2 数组比列表更节省内存

Python 中的数组要求元素具有相同的数据类型(例如，所有整数或所有浮点数)，但列表可以存储不同类型的对象，这不可避免地需要更多的内存。如果列表的元素都是相同类型，使用数组会更节省内存:

```Python
import sys
import array
 
my_lst = [i for i in range(1000)]
my_arr = array.array('i', [i for i in range(1000)])
 
print(sys.getsizeof(my_lst))  # 8856
print(sys.getsizeof(my_arr))  # 4064s
```

## 6. 字符串驻留 {collapsible="true" default-state="expanded"}

```Python
a = 'Y' * 4096
b = 'Y' * 4096
print(a is b)  # True

c = 'Y' * 4097
d = 'Y' * 4097
print(c is d)  # False
```

上述代码中，为什么 `a is b` 是真，而 `c is d` 是假呢？这在 Python 中被称作字符串驻留（string interning），如果有几个值相同的小字符串，它们将被 Python 隐式地存储并在内存中并引用相同的对象。定义小字符串阈值数字是 4096。

由于 c 和 d 的长度为4097，因此它们是内存中的两个对象而不是一个对象，不再隐式驻留字符串。所以当执行 `c = d` 时，我们得到一个 False。

驻留是一种优化内存使用的强大技术。如果我们想要显式地使用它可以使用 `sys.intern()` 方法:

```Python
import sys

c = sys.intern('Y' * 4097)
d = sys.intern('Y' * 4097)
print(c is d)  # False
```




<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/0Yuc9MyaWXFO6Px55Rlp8w">提高代码效率的6个Python内存优化技巧</a>
</category>
<category ref="ref_github">
</category>
<category ref="ref_issues">
</category>
<category ref="ref_hf">
</category>
<category ref="ref_ms">
</category>
</seealso>