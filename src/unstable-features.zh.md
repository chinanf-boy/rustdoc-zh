# 不稳定的功能

Rustdoc 正在积极开发中，与 Rust 编译器一样，某些功能仅在夜间版本中可用。其中一些功能是新功能，需要进一步测试才能将它们发布到全世界，其中一些功能与 Rust 编译器中不稳定的功能相关联。这里的一些功能需要有匹配`#![feature(...)]`启用，而更全面的文档，记录在[不稳定的书][unstable book]。文档有些部分会根据需要链接到那里。

> 译者：crate - 箱子；trait - 特质；

[unstable book]: https://doc.rust-lang.org/unstable-book/index.html

## 夜间(nightly)功能

以下功能只需要夜间构建即可运行。与此页面上的其他功能不同，这些功能不需要用命令行标志“打开”，或在箱子上写有`#![feature(...)]`属性。若是在稳定(stable)版本上使用(这些功能)时，会留一些后路模式，所以还是要小心！

### `compile-fail`文档测试的错误编号

详见[文档测试的章节][doctest-attributes]，你可以为 doctest 添加一个`compile_fail`属性，表明测试应该无法通过编译。但是，在夜间，您可以选择添加错误编号，以声明 doctest 应该发出特定的错误编号：

[doctest-attributes]: documentation-tests.zh.html#属性

````markdown
```compile_fail,E0044
extern { fn some_func<T>(x: T); }
```
````

错误索引(error index)会使用它(E0044)，确保样本正确地发出对应错误代码。但是，这些错误代码不能保证的唯一一件事是，这一段代码在版本之间没有问题，因此将来不太可能稳定。

尝试在 stable 版本上使用这些错误号，将导致代码示例被解释为纯文本。

### 按类型，链接到某项

按照[RFC 1946]的设计，当您将某项用作链接时，Rustdoc 可以解析项的路径。要解析这些类型名称，它通过声明或通过`use`，使用当前在范围内的项。对于模块，“活动范围”取决于文档是否写在模块外部（如`mod`内的`///`注释）或模块内部（文件或块内的`//!`注释）。对于所有其他项，它使用封闭模块的范围。

[rfc 1946]: https://github.com/rust-lang/rfcs/pull/1946

例如，在以下代码中：

```rust
/// Does the thing.
pub fn do_the_thing(_: SomeType) {
    println!("Let's do the thing!");
}

/// Token you use to [`do_the_thing`].
pub struct SomeType;
```

在`SomeType`的文档有`` [`do_the_thing`] ``链接，会正确链接到`fn do_the_thing`页面。请注意，在这里，rustdoc 将为您插入链接目标，但手动编写目标也可以：

```rust
pub mod some_module {
    /// Token you use to do the thing.
    pub struct SomeStruct;
}

/// Does the thing. 需要一个 [`SomeStruct`] for the thing to work.
///
/// [`SomeStruct`]: some_module::SomeStruct
pub fn do_the_thing(_: some_module::SomeStruct) {
    println!("Let's do the thing!");
}
```

有关详细信息，请查看[RFC][rfc 1946]，看看[跟踪问题][43466]有关该功能的哪些部分可用的更多信息。

[43466]: https://github.com/rust-lang/rust/issues/43466

## `#[doc]`属性扩展

下面功能通过扩展`#[doc]`属性来操作，因此可以被编译器捕获，并启用箱子中的`#![feature(...)]`属性。

### 记录，特定平台/功能的信息

由于 Rustdoc 记录箱子的方式，它创建的文档与特定目标 rustc 编译有关。(源代码)任何非特定的东西都会被丢弃，这是通过`#[cfg]`在编译过程的早期进行属性处理完成的。但是，Rustdoc 有一个技巧可以处理*确定*存在的特定平台代码。

因为 Rustdoc 不需要将 crate 完全编译为二进制文件，所以它会把函数体替换成`loop {}`，防止不必要的处理。这意味着，将忽略函数中，要求为特定平台部分的任何代码。结合特殊属性，`#[doc(cfg(...))]`，你可以告诉 Rustdoc 确切运行的平台，确保 doctests 只在适当的平台上运行。

该`#[doc(cfg(...))]`属性具有另一个效果：当 Rustdoc 渲染该项的文档时，它将附有一个横幅（提示），说明该项仅在某些平台上可用。

对于 Rustdoc 来记录项，它需要查看它，无论它当前在哪个平台上运行。为了解决这个问题，Rustdoc 在你的箱子上运行时，会设置标志`#[cfg(rustdoc)]`。将此与项的给定目标平台(如：windows)相结合，可以在该平台上正常构建您的箱子以及任何地方构建文档的情况下，显示它。

