# 变量与数据类型

<show-structure depth="3"/>

## 1. 变量与不可变性 {collapsible="true" default-state="expanded"}

Rust 中需要使用 `let` 关键字来声明变量，和大部分的高级语言一样，它支持自动类型推导，但我们也可以显式地指定其类型：

```Javascript
let x: i32 = 5;
```

### 1.1 命名方式

变量的命名方式与 Python 非常类似，如果是枚举或者结构体的变量命名，则与 Python 中类的命名方式一致。

```Javascript
# 不指定变量类型，Rust 会自动推断出类型
let file_count = 100;
# 手动指定变量类型
let card_num: i64 = 200; 
```

> 注意，这里声明的不可变变量不可以使用 `card_num = 10` 这样的赋值语句。此外，如果变量没有被后续使用，则可以添加 `_` 前缀来消除警告。
> 
{style="tip"}


### 1.2 类型转换

Rust 可以通过 `as` 关键词来将变量类型的进行转换，当前前提条件是这两种数据类型是可以转换的。

```Javascript
let a = 3.1;
let b = a as i32;
```

如果想要通过此方式将变量 `a` 强制转换成字符串类型是不被允许的，怎么操作我们后续再介绍。

### 1.3 输出变量

输出变量的方式与 Python 的 f-string 的用法非常相似，需要注意的是，**我们是无法像 Python 那样，直接使用 `println!(x)` 来打印变量的**。

```Javascript
println!("val: {}", x);

// 下面这种方式和 Python 中 f-string 非常像
println!("val: {x}");
```

### 1.4 不可变性

不可变性是 Rust 实现其可靠性和安全性的关键因素之一，**它有助于防止常见的错误，例如数据竞争和并发问题**。在 Rust 中变量默认是不可变的，这样的特性迫使程序员需要思考当前代码中哪些程序状态可能会发生改变。

### 1.5 可变变量

当然，某些情况下对于可变变量的操作才是更更变，在 Rust 中可以使用 `mut` 关键来进行声明，后续就可以对所定义的可变变量做更多操作：

```Javascript
let mut y = 10;
y = 20;
```

### 1.6 shadow variable

Rust 允许我们隐藏一个变量，这意味着我们可以声明一个与现有变量同名的变量，从而有效地隐藏前一个变量。显然，这样的设计就可以让我们轻松改变变量的值、类型以及可变性。

<tabs>
<tab title="代码">
<code-block lang="javascript">
<![CDATA[
let x: i32 = 5;
{
    // 只在当前命名空间中有效，外部不可调用
    let x: i32 = 10;
    println!("Inner x is {}", x);
}
println!("Outer x is {x}");

let x = "hello";
println!("New x is {x}");

let mut x = "world";
x = "rust!";
println!("Mut x is {x}");
]]>
</code-block>
</tab>
<tab title="输出结果">
<code-block lang="python">
<![CDATA[
Inner x is 5
Outer x is 10
New x is hello
Mut x is rust!
]]>
</code-block>
</tab>
</tabs>


> 既然之前同名的变量被隐藏了，那么之前的变量是否还可继续调用么？是否一直存在整个程度的生命周期内？
> 
{style="warning"}

## 2. 常量与静态变量 {collapsible="true" default-state="expanded"}

### 2.1 Const 常量

常量的值必须是在编译时已知的常量表达式，必须指定类型和值。与 C 语言的宏定义不同，Rust 的常量值被直接嵌入到生成的底层机器代码中，而不是进行简单的字符替换。

与很多编程语言一样，Rust 中常量的命名必须全部大写，单词之间通过下划线连接。另外，常量的作用域是块级作用域，只在声明它们的作用于内可见。

<tabs>
<tab title="常量">
<code-block lang="javascript">
<![CDATA[
const SECONDS_HOUR: usize = 3_600;
const SECONDS_DAY: usize = 24 * SECONDS_HOUR;
println!("Seconds in a hour: {}", SECONDS_HOUR);
]]>
</code-block>
</tab>
<tab title="输出结果">
<code-block lang="python">
<![CDATA[
Seconds in a hour: 3600
]]>
</code-block>
</tab>
</tabs>

