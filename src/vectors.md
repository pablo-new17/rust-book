# 向量

向量 `Vec<T>` 為標準函式庫中動態可增長的陣列，型別 `T` 表示我們可以宣告任何型別的向量（可參考 [泛型][generic] 章節）。
向量的資料一定會配置在堆積區 （heap），可以使用 `vec!` 巨集來生成向量。

```rust
let v = vec![1, 2, 3, 4, 5]; // v: Vec<i32>
```

（注意到不像之前使用的 `println!` 巨集，`vec!` 巨集使用方括號，Rust 允許我們使用圓括號或方括號。
在這裡使用方括號純粹是慣例。）

`vec!` 有另一種用來產生重複初始值的形式：

```rust
let v = vec![0; 10]; // ten zeroes
```

向量將內容儲存在連續的堆積之中，意味著在編譯時，型別 `T` 的大小就必須是已知的（即需要多少 bytes 來儲存一個 `T`）。
有些東西的大小在編譯期無法確定，這種情況你必須儲存一個指向該物件的指標：[Box][box] 型別對此相當合用。

## 存取物件

要取得特定位置的值，只需使用 `[]`：

```rust
let v = vec![1, 2, 3, 4, 5];

println!("The third element of v is {}", v[2]);
```

編號是由 `0` 開始計算，因此第三個元素為 `v[2]`。

同時需注意一定要用 `usize` 型別作為索引值：

```rust,ignore
let v = vec![1, 2, 3, 4, 5];

let i: usize = 0;
let j: i32 = 0;

// works
v[i];

// doesn’t
v[j];
```

使用非 `usize` 型別會產生如下的錯誤：

```text
error: the trait `core::ops::Index<i32>` is not implemented for the type
`collections::vec::Vec<_>` [E0277]
v[j];
^~~~
note: the type `collections::vec::Vec<_>` cannot be indexed by `i32`
error: aborting due to previous error
```

錯誤訊息中有許多標點符號，不過內容其實很單純：你不能使用 `i32` 作為索引值。

## 超出邊界存取

如果你試圖存取不存在的索引位置：

```rust,ignore
let v = vec![1, 2, 3];
println!("Item 7 is {}", v[7]);
```

則現行的執行緒會 [panic] 並產生如下的訊息：

```text
thread '<main>' panicked at 'index out of bounds: the len is 3 but the index is 7'
```

如果你要處理超出邊界的錯誤，可以使用 [get][get] 或 [get_mut][get_mut] 方法，當它們遇到無效的索引值會回傳 `None`：

```rust
let v = vec![1, 2, 3];
match v.get(7) {
    Some(x) => println!("Item 7 is {}", x),
    None => println!("Sorry, this vector is too short.")
}
```

## 疊代

有了向量，可以使用 `for` 來疊代它的元素，共有三種版本：

```rust
let mut v = vec![1, 2, 3, 4, 5];

for i in &v {
    println!("A reference to {}", i);
}

for i in &mut v {
    println!("A mutable reference to {}", i);
}

for i in v {
    println!("Take ownership of the vector and its element {}", i);
}
```

向量有許多有用的方法，可以參考 [向量的 API 文件][vec]。

[vec]: http://doc.rust-lang.org/std/vec/index.html
[box]: http://doc.rust-lang.org/std/boxed/index.html
[generic]: generics.html
[panic]: concurrency.html#panics
[get]: http://doc.rust-lang.org/std/vec/struct.Vec.html#method.get
[get_mut]: http://doc.rust-lang.org/std/vec/struct.Vec.html#method.get_mut


> *commit f2bea1c*