例如，`#[cfg(any(windows, rustdoc))]`将在 Windows 上或在文档化过程中，保留项。然后，添加新属性`#[doc(cfg(windows))]`将告诉 Rustdoc，该项应该在 Windows 上使用。例如：

```rust
#![feature(doc_cfg)]

/// Token struct that can only be used on Windows.
#[cfg(any(windows, rustdoc))]
#[doc(cfg(windows))]
pub struct WindowsToken;

/// Token struct that can only be used on Unix.
#[cfg(any(unix, rustdoc))]
#[doc(cfg(unix))]
pub struct UnixToken;
```

在此示例中，Token 只会出现在各自的平台上，但它们都会出现在文档中。

`#[doc(cfg(...))]`被引入标准库使用，目前需要`#![feature(doc_cfg)]`功能守卫。有关更多信息，请参阅[它在不稳定之书的章节][unstable-doc-cfg]和[它的跟踪问题][issue-doc-cfg]。

[unstable-doc-cfg]: https://doc.rust-lang.org/unstable-book/language-features/doc-cfg.html
[issue-doc-cfg]: https://github.com/rust-lang/rust/issues/43781

### 将您的 trait 添加到“Important Traits”对话框

Rustdoc 保存了一个列表，有一些 trait(特质)，这些特质在实现时被认为是给定类型的“基础”。这些特质旨在成为其类型的主要接口，并且通常是其类型的唯一可用文档。因此，Rustdoc 会跟踪给定类型何时实现其中一个特质，并在函数返回其中一个类型时，特别标住它。这是“Important Traits”对话框，在函数旁边显示为 圆圈-i 按钮，单击此按钮可显示对话框。

在标准库中，符合条件的特质是`Iterator`，`io::Read`，和`io::Write`。但是，这些特质不是硬编码列表，而是具有特殊的标记属性：`#[doc(spotlight)]`。这意味着您可以将此属性应用于您自己的特质，并将其包含在文档中的“Important Traits”对话框中。

该`#[doc(spotlight)]`属性目前需要`#![feature(doc_spotlight)]`功能守卫。有关更多信息，请参阅[它在不稳定的书中的章节][unstable-spotlight]和[它的跟踪问题][issue-spotlight]。

[unstable-spotlight]: https://doc.rust-lang.org/unstable-book/language-features/doc-spotlight.html
[issue-spotlight]: https://github.com/rust-lang/rust/issues/45040

### 从文档中，排除某些依赖项

标准库使用多个依赖项，而这些依赖项又使用标准库中的几种类型和特质。此外，编译器内部，有几个箱子，不被认为是官方标准库的一部分，因此会在文档中分散。排除他们的箱子文件是不够的，因为关于特质实现的信息，会出现在类型和特质的页面上，且可以是不同的箱子！

为防止内部类型包含在文档中，标准库会为其`extern crate`声明添加了属性：`#[doc(masked)]`。这会导致 Rustdoc 在构建特质实现列表时，“屏蔽掉”这些包中的类型。

该`#[doc(masked)]`属性旨在在内部使用，并且需要`#![feature(doc_masked)]`功能守卫。有关更多信息，请参阅[它在不稳定的书中的章节][unstable-masked]和[它的跟踪问题][issue-masked]。

[unstable-masked]: https://doc.rust-lang.org/unstable-book/language-features/doc-masked.html
[issue-masked]: https://github.com/rust-lang/rust/issues/44027

### 包含外部文件，把它作为 API 文档

按照设计[RFC 1990]，Rustdoc 可以读取外部文件以用作类型的文档。如果某些文档太长，以至于会破坏读取源代码的流程，这将非常有用。不把全部文档内联，而是用`#[doc(include = "sometype.md")]`（这里的`sometype.md`是一个箱子中`lib.rs`邻近的文件）将要求 Rustdoc 读取该文件并使用它，就好像它是内联编写的一样。

[rfc 1990]: https://github.com/rust-lang/rfcs/pull/1990

`#[doc(include = "...")]`目前需要`#![feature(external_doc)]`功能守卫。有关更多信息，请参阅[它在不稳定的书中的章节][unstable-include]和[它的跟踪问题][issue-include]。

[unstable-include]: https://doc.rust-lang.org/unstable-book/language-features/external-doc.html
[issue-include]: https://github.com/rust-lang/rust/issues/44732

### 在文档搜索中，为某项添加别名

此功能允许您为某项添加别名，这样当使用`rustdoc`搜索时，会接触到`doc(alias)`属性。例：

