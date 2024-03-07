# 枚举与匹配模式

<show-structure depth="3"/>

## 1. 枚举 {collapsible="true" default-state="expanded"}

枚举(enums) 是一种用户自定义的数据类型，用于表示具有一组离散可能的变量。合适地使用枚举，能让我们的代码更加严谨，且更易于理解。

```Javascript
enum Shape {
    Cicle(f64),
    Rectangle(f64, f64),
    Square(f64),
}
```

在 Rust 中常用的枚举类型为 `Option` 和 `Result`。

<tabs>
<tab title="Option">
<code-block lang="javascript">
<![CDATA[
pub enum Option<T> {
    None,
    Some(T),
}
]]>
</code-block>
</tab>
<tab title="Result">
<code-block lang="javascript">
<![CDATA[
pub enum Result<T, E> {
    Ok(T),
    Err(E),
}
]]>
</code-block>
</tab>
</tabs>

## 2. 匹配模式 {collapsible="true" default-state="expanded"}

Rust 中使用 `match` 关键字进行条件匹配，通常我们会和枚举一起使用。`match` 独立使用时，也结合 `_`、`..=`、三元表达式，功能非常强大，示例代码如下：

```Scala
match number {
    0 => println!("Zero"),
    1 | 2 => println!("One or Two"),
    3..=9 => println!("From Three to None"),
    n if n % 2 == 0 => println!("Even number"),
    _ => println!("Other"),
}
```


<tabs>
<tab title="枚举和匹配">
<code-block lang="scala">
<![CDATA[
enum Color {
    Red,
    Yellow,
    Blue,
}

fn print_color(color: Color) {
    match color {
        Color::Red => println!("Red"),
        Color::Yellow => println!("Yellow"),
        Color::Blue => println!("Blue"),
        _ => (),
    }
}

fn main() {
    print_color(Color::Red);
}
]]>
</code-block>
</tab>
<tab title="枚举条件">
<code-block lang="scala">
<![CDATA[
enum BuildingLocation {
    Number(i32),
    Name(String),  // 不用 &str
    Unknown,  // 其他未知信息
}

impl BuildingLocation {
    fn print_location(&self) {
        match self {
            // BuildingLocation::Number(44)
            BuildingLocation::Number(c) => println!("Building Number: {}", c),
            // BuildingLocation::Name("ok".to_string())
            BuildingLocation::Name(c) => println!("Building Name: {}", c),    
            BuildingLocation::Unknown => println!("Unknown"),    
        }
    }
}

fn main() {
    let house_name = BuildingLocation::Name("abc".to_string());
    let house_number = BuildingLocation::Number(100);
    let house_unknown = BuildingLocation::Unknown;
    house_name.print_location();
    house_num.print_location();
}
]]>
</code-block>
</tab>
</tabs>




