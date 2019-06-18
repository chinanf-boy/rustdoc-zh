# 通行证/传递参（Passes）

Rustdoc 有一个名为“passes”的概念。这些是在生成最终输出之前，`rustdoc`文档的转变方式。

除了下面描述的 passes 之外，也可以看看这些标志的文档：

- [`--passes`](command-line-arguments.zh.md#--passes-add-more-rustdoc-passes)
- [`--no-defaults`](command-line-arguments.zh.md#--no-defaults-dont-run-default-passes)

## 默认通行证

默认情况下，rustdoc 将运行一些传递参，即：

- `strip-hidden`
- `strip-private`
- `collapse-docs`
- `unindent-comments`

而，`strip-private`暗示`strip-private-imports`(剔除私有导入)，所以有效地，默认情况下运行所有传递。

## `strip-hidden`

这个传递参实现了`#[doc(hidden)]`属性。当此传递参运行时，它会检查每个项，若使用此属性进行注释，则会将其从`rustdoc`的输出中删除。

如果没有此传递参，这些项将保留在输出中。

## `unindent-comments`

当你写这样的文档注释时：

```rust,ignore
/// This is a documentation comment.
```

这`///`和`T`之间有一个空格。该空格间距不是输出的一部分；但它更利于人类阅读，帮助我们将注释语法与注释文本分开。这个传递参就是移除那个空格。

确切的数量规则未指定，所以我们可以解决我们发现的问题。

没有此传递参，将保留确切的空格数。

## `collapse-docs`

有了这个传递参，多个`#[doc]`属性会转换为单个文档字符串。

例如：

```rust,ignore
#[doc = "This is the first line."]
#[doc = "This is the second line."]
```

会折叠成单个文档字符串

```text
This is the first line.
This is the second line.
```

## `strip-private`

这将删除任何非-公共项的文档，例如：

```rust,ignore
/// These are private(私有) docs.
struct Private;

/// These are public(公共) docs.
pub struct Public;
```

此传递参删除了`Private`文档，因为他们不公开。

这个传递参，意味着`strip-priv-imports`。

## `strip-priv-imports`

这与`strip-private`一样的，但针对`extern crate`和`use`语句，而不是项。