```rust,no_run
#![feature(doc_alias)]

#[doc(alias = "x")]
#[doc(alias = "big")]
pub struct BigX;
```

然后，输入“x”或“big”，`rustdoc`搜索会找到，并优先显示`BigX`结构。

## 不稳定的命令行参数

通过将命令行标志传递给 Rustdoc 来启用这些功能，但是有问题的标志本身被标记为不稳定。要使用这些选项中的任何一个，请在命令行上传递`-Z unstable-options`和问题标志。要从 Cargo 执行此操作，您可以使用`RUSTDOCFLAGS`环境变量，或`cargo rustdoc`命令。

> 译者：（下面的）内容 - content: 代指要渲染的 HTML 内容。

### `--markdown-before-content`：在内容之前，包含渲染的 Markdown

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --markdown-before-content extra.md
$ rustdoc README.md -Z unstable-options --markdown-before-content extra.md
```

就像`--html-before-content`，这允许您在`<body>`标签内部，但在其他`rustdoc`正常生成内容之前，插入额外的内容(extra.md)。但不是直接插入文件，`rustdoc`会通过 Markdown 渲染器过滤文件，再把结果插入。

### `--markdown-after-content`：在内容之后，包含渲染的 Markdown

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --markdown-after-content extra.md
$ rustdoc README.md -Z unstable-options --markdown-after-content extra.md
```

就像`--html-after-content`，这允许您在`<body>`标签内部，但在其他`rustdoc`正常生成内容之后，插入额外的内容(extra.md)。但不是直接插入文件，`rustdoc`会通过 Markdown 渲染器过滤文件，再把结果插入。

### `--playground-url`：控制游乐场的位置

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --playground-url https://play.rust-lang.org/
```

渲染一个箱子的文档时，此标志提供 Rust Playground 的基本 URL，用于生成`Run`纽扣。不像`--markdown-playground-url`，此参数适用于独立的 Markdown 文件*和*Rust 箱。这与添加`#![doc(html_playground_url = "url")]`到你箱子根目录的方式相同。在[章节`#[doc]`属性][doc-playground]有提到。请注意正式的 Rust Playground<https://play.rust-lang.org>，它没有任何可用的非标准库箱子，所以，如果你的例子需要你的箱子，请确保你提供的游乐场有你的箱子。

[doc-playground]: the-doc-attribute.zh.html#html_playground_url

如果`--playground-url`和`--markdown-playground-url`两者在渲染独立的 Markdown 文件时出现，且 URL 都给定，`--markdown-playground-url`将优先考虑。如果`--playground-url`和`#![doc(html_playground_url = "url")]`两者在渲染箱子文档时存在，属性（后者）将优先。

### `--crate-version`：控制箱子版本

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --crate-version 1.3.37
```

当`rustdoc`收到这个标志后，它会在箱子根文档的侧边栏中，打印一个额外的“Version (version)”。您可以使用此标志来区分库文档的不同版本。

### `--linker`：控制用于文档测试的链接员

使用此标志如下所示：

```bash
$ rustdoc --test src/lib.rs -Z unstable-options --linker foo
$ rustdoc --test README.md -Z unstable-options --linker foo
```

当`rustdoc`运行您的文档测试，它需要在运行它们之前，编译并将测试链接为可执行文件。此标志可用来更改，这些可执行文件上使用的链接员。这相当于传递`-C linker=foo`到`rustc`。

### `--sort-modules-by-appearance`：控制模块页面上的项的排序方式

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --sort-modules-by-appearance
```

通常，当`rustdoc`在模块页面中打印项，它将按字母顺序排序（考虑它们的稳定性，以及以数字结尾的名称）。给`rustdoc`这个标志将禁用这种排序，选择按照它们在源代码中出现的顺序打印项。

### `--themes`：提供其他主题

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --themes theme.css
```

给`rustdoc`这个标志将使您的主题复制到生成的箱子文档中，并在主题选择器中启用。注意`rustdoc`将拒绝您的主题文件，如果它没有“light”主题的所有样式(元素名)。欲知详情，看下面的`--theme-checker`。

### `--theme-checker`：验证主题 CSS 的有效性

使用此标志如下所示：

```bash
$ rustdoc -Z unstable-options --theme-checker theme.css
```

在将您的主题包含在箱子文档中之前，`rustdoc`会把它包含的所有 CSS 规则，与默认包含的“light”主题进行比较。使用此标志将允许您在`rustdoc`拒绝你的主题，能够查看缺少哪些规则。

### `--resource-suffix`：修改箱子文档中的 CSS / JavaScript 名称

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --resource-suffix suf
```

