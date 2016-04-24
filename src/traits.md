# Traits

trait 為Rust 語言特徵，告知Rust compiler 一個型別必須滿足的功能性

記得提過的 `impl` 關鍵字，用在使用[method syntax][methodsyntax]
呼叫函數：

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}
```

[methodsyntax]: method-syntax.html

Traits 很相似，除了我們首先先定義包含一個函式特徵的trait，接著為型別實作該trait
在本例中，我們為型別 `Circle` 實作 trait `HasArea`：

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

trait HasArea {
    fn area(&self) -> f64;
}

impl HasArea for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}
```

可以看到，`trait` 區塊和 `impl` 區塊看起來十分相似，但只需定義型別特徵，不需要定義方法本體
當我們 `impl` 一個trait 時，使用 `impl Trait for Item` 而非 `impl Item`。

## 泛型函式的 Trait 限制

Traits 保證一個型別應有的行為，因此非常有用，泛型函式能利用 trait 作為[限制(bound)][bounds]
用以限制他們接受的型別。
下面的函式無法成功編譯：

[bounds]: glossary.html#bounds

```rust,ignore
fn print_area<T>(shape: T) {
    println!("This shape has an area of {}", shape.area());
}
```

Rust 會警告：

```text
error: no method named `area` found for type `T` in the current scope
```

因為型別 `T` 可以為任何型態，我們無法保證它一定實作了 `area` 方法，但我們可以對泛型 `T`
加上trait 限制，確保它實作該方法：

```rust
# trait HasArea {
#     fn area(&self) -> f64;
# }
fn print_area<T: HasArea>(shape: T) {
    println!("This shape has an area of {}", shape.area());
}
```

`<T: HasArea>` 語法意味：「任何實作了 `HasArea` trait 的型別」
因為 trait 定義了函式特徵，我們可以確認任何實作了 `HasArea` trait 的型別
一定有 `.area()` 方法。

以下是修改後，說明trait bound 如何運作的例子

```rust
trait HasArea {
    fn area(&self) -> f64;
}

struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl HasArea for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}

struct Square {
    x: f64,
    y: f64,
    side: f64,
}

impl HasArea for Square {
    fn area(&self) -> f64 {
        self.side * self.side
    }
}

fn print_area<T: HasArea>(shape: T) {
    println!("This shape has an area of {}", shape.area());
}

fn main() {
    let c = Circle {
        x: 0.0f64,
        y: 0.0f64,
        radius: 1.0f64,
    };

    let s = Square {
        x: 0.0f64,
        y: 0.0f64,
        side: 1.0f64,
    };

    print_area(c);
    print_area(s);
}
```

程式輸出：

```text
This shape has an area of 3.141593
This shape has an area of 1
```

如你所見，`print_area` 是泛型同時我確保我們傳入正確的型別
如果我們傳入不正確的型別：

```rust,ignore
print_area(5);
```

會造成編譯時期錯誤：

```text
error: the trait `HasArea` is not implemented for the type `_` [E0277]
```

## 泛型結構體的 Trait 限制

泛型結構體同樣能夠利用 trait 限制，需要做的只是在宣告型別參數時添加限制
這裡有個新的型別 `Rectangle<T>` 和它的操作 `is_square()`：

```rust
struct Rectangle<T> {
    x: T,
    y: T,
    width: T,
    height: T,
}

impl<T: PartialEq> Rectangle<T> {
    fn is_square(&self) -> bool {
        self.width == self.height
    }
}

fn main() {
    let mut r = Rectangle {
        x: 0,
        y: 0,
        width: 47,
        height: 47,
    };

    assert!(r.is_square());

    r.height = 42;
    assert!(!r.is_square());
}
```

`is_square()` 需要確認邊是否相等，因此邊的型別一定要實作 trait
[`core::cmp::PartialEq`][PartialEq]：

```ignore
impl<T: PartialEq> Rectangle<T> { ... }
```

現在，一個 rectangle 可用任何可以比較是否相等的型別來定義

[PartialEq]: ../core/cmp/trait.PartialEq.html

