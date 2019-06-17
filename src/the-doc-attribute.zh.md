# 该`#[doc]`属性

该`#[doc]`属性可以让你控制各个方面`rustdoc`做它的工作。

最基本的功能`#[doc]`是处理实际的文档文本。那是，`///`是语法糖`#[doc]`。这意味着这两个是相同的：

```rust,ignore
/// This is a doc comment.
#[doc = " This is a doc comment."]
```

（注意属性版本中的前导空格。）

在多数情况下，`///`比起来更容易使用`#[doc]`。后者更容易的一种情况是在宏中生成文档;该`collapse-docs`传球将结合多个`#[doc]`属性到单个doc注释中，让您生成如下代码：

```rust,ignore
#[doc = "This is"]
#[doc = " a "]
#[doc = "doc comment"]
```

哪个可以感觉更灵活。请注意，这会生成：

```rust,ignore
#[doc = "This is\n a \ndoc comment"]
```

但鉴于文档是通过Markdown呈现的，它将删除这些换行符。

该`doc`属性有更多的选择！这些不涉及输出的文本，而是涉及输出的呈现的各个方面。我们将它们分为以下两种：在箱子级别有用的属性，以及在项目级别有用的属性。

## 在板条箱水平

这些选项控制文档在宏观层面的外观。

### `html_favicon_url`

这种形式`doc`属性可让您控制文档的图标。

```rust,ignore
#![doc(html_favicon_url = "https://example.com/favicon.ico")]
```

这将提出`<link rel="shortcut icon" href="{}">`进入你的文档，属性的字符串进入你的文档`{}`。

如果您不使用此属性，则不会有图标。

### `html_logo_url`

这种形式`doc`属性允许您控制文档左上角的徽标。

```rust,ignore
#![doc(html_logo_url = "https://example.com/logo.jpg")]
```

这将提出`<a href='index.html'><img src='{}' alt='logo' width='100'></a>`进入你的文档，属性的字符串进入你的文档`{}`。

如果您不使用此属性，则不会有徽标。

### `html_playground_url`

这种形式`doc`使用属性可以控制文档示例中的“运行”按钮向其发出请求的位置。

```rust,ignore
#![doc(html_playground_url = "https://playground.example.com/")]
```

现在，当您按“运行”时，该按钮将向此域发出请求。

如果您不使用此属性，则不会有运行按钮。

### `issue_tracker_base_url`

这种形式`doc`属性大多只对标准库有用;当要素不稳定时，必须提供跟踪要素的问题编号。`rustdoc`使用此数字加上此处给出的基本URL链接到跟踪问题。

```rust,ignore
#![doc(issue_tracker_base_url = "https://github.com/rust-lang/rust/issues/")]
```

### `html_root_url`

该`#[doc(html_root_url = "…")]`attribute value表示用于生成指向外部包的链接的URL。当rustdoc需要生成指向外部包中项目的链接时，它将首先检查外部包装箱是否已在磁盘上本地记录，如果是，则直接链接到它。如果不这样做，它将使用由...给出的URL`--extern-html-root-url`命令行标志（如果可用）。如果没有，那么它将使用`html_root_url`extern crate中的值如果可用的话。如果不可用，则不会链接extern项。

```rust,ignore
#![doc(html_root_url = "https://docs.rs/serde/1.0")]
```

### `html_no_source`

默认情况下，`rustdoc`将包含您的程序的源代码，以及在文档中的链接。但如果你包括这个：

```rust,ignore
#![doc(html_no_source)]
```

它不会。

### `test(no_crate_inject)`

默认情况下，`rustdoc`会自动添加一行`extern crate my_crate;`进入每个doctest。但如果你包括这个：

```rust,ignore
#![doc(test(no_crate_inject))]
```

它不会。

### `test(attr(...))`

这种形式`doc`属性允许您向所有doctests添加任意属性。例如，如果您希望您的doctests在产生任何警告时失败，您可以添加以下内容：

```rust,ignore
#![doc(test(attr(deny(warnings))))]
```

## 在项目级别

这些形式的`#[doc]`属性用于单个项目，以控制它们的记录方式。

## `#[doc(no_inline)]`/`#[doc(inline)]`

使用这些属性`use`声明和控制文档出现的位置。例如，考虑这个Rust代码：

```rust,ignore
pub use bar::Bar;

/// bar docs
pub mod bar {
    /// the docs for Bar
    pub struct Bar;
}
```

该文档将生成“再出口”部分，并说`pub use bar::Bar;`，哪里`Bar`是其页面的链接。

如果我们改变了`use`像这样的行：

```rust,ignore
#[doc(inline)]
pub use bar::Bar;
```

代替，`Bar`将出现在`Structs`部分，就像`Bar`被定义在顶层，而不是`pub use`倒是。

让我们通过制作来改变我们原来的例子`bar`私人的：

```rust,ignore
pub use bar::Bar;

/// bar docs
mod bar {
    /// the docs for Bar
    pub struct Bar;
}
```

在这里，因为`bar`不公开，`Bar`没有自己的页面，所以没有地方可以链接到。`rustdoc`将内联这些定义，因此我们最终的结果与`#[doc(inline)]`以上;`Bar`在...`Structs`部分，好像它是在顶层定义的。如果我们添加`no_inline`属性的形式：

```rust,ignore
#[doc(no_inline)]
pub use bar::Bar;

/// bar docs
mod bar {
    /// the docs for Bar
    pub struct Bar;
}
```

现在我们将有一个`Re-exports`线，和`Bar`不会链接到任何地方。

一个特例：在Rust 2018及以后，如果你`pub use`你的一个依赖，`rustdoc`除非你添加，否则不会急切地将其作为模块内联`#[doc(inline)]`。

## `#[doc(hidden)]`

任何带注释的项目`#[doc(hidden)]`不会出现在文档中，除非`strip-hidden`传球被删除。

## `#[doc(primitive)]`

由于原始类型是在编译器中定义的，因此无法附加文档属性。标准库使用此属性来提供生成基本类型的文档的方法。