此外，如果在命名空间中定义的常量，该常量在外部也是不能被调用的。


### 2.2 Static 静态变量

与 `const` 常量不一样的是，`static` 变量实在运行时分配内存的，并且只能在 unsafe code 中改变变量值，静态变量的生命周期为整个程度的运行时间。


<tabs>
<tab title="代码">
<code-block lang="javascript">
<![CDATA[
static STATIC_VALUE: i32 = 42;
static mut STATIC_MUT_VALUE: i32 = 43;

fn main() {
   println!("STATIC_VALUE is {STATIC_VALUE}");

    // 使用 unsafe 代码修改静态变量的值
    unsafe {
        STATIC_MUT_VALUE = 30;
        println!("STATIC_MUT_VALUE is {STATIC_MUT_VALUE}");
    }
}
]]>
</code-block>
</tab>
<tab title="输出">
<code-block lang="python">
<![CDATA[
STATIC_VALUE is 42
STATIC_MUT_VALUE is 30
]]>
</code-block>
</tab>
</tabs>

> 所定义的静态可变变量 `STATIC_MUT_VALUE` 除了在 main 函数中 unsafe 代码中能使用，其他地方是不允许使用的。
> 
{style="warning"}


## 3. 基础数据类型 {collapsible="true" default-state="expanded"}

和大多数语言一样，Rust 的基础数据类型有整型、无符号整型、浮点型、布尔型、字符型，需要注意的是，字符串并不是 Rust 的基础类型。

| 分类       | 类型                               |
|----------|----------------------------------|
| 整型       | 默认是 i32，其他还有 i8、i16、i64、i128     |
| 无符号整型    | u8、u16、u32、u64、u128              |
| 与平台相关的整型 | usize、isize                      |
| 浮点型      | f32、f64，在不清楚边界所需空间时请尽量用 f64      |
| 布尔型      | true、false                       |
| 字符型      | 支持 Unicode 字符，需要使用单引号，双引号表示的是字符串 |

尽管非常简单，我们这里还是给一些代码示例:


<tabs>
<tab title="不同进制的整型">
<code-block lang="javascript">
<![CDATA[
let a1: i32 = -125;
let a2: i32 = 0xFF;
let a3: i32 = 0o13;
let a4: i32 = 0b10;
]]>
</code-block>
</tab>
<tab title="最大最小值">
<code-block lang="javascript">
<![CDATA[
println!("u32 max value: {}", u32::MAX);
println!("u32 min value: {}", u32::MIN);

println!("i32 max value: {}", i32::MAX);
println!("i32 min value: {}", i32::MIN);
println!("usize max value: {}", usize::MAX);
]]>
</code-block>
</tab>
<tab title="占用字节">
<code-block lang="javascript" ignore-vars="true">
<![CDATA[
println!("isize is {} bytes", std::mem::size_of::<isize>());
println!("usize is {} bytes", std::mem::size_of::<usize>());
println!("u64 is {} bytes", std::mem::size_of::<u64>());
]]>
</code-block>
</tab>

<tab title="浮点型">
<code-block lang="javascript">
<![CDATA[
let f1: f32 = 1.123010123;
let f2: f64 = 9.88123123;
println!("Float are {:.2} {f2:.3}", f1);
]]>
</code-block>
</tab>

<tab title="布尔型">
<code-block lang="javascript">
<![CDATA[
let is_true: bool = true;
let is_false: bool = false;
]]>
</code-block>
</tab>

<tab title="字符型">
<code-block lang="javascript">
<![CDATA[
let char_c: char = 'c';
let char_emo: char = '😀';
]]>
</code-block>
</tab>
</tabs>



## 4. 元组与数组 {collapsible="true" default-state="expanded"}

