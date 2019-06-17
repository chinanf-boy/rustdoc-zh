# 不稳定的功能

Rustdoc正在积极开发中，与Rust编译器一样，某些功能仅在夜间版本中可用。其中一些功能是新功能，需要进一步测试才能将它们发布到全世界，其中一些功能与Rust编译器中不稳定的功能相关联。这里的一些功能需要匹配`#![feature(...)]`要启用的属性，因此更全面地记录在[不稳定的书]。那些部分将根据需要链接到那里。

[unstable book]: ../unstable-book/index.html

## 夜间功能

这些功能只需要每晚构建即可运行。与此页面上的其他功能不同，这些功能不需要使用命令行标志或“打开”`#![feature(...)]`您的箱子中的属性。当在稳定版本上使用时，这可以给它们一些微妙的后退模式，所以要小心！

### 错误号码`compile-fail`文档测试

详见[关于文档测试的章节][doctest-attributes]，你可以添加一个`compile_fail`属性为doctest，表明测试应该无法编译。但是，在每晚，您可以选择添加错误编号，以声明doctest应该发出特定的错误编号：

[doctest-attributes]: documentation-tests.html#attributes

````markdown
```compile_fail,E0044
extern { fn some_func<T>(x: T); }
```
````

错误索引使用此方法来确保与给定错误号对应的样本正确地发出该错误代码。但是，这些错误代码不能保证是一段代码从版本到版本发出的唯一内容，因此将来不太可能稳定。

尝试在stable上使用这些错误号将导致代码示例被解释为纯文本。

### 按类型链接到项目

按照设计[RFC 1946]，当您将项目用作链接时，Rustdoc可以解析项目的路径。要解析这些类型名称，它将使用当前在范围内的项目，通过声明或通过`use`声明。对于模块，“活动范围”取决于文档是否写在模块外部（如`///`评论`mod`声明）或模块内部（at`//!`文件或块内的注释）。对于所有其他项目，它使用封闭模块的范围。

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

链接到``[`do_the_thing`]``在`SomeType`的文档将正确链接到页面`fn
do_the_thing`。请注意，在这里，rustdoc将为您插入链接目标，但手动编写目标也可以：

```rust
pub mod some_module {
    /// Token you use to do the thing.
    pub struct SomeStruct;
}

/// Does the thing. Requires one [`SomeStruct`] for the thing to work.
///
/// [`SomeStruct`]: some_module::SomeStruct
pub fn do_the_thing(_: some_module::SomeStruct) {
    println!("Let's do the thing!");
}
```

有关详细信息，请查看[RFC][rfc 1946]，看看[跟踪问题][43466]有关该功能的哪些部分可用的更多信息。

[43466]: https://github.com/rust-lang/rust/issues/43466

## 延伸到`#[doc]`属性

这些功能通过扩展来实现`#[doc]`属性，因此可以被编译器捕获并启用`#![feature(...)]`您的箱子中的属性。

### 记录平台/特定功能的信息

由于Rustdoc记录包的方式，它创建的文档特定于目标rustc编译。任何特定于任何其他目标的东西都会被丢弃`#[cfg]`在编译过程的早期进行属性处理。但是，Rustdoc有一个技巧可以处理特定于平台的代码*不*收到它。

因为Rustdoc不需要将crate完全编译为二进制文件，所以它会替换函数体`loop {}`防止必须处理超过必要的。这意味着将忽略函数中需要特定于平台的部分的任何代码。结合特殊属性，`#[doc(cfg(...))]`，你可以告诉Rustdoc确切地运行哪个平台，确保doctests只在适当的平台上运行。

该`#[doc(cfg(...))]`属性具有另一个效果：当Rustdoc呈现该项目的文档时，它将附有一个横幅，说明该项目仅在某些平台上可用。

对于Rustdoc来记录项目，它需要查看它，无论它当前在哪个平台上运行。为了解决这个问题，Rustdoc设置了标志`#[cfg(rustdoc)]`在你的板条箱上运行时。将此与给定项目的目标平台相结合，可以在正常情况下在该平台上构建您的箱子时以及在任何地方构建文档时显示它。

例如，`#[cfg(any(windows, rustdoc))]`将在Windows上或在文档过程中保留项目。然后，添加新属性`#[doc(cfg(windows))]`将告诉Rustdoc该项应该在Windows上使用。例如：

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

