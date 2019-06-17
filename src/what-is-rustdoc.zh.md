# 什么是rustdoc？

标准的Rust发行版附带了一个名为的工具`rustdoc`。它的工作是为Rust项目生成文档。在基础层面上，Rustdoc将crate root或Markdown文件作为参数，并生成HTML，CSS和JavaScript。

## 基本用法

试一试吧！让我们用Cargo创建一个新项目：

```bash
$ cargo new docs
$ cd docs
```

在`src/lib.rs`，你会发现Cargo已经生成了一些示例代码。删除它并替换为：

```rust
/// foo is a function
fn foo() {}
```

我们跑吧`rustdoc`在我们的代码上。为此，我们可以使用我们的crate root的路径来调用它，如下所示：

```bash
$ rustdoc src/lib.rs
```

这将创建一个新目录，`doc`，里面有一个网站！在我们的例子中，主页位于`doc/lib/index.html`。如果你在网络浏览器中打开它，你会看到一个带有搜索栏的页面，顶部有“Crate lib”，没有内容。这有两个问题：首先，为什么我们认为我们的包名为“lib”？第二，为什么它没有任何内容？

第一个问题是由于`rustdoc`试图提供帮助;喜欢`rustc`，它假定我们的箱子名称是箱子根目录的文件名。要解决这个问题，我们可以传入一个命令行标志：

```bash
$ rustdoc src/lib.rs --crate-name docs
```

现在，`doc/docs/index.html`将生成，页面显示“Crate docs”。

对于第二个问题，这是因为我们的功能`foo`不公开;`rustdoc`默认为仅为公共函数生成文档。如果我们改变我们的代码......

```rust
/// foo is a function
pub fn foo() {}
```

......然后重新运行`rustdoc`：

```bash
$ rustdoc src/lib.rs --crate-name docs
```

我们将有一些生成的文档。打开`doc/docs/index.html`并检查出来！它应该显示一个链接`foo`功能页面，位于`doc/docs/fn.foo.html`。在那个页面上，你会看到“foo是一个函数”，我们把它放在我们的箱子里的文档注释中。

## 使用rustdoc和货物

货物也有整合`rustdoc`使生成文档更容易。而不是`rustdoc`命令，我们可以做到这一点：

```bash
$ cargo doc
```

在内部，这呼吁`rustdoc`像这样：

```bash
$ rustdoc --crate-name docs srclib.rs -o <path>\docs\target\doc -L
dependency=<path>docs\target\debug\deps
```

你可以看到这个`cargo doc --verbose`。

它生成正确的`--crate-name`对我们来说，以及指向`src/lib.rs`但那些其他论点呢？`-o`控制*Ø*我们的文档。而不是顶级`doc`目录，您会注意到Cargo将生成的文档放在`target`。这是Cargo项目中生成文件的惯用场所。它也通过了`-L`，一个标志，帮助rustdoc找到您的代码所依赖的依赖项。如果我们的项目使用依赖项，我们也会获得它们的文档！

## 使用独立的Markdown文件

`rustdoc`也可以从独立的Markdown文件生成HTML。让我们试一试：创建一个`README.md`包含以下内容的文件：

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

并致电`rustdoc`在上面：

```bash
$ rustdoc README.md
```

你会找到一个HTML文件`docs/doc/README.html`从其Markdown内容生成。

不幸的是，Cargo目前无法理解独立的Markdown文件。

## 摘要

这涵盖了最简单的用例`rustdoc`。本书的其余部分将解释所有选项`rustdoc`有，以及如何使用它们。
