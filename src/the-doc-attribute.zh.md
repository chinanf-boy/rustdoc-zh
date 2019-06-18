# `#[doc]`属性

`#[doc]`属性可以让你控制`rustdoc`各个方面的工作。

`#[doc]`最基本的功能就是处理实际的文档文本。`///`，就是`#[doc]`的语法糖。这意味着，这两个是相同的：

```rust,ignore
/// This is a doc comment.
#[doc = " This is a doc comment."]
```

（注意属性版本中，字符串的前空格。）

在多数情况下，`///`比起`#[doc]`更容易使用。后者更容易使用的一种情况，是在宏中生成文档；`collapse-docs`通行证参数会把多个`#[doc]`属性，结合到单个 doc 注释中，让您生成如下代码：

```rust,ignore
#[doc = "This is"]
#[doc = " a "]
#[doc = "doc comment"]
```

能感觉更加灵活。注意这会生成：

```rust,ignore
#[doc = "This is\n a \ndoc comment"]
```

但鉴于文档是通过 Markdown 呈现的，它将删除这些换行符。

`doc`属性版本会有更多的选项！不是涉及输出文本的，而是有关输出描述的各个方面。我们把它们分为以下两种：箱子级别有用的属性，以及项目级别有用的属性。

## 在箱子级别

这些选项控制文档，在宏观层面的外观。

### `html_favicon_url`

`doc`属性的这种形式，可让您控制文档的图标。

```rust,ignore
#![doc(html_favicon_url = "https://example.com/favicon.ico")]
```

这会将`<link rel="shortcut icon" href="{}">`放入你的文档(HTML)，`{}`就是你的字符串。

如果您不使用此属性，则不会有图标。

### `html_logo_url`

`doc`属性的这种形式，允许您控制文档左上角的徽标。

```rust,ignore
#![doc(html_logo_url = "https://example.com/logo.jpg")]
```

这将提出`<a href='index.html'><img src='{}' alt='logo' width='100'></a>`进入你的文档，`{}`就是你的字符串。

如果您不使用此属性，则不会有徽标。

### `html_playground_url`

`doc`属性的这种形式，可以控制文档示例中的“Run”按钮，发出请求的位置。

```rust,ignore
#![doc(html_playground_url = "https://playground.example.com/")]
```

现在，当您按“Run”时，该按钮将向此域名发出请求。

如果您不使用此属性，则不会有 Run 按钮。

### `issue_tracker_base_url`

`doc`属性的这种形式，大多只对标准库有用；当一个功能不稳定时，必须提供跟踪该功能的问题编号。`rustdoc`使用编号，加上此处给出的基本 URL，链接到跟踪问题。

```rust,ignore
#![doc(issue_tracker_base_url = "https://github.com/rust-lang/rust/issues/")]
```

### `html_root_url`

`#[doc(html_root_url = "…")]`属性的值，表示生成指向外部箱子的 URL 链接。当 rustdoc 需要生成指向一个外部箱子的项的链接时，它将首先检查外部箱子，是否已在本地磁盘上记录，如果是，则直接链接到它。如果不是，它将使用`--extern-html-root-url`命令行标志给出的 URL（如果可用）。如果也没有，那么它将使用 extern crate 中的`html_root_url`值(如果可用)。如果还不可用，则不会链接 extern 项。

```rust,ignore
#![doc(html_root_url = "https://docs.rs/serde/1.0")]
```

### `html_no_source`

默认情况下，`rustdoc`将包含您的程序的源代码，以及在文档中的链接。但如果你包括这个：

```rust,ignore
#![doc(html_no_source)]
```

就不默认。

### `test(no_crate_inject)`

默认情况下，`rustdoc`会自动添加一行`extern crate my_crate;`进入每个 doctest(文档测试)。但如果你包括这个：

```rust,ignore
#![doc(test(no_crate_inject))]
```

它不会。

### `test(attr(...))`

`doc`属性的这种形式，允许您向所有 doctests 添加任意属性。例如，如果您希望您的 doctests 在产生任何警告时失败，您可以添加以下内容：

```rust,ignore
#![doc(test(attr(deny(warnings))))]
```

## 项目级别

`#[doc]`属性的这些形式，用于单个项目，以控制它们的文档化方式。

## `#[doc(no_inline)]`/`#[doc(inline)]`

这些属性用在`use`声明语句，和控制文档出现的位置。例如，考虑这个 Rust 代码：

```rust,ignore
pub use bar::Bar;

/// bar docs
pub mod bar {
    /// the docs for Bar
    pub struct Bar;
}
```

该文档将生成“Re-exports”部分，并给出`pub use bar::Bar;`，那这个`Bar`会是它自身页面的链接。

但如果我们像这样，改变了`use`行：

```rust,ignore
#[doc(inline)]
pub use bar::Bar;
```

`Bar`就变成，出现在`Structs`部分，`Bar`就像是在顶层定义的，而不是`pub use`形式。

让我们改变我们原来的例子，把`bar`模块变为私有：

```rust,ignore
pub use bar::Bar;

/// bar docs
mod bar {
    /// the docs for Bar
    pub struct Bar;
}
```

在这里，因为`bar`不公开，`Bar`就不会有自己的页面，所以没有地方可以链接。`rustdoc`将内联(inline)这些定义，因此我们最终的结果与上面`#[doc(inline)]`一样；`Bar`在`Structs`部分，好像它是在顶层定义的。如果，我们换成`no_inline`：

```rust,ignore
#[doc(no_inline)]
pub use bar::Bar;

/// bar docs
mod bar {
    /// the docs for Bar
    pub struct Bar;
}
```

现在变成，有一个`Re-exports`部分，但`Bar`不会链接到任何地方。

一个特例：在 Rust 2018 及以后，如果你`pub use`你的一个依赖，除非你添加`#[doc(inline)]`，否则`rustdoc`不会急切地将其作为模块内联。

## `#[doc(hidden)]`

任何带`#[doc(hidden)]`注释的项不会出现在文档中，除非`strip-hidden`通行证参数被删除。

## `#[doc(primitive)]`

由于原始类型是在编译器中定义的，因此无法附加文档属性。标准库会用此属性提供的方法，生成原始类型的文档。
