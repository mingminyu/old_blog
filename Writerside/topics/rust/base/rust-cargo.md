# Cargo 包管理器

<show-structure depth="2"/>

## 1. 简介

现在主流的编程语言都有自己的包管理器，比如 Python 中的 pip，Java 中的 maven（实际上不是自带的），Rust 作为一根极具开发效率的一门语言，自然也有自己的包管理器——Cargo。

Cargo 可以帮助我们创建、编译、检测、测试以及运行 Rust 项目，在编译的过程中，它会隐式地调用 `rustc`。

| 命令                       | 描述                                | 
|--------------------------|-----------------------------------|
| cargo new <project_name> | 创建新项目，也可加上 `--lib` 参数             |
| cargo build              | 构建项目，生成二进制可执行文件或库文件               |
| cargo build --release    | 优化 `cargo build` 生成的可执行文件，常用于生产环境 |
| cargo check              | 检测                                |
| cargo test               | 测试                                |
| cargo run                | 运行                                |


## 2. 项目结构

