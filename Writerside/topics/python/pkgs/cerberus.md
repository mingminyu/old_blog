# Cerberus 教程

<show-structure depth="2"/>

数据验证是任何应用程序的重要组成部分，它有助于确保数据的完整性和准确性。Python 中有许多用于数据验证的库，其中 Cerberus 是一个强大的选择。它可以帮助你定义数据验证规则，然后验证数据是否符合这些规则。

本文将深入探讨 Cerberus 库的基本概念、安装方法以及详细的示例代码，以帮助大家了解如何使用 Cerberus 进行数据验证。

> 如果你还没使用过 Pydantic 的话，强烈建议取学习下，Pydantic 有着更强大的功能。
{style="note"}

## 1. 简介

Cerberus 是一个 Python 数据验证库，它旨在帮助你定义和执行数据验证规则。它提供了一个灵活的验证引擎，可以轻松地验证各种数据，包括嵌套结构和复杂的数据模式。

Cerberus 的主要特点包括：
- **灵活的验证规则**：Cerberus 可以定义各种验证规则，包括数据类型、最小/最大值、正则表达式、自定义验证函数等。
- **嵌套结构验证**：Cerberus 可以轻松验证嵌套结构的数据，例如嵌套字典或列表。
- **多种数据源支持**：Cerberus 支持验证不同来源的数据，包括字典、JSON、MongoDB 文档等。
- **可扩展性**：我们可以轻松地扩展 Cerberus，添加自定义验证规则和转换函数，以满足特定的验证需求。
- **错误报告**：Cerberus 提供了详细的错误报告，可以理解验证失败的原因。

## 2. 安装

要开始使用 Cerberus，首先需要安装它。可以使用 `pip` 来安装 Cerberus：

```Bash
pip install cerberus
```

## 3. 基础使用

### 3.1 定义验证规则

Cerberus 使用一种称为验证规则的方式来定义数据验证条件，可以为每个字段定义验证规则，规定了该字段应满足的条件。

以下是一个简单的示例，演示如何定义验证规则：

```Python
from cerberus import Validator


# 定义验证规则
schema = {
    'name': {
        'type': 'string',
        'minlength': 3,
        'maxlength': 20,
        'required': True,
    },
    'age': {
        'type': 'integer',
        'min': 0,
        'max': 120,
        'required': True,
    },
    'email': {
        'type': 'string',
        'regex': r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$',
    },
}

# 创建验证器对象
validator = Validator(schema)
```

在这个示例中，定义了一个包含三个字段的验证规则：`name`、`age` 和 `email`。每个字段都具有不同的验证条件，例如 `type` 规定了字段的数据类型，`minlength` 和 `maxlength` 规定了字符串长度范围，`required` 规定了字段是否为必填项，`regex` 规定了邮箱字段的正则表达式验证。

### 3.2 验证数据

一旦定义了验证规则，就可以使用 Cerberus 对数据进行验证。 以下是一个示例，演示如何验证数据是否符合规则：

```Python
data = {
    'name': 'John Doe',
    'age': 30,
    'email': 'john@example.com',
}

# 验证数据
if validator.validate(data):
    print("Data is valid!")
else:
    print("Validation errors:", validator.errors)
```

在这个示例中，创建了一个包含数据的字典，并使用 `validator.validate(data)` 方法来验证数据是否符合规则。如果数据有效，将打印 "Data is valid!"，否则将打印验证错误信息。

### 3.3 获取详细错误信息

如果数据验证失败，可以使用 `validator.errors` 属性来获取详细的错误信息，以便更容易地识别问题所在。

```Python
data = {
    'name': 'Jo',
    'age': 150,
    'email': 'john',
}

if validator.validate(data):
    print("Data is valid!")
else:
    print("Validation errors: ")
    for field, errors in validator.errors.items():
        print(f"Field '{field}': {', '.join(errors)}")
```

在这个示例中，故意提供了不符合规则的数据，然后遍历 `validator.errors` 获取每个字段的错误信息并打印出来。

## 4. 高级用法

### 4.1 嵌套结构验证

Cerberus 可以验证嵌套结构的数据，例如嵌套的字典或列表，以下是一个示例：

```Python
schema = {
    'person': {
        'type': 'dict',
        'schema': {
            'name': {'type': 'string', 'required': True},
            'age': {'type': 'integer', 'required': True},
        },
        'required': True,
    },
}

validator = Validator(schema)

data = {
    'person': {
        'name': 'John Doe',
        'age': 30,
    },
}

if validator.validate(data):
    print("Data is valid!")
else:
    print("Validation errors: ", validator.errors)
```

在这个示例中，定义了一个包含嵌套字典的验证规则，然后提供了符合规则的数据。Cerberus 会递归验证嵌套的数据结构。

### 4.2 自定义验证函数

除了内置的验证规则，还可以使用自定义验证函数来执行特定的验证操作，以下是一个示例：

```Python

def custom_validation(field, value, error):
    if len(value) < 5:
        error(field, "Must be at least 5 characters")

schema = {
    'password': {
        'type': 'string', 'custom': custom_validation
    },
}

validator = Validator(schema)

data = {
    'password': '1234',
}

if validator.validate(data):
    print("Data is valid!")
else:
    print("Validation errors: ", validator.errors))
```

在这个示例中，定义了一个自定义验证函数 `custom_validation`，它检查密码字段是否至少包含 5 个字符。然后，在验证规则中使用了这个自定义验证函数。

## 5. 总结

Cerberus 是一个功能强大且灵活的 Python 数据验证库，可以帮助大家确保应用程序接收到的数据符合预期的格式和条件。通过定义验证规则，可以轻松地验证各种数据，从简单的字典到复杂的嵌套结构。


<seealso>
<category ref="ref_github">
    <a href="https://github.com/pyeve/cerberus">Cerberus</a>
</category>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/vAt95wDdC_L1WMY6UbRH7A">Cerberus 介绍</a>
    <a href="https://mp.weixin.qq.com/s/KnRnHqLDTHBOzg0NkO_Ang">厉害的 Python 库: Cerberus</a>
    <a href="https://docs.python-cerberus.org/">Cerberus 文档</a>
</category>
</seealso>