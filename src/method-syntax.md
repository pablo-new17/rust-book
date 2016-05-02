# 方法語法（Method Syntax）


函式（functions）非常好用，但是如果當你想要在一些資料上呼叫一堆函式時，它並不是那麼方便。
看看以下的程式碼：

```rust,ignore
baz(bar(foo));
```

我們可以從左讀到右，然後我們就會看到「baz bar foo」的結果。
但這樣的順序並不是函式被呼叫的次序，函式的呼叫是由內而外的：「foo bar baz」才對。
如果我們能像以下這樣不是很好嗎？

```rust,ignore
foo.bar().baz();
```

幸運的是，正如你所猜的那樣，的確可以這樣做！
Rust 透過 `impl` 關鍵字提供了「方法呼叫語法」（method call syntax）的功能。

## 方法呼叫

這是它的運作方式：

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

fn main() {
    let c = Circle { x: 0.0, y: 0.0, radius: 2.0 };
    println!("{}", c.area());
}
```

這會印出 `12.566371`。

我們建立一個代表園圈的 `struct`。
然後撰寫一個 `impl` 區塊，在其中定義一個方法 `area`。

方法的第一個參數比較特別，它有三種變體：`self`、`&self` 及 `&mut self`。
你可以把第一個參數想成 `foo.bar()` 中的 `foo`。
而三種不同的變體則對應到三種可能的 `foo` 類型：如果是堆疊中的一個值就使用 `self`；如果是 reference 就使用 `&self`；如果是可變 reference 就使用 `&mut self`。
範例中因為 `area` 使用 `&self` 作為參數，我們可以像使用其他參數一樣使用它。
而我們清楚它是個 `Circle`，所以我們可以存取它的 `radius`。

我們預設應該使用 `&self`，因為與其取得所有權，你應該更傾向使用借用（borrowing），正如同與其使用可變 references，應該更傾向使用不可變 reference。
以下是全部三種變體的範例：

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn reference(&self) {
       println!("taking self by reference!");
    }

    fn mutable_reference(&mut self) {
       println!("taking self by mutable reference!");
    }

    fn takes_ownership(self) {
       println!("taking ownership of self!");
    }
}
```

你可以盡情使用多個 `impl` 區塊。
所以上述範例也能改寫成這樣：

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn reference(&self) {
       println!("taking self by reference!");
    }
}

impl Circle {
    fn mutable_reference(&mut self) {
       println!("taking self by mutable reference!");
    }
}

impl Circle {
    fn takes_ownership(self) {
       println!("taking ownership of self!");
    }
}
```

## 鍊接方法呼叫（Chaining method calls）

所以現在我們了解如何呼叫方法了，像是 `foo.bar()` 這樣。
但是最一開始的那個例子 `foo.bar().baz()` 呢？
這個被稱為「方法鍊接」（method chaining）。
讓我們來看個例子：

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

    fn grow(&self, increment: f64) -> Circle {
        Circle { x: self.x, y: self.y, radius: self.radius + increment }
    }
}

fn main() {
    let c = Circle { x: 0.0, y: 0.0, radius: 2.0 };
    println!("{}", c.area());

    let d = c.grow(2.0).area();
    println!("{}", d);
}
```

注意回傳值的型別：

```rust
# struct Circle;
# impl Circle {
fn grow(&self, increment: f64) -> Circle {
# Circle } }
```

我們回傳了一個 `Circle`。
透過這個方法，我們可以把 `Circle` 成長到任意的大小。

## 關聯函式（Associated functions）

你也能定義不需要 `self` 參數的關聯函式。
這在 Rust 中是非常常見的模式：

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn new(x: f64, y: f64, radius: f64) -> Circle {
        Circle {
            x: x,
            y: y,
            radius: radius,
        }
    }
}

fn main() {
    let c = Circle::new(0.0, 0.0, 2.0);
}
```

這個「關聯函式」（associated function）替我們建立了一個新的 `Circle`。
請注意關聯函式是透過 `Struct::function()` 語法被呼叫的，而不是透過 `ref.method()` 語法。
一些其他程式語言會把關聯函式稱為「靜態方法」（static methods）。

## 生成器模式（Builder Pattern）

我們希望使用者可以建立 `Circle`，而且我們將允許他們只設定他們所關心的屬性。
例如 `x` 和 `y` 是 `0.0`，而 `radius` 是 `1.0`。
Rust 並沒有方法重載、命名參數、或可變參數的功能。
但我們利用生成器模式（Builder Pattern）來取代。
它看起來像這樣：

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

struct CircleBuilder {
    x: f64,
    y: f64,
    radius: f64,
}

impl CircleBuilder {
    fn new() -> CircleBuilder {
        CircleBuilder { x: 0.0, y: 0.0, radius: 1.0, }
    }

    fn x(&mut self, coordinate: f64) -> &mut CircleBuilder {
        self.x = coordinate;
        self
    }

    fn y(&mut self, coordinate: f64) -> &mut CircleBuilder {
        self.y = coordinate;
        self
    }

    fn radius(&mut self, radius: f64) -> &mut CircleBuilder {
        self.radius = radius;
        self
    }

    fn finalize(&self) -> Circle {
        Circle { x: self.x, y: self.y, radius: self.radius }
    }
}

fn main() {
    let c = CircleBuilder::new()
                .x(1.0)
                .y(2.0)
                .radius(2.0)
                .finalize();

    println!("area: {}", c.area());
    println!("x: {}", c.x);
    println!("y: {}", c.y);
}
```

我們在此處又建立了另一個名為 `CircleBuilder` 的 `struct`。
我們替它定義了生成器方法（builder methods）。
我們也定義了 `Circle` 上的 `area()` 方法。
另外，我們也替 `CircleBuilder` 多定義了一個方法：`finalize()`。
這個方法會從生成器產生最終的 `Circle`。
現在我們可以使用型別系統來執行我們所關心的事：使用 `CircleBuilder` 上的方法來產生我們所想要的 `Circle`。


> *commit 6ba9520*
