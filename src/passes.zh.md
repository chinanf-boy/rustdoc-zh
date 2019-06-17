# 通行证

Rustdoc 有一个名为“pass”的概念。这些是转变`rustdoc`在生成最终输出之前运行文档。

除了以下传递之外，请查看这些标志的文档：

- [`--passes`](command-line-arguments.md#--passes-add-more-rustdoc-passes)
- [`--no-defaults`](command-line-arguments.md#--no-defaults-dont-run-default-passes)

## 默认通过

默认情况下，rustdoc 将运行一些传递，即：

- `strip-hidden`
- `strip-private`
- `collapse-docs`
- `unindent-comments`

然而，`strip-private`暗示`strip-private-imports`，所以有效地，默认情况下运行所有 ​​ 传递。

## `strip-hidden`

这个传递实现了`#[doc(hidden)]`属性。当此传递运行时，它会检查每个项目，如果使用此属性进行注释，则会将其从中删除`rustdoc`的输出。

如果没有此过程，这些项目将保留在输出中。

## `unindent-comments`

当你写这样的文档评论时：

```rust,ignore
/// This is a documentation comment.
```

这之间有一个空间`///`然后`T`。该间距不是输出的一部分;它适用于人类，帮助将评论语法与评论文本分开。这个过程就是移除那个空间的东西。

确切的规则保留不足，以便我们可以解决我们发现的问题。

没有此传递，将保留确切的空格数。

## `collapse-docs`

有了这个传球，多个`#[doc]`属性转换为单个文档字符串。

例如：

```rust,ignore
#[doc = "This is the first line."]
#[doc = "This is the second line."]
```

获取折叠为单个文档字符串

```text
This is the first line.
This is the second line.
```

## `strip-private`

这将删除任何非公共项目的文档，例如：

```rust,ignore
/// These are private docs.
struct Private;

/// These are public docs.
pub struct Public;
```

此过程删除了文档`Private`因为他们不公开。

这个传球意味着`strip-priv-imports`。

## `strip-priv-imports`

这是一样的`strip-private`， 但对于`extern crate`和`use`语句而不是项目。
