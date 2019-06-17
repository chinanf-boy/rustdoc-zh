# 文档测试

`rustdoc`支持执行文档示例作为测试。这可以确保您的测试是最新的和有效的。

基本的想法是这样的：

````ignore
/// # Examples
///
/// ```
/// let x = 5;
/// ```
````

三重反引号开始和结束代码块。如果这是在一个名为的文件中`foo.rs`，跑步`rustdoc --test foo.rs`将提取此示例，然后将其作为测试运行。

请注意，默认情况下，如果没有为块代码设置语言，`rustdoc`假设它是`Rust`码。所以以下内容：

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

虽然有些微妙！请阅读以获得更多详情。

## 通过或未通过doctest

与常规单元测试一样，如果编译和运行没有恐慌，常规doctests被认为是“通过”。因此，如果你想证明一些计算给出一定的结果，那么`assert!`宏系列的工作原理与其他Rust代码相同：

```rust
let foo = "foo";

assert_eq!(foo, "foo");
```

这样，如果计算返回不同的东西，代码恐慌并且doctest失败。

## 预处理示例

在上面的例子中，你会注意到一些奇怪的事情：没有`main`功能！强迫你写`main`对于每个例子，无论多小，都会增加摩擦力。所以`rustdoc`在运行之前稍微处理您的示例。这是rustdoc用于预处理示例的完整算法：

1.  有些常见`allow`插入属性，包括`unused_variables`，`unused_assignments`，`unused_mut`，`unused_attributes`，和`dead_code`。小例子经常触发这些lint。
2.  使用指定的任何属性`#![doc(test(attr(...)))]`被添加。
3.  任何领先`#![foo]`属性保留为crate属性。
4.  如果示例不包含`extern crate`，和`#![doc(test(no_crate_inject))]`那时候没有说明`extern crate
    <mycrate>;`插入（注意缺乏`#[macro_use]`）。
5.  最后，如果示例不包含`fn main`，文本的其余部分包含在内`fn main() { your_code }`。

有关规则4中的警告的更多信息，请参阅下面的“记录宏”。

## 隐藏示例的部分内容

有时，您需要一些设置代码或其他会分散您示例的内容，但对于使测试有效非常重要。考虑一个如下所示的示例块：

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

是的，这是正确的：你可以添加以开头的行`#`，它们将从输出中隐藏，但在编译代码时将被使用。你可以利用这个优势。在这种情况下，文档注释需要应用于某种函数，所以如果我只想向您展示文档注释，我需要在它下面添加一个小函数定义。同时，它只是满足编译器，所以隐藏它使示例更清晰。您可以使用此技术详细解释更长的示例，同时仍保留文档的可测试性。

例如，假设我们想要记录此代码：

```rust
let x = 5;
let y = 6;
println!("{}", x + y);
```

我们可能希望文档最终看起来像这样：

> 首先，我们设定`x`到五：
>
> ```rust
> let x = 5;
> # let y = 6;
> # println!("{}", x + y);
> ```
>
> 接下来，我们设置`y`到六：
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

为了保持每个代码块可测试，我们希望每个块中的整个程序，但我们不希望读者每次都看到每一行。这是我们在源代码中添加的内容：

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

该`#`通过使用两个连续的哈希值可以防止隐藏线条`##`。这只需要用第一个完成`#`否则会导致隐藏。如果我们有一个像下面这样的字符串文字，它有一行以a开头`#`：

```rust
let s = "foo
## bar # baz";
```

我们可以通过逃避初始来记录它`#`：

```text
/// let s = "foo
/// ## bar # baz";
```

## 运用`?`在doc测试中

在编写示例时，包含完整的错误处理很少有用，因为它会添加大量的样板代码。相反，您可能需要以下内容：

````ignore
/// ```
/// use std::io;
/// let mut input = String::new();
/// io::stdin().read_line(&mut input)?;
/// ```
````

问题是`?`返回一个`Result<T, E>`和测试函数不返回任何内容，因此这将给出不匹配的类型错误。

你可以通过手动添加一个来解决这个限制`main`返回`Result<T, E>`因为`Result<T, E>`实现`Termination`特征：

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

再加上`#`从上面的部分中，您可以得出一个解决方案，在读者看来它是最初的想法，但可以与文档测试一起使用：

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

从1.34.0版开始，还可以省略`fn main()`，但必须消除错误类型的歧义：

````ignore
/// ```
/// use std::io;
/// let mut input = String::new();
/// io::stdin().read_line(&mut input)?;
/// # Ok::<(), io::Error>(())
/// ```
````

这是一个不幸的结果`?`运算符添加隐式转换，因此类型推断失败，因为类型不是唯一的。请注意，您必须`(())`在一个没有中间空白的序列中，以便RustDoc了解您需要一个隐式`Result`-返回函数。

从1.37.0版开始，这种简化也适用于`Option`s，可以方便地测试，例如迭代器或校验算术，例如：

````ignore
/// ```
/// let _ = &[].iter().next()?;
///# Some(())
/// ```
````

请注意，结果必须是`Some(())`这必须一次完成。在这种情况下，不需要消除结果的歧义。

## 记录宏

以下是记录宏的示例：

````rust
/// Panic with a given message unless an expression evaluates to true.
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

你会注意到三件事：我们需要增加我们自己的`extern crate`行，以便我们可以添加`#[macro_use]`属性。第二，我们需要增加我们自己的`main()`以及（出于上述原因）。最后，明智地使用`#`注释掉这两件事，这样它们就不会出现在输出中。

## 属性

有一些注释对帮助`rustdoc`测试代码时要做正确的事情：

````rust
/// ```ignore
/// fn foo() {
/// ```
# fn foo() {}
````

这个`ignore`指令告诉Rust忽略您的代码。这几乎不是你想要的，因为它是最通用的。相反，考虑用注释`text`如果不是代码，或者使用`#`得到一个只显示你关心的部分的工作示例。

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

这个`no_run`属性将编译代码，但不运行它。这对于“以下是如何检索网页”等示例很重要，您可能希望确保编译，但可能在没有网络访问的测试环境中运行。

````text
/// ```compile_fail
/// let x = 5;
/// x += 2; // shouldn't compile!
/// ```
````

`compile_fail`讲述`rustdoc`编译应该失败。如果它编译，那么测试将失败。但是，请注意，随着新特性的添加，当前的铁锈释放失败的代码可能在将来的版本中起作用。

````text
/// Only runs on the 2018 edition.
///
/// ```edition2018
/// let result: Result<i32, ParseIntError> = try {
///     "1".parse::<i32>()?
///         + "2".parse::<i32>()?
///         + "3".parse::<i32>()?
/// };
/// ```
````

`edition2018`讲述`rustdoc`代码样本应编制2018版Rust。同样，您可以指定`edition2015`以2015年版编制本规范。

## 语法参考

这个*准确的*代码块（包括边缘大小写）的语法可以在[用栅栏围起来的代码块](https://spec.commonmark.org/0.28/#fenced-code-blocks)通用标记规范的章节。

RustDoc也接受*缩进*代码块作为隔离代码块的替代：您可以将每行缩进四个或更多的空格，而不是用三个反勾号包围代码。

```markdown
    let foo = "foo";
    assert_eq!(foo, "foo");
```

这些也记录在通用标记规范中，[缩进代码块](https://spec.commonmark.org/0.28/#indented-code-blocks)第节。

但是，最好使用隔离代码块而不是缩进代码块。不仅围栏代码块被认为是更惯用的锈代码，而且没有办法使用诸如`ignore`或`should_panic`使用缩进代码块。
