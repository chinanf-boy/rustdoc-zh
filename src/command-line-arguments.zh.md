# 命令行参数

这是您可以传递给`rustdoc`的参数列表：

## `-h`/`--help`：帮助

使用此标志如下：

```bash
$ rustdoc -h
$ rustdoc --help
```

这将显示`rustdoc`的内置帮助，主要由能用的命令行标志列表组成。

会有一些`rustdoc`的标志参数不稳定；此页仅显示稳定的选项，`--help`则会显示全部。

## `-V`/`--version`：版本信息

使用此标志如下：

```bash
$ rustdoc -V
$ rustdoc --version
```

这将显示`rustdoc`的版本，其外观如下：

```text
rustdoc 1.17.0 (56124baa9 2017-04-24)
```

## `-v`/`--verbose`：更详细的输出

使用此标志如下：

```bash
$ rustdoc -v src/lib.rs
$ rustdoc --verbose src/lib.rs
```

这将启用“详细模式”，这意味着更多信息将写入标准输出。所写的内容取决于您传入的其他标志。例如，使用`--version`：

```text
$ rustdoc --verbose --version
rustdoc 1.17.0 (56124baa9 2017-04-24)
binary: rustdoc
commit-hash: hash
commit-date: date
host: host-triple
release: 1.17.0
LLVM version: 3.9
```

## `-r`/`--input-format`：输入格式

此标志当前被忽略；想法是`rustdoc`将支持各种输入格式，您可以通过这个标志来指定它们。

RustDoc 只支持 Rust 源代码和 markdown 输入格式。如果文件以`.md`或`.markdown`，`rustdoc`将其视为 markdown 文件。否则，它假定输入文件是 Rust 的。

## `-w`/`--output-format`：输出格式

此标志当前被忽略；其想法是`rustdoc`将支持各种输出格式，您可以通过此标志指定它们。

RustDoc 只支持 HTML 输出，所以这个标志在今天来说是多余的。

## `-o`/`--output`：输出路径

使用此标志如下：

```bash
$ rustdoc src/lib.rs -o target\\doc
$ rustdoc src/lib.rs --output target\\doc
```

默认情况下，`rustdoc`的输出显示在当前工作目录中的`doc`目录。使用这个标志，它将把所有输出放到您指定的目录中。

## `--crate-name`：控制箱子的名称

使用此标志如下：

```bash
$ rustdoc src/lib.rs --crate-name mycrate
```

默认情况下，`rustdoc`假设箱子的名称与`.rs`文件名一样。`--crate-name`允许您用您选择的任何名称覆盖这个假设。

## `-L`/`--library-path`：查找依赖项的位置

使用此标志如下：

```bash
$ rustdoc src/lib.rs -L target/debug/deps
$ rustdoc src/lib.rs --library-path target/debug/deps
```

如果你的箱子有依赖性，`rustdoc`需要知道在哪里找到他们。传递`--library-path`参数，让`rustdoc`可以查找这些依赖项的位置列表。

此标志接受任意数量的目录作为参数，并在搜索时，使用所有文档。

## `--cfg`：传递配置标志

使用此标志如下：

```bash
$ rustdoc src/lib.rs --cfg feature="foo"
```

此标志接受的值与`rustc --cfg`一样，也可以用来配置编译。上面的示例使用`feature`，但任何的`cfg`值都是可以接受的。

## `--extern`：指定依赖项的位置

使用此标志如下：

```bash
$ rustdoc src/lib.rs --extern lazy-static=/path/to/lazy-static
```

类似`--library-path`，`--extern`是关于指定依赖项的位置。`--library-path`提供要搜索的目录，`--extern`相反，您可以确切地指定依赖项位于何处。

## `-C`/`--codegen`：将 codegen 选项传递给 rustc

使用此标志如下：

```bash
$ rustdoc src/lib.rs -C target_feature=+avx
$ rustdoc src/lib.rs --codegen target_feature=+avx

$ rustdoc --test src/lib.rs -C target_feature=+avx
$ rustdoc --test src/lib.rs --codegen target_feature=+avx

$ rustdoc --test README.md -C target_feature=+avx
$ rustdoc --test README.md --codegen target_feature=+avx
```

当 RustDoc 生成文档、查找文档测试或执行文档测试时，它需要编译一些 Rust 代码，至少是部分编译。此标志允许您告诉 RustDoc 在运行这些编译时，向 RustC 提供一些额外的 codegen 选项。大多数情况下，这些选项不会影响常规文档运行，但如果某些内容会依赖要启用的目标功能(target feature)，或者文档测试需要使用一些其他选项，这时，这个标志就能让您影响这些内容。

此标志的参数与 rustc 上的`-C`标志一样。运行`rustc -C help`获取完整的列表。

## `--passes`：添加更多 RustDoc psses（通行证）

