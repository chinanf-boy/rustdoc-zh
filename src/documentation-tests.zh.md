# 文档测试

`rustdoc`支持执行文档示例，作为测试。这可以确保您的测试是最新的和有效的。

基本的想法是这样的：

````ignore
/// # Examples
///
/// ```
/// let x = 5;
/// ```
````

三个反引号开始和结束的代码块。如果这是在一个名为`foo.rs`的文件中，运行`rustdoc --test foo.rs`将提取此示例，然后将其作为测试运行。

请注意，默认情况下，如果没有为代码块设置语言，`rustdoc`假设它是`Rust`码。所以，以下内容：

````markdown
```rust
let x = 5;
```
````

严格等同于：

````markdown
```
let x = 5;
```
````

虽然有些微妙！但请继续。

## 通过或未通过一个 doctest(文档测试)

与常规单元测试一样，如果编译和运行没有恐慌，常规 doctests 被认为是“通过”。因此，如果你想证明一些计算给出一定的结果，那么`assert!`宏家族的工作与常规 Rust 代码相同：

```rust
let foo = "foo";

assert_eq!(foo, "foo");
```

这样，如果计算返回不同的东西，代码恐慌，并且 doctest 失败。

## 预处理示例

在上面的例子中，你会注意到一些奇怪的事情：没有`main`函数！若是强迫你在每个示例中，都要写`main`，无论这举动多小，都不太懒人化。所以`rustdoc`会在运行之前稍微处理您的示例。这就是 rustdoc 预处理示例的完整算法：

1.  插入些常见`allow`属性，包括`unused_variables`，`unused_assignments`，`unused_mut`，`unused_attributes`，和`dead_code`。小例子经常触发这些 lint。
2.  使用`#![doc(test(attr(...)))]`指定的任何属性，会被添加。
3.  任何`#![foo]`前缀属性保留为 箱子属性。
4.  如果示例不包含`extern crate`，和没有指定`#![doc(test(no_crate_inject))]`，那就插入`extern crate <mycrate>;`（注意`#[macro_use]`的缺乏）。
5.  最后，如果示例不包含`fn main`，文本的其余部分包含在`fn main() { your_code }`。

有关规则 4 中警告的更多信息，请参阅下面的“宏文档”。

## 隐藏示例的部分内容

有时，您需要一些设置代码，或其他会分散您示例，但对有效测试非常重要的的内容。考虑一个如下所示的示例块：

```text
/// /// Some documentation.
/// # fn foo() {} // this function will be hidden
/// println!("Hello, World!");
```

它将呈现如下：

```rust
/// Some documentation.
# fn foo() {}
println!("Hello, World!");
```

是的，这是正确的：你可以添加以`#`开头的行，它们将从输出中隐藏，但在编译代码时将被使用。你可以利用这个优势。设想下，当文档注释需要应用于某种函数，所以如果我只想向您展示文档注释，我需要在它下面添加一个小函数定义。同时，它只是满足编译器，所以隐藏它，会使示例更清晰。您可以使用此技术，详细解释更长的示例，同时仍保留文档的可测试性。

例如，假设我们想要文档化此代码：

```rust
let x = 5;
let y = 6;
println!("{}", x + y);
```

我们可能希望文档最终看起来像这样：

> 首先，我们设定`x`到 5：
>
> ```rust
> let x = 5;
> # let y = 6;
> # println!("{}", x + y);
> ```
>
> 接下来，我们设置`y`到 6：
>
> ```rust
> # let x = 5;
> let y = 6;
> # println!("{}", x + y);
> ```
>
> 最后，我们打印出的总和`x`和`y`：
>
> ```rust
> # let x = 5;
> # let y = 6;
> println!("{}", x + y);
> ```

为了保持每个代码块可测试，我们希望每个块，都有整个程序，但我们不希望读者每次都看到每一行。这是我们在源代码中添加的内容：

````markdown
First, we set `x` to five:

```
let x = 5;
# let y = 6;
# println!("{}", x + y);
```

Next, we set `y` to six:

```
# let x = 5;
let y = 6;
# println!("{}", x + y);
```

Finally, we print the sum of `x` and `y`:

```
# let x = 5;
# let y = 6;
println!("{}", x + y);
```
````

通过重复示例的所有部分，您可以确保您的示例仍然编译，同时仅显示与您的解释部分相关的部分。

通过使用两个连续的`##`哈希值，可以防止隐藏。这只需要用第一个完成`#`否则会导致隐藏。如果我们有一个像下面这样的字符串文字，它有一行是以一个`#`开头的字符串：

```rust
let s = "foo
## bar # baz";
```

我们可以通过转义头个`#`来文档化它：

```text
/// let s = "foo
/// ## bar # baz";
```

## 在文档测试中运用`?`

在编写示例时，包含完整的错误处理很少有用，因为它会添加大量的样板代码。相反，您可能需要以下内容：

````ignore
/// ```
/// use std::io;
/// let mut input = String::new();
/// io::stdin().read_line(&mut input)?;
/// ```
````