我們定義一個新的結構體 `Rectangle`，能夠接受任意精確度的數字-幾乎可算是任意型別-
只要能夠比較是否相等，我們是否能對其他結構體 `HasArea`、`Square`、`Circle`
做相同的事？可以，但他們需要實作乘法，要實作它可以參考 [operator traits][operators-and-overloading].

[operators-and-overloading]: operators-and-overloading.html

# 實作 traits 的規則

到目前為止，我們只對結構體加上 trait 實作，但我們可以對任何型別實作 trait
技術上來說，我們「可以」對 `i32` 實作 `HasArea`：

```rust
trait HasArea {
    fn area(&self) -> f64;
}

impl HasArea for i32 {
    fn area(&self) -> f64 {
        println!("this is silly");

        *self as f64
    }
}

5.area();
```

儘管可以這麼做，對這類基本型別實作方法並不算是好的風格

這看起來有點像毫無規則的「瘋狂西部大拓荒」，但實際上實作 trait 時有兩條規則

第一：實作的trait 必須在你定義的有效範圍，否則無法實作
這裡有個例子：標準函式庫提供了[`Write`][write] trait，對 `File`s 新增 I/O 用的功能
`File` 在預設並沒有這些方法：

[write]: ../std/io/trait.Write.html

```rust,ignore
let mut f = std::fs::File::open("foo.txt").expect("Couldn’t open foo.txt");
let buf = b"whatever"; // byte string literal. buf: &[u8; 8]
let result = f.write(buf);
# result.unwrap(); // ignore the error
```

以下為編譯錯諤：

```text
error: type `std::fs::File` does not implement any method in scope named `write`
let result = f.write(buf);
               ^~~~~~~~~~
```

我們需要先 `use` `Write` trait：

```rust,ignore
use std::io::Write;

let mut f = std::fs::File::open("foo.txt").expect("Couldn’t open foo.txt");
let buf = b"whatever";
let result = f.write(buf);
# result.unwrap(); // ignore the error
```

即可成功編譯

這表示即便有人做了對 `i32` 新增一個方法的蠢事，它也不會影響到你，除非你引入該 trait

另外一個限制為：你實作的trait 和 type，其中一個必須是你定義的，更精確的說
其中一個一定要和寫 `impl` 同個crate 中定義，關於 Rust 模組和套件系統的資訊，請見
[crates and modules][cm]。

我們可以對 `i32` 實作 `HasArea`，因為我們定義自己了 `HasArea`，但我們無法
對 `i32` 實作Rust 提供的 `ToString`，因為無論 trait 或型別都不是我們這個crate 定義的

最後一件關於 traits 的事：包含trait限制的泛型函式使用'單型'（monomorphization：「Mono」為單，「morph」指型態）
因些他們是靜態分派，要知道更多細節請參考 [trait objects][to]

[cm]: crates-and-modules.html
[to]: trait-objects.html

# 多重 trait 限制

上文已經我們可以使用trait 限制泛型參數：

```rust
fn foo<T: Clone>(x: T) {
    x.clone();
}
```

若需要超過一個限制，可以使用 `+`:

```rust
use std::fmt::Debug;

fn foo<T: Clone + Debug>(x: T) {
    x.clone();
    println!("{:?}", x);
}
```

現在型別 `T` 需要 `Clone` 和 `Debug` traits

# Where 子句

撰寫少量泛型和 trait 限制的函式不算大問題，但當數量增加時，語法即變得不簡潔：

```rust
use std::fmt::Debug;

fn foo<T: Clone, K: Clone + Debug>(x: T, y: K) {
    x.clone();
    y.clone();
    println!("{:?}", y);
}
```

限制語法卡在中間，分隔了最左的函式名稱與最右的參數列：

Rust 的解法為'`where` 子句'：

```rust
use std::fmt::Debug;

fn foo<T: Clone, K: Clone + Debug>(x: T, y: K) {
    x.clone();
    y.clone();
    println!("{:?}", y);
}

fn bar<T, K>(x: T, y: K) where T: Clone, K: Clone + Debug {
    x.clone();
    y.clone();
    println!("{:?}", y);
}

fn main() {
    foo("Hello", "world");
    bar("Hello", "world");
}
```

`foo()` 用了先前的語法，`bar()` 則使用 `where` 子句
需要做的僅是在參數列後加入 `where`，並將限制從型別參數中移出
對更長的限制列可以上空白：

