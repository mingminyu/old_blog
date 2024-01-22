# SQLGlot 教程

<show-structure depth="2"/>

## 1. 简介

[SQLGlot](https://github.com/tobymao/sqlglot) 是一个无依赖的 SQL 解析器、转译器、优化器和引擎。它可用于格式化 SQL 或在 20 种不同的 SQL 方言之间进行转换。它的目标是读取各种 SQL 输入，并以目标方言输出语法和语义正确的 SQL。

它是一个非常全面的通用 SQL 解析器，具有强大的测试套件。它的性能也相当高，而且完全是用 Python 编写的。我们可以轻松自定义解析器、分析查询、遍历表达式树以及以编程方式构建 SQL。

## 2. 安装

可以直接通过 `pip` 进行安装:

```Bash
pip install "sqlglot[rs]"
```

## 3. SQL 方言之间的转换

假设我们有一个 Databricks SQL 查询，大多数语法应该是通用的，但 `LIMIT 10` 是专门针对某些数据库管理系统的。

```Python
databricks_query = "SELECT * FROM Customer LIMIT 10"
```

如果你熟悉 SQL Server 或 Azure Synapse，它们会使用不同的语法将结果集限制为一定数量的行。那就是 `SELECT TOP n ... FROM ...`。现在，让我们看看如何使用 sqlglot 非常轻松地转换查询。

```Python
import sqlglot as sg

print(sg.transpile(databricks_query, read="databricks", write="tsql")[0])
```

在上面的代码中，我们调用了 sqlglot 中的 `transpile()` 函数。然后，我们提供 SQL 语句 databricks_query。之后告诉函数这是 Databricks 方言，我们希望它将其转换为 T-SQL。

值得一提的是，该函数实际上会返回一个列表，这是为了防止一个查询字符串中包含多个 SQL 语句。 当然，我们也可以将其从 T-SQL 转译回 Databricks SQL，如下所示。

```Python
tsql_query = "SELECT TOP 10 * FROM Customer"
print(sg.transpile(tsql_query, read="tsql", write="databricks")[0])
```

## 4. 编程方式构建 SQL

通过 sqlglot 来为构建 SQL 语句有以下有点：
- 降低SQL注入攻击的风险
- 错误处理和验证
- 可读性
- 可维护性
- 可扩展性

代码中使用 `WHERE 1=1` 的情况并不少见，事实上，在处理没有过滤条件的情况时，这确实是一个聪明的解决方案。

然而，这是一个很好的例子，证明使用字符串构造技术来构建 SQL 语句是有局限性的。也就是说，我们需要处理语法不允许将一致模式附加到查询字符串的情况。

现在，让我们构建一个简单的查询来尝试 sqlglot 的此功能。

```Python
query = (
    sg.select("*").from_("Customer")
      .where(
            sg.condition("age > 18").and_("is_vip = 'Y'")
            )
        )
print(query.sql())
```

带下划线的 `from_` 和 `and_` 函数只是为了避免与 Python 原生函数的命名冲突，所以请不要混淆，这里关于下划线并没有什么神奇之处。`.sql()` 可用于将构建的查询转换为字符串。

此功能的另一个用例可能是多语言要求。也就是说，一旦我们构建了查询，我们就可以轻松地将其输出到不同的 SQL 方言。让我们使用一开始的相同示例，构建一个查询，如下所示。

```Python
query = sg.select("*").from_("Customer").limit(10)
```

现在，我们可以在 Databricks SQL 或者 T-SQL 中输出查询：

```Python
sql_databricks = query.sql(dialect="databricks")
sql_tsql = query.sql(dialect="databricks")
print(sql_databricks)
print(sql_tsql)
```

## 5. 查询优化

最后但并非最不重要的一点是，该库还具有某种查询优化功能。 例如，如果我们定义了非常复杂的条件，如下所示。

```Python
bad_sql = sg.parse_one(
    """
    SELECT *
    FROM my_table
    WHERE a=1 OR (b=2 OR (c=3 AND d=4))
    """
)
```

现在，让我们看看 sqlglot 是如何优化它的。


```Python
good_sql = sg.optimizer.optimize(bad_sql)
print(good_sql.sql(pretty=True))
```

给出的 SQL 仍然很复杂，虽然优化的 SQL 有着更好的可读性，但是实际上这会造成 SQL 文本过长，**慎重使用**。


> 更多使用案例，请参阅[SQLGlot 文档](https://github.com/tobymao/sqlglot)



<seealso>
<category ref="ref_docs">
    <a href="https://mp.weixin.qq.com/s/TEXT7m1vjWGX9_GoXHAJDQ">SQLGlot 介绍</a>
    <a href="https://sqlglot.com/sqlglot.html">SQLGlot 文档</a>
</category>
<category ref="ref_github">
    <a href="https://github.com/tobymao/sqlglot">SQLGlot Github</a>
</category>
</seealso>