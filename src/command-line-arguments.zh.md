# 命令行参数

这是您可以传递给的参数列表`rustdoc`：

## `-h`/`--help`帮助

使用此标志如下：

```bash
$ rustdoc -h
$ rustdoc --help
```

这将显示`rustdoc`的内置帮助，主要由可能的命令行标志列表组成。

一些`rustdoc`的标志不稳定；此页仅显示稳定的选项，`--help`将向他们展示所有。

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

此标志当前被忽略；其思想是`rustdoc`将支持各种输入格式，您可以通过这个标志来指定它们。

RustDoc只支持Rust源代码和降价输入格式。如果文件以`.md`或`.markdown`，`rustdoc`将其视为降价文件。否则，它假定输入文件是生锈的。

## `-w`/`--output-format`：输出格式

此标志当前被忽略；其思想是`rustdoc`将支持各种输出格式，您可以通过此标志指定它们。

RustDoc只支持HTML输出，所以这个标志在今天是多余的。

## `-o`/`--output`：输出路径

使用此标志如下：

```bash
$ rustdoc src/lib.rs -o target\\doc
$ rustdoc src/lib.rs --output target\\doc
```

默认情况下，`rustdoc`的输出显示在名为`doc`在当前工作目录中。使用这个标志，它将把所有输出放到您指定的目录中。

## `--crate-name`：控制板条箱的名称

使用此标志如下：

```bash
$ rustdoc src/lib.rs --crate-name mycrate
```

默认情况下，`rustdoc`假设板条箱的名称与`.rs`文件。`--crate-name`允许您用您选择的任何名称覆盖此假设。

## `-L`/`--library-path`：查找依赖项的位置

使用此标志如下：

```bash
$ rustdoc src/lib.rs -L target/debug/deps
$ rustdoc src/lib.rs --library-path target/debug/deps
```

如果你的板条箱有依赖性，`rustdoc`需要知道在哪里找到他们。经过`--library-path`给予`rustdoc`查找这些依赖项的位置列表。

此标志接受任意数量的目录作为参数，并在搜索时使用所有目录。

## `--cfg`：传递配置标志

使用此标志如下：

```bash
$ rustdoc src/lib.rs --cfg feature="foo"
```

此标志接受的值与`rustc --cfg`，并使用它来配置编译。上面的示例使用`feature`但是`cfg`值可以接受。

## `--extern`：指定依赖项的位置

使用此标志如下：

```bash
$ rustdoc src/lib.rs --extern lazy-static=/path/to/lazy-static
```

类似`--library-path`，`--extern`是关于指定依赖项的位置。`--library-path`提供要搜索的目录，`--extern`相反，您可以确切地指定依赖项位于何处。

## `-C`/`--codegen`：将codegen选项传递给rustc

使用此标志如下：

```bash
$ rustdoc src/lib.rs -C target_feature=+avx
$ rustdoc src/lib.rs --codegen target_feature=+avx

$ rustdoc --test src/lib.rs -C target_feature=+avx
$ rustdoc --test src/lib.rs --codegen target_feature=+avx

$ rustdoc --test README.md -C target_feature=+avx
$ rustdoc --test README.md --codegen target_feature=+avx
```

当RustDoc生成文档、查找文档测试或执行文档测试时，它需要编译一些Rust代码，至少是部分编译。此标志允许您告诉RustDoc在运行这些编译时向RustC提供一些额外的codegen选项。大多数情况下，这些选项不会影响常规文档运行，但如果某些内容依赖于要启用的目标功能，或者文档测试需要使用一些其他选项，则此标志允许您影响这些内容。

此标志的参数与`-C`Rustc上的标志。跑`rustc -C help`获取完整的列表。

## `--passes`：添加更多RustDoc通行证

使用此标志如下：

```bash
$ rustdoc --passes list
$ rustdoc src/lib.rs --passes strip-priv-imports
```

“list”参数将打印一个可能的“rustdoc passes”列表，除默认值外，其他参数将是要运行的passes的名称。

有关通行证的详细信息，请参见[关于它们的章节](passes.md).

另请参见`--no-defaults`.

## `--no-defaults`：不运行默认过程

使用此标志如下：

