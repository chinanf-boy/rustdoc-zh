# 什么是 rustdoc？

标准的 Rust 发行版，附带了一个名为`rustdoc`的工具。它的任务是为 Rust 项目，生成(示例/代码)文档。基本层面理解为，Rustdoc 将 一个 (crate root)箱子的根目录 或 Markdown 文件作为参数，并生成 HTML，CSS 和 JavaScript。

## 基本用法

试一试吧！让我们用 Cargo 创建一个新项目：

```bash
$ cargo new docs
$ cd docs
```

在`src/lib.rs`，你会发现 Cargo 已经生成了一些示例代码。删除它并替换为：

```rust
/// foo 是一个函数
fn foo() {}
```

让我们用`rustdoc`对准我们的代码。为此，到达我们的 crate root 的路径上，调用它，如下所示：

```bash
$ rustdoc src/lib.rs
```

这将创建一个新目录，`doc`，里面有一个网站！在我们的例子中，主网页是`doc/lib/index.html`。如果你在网络浏览器中打开它，你会看到一个带有搜索栏的页面，顶部有“Crate lib”，没有内容。这有两个问题：

- 首先，为什么我们的包名是“lib”？
- 第二，为什么它没有任何内容？

第一个问题是由于，`rustdoc`想要给出帮助信息；如同`rustc`一样，它会假定我们的箱子名，就是箱子根目录的文件名。要解决这个问题，我们可以传入一个命令行标志：

```bash
$ rustdoc src/lib.rs --crate-name docs
```

现在，`doc/docs/index.html`将生成，页面显示“Crate docs”。

对于第二个问题，是因为我们的函数`foo`是不公开的；`rustdoc`默认仅为公开函数，生成文档。如果我们改变我们的代码...

```rust
/// foo 是一个函数
pub fn foo() {}
```

...然后重新运行`rustdoc`：

```bash
$ rustdoc src/lib.rs --crate-name docs
```

我们将有一些生成的文档。打开`doc/docs/index.html`并检查出来！它应该显示一个链接`foo`函数页面，位于`doc/docs/fn.foo.html`。在那个页面上，你会看到“foo 是一个函数”，正是我们放在箱子的文档注释。

##  rustdoc 搭配 Cargo

Cargo 也有整合`rustdoc`使生成文档更容易。不是`rustdoc`命令，我们会使用：

```bash
$ cargo doc
```

在内部，会像这样调用`rustdoc`：

```bash
$ rustdoc --crate-name docs srclib.rs -o <path>\docs\target\doc -L
dependency=<path>docs\target\debug\deps
```

用`cargo doc --verbose`，能看到这个详细信息。

可以看到，内部命令为我们生成了正确的`--crate-name`，以及指向`src/lib.rs`，但那些其他参数呢？`-o`控制我们文档的*o*utput(输出目录)。不再是`doc`顶级目录，您会注意到 Cargo 把生成的文档放在`target`。这是 Cargo 项目中，生成文件的惯用场所。还有，一个`-L`标志，它帮助 rustdoc 找到您的代码所依赖的依赖项。如果我们的项目使用依赖项，我们也会获得它们的文档！

## 使用独立的 Markdown 文件

`rustdoc`也可以从独立的 Markdown 文件生成 HTML。让我们试一试：创建一个`README.md`，包含以下内容的文件：

````text
# Docs

This is a project to test out `rustdoc`.

[Here is a link!](https://www.rust-lang.org)

## Subheading

```rust
fn foo() -> i32 {
    1 + 1
}
```
````

调用`rustdoc`：

```bash
$ rustdoc README.md
```

你会找到一个 HTML 文件`docs/doc/README.html`，源自 Markdown 内容。

不幸的是，Cargo 目前无法理解独立的 Markdown 文件。

## 摘要

这涵盖了最简单的`rustdoc`用例。本书的其余部分，将解释`rustdoc`所有的选项，以及如何使用它们。
