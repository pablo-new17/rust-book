# if let

`if let` 讓你將 `if` 和 `let` 結合在一起，用以減少某些模式匹配的開銷。

例如，假設我們有某個類型的 `Option<T>` ，若它是 `Some<T>` 我們希望對它呼叫一個函式，
但若是 `None` 就什麼都不做，寫起來像這樣：

```rust
# let option = Some(5);
# fn foo(x: i32) { }
match option {
    Some(x) => { foo(x) },
    None => {},
}
```

這裡不需要用到 `match` ，例如我們可以用 `if` ：

```rust
# let option = Some(5);
# fn foo(x: i32) { }
if option.is_some() {
    let x = option.unwrap();
    foo(x);
}
```

這兩種語法都不特別令人討喜，用了 `if let` ，可以做得比較漂亮一點：

```rust
# let option = Some(5);
# fn foo(x: i32) { }
if let Some(x) = option {
    foo(x);
}
```

若一個 [模式][patterns](pattern) 成功匹配，它會將模式中的標識符與值中任何合適的部分連結起來，
隨後對表達式求值，若模式不匹配則什麼都不做。

若想在模式不匹配時做其他事，可以使用 `else` ：

```rust
# let option = Some(5);
# fn foo(x: i32) { }
# fn bar() { }
if let Some(x) = option {
    foo(x);
} else {
    bar();
}
```

## `while let`

相似的， `while let` 用在以「值符合某個模式」做為條件執行迴圈，它將下列的程式碼：

```rust
let mut v = vec![1, 3, 5, 7, 11];
loop {
    match v.pop() {
        Some(x) =>  println!("{}", x),
        None => break,
    }
}
```

轉成這樣：

```rust
let mut v = vec![1, 3, 5, 7, 11];
while let Some(x) = v.pop() {
    println!("{}", x);
}
```

[patterns]: patterns.html


> *commit 797a0bd*