在此示例中，令牌只会出现在各自的平台上，但它们都会出现在文档中。

`#[doc(cfg(...))]`被引入标准库使用，目前需要`#![feature(doc_cfg)]`特色门。有关更多信息，请参阅[它在不稳定的书中的一章][unstable-doc-cfg]和[它的跟踪问题][issue-doc-cfg]。

[unstable-doc-cfg]: ../unstable-book/language-features/doc-cfg.html

[issue-doc-cfg]: https://github.com/rust-lang/rust/issues/43781

### 将您的特征添加到“重要特征”对话框中

Rustdoc保留了一些特性列表，这些特征在实现时被认为是给定类型的“基础”。这些特征旨在成为其类型的主要接口，并且通常是唯一可用于其类型的文档。因此，Rustdoc将跟踪给定类型何时实现其中一个特征，并在函数返回其中一个类型时特别注意它。这是“重要特征”对话框，在功能旁边显示为圆圈-i按钮，单击此按钮可显示对话框。

在标准库中，符合条件的特征是`Iterator`，`io::Read`，和`io::Write`。但是，这些特征不是被实现为硬编码列表，而是具有特殊的标记属性：`#[doc(spotlight)]`。这意味着您可以将此属性应用于您自己的特征，以将其包含在文档中的“重要特征”对话框中。

该`#[doc(spotlight)]`属性目前需要`#![feature(doc_spotlight)]`特色门。有关更多信息，请参阅[它在不稳定的书中的一章][unstable-spotlight]和[它的跟踪问题][issue-spotlight]。

[unstable-spotlight]: ../unstable-book/language-features/doc-spotlight.html

[issue-spotlight]: https://github.com/rust-lang/rust/issues/45040

### 从文档中排除某些依赖项

标准库使用多个依赖项，而这些依赖项又使用标准库中的几种类型和特征。此外，有几个编译器内部的包装箱不被认为是官方标准库的一部分，因此会分散在文档中。排除他们的箱子文件是不够的，因为关于特征实施的信息出现在类型和特征的页面上，可以在不同的箱子中！

为防止内部类型包含在文档中，标准库会为其添加属性`extern crate`声明：`#[doc(masked)]`。这会导致Rustdoc在构建特征实现列表时从这些包中“屏蔽掉”类型。

该`#[doc(masked)]`属性旨在在内部使用，并且需要`#![feature(doc_masked)]`特色门。有关更多信息，请参阅[它在不稳定的书中的一章][unstable-masked]和[它的跟踪问题][issue-masked]。

[unstable-masked]: ../unstable-book/language-features/doc-masked.html

[issue-masked]: https://github.com/rust-lang/rust/issues/44027

### 将外部文件包含为API文档

按照设计[RFC 1990]，Rustdoc可以读取外部文件以用作类型的文档。如果某些文档太长以至于会破坏读取源的流程，这将非常有用。写作而不是全部内联`#[doc(include = "sometype.md")]`（哪里`sometype.md`是一个邻近的文件`lib.rs`对于crate）将要求Rustdoc读取该文件并使用它，就好像它是内联编写的一样。

[rfc 1990]: https://github.com/rust-lang/rfcs/pull/1990

`#[doc(include = "...")]`目前需要`#![feature(external_doc)]`特色门。有关更多信息，请参阅[它在不稳定的书中的一章][unstable-include]和[它的跟踪问题][issue-include]。

[unstable-include]: ../unstable-book/language-features/external-doc.html

[issue-include]: https://github.com/rust-lang/rust/issues/44732

### 在文档搜索中为项添加别名

此功能允许您在使用时为项目添加别名`rustdoc`搜索`doc(alias)`属性。例：

```rust,no_run
#![feature(doc_alias)]

#[doc(alias = "x")]
#[doc(alias = "big")]
pub struct BigX;
```

然后，当通过它寻找它`rustdoc`搜索，如果输入“x”或“大”，搜索将显示`BigX`结构优先。

## 不稳定的命令行参数

通过将命令行标志传递给Rustdoc来启用这些功能，但是有问题的标志本身被标记为不稳定。要使用这些选项中的任何一个，请传递`-Z unstable-options`以及在命令行上对Rustdoc提出的问题。要从Cargo执行此操作，您可以使用`RUSTDOCFLAGS`环境变量或`cargo rustdoc`命令。