使用此标志如下：

```bash
$ rustdoc --passes list
$ rustdoc src/lib.rs --passes strip-priv-imports
```

“list”参数将打印一个可能的“rustdoc passes”列表，除默认值外，其他参数则是功能与其名。

有关通行证的详细信息，请参见[关于它们的章节](passes.zh.md).

另请参见`--no-defaults`.

## `--no-defaults`：不运行默认通行证

使用此标志如下：

```bash
$ rustdoc src/lib.rs --no-defaults
```

默认情况下，`rustdoc`在代码上运行多个通行证。此标志就是删除这些默认值的，让你自行重新传递`--passes`，以准确指定所需的通行证。

有关通行证的详细信息，请参见[关于它们的章节](passes.zh.md).

另请参见`--passes`.

## `--test`：将代码示例作为测试运行

使用此标志如下：

```bash
$ rustdoc src/lib.rs --test
```

此标志将运行代码示例作为测试。有关更多信息，请参阅[文档测试章节](documentation-tests.zh.md)。

另请参见`--test-args`.

## `--test-args`：传递选项给测试运行程序

使用此标志如下：

```bash
$ rustdoc src/lib.rs --test --test-args ignored
```

此标志将在运行文档测试时，将选项传递给测试运行程序。有关更多信息，请参阅[文档测试章节](documentation-tests.zh.md)。

另请参见`--test`.

## `--target`：为指定的三元目标，生成文档

使用此标志如下：

```bash
$ rustdoc src/lib.rs --target x86_64-pc-windows-gnu
```

类似于`rustc`的`--target`标志，生成指定三元目标的文档。

所有跨平台编译代码的常见注意事项都适用。

## `--markdown-css`：渲染 markdown 时，包含更多 CSS 文件

使用此标志如下：

```bash
$ rustdoc README.md --markdown-css foo.css
```

渲染 markdown 文件时，这将在生成 HTML 的`<head>`元素内，创建一个`<link>`元素。例如，上面的调用会生成，

```html
<link rel="stylesheet" type="text/css" href="foo.css" />
```

添加到 HTML。

渲染 Rust 文件时，此标志将被忽略。

## `--html-in-header`：在 <head> 中 包含更多 HTML

使用此标志如下：

```bash
$ rustdoc src/lib.rs --html-in-header header.html
$ rustdoc README.md --html-in-header header.html
```

此标志获取文件列表，并将其插入到渲染文档的`<head>`。

## `--html-before-content`：在 content 之前，包含更多 HTML

使用此标志如下：

```bash
$ rustdoc src/lib.rs --html-before-content extra.html
$ rustdoc README.md --html-before-content extra.html
```

此标志获取文件列表，并将其插入到渲染文档的`<body>`标记元素，但排在其他 content(内容) 之前。

## `--html-after-content`：在 content 之后，包含更多 HTML

使用此标志如下：

```bash
$ rustdoc src/lib.rs --html-after-content extra.html
$ rustdoc README.md --html-after-content extra.html
```

此标志获取文件列表，并将其插入到渲染文档的`</body>`标记元素，但排在其他 content 之后。

## `--markdown-playground-url`：控制游乐场的位置

使用此标志如下：

```bash
$ rustdoc README.md --markdown-playground-url https://play.rust-lang.org/
```

在渲染 markdown 文件时，此标志提供 Rust Playground 的基 URL，用于生成`Run`按钮。

## `--markdown-no-toc`：不生成文件目录超链接

使用此标志如下：

```bash
$ rustdoc README.md --markdown-no-toc
```

从 markdown 文件生成文档时，默认情况下，`rustdoc`将生成目录。此标志禁止此操作，不会生成 TOC（目录超链接）。

## `-e`/`--extend-css`：扩展 RustDoc 的 CSS

使用此标志如下：

```bash
$ rustdoc src/lib.rs -e extra.css
$ rustdoc src/lib.rs --extend-css extra.css
```

使用此标志，您传递文件的内容，将添加到 RustDoc `theme.css`文件的下面。

当此标志稳定时，`theme.css`的内容会不在了，所以小心点！更新可能会破坏你的主题扩展。

## `--sysroot`：重写系统根目录

使用此标志如下：

```bash
$ rustdoc src/lib.rs --sysroot /path/to/sysroot
```

类似`rustc --sysroot`，这允许您在编译代码时，更改`rustdoc`使用的 sysroot。

### `--edition`：控制文档和文档测试的版本(edition)

使用此标志如下：

```bash
$ rustdoc src/lib.rs --edition 2018
$ rustdoc --test src/lib.rs --edition 2018
```

此标志允许 RustDoc 将您的 Rust 代码视为给定版本。它也将用给定的版本编译文档测试。与`rustc`一样，`rustdoc`使用的默认版本是`2015`（第一版）。