问题是，`?`会返回一个`Result<T, E>`，而测试函数是不返回任何内容，因此这会给出不匹配的类型错误。

你可以通过手动添加一个`main`，它返回`Result<T, E>`，因为`Result<T, E>`实现`Termination`trait，自然解决了这个限制：

````ignore
/// A doc test using ?
///
/// ```
/// use std::io;
///
/// fn main() -> io::Result<()> {
///     let mut input = String::new();
///     io::stdin().read_line(&mut input)?;
///     Ok(())
/// }
/// ```
````

再加上从上面的`#`部分，您可以得出一个解决方案，在读者看来它就是最初的想法，但可以与文档测试一起使用：

````ignore
/// ```
/// use std::io;
/// # fn main() -> io::Result<()> {
/// let mut input = String::new();
/// io::stdin().read_line(&mut input)?;
/// # Ok(())
/// # }
/// ```
````

从 1.34.0 版开始，还可以省略`fn main()`，但必须消除错误类型的歧义：

````ignore
/// ```
/// use std::io;
/// let mut input = String::new();
/// io::stdin().read_line(&mut input)?;
/// # Ok::<(), io::Error>(())
/// ```
````

`?`运算符要添加隐式转换，是一个不幸的结果。是因为类型不是唯一的，所以不这样的话，会导致类型推断失败。请注意，您必须写下`(())`序列，中间没有空格，以便 RustDoc 了解您需要一个隐式`Result`——返回函数。

从 1.37.0 版开始，这种简化也适用于`Option`s，可以方便地测试，例如迭代器或校验算术，例如：

````ignore
/// ```
/// let _ = &[].iter().next()?;
///# Some(())
/// ```
````

请注意，结果必须是`Some(())`，且必须一次。在这种情况下，不需要消除结果的歧义。

## 宏的文档

以下是一个宏的文档示例：

````rust
/// 除非表达式正确，不然就信息性恐慌
///
/// # Examples
///
/// ```
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(1 + 1 == 2, “Math is broken.”);
/// # }
/// ```
///
/// ```should_panic
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(true == false, “I’m broken.”);
/// # }
/// ```
#[macro_export]
macro_rules! panic_unless {
    ($condition:expr, $($rest:expr),+) => ({ if ! $condition { panic!($($rest),+); } });
}
# fn main() {}
````

你会注意到三件事：

- 1，我们需要增加我们自己的`extern crate`行，以便我们可以添加`#[macro_use]`属性。
- 2，我们需要增加我们自己的`main()`以及（出于上述原因）。
- 3，最后，明智地使用`#`注释掉这两件事，这样它们就不会出现在输出中。

## 属性

有一些注释对`rustdoc`正确测试代码的事情上有帮助：

````rust
/// ```ignore
/// fn foo() {
/// ```
# fn foo() {}
````

这个`ignore`指令告诉 Rust 忽略您的代码。大概是你不想要的(代码)，因为它太普通了。相反，如果不是代码，考虑用`text`文本，或者使用`#`得到一个只显示你关心的部分的工作示例。

````rust
/// ```should_panic
/// assert!(false);
/// ```
# fn foo() {}
````

`should_panic`讲述`rustdoc`代码应该正确编译，但不能作为测试通过。

````text
/// ```no_run
/// loop {
///     println!("Hello, world");
/// }
/// ```
# fn foo() {}
````

这个`no_run`属性将编译代码，但不运行它。这对“以下是如何检索网页”等示例很重要，您可能希望确保编译，但可能是在没有网络访问的测试环境中运行。

````text
/// ```compile_fail
/// let x = 5;
/// x += 2; // shouldn't compile!
/// ```
````

`compile_fail`讲述`rustdoc`编译应该失败。如果它编译，那么测试将失败。但是，请注意，随着新特性的添加，在当前 Rust 标准版失败的代码，可能在将来的版本中工作。

````text
/// 只在 2018 edition 运行.
///
/// ```edition2018
/// let result: Result<i32, ParseIntError> = try {
///     "1".parse::<i32>()?
///         + "2".parse::<i32>()?
///         + "3".parse::<i32>()?
/// };
/// ```
````

`edition2018`讲述`rustdoc`代码样本，应在 2018 版 Rust 下编译。同样，您可以指定`edition2015`以 2015 年版编制本规范。

## 语法参考

*准确的*代码块（包括边缘大小写）的语法可以在通用标记规范的[用栅栏围起来的代码块](https://spec.commonmark.org/0.28/#fenced-code-blocks)章节找到。

RustDoc 也接受*缩进*代码块，作为栅栏代码块的替代：您可以将每行缩进四个或更多的空格，而不是用三个反勾号包围代码。

```markdown
    let foo = "foo";
    assert_eq!(foo, "foo");
```

这些也记录在通用标记规范中，[缩进代码块](https://spec.commonmark.org/0.28/#indented-code-blocks)章节。

但是，最好使用栅栏代码块，而不是缩进代码块。不仅仅是栅栏代码块被认为是 Rust 代码的习惯，还因为缩进代码块没有办法使用诸如`ignore`或`should_panic`。