### `--markdown-before-content`：在内容之前包括呈现的Markdown

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --markdown-before-content extra.md
$ rustdoc README.md -Z unstable-options --markdown-before-content extra.md
```

就像`--html-before-content`，这允许您在内部插入额外的内容`<body>`标签但在其他内容之前`rustdoc`通常会在提交的文档中生成。但是，不是直接插入文件，而是`rustdoc`在将结果插入文件之前，将通过Markdown渲染器传递文件。

### `--markdown-after-content`：包含内容后的渲染Markdown

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --markdown-after-content extra.md
$ rustdoc README.md -Z unstable-options --markdown-after-content extra.md
```

就像`--html-after-content`，这允许您在之前插入额外的内容`</body>`标签但是在其他内容之后`rustdoc`通常会在提交的文档中生成。但是，不是直接插入文件，而是`rustdoc`在将结果插入文件之前，将通过Markdown渲染器传递文件。

### `--playground-url`：控制操场的位置

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --playground-url https://play.rust-lang.org/
```

渲染crate文档时，此标志提供Rust Playground的基本URL，用于生成`Run`纽扣。不像`--markdown-playground-url`，此参数适用于独立的Markdown文件*和*铁锈箱。这与添加的方式相同`#![doc(html_playground_url =
"url")]`如你所述，你的箱子根[关于这个的章节`#[doc]`属性][doc-playground]。请注意正式的Rust Playground<https://play.rust-lang.org>没有可用的箱子，所以如果你的例子需要你的箱子，请确保你提供的游乐场有你的箱子。

[doc-playground]: the-doc-attribute.html#html_playground_url

如果两者`--playground-url`和`--markdown-playground-url`在呈现独立的Markdown文件时出现，URL是给定的`--markdown-playground-url`将优先考虑。如果两者`--playground-url`和`#![doc(html_playground_url = "url")]`在渲染包文档时存在，该属性将优先。

### `--crate-version`：控制箱子版本

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --crate-version 1.3.37
```

什么时候`rustdoc`收到这个标志后，它会在包根文档的侧边栏中打印一个额外的“版本（版本）”。您可以使用此标志来区分库文档的不同版本。

### `--linker`：控制用于文档测试的链接器

使用此标志如下所示：

```bash
$ rustdoc --test src/lib.rs -Z unstable-options --linker foo
$ rustdoc --test README.md -Z unstable-options --linker foo
```

什么时候`rustdoc`运行您的文档测试，它需要在运行它们之前编译并将测试链接为可执行文件。此标志可用于更改这些可执行文件上使用的链接器。这相当于传球`-C linker=foo`至`rustc`。

### `--sort-modules-by-appearance`：控制模块页面上的项目的排序方式

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --sort-modules-by-appearance
```

通常，什么时候`rustdoc`在模块页面中打印项目，它将按字母顺序排序（考虑它们的稳定性，以及以数字结尾的名称）。给这个标志`rustdoc`将禁用此排序，而是使其按照它们在源中出现的顺序打印项目。

### `--themes`：提供其他主题

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --themes theme.css
```

给这个标志`rustdoc`将使您的主题复制到生成的包文档中并在主题选择器中启用它。注意`rustdoc`将拒绝您的主题文件，如果它没有样式“轻”主题的样式。看到`--theme-checker`以下是详情。

### `--theme-checker`：验证主题CSS的有效性

使用此标志如下所示：

```bash
$ rustdoc -Z unstable-options --theme-checker theme.css
```

在将您的主题包含在包文档中之前，`rustdoc`将它包含的所有CSS规则与默认包含的“light”主题进行比较。使用此标志将允许您查看缺少哪些规则`rustdoc`拒绝你的主题。

### `--resource-suffix`：修改包文档中的CSS / JavaScript名称

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --resource-suffix suf
```

渲染文档时，`rustdoc`创建几个CSS和JavaScript文件作为输出的一部分。由于所有这些文件都是从每个页面链接的，因此如果您需要专门缓存它们，更改它们的位置可能会很麻烦。此标志将重命名输出中的所有这些文件，以在文件名中包含后缀。例如，`light.css`会成为`light-suf.css`用上面的命令。