渲染文档时，`rustdoc`创建几个 CSS 和 JavaScript 文件作为输出的一部分。由于所有这些文件都是每个页面要链接的，因此如果您需要专门缓存它们，更改它们的位置可能会很麻烦。此标志将重命名输出中的所有这些文件，以在文件名中包含后缀。例如用上面的命令，`light.css`会成为`light-suf.css`。

### `--display-warnings`：记录或运行文档测试时显示警告

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --display-warnings
$ rustdoc --test src/lib.rs -Z unstable-options --display-warnings
```

此标志背后的意图是允许用户查看其库或其文档测试中，发生的警告，这些警告通常会被抑制。然而，[由于一个错误][issue-display-warnings]，这个标志不是 100％按预期工作。有关详细信息，请参阅问题链接

[issue-display-warnings]: https://github.com/rust-lang/rust/issues/41574

### `--extern-html-root-url`：控制 rustdoc 如何链接到非本地箱子

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --extern-html-root-url some-crate=https://example.com/some-crate/1.0.1
```

通常，当 rustdoc 想要链接到来自不同箱子的类型时，它会在两个地方查找：输出目录中已存在的文档，或者是另一个箱子中的`#![doc(doc_html_root)]`。但是，如果要链接到这两个位置中都不存在的文档，可以使用这些标志来控制该行为。当`--extern-html-root-url`使用，并与您的某个依赖项(名称)匹配，rustdoc 会使用这些文档的 URL。请记住，如果这些文档存在于输出目录中，那些本地文档仍将覆盖此标志。

### `-Z force-unstable-if-unmarked`

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z force-unstable-if-unmarked
```

这是一个内部标志，用于标准库和应用程序的编译器，将`#[unstable]`属性应用到任何没有其他稳定属性的依赖箱子。这允许`rustdoc`能够为编译器包和标准库生成文档，作为建造这些箱子时，提供给`rustc`的等效命令行参数。

### `--index-page`：为文档提供顶层网面

此功能允许您生成，给定 markdown 文件的索引页面。一个很好的例子是[rust 文档索引页](https://doc.rust-lang.org/index.html)。

有了这个，你将有一个页面，你可以在你的箱子顶部，尽可能自定义定制。

使用`index-page`选项，也会启用`enable-index-page`。

### `--enable-index-page`：为文档生成默认索引页

此功能允许，生成列出生成箱子的默认索引页。

### `--static-root-path`：控制静态文件在 HTML 输出中的加载方式

使用此标志如下：

```bash
$ rustdoc src/lib.rs -Z unstable-options --static-root-path '/cache/'
```

此标志控制 RustDoc 如何链接到 HTML 页面上的静态文件。如果您托管了许多，由同一版本的 RustDoc 生成的箱子文档，则可以使用此标志将 RustDoc 的 CSS、javascript 和字体文件缓存在单个位置，而不是每个“Doc root”（将生成的箱子文档，分组到同一输出目录，如`cargo doc`）每个箱子文件（如搜索索引）仍将从文档根目录加载，但从给定路径，加载并重命名为`--resource-suffix`。

### `--persist-doctests`：运行后保留 doctest 可执行文件

使用此标志如下：

```bash
$ rustdoc src/lib.rs --test -Z unstable-options --persist-doctests target/rustdoctest
```

此标志允许您在编译或运行 doctest 可执行文件之后保留它们。通常，RustDoc 会在测试后立即丢弃编译好的 doctest，但使用此选项，您可以保留这些二进制文件，以进行进一步的测试。

### `--show-coverage`：计算文档的项(item)百分比

使用此标志如下：

```bash
$ rustdoc src/lib.rs -Z unstable-options --show-coverage
```

如果您想确定箱子中，文档化了多少项，请将此标志传递给 RustDoc。当它收到这个标志时，它将统计您的箱子中，有文档的公共项，并打印出计数和百分比，而不是生成文档。

一些方法说明了 RustDoc 在该指标中的重要性：

- RustDoc 只计算您箱子中的项（即从其他箱子重新导出的项，不计算在内）。
- 直接写入内在 impl 块的文档不计算在内，即使显示了它们的文档注释，因为 Rust 代码中的常见模式是将所有内在方法写入同一 impl 块。
- trait 实现中的项不计算在内，因为这些 impl 将从 trait 本身继承任何文档。
- 默认情况下，只计算公共项。要计算私有项，请同时传递`--document-private-items`。

没有文档(注释)的公共项，可以通过内置的`missing_docs`lint 看到，没有文档(注释)的私有项，可通过 clippy `missing_docs_in_private_items`lint 查看。
