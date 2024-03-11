# Dataclasses 教程

<show-structure depth="3"/>

在 Python 3.7（[PEP 557](https://peps.python.org/pep-0557)）后引入一个新功能是装饰器 `@dataclass`，它通过自动生成特殊方法（如 `__init__()` 和 `__repr__()` 等魔术方法 ）来简化数据类的创建。

数据类和普通类一样，但在存储数据以及将相关的数据组织在一起，数据类以其结构简单、字段清晰深受广大 Pythoner 的喜爱。当然，Python 还有比 `dataclasses` 更强大的第三方库，例如 Pydantic，但 `dataclass` 优势在于使用方式简单、高效，因为它是内置库的原因，能让项目变得更加简洁，减少依赖。

在实际应用中，特别是在数据处理和对象建模方面，使用 `@dataclass` 装饰器能够极大地提升代码的清晰度，减少冗余的样板代码。深入理解 `dataclass` 的各项特性将帮助我们更灵活地运用这一功能，从而提高代码的质量和开发效率。

## 1. 创建数据类

除了传统的 `tuple`、`list` 以及 `dict` 来存储数据外，我们还可以借助以下这些方式来创建数据类，相对及 dataclass，但在其使用上依旧存在劣势。

### 1.1 namedtuple

`namedtuple` 是内置库 `collections` 中一个功能，也可以用来创建数据类。

```Python
from collections import namedtuple

Player = namedtuple('Player', ['name', 'number', 'position', 'age', 'grade'])
jordan = Player('James Harden', 1, 'PG', 1, 'S+')
print(jordan)  
# Player(name='James Harden', number=1, position='PG', age=1, grade='S+')
print(jordan.name)  # James Harden
```

使用 `namedtuple` 可以使用 `.` 获取数据的属性，可以明确数据的属性名称，但是仍然存在一些问题，比如：
- 数据无法修改
- 无法自定义数据比较，没有默认值，没有函数支持。

### 1.2 typing.NamedTuple


`typing.NamedTuple` 是 `typing` 库提供的一个数据类型，可以用来创建数据类。通过类型提示让代码更具可读性和可维护性，但同样和 `namedtuple` 一样也存在一些问题，如不可变性等。

```Python
from typing import NamedTuple

class Player(NamedTuple):
    name: str
    number: int
    position: str
    age: int
    grade: str

jordan = Player('James Harden', 1, 'PG', 1, 'S+')
print(jordan)
# Player(name='James Harden', number=1, position='PG', age=1, grade='S+')
print(jordan.name)  # James Harden
```

### 1.3 typing.TypedDict

`typing.TypedDict` 是 `typing` 库提供的另外一个数据类型，可以用来创建数据类。它可以更多的利用类型检查来帮助减少错误发生的可能，同时也能帮助其他开发者理解复杂数据结构。

```Python

from typing import TypedDict

class Player(TypedDict):
    name: str
    number: int
    position: str
    age: int

jordan: Player = {'name': 'James Harden', 'number': 1, 'position': 'PG', 'age': 34}
print(jordan['position'])  # Output: PG
```

总的来说，对于一些简单的场景，上述方法还是有一席用武之地的，但是在一些更复杂的场景中，这三者就显得没那么好用了，比如：数据比较，设置默认值等。

通常我们一般会通过自定义类来实现复杂场景的数据类，然而即使这样定义的类还是有以下问题：不支持比较、对于对象的描述不太友好。

```Python
class Player:
    def __init__(self, name, number, position, age, grade):
        self.name = name
        self.number = number
        self.position = position
        self.age = age
        self.grade = grade


harden = Player('James Harden', 1, 'PG', 34, 'S+')
bryant = Player(name='Kobe Bryant', number=24, position='PG', age=41, grade='S+')
print(harden.name)  # James Harden
print(bryant.name)  # Kobe Bryant
print(harden)  # <__main__.Player object at 0x000002431AFC6E00>

# print(harden < bryant)  # 报错
```

为了解决上面两个问题，可以通过实现 `__repr__` 方法来自定义描述， 实现 `__gt__` 方法来支持比较的功能。

```Python
class Player:
    def __init__(self, name, number, position, age, grade):
        self.name = name
        self.number = number
        self.position = position
        self.age = age
        self.grade = grade
        
    def __repr__(self):
        return f'Player: {self.name} : {self.age}'

    def __gt__(self, other):
        return self.age > other.age

    def __eq__(self, other):
        return self.age == other.age


harden = Player('James Harden', 1, 'PG', 34, 'S+')
bryant = Player(name='Kobe Bryant', number=24, position='PG', age=41, grade='S+')
print(harden.name)  # James Harden
print(bryant.name)  # Kobe Bryant
print(harden)  # <__main__.Player object at 0x000002431AFC6E00>
print(harden < bryant)  # True
```

这里缺点也非常明显，我们经常需要添加构造函数、表示方法、比较函数等。这些函数很麻烦，而这正是语言应该透明地处理的。

### 1.4 dataclass

```Python
from dataclasses import dataclass

@dataclass(order=True)
class Player:
    name: str
    number: int
    position: str
    grade: str
    age: int = 18  # 默认值，跟函数定义一样，需要往后放


harden = Player('James Harden', 1, 'PG', 'S+', 34)
bryant = Player(name='Kobe Bryant', number=24, position='PG', grade='S+', age=41)

print(harden.name)  # James Harden
print(bryant.name)  # Kobe Bryant
print(harden)  
## Player(name='James Harden', number=1, position='PG', grade='S+', age=34)

# 比较, 默认按照属性定义的顺序比较的
print(harden < bryant)  # True
```

上述代码可以明显看出，`dataclass` 具有明显的优势，它能更精确地指定每个成员变量的类型，同时提供字段名的检查，大大降低了出错的可能性。相对于传统的类定义，使用 `dataclass` 更加简洁，省去了冗长的 `__init__` 方法等，只需直接列出成员变量即可。


我们也可以通过实现 `__eq__`、`__lt__` 这些魔术方法，来达到自定义的精准控制:

```Python
from dataclasses import dataclass


@dataclass
class Player:
    name: str
    number: int
    position: str
    grade: str
    age: int = 18

    def __eq__(self, other):
        return self.age == other.age  # 只比较age

    def __lt__(self, other):
        return self.age < other.age  # 只比较 age


# 示例使用
harden = Player('James Harden', 1, 'PG', 'S+', 34)
bryant = Player(name='Kobe Bryant', number=24, position='PG', grade='S+', age=41)
result = harden < bryant  # 按照 age 进行比较
print(result)  # 输出 True，因为 34 < 41
print(harden.name)  # James Harden
print(bryant.name)  # Kobe Bryant
print(bryant == harden)  # False
```

当然，如果都要自己重载实现，那 `dataclass` 看起来也是不太聪明的样子。不想全部的字段都参与，`dataclass` 也是提供了 `field` 对象用于简化。


## 2. dataclass 使用

通过上面的示例，我们了解到，`dataclass` 帮我们模板化的实现了一批魔术方法，而我们要做的仅仅是根据需求调整`dataclass` 的参数或者在适当的时候进行部分重载以满足我们的实际场景。

### 2.1 类型提示和默认值

与函数参数规则一样，具有默认值的属性必须出现在没有默认值的属性之后。

```Python
from typing import Any
from dataclasses import dataclass


@dataclass
class Player:
    name: str
    number: int
    position: str
    grade: str
    age: int = 18
    team: Any = "nba"


# 示例使用
harden = Player('James Harden', 1, 'PG', 'S+', 34)
bryant = Player(name='Kobe Bryant', number=24, position='PG', grade='S+')

print(harden.name)  # James Harden
print(bryant.name)  # Kobe Bryant
print(bryant.age)   # 18
print(bryant.team)  # nba
```


### 2.2 数据嵌套


数据类可以嵌套为其他数据类的字段，可以简单创建一个有 2 个队员的球队。

```Python
from typing import List
from dataclasses import dataclass


@dataclass
class Player:
    name: str
    number: int
    position: str
    grade: str
    age: int = 18


@dataclass
class Team:
    name: str
    players: List[Player]


# 示例使用
harden = Player('James Harden', 1, 'PG', 'S+', 34)
leonard = Player(name='Kawhi Leonard', number=2, position='SF', grade='S+')

clippers = Team("clippers", [harden, leonard])
print(harden.name)  # James Harden
print(leonard.name)  # Kawhi Leonard
print(leonard.age)  # 18

print(clippers)  
# Team(name='clippers', players=[
    Player(name='James Harden', number=1, position='PG', grade='S+', age=34), 
    Player(name='Kawhi Leonard', number=2, position='SF', grade='S+', age=18)
    ])
```


> 使用过 Pydantic 的同学都知道，Pydantic 是会直接实例化对象的子数据类的，但是这一点在 dataclass 中并没有，我们得通过先实例话子数据类，再实例化父数据类，这一点确实不太友好，后面我们说明下如何解决这个问题。
> 
{style="warning"}


### 2.3 继承

通过 dataclass 定义的数据类也是可以被其他类所继承的，类中定义的字段的顺序为先父类，再当前类。

```Python
from dataclasses import dataclass, field


@dataclass(order=True)
class Person:
    name: str
    age: int


@dataclass(order=True)
class Player(Person):
    number: int
    position: str
    grade: str
    team: str = "nba"


# 示例使用
harden = Player(name='James Harden', age=34, number=1, position='PG', grade='S+')
bryant = Player(name='Kobe Bryant', age=41, number=24, position='PG', grade='S+')

print(harden.name)  # James Harden
print(bryant.name)  # Kobe Bryant
print(bryant.age)  # 41
print(bryant.team)  # nba

# 使用 order 参数，可以比较对象的大小（用于排序）
print(harden < bryant)  # True
```


### 2.4 不定长参数

数据类一般建议是显示声明属性，如果你想额外接收一些参数，可能以下方法可以满足你。

```Python
from dataclasses import dataclass, field


@dataclass
class Player:
    name: str
    number: int
    position: str
    grade: str
    age: int = 18
    args: tuple = ()
    kwargs: dict = field(default_factory=dict)


# 示例使用
harden = Player('James Harden', 1, 'PG', 'S+', 34)
bryant = Player(
    name='Kobe Bryant', number=24, position='PG', grade='S+', 
    args=(1, 2), kwargs={"hello": "world"}
    )

print(bryant)
```


### 2.5 field 对象

如果数据类的属性是不可变类型，可以直接为其赋默认值，`dataclass` 默认阻止使用可变数据做默认值。然而当属性是不可变类型时，直接给定默认值时会报错。

#### I) 基础使用

```Python
from dataclasses import dataclass
from typing import List


@dataclass
class Player:
    name: str
    number: int
    position: str
    grade: str
    age: int = 18


# 示例使用
harden = Player('James Harden', 1, 'PG', 'S+', 34)
leonard = Player(name='Kawhi Leonard', number=2, position='SF', grade='S+')


@dataclass
class Team:
    name: str
    players: List[Player] = [leonard]  # 这里会报错
```

这个时候就可以借助 `field` 函数中的 `default_factory` 参数来完成：

<tabs>
<tab title="default_factory">
<code-block lang="python">
<![CDATA[
from typing import List
from dataclasses import dataclass, field, fields

@dataclass
    class Player:
    name: str
    number: int
    position: str
    grade: str
    age: int = 18

harden = Player('James Harden', 1, 'PG', 'S+', 34)
leonard = Player(name='Kawhi Leonard', number=2, position='SF', grade='S+')


@dataclass
class Team:
    name: str = field(metadata={'unit': 'name'})
    players: List[Player] = field(
        default_factory=lambda: [leonard],
        metadata={'unit': 'players'}
    )

clippers = Team("clippers", [harden])
clippers1 = Team("clippers")
print(harden.name)
print(leonard.name)
print(leonard.age)
print(clippers.players)
print(clippers1.players)
print(fields(clippers)[1].metadata)
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
James Harden
Kawhi Leonard
18
[Player(name='James Harden', number=1, position='PG', grade='S+', age=34)]
[Player(name='Kawhi Leonard', number=2, position='SF', grade='S+', age=18)]
{'unit': 'players'}
]]>
</code-block>
</tab>
</tabs>


除此之外，`field` 函数还有以下参数:
- `default`: 指定字段默认值
- `default_factory`: 与 default 相似，但是是一个可调用对象，用于提供默认值。每次创建实例时，都会重新调用工厂函数以获取新的默认值。
- `init`: 控制是否在 `__init__` 方法中包含该字段，默认为 True
- `repr`: 是否在 `__repr__()` 方法中使用字段，默认为 True
- `compare`: 是否在比较对象时, 包括该字段，默认为 True
- `hash`: 计算hash时, 是否包括字段，默认为 True
- `metadata`: 包含字段信息的映射


上面示例中，如果不想 `name` 加入比较，则可以设置：`name: str = field(compare=False)`。

#### II) 元数据 metadata

基于 `field` 可以基于元数据进行数据校验：

```Python
from dataclasses import dataclass, field, fields
from datetime import datetime


class ValidationError(Exception):
    def __init__(self, field_name, condition, actual_value):
        self.field_name = field_name
        self.condition = condition
        self.actual_value = actual_value
        super().__init__(
            f"{field_name} validation failed: {condition} (Actual value: {actual_value})"
            )


class Color:
    RED = '\033[91m'
    END = '\033[0m'


@dataclass
class Player:
    name: str = field(default="", metadata={"validation": [lambda x: len(x) == 0]})
    number: int = field(default=0, metadata={"validation": [lambda x: not 0 < x <= 100]})
    position: str = field(default="", metadata={"validation": [lambda x: len(x) == 0]})
    grade: str = field(default="", metadata={"validation": [lambda x: x in {'S+', 'S', 'A', 'B', 'C'}]})
    age: int = field(default=0, metadata={"validation": [lambda x: not 0 < x <= 150]})
    foundation_date: datetime = field(default_factory=datetime.now)

    def validation(self):
        for field_ in fields(self):
            validations = field_.metadata.get("validation", [])
            for validation in validations:
                if validation(getattr(self, field_.name)):
                    raise ValidationError(field_.name, str(validation), getattr(self, field_.name))


harden = Player(name='James Harden', number=13, position='PG', grade='S+', age=32)
bryant = Player(name='Kobe Bryant', number=24, position='SG', grade='S', age=41)

# 无效的数据，引发异常
try:
    harden.validation()
except ValidationError as e:
    print(f"{Color.RED}{e}{Color.END}")

try:
    bryant.validation()
except ValidationError as e:
    print(f"{Color.RED}{e}{Color.END}")
```

#### III) 自定义属性

通过对 `field()` 对象的剖析，我们可以指定属性：是否参与比较，是否参与 hash 计算等等。不过我们知道默认的比较顺序，我们也可以通过增加属性以实现按需比较的功能。而这个用于比较的属性位于数据类的第一个属性，并可以借助 `__post_init__` 魔法函数实现灵活赋值。

```Python
from dataclasses import dataclass, field


@dataclass(order=True)
class Player:
     # 添加一个 sort_index 字段，并设置为不在 __init__ 方法中初始化
    sort_index: tuple = field(init=False)
    name: str
    number: int
    position: str
    grade: str
    age: int = 18

    def __post_init__(self):
        self.sort_index = (self.age, self.grade)  # 在 __post_init__ 方法中计算 sort_index


# 示例使用
harden = Player('James Harden', 1, 'PG', 'S+', 34)
bryant = Player(name='Kobe Bryant', number=24, position='PG', grade='S+', age=41)

result = harden < bryant  # 按照 age 进行比较
print(result)  # 输出 True，因为 34 < 41
print(harden.name)  # James Harden
print(bryant.name)  # Kobe Bryant
print(bryant == harden)  # False
```


### 2.6 不可变数据类

使用 `dataclass` 实现的数据类默认是可变的，要使数据类不可变，需要在创建类时设置 `frozen=True`。

```Python
from dataclasses import dataclass, field


@dataclass(order=True, frozen=True)
class Player:
    name: str
    number: int
    position: str
    grade: str
    age: int = 18
```

### 2.7 实现数据类去重

当 `unsafe_hash=True` 时，可以实现数据类的去重，参与的字段同样可由 `field` 对象控制。

```Python
from dataclasses import dataclass, field


@dataclass(order=True, unsafe_hash=True)
class Player:
    name: str
    number: int
    position: str = field(hash=False)  # 不参与hash
    grade: str
    age: int = 18


# 示例使用
harden = Player('James Harden', 1, 'PG', 'S+', 34)
harden2 = Player('James Harden', 1, 'PG', 'S+', 34)
harden3 = Player('James Harden', 1, 'SG', 'S+', 34)

print({harden, harden2})
print({harden, harden3})
```

### 2.8 数据类转换为元组或字典


`dataclass` 提供了 `asdict` 和 `astuple` 方法来将数据类转换为元组或字典。


<tabs>
<tab title="转换元组或字典">
<code-block lang="python">
<![CDATA[
from dataclasses import dataclass, field, asdict, astuple


@dataclass
class Player:
name: str
number: int
position: str
grade: str
age: int = 18
args: tuple = ()
kwargs: dict = field(default_factory=dict)


# 示例使用
harden = Player('James Harden', 1, 'PG', 'S+', 34)
bryant = Player(name='Kobe Bryant', number=24, position='PG', grade='S+', args=(1, 2), kwargs={"hello": "world"})

alist = [harden, bryant]

print(sorted(alist, key=lambda x: x.age))
print(asdict(bryant))
print(astuple(harden))
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
[
    Player(name='Kobe Bryant', number=24, position='PG', grade='S+', age=18, args=(1, 2), kwargs={'hello': 'world'}), 
    Player(name='James Harden', number=1, position='PG', grade='S+', age=34, args=(), kwargs={})
]
{'name': 'Kobe Bryant', 'number': 24, 'position': 'PG', 'grade': 'S+', 'age': 18, 'args': (1, 2), 'kwargs': {'hello': 'world'}}
('James Harden', 1, 'PG', 'S+', 34, (), {})
]]>
</code-block>
</tab>
</tabs>

### 2.9 replace方法

这个方法允许你创建一个新的实例，其中某些字段的值被更改，而其他字段的值保持不变。


```Python
from dataclasses import dataclass, field, fields, replace
from typing import List


@dataclass
class Player:
    name: str
    number: int
    position: str
    grade: str
    age: int = 18


# 示例使用
harden = Player('James Harden', 1, 'PG', 'S+', 34)
leonard = Player(name='Kawhi Leonard', number=2, position='SF', grade='S+')


@dataclass
class Team:
    name: str = field(metadata={'unit': 'name'})
    players: List[Player] = field(
        default_factory=lambda: [leonard], metadata={'unit': 'players'}
        )


clippers = Team("clippers", [leonard])
# 使用 replace() 替换 Team 实例中的字段值
new_clippers = replace(clippers, name="new_clippers", players=[leonard, harden])
print("Original Clippers: ", clippers)
print("New Clippers: ", new_clippers)
```

### 2.10 自动化创建子数据类

我们可以通过 `from_dict` 方法，来自动创建子数据类。

```Python
from dataclasses import dataclass, field, fields, replace
from typing import Any

@dataclass
class WorkInfo:
    company_name: str
    level: str
    years: int


@dataclass
class Person:
    name: str
    age: int
    work_info: WorkInfo
    
    def from_dict(cls, data: Dict[str, Any]) -> "Person":
        return cls(
            name=data["name"],
            age=data["age"],
            work_info=WorkInfo(**data["work_info"])
        )
    
    
p = Person(**{
        "name": "draven",
        "age": 28,
        "work_info": {"company_name": "Google", "level": "staff", "years": 3}
    })
print(p.work_info)
```

## 3. 应用示例

### 3.1 数据提取、参数校验

`dataclass` 数据类可以配合一些校验工具包和数据提取工具包以实现数据提取或参数校验的工作，以下是配合 `marshmallow`、`desert` 实现数据校验提取工作的示例：

```Python
import requests
import dataclasses
import desert
from dataclasses import dataclass, field
from marshmallow import fields, EXCLUDE, validate


@dataclass
class Activity:
    activity: str
    participants: int = field(
        metadata=desert.metadata(
            fields.Int(
                required=True,
                validate=validate.Range(min=1, max=50, error="Participants must be between 1 and 50 people"
                )
            )
        )
    )
    price: float = field(
        metadata=desert.metadata(
        fields.Float(
            required=True,
            validate=validate.Range(min=0, max=50, error="Price must be between $1 and $50")
            )
        )
    )

    def __post_init__(self):
        self.price = self.price * 100


def get_activity():
    # resp = requests.get("https://www.boredapi.com/api/activity").json()
    resp = {
        "activity": "Improve your touch typing",
        "type": "busywork",
        "participants": 1,
        "price": 1.0,
        # "price": 51,
        "link": "https://en.wikipedia.org/wiki/Touch_typing",
        "key": "2526437",
        "accessibility": 0.8
    }
    # 只提取关心的部分，未知内容选择忽略
    schema = desert.schema(Activity, meta={"unknown": EXCLUDE})
    return schema.load(resp)


print(get_activity())
```

### 3.2 存储数据的简单对象

`dataclasses` 在许多情境下都表现出色，尤其是在定义用于存储数据的简单对象时。它特别适用于处理配置信息、数据传输对象（DTO）、领域对象以及其他仅包含数据的结构。

**需求**：程序退出前自动持久化配置对象到配置文件。


```Python
import json
import atexit
import logging
import threading
from pathlib import Path
from dataclasses import dataclass, asdict


@dataclass
class Config:
    name: str = "mysql"
    port: int = 3306

    _instance = None
    _lock = threading.Lock()
    _registered = False  # 新增类属性

    def __new__(cls, *args, **kw):
        with cls._lock:
            if cls._instance is None:
                cls._instance = super().__new__(cls)
            return cls._instance

    def load_from_file(self, file_path):
        """从配置文件加载配置，如果文件不存在或加载失败，保持默认值。
        """
        if file_path.exists():
            try:
                with file_path.open() as f:
                    json_data = json.load(f)
                    for key, value in json_data.items():
                        setattr(self, key, value)
            except Exception as err:
                logging.error(f"Failed to load config from file: {err}")
        else:
            logging.warning(f"Config file '{file_path}' not exists. Using default values.")

    def save_to_file(self, file_path):
        """保存配置到文件
        """
        json_str = json.dumps(asdict(self), indent=4)
        with file_path.open('w') as f:
            logging.warning(f"Saving configs to '{file_path}'")
            f.write(json_str)

    @classmethod
    def register_atexit(cls):
        """注册在程序退出时保存配置到配置文件"""
        with cls._lock:
            if not cls._registered:
                atexit.register(cls._instance.save_to_file, Path("./config.json"))
                cls._registered = True

    # 读取配置文件和保存配置的逻辑分离
    def __post_init__(self):
        config_file = Path("./config.json")

        # 从配置文件加载配置
        self.load_from_file(config_file)

        # 注册在程序退出时保存配置到配置文件
        self.register_atexit()


if __name__ == "__main__":
    # 创建一个 Config 实例
    config_instance = Config(name="redis", port=6379)
    # 打印当前配置
    print("Current Config:", config_instance)
    # 修改配置并再次打印
    config_instance.port = 8080
    print("Updated Config:", config_instance)
    # 创建另一个 Config 实例，演示单例模式
    another_instance = Config()
    print("Another Instance Config:", another_instance)
    # 保存配置到文件
    another_instance.save_to_file(Path("./another_config.json"))
    # 从文件加载配置
    another_instance.load_from_file(Path("./another_config.json"))
    print("Loaded Config from File:", another_instance)
```

### 3.3 让函数返回值更明确清晰


```Python
from dataclasses import dataclass
from enum import Enum
from typing import Tuple, Dict, Union

class Grade(Enum):
    S_PLUS = 'S+'
    # 定义其他等级...

@dataclass
class Player:
    name: str
    number: int
    position: str
    grade: Grade
    age: int = 18

def create_player(name: str, number: int, position: str, grade: Grade, age: int) -> Player:
    return Player(name, number, position, grade, age)

# 示例使用
harden = create_player('詹姆斯·哈登', 1, '控球后卫', Grade.S_PLUS, 34)
bryant = create_player('科比·布莱恩特', 24, '得分后卫', Grade.S_PLUS, 41)

print(harden)
print(bryant)
```


## 4. 源码解析


## 5. 替代方案

`dataclasses` 提供了许多方便的功能，但是 PEP 557 中还提到一个同样强大的数据类库 `attrs`，并且这个库支持验证器等功能。

```Python
import attr

@attr.s
class Point:
    x = attr.ib(type=int)
    y = attr.ib(type=int)

p = Point(1, 2)
print(p)
```


在选择使用 `dataclasses` 还是 `attrs` 时，取决于项目的需求和个人喜好。`dataclasses` 更简单直观，而 `attrs` 提供了更多的扩展性。如果只需要一些基本的自动生成特殊方法的功能，`dataclasses` 是个不错的选择。如果你需要更高级的功能和更多的定制选项，可以考虑使用 `attrs`。

此外，还有 Box 这样的第三方库可以创建数据类，Pydantic 也是更为强大的库。









<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/RMmsD_CwpJ2zuqW4xNrggw">实用的标准库装饰器: dataclass</a>
    <a href="https://docs.python.org/zh-cn/3/library/dataclasses.html">dataclasses 文档</a>
</category>
<category ref="ref_github">
</category>
<category ref="ref_issues"></category>
<category ref="ref_hf"></category>
<category ref="ref_ms"></category>
</seealso>