### `--display-warnings`：记录或运行文档测试时显示警告

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --display-warnings
$ rustdoc --test src/lib.rs -Z unstable-options --display-warnings
```

此标志背后的意图是允许用户查看其库或其文档测试中发生的警告，这些警告通常会被抑制。然而，[由于一个错误][issue-display-warnings]，这个标志不是100％按预期工作。有关详细信息，请参阅链接问

[issue-display-warnings]: https://github.com/rust-lang/rust/issues/41574

### `--extern-html-root-url`：控制rustdoc如何链接到非本地包

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z unstable-options --extern-html-root-url some-crate=https://example.com/some-crate/1.0.1
```

通常，当rustdoc想要链接到来自不同包的类型时，它会在两个地方查找：输出目录中已存在的文档，或者`#![doc(doc_html_root)]`放在另一个箱子里。但是，如果要链接到这两个位置中都不存在的文档，可以使用这些标志来控制该行为。当。。。的时候`--extern-html-root-url`使用与您的某个依赖项匹配的名称给出flag，rustdoc使用这些文档的URL。请记住，如果这些文档存在于输出目录中，那些本地文档仍将覆盖此标志。

### `-Z force-unstable-if-unmarked`

使用此标志如下所示：

```bash
$ rustdoc src/lib.rs -Z force-unstable-if-unmarked
```

这是一个内部标志，用于标准库和应用程序的编译器`#[unstable]`属性为任何没有其他稳定属性的依赖包。这允许`rustdoc`能够为编译器包和标准库生成文档，因为提供了等效的命令行参数`rustc`在建造这些板条箱时。

### `--index-page`：为文档提供顶级登录页面

此功能允许您生成带有给定markdown文件的索引页面。一个很好的例子是[生锈文件索引](https://doc.rust-lang.org/index.html)。

有了这个，你将有一个页面，你可以定制尽可能多的在你的板条箱顶部。

使用`index-page`选项启用`enable-index-page`也可以选择。

### `--enable-index-page`：为文档生成默认索引页

此功能允许生成列出生成板条箱的默认索引页。

### `--static-root-path`：控制静态文件在HTML输出中的加载方式

使用此标志如下：

```bash
$ rustdoc src/lib.rs -Z unstable-options --static-root-path '/cache/'
```

此标志控制RustDoc如何链接到HTML页面上的静态文件。如果您托管了许多由同一版本的RustDoc生成的板条箱文档，则可以使用此标志将RustDoc的CSS、javascript和字体文件缓存在单个位置，而不是每个“Doc根”（将生成的板条箱文档分组到同一输出目录，如`cargo doc`）每个板条箱文件（如搜索索引）仍将从文档根目录加载，但重命名为`--resource-suffix`将从给定路径加载。

### `--persist-doctests`：运行后保留doctest可执行文件

使用此标志如下：

```bash
$ rustdoc src/lib.rs --test -Z unstable-options --persist-doctests target/rustdoctest
```

此标志允许您在编译或运行doctest可执行文件之后保留它们。通常，RustDoc会在测试后立即丢弃编译好的doctest，但使用此选项，您可以保留这些二进制文件以进行进一步的测试。

### `--show-coverage`：使用文档计算项目百分比

使用此标志如下：

```bash
$ rustdoc src/lib.rs -Z unstable-options --show-coverage
```

如果您想确定板条箱中记录了多少项目，请将此标志传递给RustDoc。当它收到这个标志时，它将统计您的板条箱中有文档的公共项目，并打印出计数和百分比，而不是生成文档。

一些方法说明了RustDoc在该指标中的重要性：

-   RustDoc只计算您板条箱中的物品（即从其他板条箱重新出口的物品不计算在内）。
-   直接写入固有IMPL块的文档不计算在内，即使显示了它们的文档注释，因为Rust代码中的常见模式是将所有固有方法写入同一IMPL块。
-   特性实现中的项不计算在内，因为这些impl将从特性本身继承任何文档。
-   默认情况下，只计算公共项。要计算私人物品，请通过`--document-private-items`同时。

未记录的公共项目可以通过内置的`missing_docs`绒布未记录的私人物品可通过clippy查看。`missing_docs_in_private_items`绒布