```bash
$ rustdoc src/lib.rs --no-defaults
```

默认情况下，`rustdoc`将对代码进行多次传递。这将删除这些默认值，允许您使用`--passes`以准确指定所需的过程。

有关通行证的详细信息，请参见[关于它们的章节](passes.md).

另请参见`--passes`.

## `--test`：将代码示例作为测试运行

使用此标志如下：

```bash
$ rustdoc src/lib.rs --test
```

此标志将运行代码示例作为测试。有关更多信息，请参阅[文件测试章节](documentation-tests.md).

另请参见`--test-args`.

## `--test-args`：通过测试运行程序选项

使用此标志如下：

```bash
$ rustdoc src/lib.rs --test --test-args ignored
```

此标志将在运行文档测试时将选项传递给测试运行程序。有关更多信息，请参阅[文件测试章节](documentation-tests.md).

另请参见`--test`.

## `--target`：为指定的目标三重生成文档

使用此标志如下：

```bash
$ rustdoc src/lib.rs --target x86_64-pc-windows-gnu
```

类似于`--target`标志`rustc`，这将为不同于主机三倍的目标三倍生成文档。

交叉编译代码的所有常见注意事项都适用。

## `--markdown-css`：呈现降价时包含更多CSS文件

使用此标志如下：

```bash
$ rustdoc README.md --markdown-css foo.css
```

渲染降价文件时，这将创建一个`<link>`中的元素`<head>`生成的HTML的节。例如，使用上面的调用，

```html
<link rel="stylesheet" type="text/css" href="foo.css">
```

将添加。

渲染防锈文件时，此标志将被忽略。

## `--html-in-header`：在中包含更多HTML<head>

使用此标志如下：

```bash
$ rustdoc src/lib.rs --html-in-header header.html
$ rustdoc README.md --html-in-header header.html
```

此标志获取文件列表，并将其插入到`<head>`提供的文档的节。

## `--html-before-content`：在内容之前包含更多HTML

使用此标志如下：

```bash
$ rustdoc src/lib.rs --html-before-content extra.html
$ rustdoc README.md --html-before-content extra.html
```

此标志获取文件列表，并将其插入到`<body>`标记，但在其他内容之前`rustdoc`通常会在呈现的文档中生成。

## `--html-after-content`：在内容后包含更多HTML

使用此标志如下：

```bash
$ rustdoc src/lib.rs --html-after-content extra.html
$ rustdoc README.md --html-after-content extra.html
```

此标志获取文件列表，并在`</body>`标记，但在其他内容之后`rustdoc`通常会在呈现的文档中生成。

## `--markdown-playground-url`：控制操场的位置

使用此标志如下：

```bash
$ rustdoc README.md --markdown-playground-url https://play.rust-lang.org/
```

在呈现降价文件时，此标志提供Rust Playground的基本URL，用于生成`Run`按钮。

## `--markdown-no-toc`：不生成目录

使用此标志如下：

```bash
$ rustdoc README.md --markdown-no-toc
```

从降价文件生成文档时，默认情况下，`rustdoc`将生成目录。此标志禁止此操作，不会生成TOC。

## `-e`/`--extend-css`：扩展RustDoc的CSS

使用此标志如下：

```bash
$ rustdoc src/lib.rs -e extra.css
$ rustdoc src/lib.rs --extend-css extra.css
```

使用此标志，您传递的文件的内容将包含在RustDoc的底部`theme.css`文件。

当此标志稳定时，`theme.css`不是，所以小心点！更新可能会破坏主题扩展。

## `--sysroot`：重写系统根目录

使用此标志如下：

```bash
$ rustdoc src/lib.rs --sysroot /path/to/sysroot
```

类似`rustc --sysroot`，这允许您更改sysroot`rustdoc`在编译代码时使用。

### `--edition`：控制文档和文档的版本

使用此标志如下：

```bash
$ rustdoc src/lib.rs --edition 2018
$ rustdoc --test src/lib.rs --edition 2018
```

此标志允许RustDoc将您的锈代码视为给定版本。它也将用给定的版本编译教义。和…一样`rustc`，默认版本`rustdoc`将使用的是`2015`（第一版）。