```rust
use std::fmt::Debug;

fn bar<T, K>(x: T, y: K)
    where T: Clone,
          K: Clone + Debug {

    x.clone();
    y.clone();
    println!("{:?}", y);
}
```

這個彈性能讓複雜的語法更加清楚

`where` 並不僅是更簡單的語法，例如：

```rust
trait ConvertTo<Output> {
    fn convert(&self) -> Output;
}

impl ConvertTo<i64> for i32 {
    fn convert(&self) -> i64 { *self as i64 }
}

// can be called with T == i32
fn normal<T: ConvertTo<i64>>(x: &T) -> i64 {
    x.convert()
}

// can be called with T == i64
fn inverse<T>() -> T
        // this is using ConvertTo as if it were "ConvertTo<i64>"
        where i32: ConvertTo<T> {
    42.convert()
}
```

這展示 `where` 額外的特性，它們除了型別參數 `T`，亦可限制型別（範例中的 `i32`)
在範例中，`i32` 必須實作 `ConvertTo<T>`，這裡的 `where` 限制 `T`而不是定義 `i32`是什麼

# 預設方法

如果已知一般的實作是如何，可在trait 中加入預設方法，例如
`is_invalid()` 和　`is_valid()` 是相反的：

```rust
trait Foo {
    fn is_valid(&self) -> bool;

    fn is_invalid(&self) -> bool { !self.is_valid() }
}
```

因為加上預設方法，實作 `Foo` 的人必須實作 `is_valid()` 但不需要實作 `is_invalid()`
這樣的方法仍可被覆寫：

```rust
# trait Foo {
#     fn is_valid(&self) -> bool;
#
#     fn is_invalid(&self) -> bool { !self.is_valid() }
# }
struct UseDefault;

impl Foo for UseDefault {
    fn is_valid(&self) -> bool {
        println!("Called UseDefault.is_valid.");
        true
    }
}

struct OverrideDefault;

impl Foo for OverrideDefault {
    fn is_valid(&self) -> bool {
        println!("Called OverrideDefault.is_valid.");
        true
    }

    fn is_invalid(&self) -> bool {
        println!("Called OverrideDefault.is_invalid!");
        true // overrides the expected value of is_invalid()
    }
}

let default = UseDefault;
assert!(!default.is_invalid()); // prints "Called UseDefault.is_valid."

let over = OverrideDefault;
assert!(over.is_invalid()); // prints "Called OverrideDefault.is_invalid!"
```

# 繼承

有時，實作一個trait 要求實作另一個：

```rust
trait Foo {
    fn foo(&self);
}

trait FooBar : Foo {
    fn foobar(&self);
}
```

實作 `FooBar` 必須同時實作 `Foo`，像這樣：

```rust
# trait Foo {
#     fn foo(&self);
# }
# trait FooBar : Foo {
#     fn foobar(&self);
# }
struct Baz;

impl Foo for Baz {
    fn foo(&self) { println!("foo"); }
}

impl FooBar for Baz {
    fn foobar(&self) { println!("foobar"); }
}
```

如果我們忘了實作 `Foo`，Rust 會告訴我們：

```text
error: the trait `main::Foo` is not implemented for the type `main::Baz` [E0277]
```

# 推導

重複實作例如 `Debug` 和`Default` 相當無聊，因此Rust 提供了[屬性][attributes]
讓Rust 自動為你實作這些 traits：

```rust
#[derive(Debug)]
struct Foo;

fn main() {
    println!("{:?}", Foo);
}
```

[attributes]: attributes.html

不過，推導只適用在一些 traits 上：

- [`Clone`](../core/clone/trait.Clone.html)
- [`Copy`](../core/marker/trait.Copy.html)
- [`Debug`](../core/fmt/trait.Debug.html)
- [`Default`](../core/default/trait.Default.html)
- [`Eq`](../core/cmp/trait.Eq.html)
- [`Hash`](../core/hash/trait.Hash.html)
- [`Ord`](../core/cmp/trait.Ord.html)
- [`PartialEq`](../core/cmp/trait.PartialEq.html)
- [`PartialOrd`](../core/cmp/trait.PartialOrd.html)


> *commit a559577*