Rust 中的元组以及数组都是复合类型，且**它们的长度都是固定的**。不一样的地方在于，元组它可以容纳不同类型的数据，而数组则只能容纳同一类型的数据。此外，在 Rust 中还有集合类型 Vec 和 Map。


> 无论是数组还是元组，都无法直接使用 `println!("{}", x)` 这种方式打印变量，但是可以使用 `println!("{:?}", x)` 来打印出其结构。
>
{style="warning"}


### 4.1 数组

数组只能包含同一类型的数据，与其他语言一样，获取数组元素的值需要使用下标索引，可以通过其 `.len()` 方法来获取长度。它的创建方式、获取元素值、获取数组长度示例代码如下所示。

> 数组的长度是不支持 `println!("{arr.len()}")` 这种写法的，这种确实还是不如 Python 方便。
>
{style="warning"}

<tabs>
<tab title="定义数组">
<code-block lang="javascript">
<![CDATA[
let arr = [1, 2, 3];
let mut arr2 = [11, 12, 13];
arr2[0] = 111;
let arr3 = [1;3]  // 相当于 [1, 1, 1]，其中 `3` 表示数组个数
]]>
</code-block>
</tab>
<tab title="获取元素值">
<code-block lang="python">
<![CDATA[
let arr = [1, 2, 3];
println!("{}", arr[0]);
]]>
</code-block>
</tab>
<tab title="获取数组长度">
<code-block lang="python">
<![CDATA[
let arr = [1, 2, 3];
println!("arr length is {}", arr.len());
]]>
</code-block>
</tab>
<tab title="遍历元素">
<code-block lang="python">
<![CDATA[
let arr = [1, 2, 3];
for ele in arr {
   println!("{}", ele);
}
]]>
</code-block>
</tab>
</tabs>


### 4.2 元组

元组可以包含不同类型的数据，与数组不一样的是，元组没有 `.len()` 方法可以获取长度，且获取元素值的方法也不一样。此外，`()` 是空元组，通常函数默认返回值就是空元组。

<tabs>
<tab title="定义元组">
<code-block lang="javascript">
<![CDATA[
let tup = (0, "hi", 3.4);
// 当下标超过元组长度是会报错的，例如 `tup.3`
println!("Tup elements: {} {} {}", tup.0, tup.1, tup.2);

// 空元组的打印需要使用 `{:?}`
let empty_tup = ();
println!("{:?}", empty_tup); // 打印结构
]]>
</code-block>
</tab>

<tab title="可变元组">
<code-block lang="javascript">
<![CDATA[
let mut tup2 = (0, "hi", 3.4);
println!("Tup elements: {} {} {}", tup2.0, tup2.1, tup2.2);
tup2.1 = "hello";
println!("Tup elements: {} {} {}", tup2.0, tup2.1, tup2.2);
]]>
</code-block>
</tab>
</tabs>


### 4.3 所有权机制

在 Python 语言中，如果将变量 `a` 的值赋值给变量 `b`，实际上执行的是复制操作（复制操作的本质实际上就是值传递），在 Rust 中除了复制操作外，变量间的赋值还可以转移所有权，这种操作可以大大节省出空间。

> 通过 `String::from` 构建的字符串没有复制操作，所以才会有所有权转移的操作。如果使用是 `let a = "abc"` 所构建的字符串变量的话，也是有复制操作的。


<tabs>
<tab title="复制">
<code-block lang="javascript">
let a = "aa";
let b = a;
println!("{}", b);
// 与所有权转移不一样的是，这里的 `a` 并不会被销毁，依然可以使用
println!("{}", a);
</code-block>
</tab>
<tab title="转移所有权">
<code-block lang="javascript">
let str_item = String::from("aa");
// String 类型就将所有权进行转移操作，前面的 `str_item` 就不存在了
let str_item_mv = str_item;
// 也就是说，在上面一行代码后，你无法再使用 `str_item` 变量了
println!("{}", str_item_mv);
</code-block>
</tab>
</tabs>
