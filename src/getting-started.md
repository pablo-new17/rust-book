# 準備

本書第一章，將會帶領我們使用 Rust 及它的相關工具。
首先，我們會安裝 Rust。
接著，撰寫經典的 "Hello World" 程式。
最後，我們將談到 Cargo，它是 Rust 的建置系統跟套件管理器。

## 安裝 Rust

使用 Rust 的第一步就是安裝它。
一般來說，在本節你需要網路連線以執行指令，因為我們會從網路上下載 Rust。

我們將會提供許多終端機指令，這些指令每行都會以 `$` 開頭。
你不用輸入這些 `$` 符號，它們只是用來標示每一行指令。
在網路上的教學及範例中常使用以下的慣例：以一般使用者身分執行的指令使用 `$` 標示，而以系統管理員身分執行的指令則使用 `#` 標示。

### 平台支援

Rust 編譯器能編譯並執行在大多數的平台上，但不是所有的平台都有相同的支援性。
Rust 的支援程度被分為三級。每一級都有不同的保證。

平台將以 "target triple" 的方式標識，這個字串是用來告知編譯器最終該產出怎樣的輸出。
而各列代表對應的元件在特定平台的支援性。

> 譯註：`target triple` 通常會依 `<arch><sub>-<vendor>-<sys>-<abi>` 的規則表示。
> 可以參考 Clang 的[說明文件][target triple]。

[target triple]: http://clang.llvm.org/docs/CrossCompilation.html#target-triple

#### Tier 1 一級

一級平台可被認為是"被保證可建置且運行"的。
具體來說，它們都滿足以下要求：

* 此平台的已建有自動化測試。
* 通過測試，是能否將修改進到 `rust-lang/rust` repository master 分支的門檻。
* 此平台提供官方發行版。
* 已經有如何使用及建置此平台的文件。

|  Target                       | std |rustc|cargo| notes                      |
|-------------------------------|-----|-----|-----|----------------------------|
| `i686-pc-windows-msvc`        |  ✓  |  ✓  |  ✓  | 32-bit MSVC (Windows 7+)   |
| `x86_64-pc-windows-msvc`      |  ✓  |  ✓  |  ✓  | 64-bit MSVC (Windows 7+)   |
| `i686-pc-windows-gnu`         |  ✓  |  ✓  |  ✓  | 32-bit MinGW (Windows 7+)  |
| `x86_64-pc-windows-gnu`       |  ✓  |  ✓  |  ✓  | 64-bit MinGW (Windows 7+)  |
| `i686-apple-darwin`           |  ✓  |  ✓  |  ✓  | 32-bit OSX (10.7+, Lion+)  |
| `x86_64-apple-darwin`         |  ✓  |  ✓  |  ✓  | 64-bit OSX (10.7+, Lion+)  |
| `i686-unknown-linux-gnu`      |  ✓  |  ✓  |  ✓  | 32-bit Linux (2.6.18+)     |
| `x86_64-unknown-linux-gnu`    |  ✓  |  ✓  |  ✓  | 64-bit Linux (2.6.18+)     |

#### Tier 2 二級

二級平台可被認為是"被保證可以建置"的。
但因為沒有執行自動化測試，所以不保證能產生可以運行的 build，不過此平台通常運行良好，且永遠歡迎上 patches！
具體來說，這些平台被要求要符合以下各項：

* 有自動建置，但可能沒有執行測試。
* 能建置成功，是能否把將修改進到 `rust-lang/rust` repository master 分支的門檻。
  請注意，這代表在某些平台上，只要標準函式庫被編譯成功即可，但其他平台將需要執行整個 bootstrap。
* 此平台提供官方發行版。

|  Target                       | std |rustc|cargo| notes                      |
|-------------------------------|-----|-----|-----|----------------------------|
| `x86_64-unknown-linux-musl`   |  ✓  |     |     | 64-bit Linux with MUSL     |
| `arm-linux-androideabi`       |  ✓  |     |     | ARM Android                |
| `arm-unknown-linux-gnueabi`   |  ✓  |  ✓  |     | ARM Linux (2.6.18+)        |
| `arm-unknown-linux-gnueabihf` |  ✓  |  ✓  |     | ARM Linux (2.6.18+)        |
| `aarch64-unknown-linux-gnu`   |  ✓  |     |     | ARM64 Linux (2.6.18+)      |
| `mips-unknown-linux-gnu`      |  ✓  |     |     | MIPS Linux (2.6.18+)       |
| `mipsel-unknown-linux-gnu`    |  ✓  |     |     | MIPS (LE) Linux (2.6.18+)  |

#### Tier 3 三級

三級平台是那些 Rust 支援的平台，但能否成功建置或通過測試不會阻礙提交修改。
在這些平台上可運行的 builds 可能會有一些小問題，因為它們的可靠度通常由社群貢獻所決定。
此外，並不提供發行版及安裝檔，但可能在非官方的地方有提供社群產出的版本。

|  Target                       | std |rustc|cargo| notes                      |
|-------------------------------|-----|-----|-----|----------------------------|
| `i686-linux-android`          |  ✓  |     |     | 32-bit x86 Android         |
| `aarch64-linux-android`       |  ✓  |     |     | ARM64 Android              |
| `powerpc-unknown-linux-gnu`   |  ✓  |     |     | PowerPC Linux (2.6.18+)    |
| `powerpc64-unknown-linux-gnu` |  ✓  |     |     | PPC64 Linux (2.6.18+)      |
|`powerpc64le-unknown-linux-gnu`|  ✓  |     |     | PPC64LE Linux (2.6.18+)    |
|`armv7-unknown-linux-gnueabihf`|  ✓  |     |     | ARMv7 Linux (2.6.18+)      |
| `i386-apple-ios`              |  ✓  |     |     | 32-bit x86 iOS             |
| `x86_64-apple-ios`            |  ✓  |     |     | 64-bit x86 iOS             |
| `armv7-apple-ios`             |  ✓  |     |     | ARM iOS                    |
| `armv7s-apple-ios`            |  ✓  |     |     | ARM iOS                    |
| `aarch64-apple-ios`           |  ✓  |     |     | ARM64 iOS                  |
| `i686-unknown-freebsd`        |  ✓  |  ✓  |     | 32-bit FreeBSD             |
| `x86_64-unknown-freebsd`      |  ✓  |  ✓  |     | 64-bit FreeBSD             |
| `x86_64-unknown-openbsd`      |  ✓  |  ✓  |     | 64-bit OpenBSD             |
| `x86_64-unknown-netbsd`       |  ✓  |  ✓  |     | 64-bit NetBSD              |
| `x86_64-unknown-bitrig`       |  ✓  |  ✓  |     | 64-bit Bitrig              |
| `x86_64-unknown-dragonfly`    |  ✓  |  ✓  |     | 64-bit DragonFlyBSD        |
| `x86_64-rumprun-netbsd`       |  ✓  |     |     | 64-bit NetBSD Rump Kernel  |
| `x86_64-sun-solaris`          |  ✓  |  ✓  |     | 64-bit Solaris/SunOS       |
| `i686-pc-windows-msvc` (XP)   |  ✓  |     |     | Windows XP support         |
| `x86_64-pc-windows-msvc` (XP) |  ✓  |     |     | Windows XP support         |

請注意，這個表格可能會隨著時間而增長，這永遠不是三級平台最終的列表！

### 安裝在 Linux 或 Mac 上

如果在 Linux 或 Mac 上，我們只需要開啟終端機並輸入：

```bash
$ curl -sSf https://static.rust-lang.org/rustup.sh | sh
```

這樣就會下載安裝腳本，然後開始安裝。當全部完成後，你將會看到：

```text
Welcome to Rust.

This script will download the Rust compiler and its package manager, Cargo, and
install them to /usr/local. You may install elsewhere by running this script
with the --prefix=<path> option.

The installer will run under ‘sudo’ and may ask you for your password. If you do
not want the script to run ‘sudo’ then pass it the --disable-sudo flag.

You may uninstall later by running /usr/local/lib/rustlib/uninstall.sh,
or by running this script again with the --uninstall flag.

Continue? (y/N)
```

接著按下 `y` 表示同意，接著按照提示安裝。

### 安裝在 Windows 上

如果你使用 Windows，請下載合適版本的[安裝檔][install-page]。

[install-page]: https://www.rust-lang.org/install.html

### 移除

移除 Rust 就跟安裝一樣簡單。在 Linux 或 Mac 上，只需執行移除指令就可以了：

```bash
$ sudo /usr/local/lib/rustlib/uninstall.sh
```

如果你是安裝 Windows 安裝檔來安裝的，請重新執行 `.msi` 檔，然後他就會出現移除的選項。

### 故障排除

當我們裝好 Rust 後，我們可以開啟 shell，然後輸入：

```bash
$ rustc --version
```

你將會看到版本號碼、commit hash 及 commit 的日期。

如果你成功了，代表 Rust 安裝成功了！恭喜你！

如果沒有成功，而你是使用 Windows 作業系統，請確認 Rust 有設定在你的 %PATH% 系統環境變數內。
如果沒有，重新執行安裝檔，在 "Change, repair, or remove installation" 頁面選擇 "Change"，接著確定 "Add to PATH" 有裝到本機端。

Rust 不自己做 linking，所以你會需要安裝一個 linker。
做這件事會相依於你所使用的作業系統，更多細節請查閱相關文件。

如果仍然沒有成功，有許多地方可以獲得協助。
最簡單的是使用 [Mibbit][mibbit] 連上 [irc.mozilla.org 的 #rust IRC 頻道][irc]。
按下連結，然後就能跟可以幫助我們的 Rustaceans（我們用以自稱的暱稱）聊天。
其他的資源還包括了[使用者論壇][users]和 [Stack Overflow][stackoverflow]。

[irc]: irc://irc.mozilla.org/#rust
[mibbit]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust
[users]: https://users.rust-lang.org/
[stackoverflow]: http://stackoverflow.com/questions/tagged/rust

安裝檔同時也安裝了一份文件的複本在本機，所以我們可以離線閱讀這份文件。
在 UNIX 系統的位置是 `/usr/local/share/doc/rust`。
在 Windows，文件則放在 Rust 安裝目錄下的 `share/doc` 目錄。

## Hello, world!

現在 Rust 已經裝好了，我們將協助你開始寫你的第一個 Rust 程式。
學習一個新語言的傳統是，撰寫一個小程式，並在螢幕上印出 "Hello, world!"。
本節我們會遵循這個傳統。

從這樣一個簡單小程式開始的好處，是你能很快地驗證編譯器已經安裝完成，而且正常運行。
把資訊印在螢幕上也是個未來會很常做的事情，早點練習沒什麼不好。

> 註：本書假設你已經有基本的指令熟悉程度。
> Rust 對你的編輯器、工具、或程式碼放在何處沒有特別要求，所以如果你喜歡使用 IDE 而非 command line 介面，使用 IDE 也是個選擇。
> 你也許會想嘗試專為 Rust 建立的 [SolidOak]。
> 社群也開發了許多擴充功能，而且 Rust 團隊也替[許多編輯器][various editors]發佈了外掛。
> 配置你的編輯器或 IDE 不在本教學的範圍內，所以請依據你的設置查閱文件。

[SolidOak]: https://github.com/oakes/SolidOak
[various editors]: https://github.com/rust-lang/rust/blob/master/src/etc/CONFIGS.md

### 建立專案檔

首先，建立一個檔案來放程式碼。
Rust 不在乎你的程式碼放在哪，但在本書中，我建議建立一個 *projects* 目錄在你的 home 目錄底下，並且把所有你的專案都放在裡面。
開啟終端機並輸入以下指令來替專案建立目錄：

```bash
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

> 註：如果你使用 Windows 而且不是使用 PowerShell，則 ~ 可能沒有作用。
> 更多細節請查閱你所使用 shell 的文件。

### 撰寫並執行 Rust 程式

接著，建立一個新的原始碼檔案，命名為 *main.rs*。
Rust 的檔案永遠以 *.rs* 副檔名為結尾。
如果你的檔案名稱中使用了超過一個英文單字，請使用底線去區分它們；舉例來說，你應該用 *hello_world.rs* 來命名，而不是 *helloworld.rs*。

現在開啟建好的 *main.rs* 檔案，然後輸入以下程式碼：

```rust
fn main() {
    println!("Hello, world!");
}
```

存檔，然後回到你的終端機視窗。
在 Linux 或 OSX 下，輸入以下指令：

```bash
$ rustc main.rs
$ ./main
Hello, world!
```

在 Windows 下，請把 `main` 換成 `main.exe`。
不論你的作業系統為何，你應該能看到字串 `Hello, world!` 印在終端機上。
如果你成功了，那麼恭喜你！
你已正式地寫出了一個 Rust 程式。
這讓你成為了 Rust 程式設計師！歡迎你！

### 剖析 Rust 程式

現在，讓我們來仔細檢視 "Hello, world!" 程式到底發生了什麼事。
這是第一塊謎團：

```rust
fn main() {

}
```

這幾行定義了 Rust 中的 *函式* (function)。
其中 `main` 函式比較特殊：他是所有 Rust 程式的起始點。
第一行意指 "我要定義一個叫 `main` 的函式，它沒有參數也沒有回傳值。"
如果函式有參數的話，參數會放在括號（`(` 及 `)`）中，且因為我們不回傳值，所以可以省略回傳的型態。

同時留意，函式的內容會被包在大括號（`{` 及 `}`）內。
Rust 要求所有函式的內容都要用它包起來。
把第一個大括號放在函式宣告的同一行，在中間留一個空白，一般被認為是好的程式風格。

在 `main()` 函式中：

```rust
    println!("Hello, world!");
```

這一行負責這個小程式的全部工作：印出文字到螢幕上。
這邊有許多很重要的細節。
首先，縮排是四個空白，而不是 tabs。

第二個重要的部分是 `println!()` 這行。
這個在 Rust 中叫做 *[巨集][macro]* (macro)，這是 Rust 達成 metaprogramming 的方式。
如果在此處改用函式的話，看起來會像是 `println()`（沒有 ! 符號）。
我們將在之後討論 Rust 巨集的更多細節，但現在你只需知道，當看到 `!` 的時候，代表你正在呼叫一個巨集，而不是一般的函式。

[macro]: macros.html

接著 `"Hello, world!"` 是一個 *字串* (string)。
在系統程式語言中，字串是個複雜到令人驚訝的主題，在此處，這是一個 *[靜態分配][statically allocated]* (statically allocated) 的字串。
我們傳遞這個字串作為參數給 `println!`，然後它將字串印在螢幕上。
夠簡單吧！

[statically allocated]: the-stack-and-the-heap.html

程式中每一行都以分號（`;`）結尾。
Rust 是個 *[表達式導向語言][expression-oriented language]* (expression-oriented language)，這代表大多數的東西都是表達式，而不是陳述式 (statements)。
`;` 代表著這個表達式已經結束，而且下一個正準備開始。
大多數 Rust 程式碼的行都是以 `;` 結尾。

[expression-oriented language]: glossary.html#expression-oriented-language

### 編譯和執行是不同的步驟

在 "撰寫並執行 Rust 程式" 中，我們告訴你如何執行一個新建的程式。
現在我們分解流程，檢查每一步。

執行 Rust 程式之前，你需要編譯它。
你可以使用 Rust 編譯器，輸入 `rustc` 指令並傳遞原始碼的檔名給它，像這樣

```bash
$ rustc main.rs
```

如果你有 C 或 C++ 的背景，你會注意到這跟 `gcc` 或 `clang` 很類似。
編譯成功之後，Rust 應該會輸出一個二進位執行檔，你可以在 Linux 或 OSX 下的 shell 輸入 `ls` 指令：

```bash
$ ls
main  main.rs
```

在 Windows 下，你可以輸入：

```bash
$ dir
main.exe  main.rs
```

這表示我們有兩個檔案：以 `.rs` 作為副檔名的原始檔，以及執行檔（在 Windows 是 `main.exe`，其他作業系統則是 `main`）。
接著我們唯一要做的就只剩下執行 `main` 或 `main.exe` 了：

```bash
$ ./main  # or main.exe on Windows
```

如果 *main.rs* 是 "Hello, world!" 程式，那它會印出 `Hello, world!` 在終端機上。

如果你以前學的是動態語言，例如 Ruby、Python、或 JavaScript，你可能會不習慣把編譯程式和執行程式的動作分開。
Rust 是個 *事先編譯* (ahead-of-time compiled) 的語言，你可以編譯程式，把程式給其他人，他們不用安裝 Rust 就能執行程式。
如果你給一個人 `.rb` 或 `.py` 或 `.js` 檔，他則必須要先裝好 Ruby、Python、或 JavaScript 應用，不過反之，他只需一個指令就可以編譯並執行程式。
這都是程式語言的設計中的取捨。

簡單的程式只需要用 `rustc` 直接編譯就好了，但隨著你的專案成長，你會希望能夠管理專案的所有選項，並且能簡單的分享程式碼給其他人和專案。
下一節，我們將會介紹給你一個叫做 Cargo 的工具，他可以幫你撰寫真實世界的 Rust 程式。

## Hello, Cargo!

Cargo 是 Rust 的建置系統跟套件管理器，而且 Rustaceans 會使用 Cargo 去管理他們的 Rust 專案。
Cargo 管理三件事：建置你的程式碼、下載你的程式碼所依賴的函式庫 (libraries)、以及建置這些函式庫。
我們把這些你的程式所依賴的函式庫叫做 "dependencies"，因為你的程式碼依賴他們。

最簡單的 Rust 程式不會有任何 dependencies，所以現在你只會用到第一部份的功能。
當你撰寫更複雜的 Rust 程式後，你將會希望加入 dependencies，如果你從 Cargo 開始的話，那就會簡單很多。

許多主要的 Rust 專案都使用 Cargo，我們假設在本書後面的章節你都會使用它。
如果你使用官方安裝檔，Cargo 將會隨著 Rust 一起裝好。
如果你用其他方法安裝 Rust，你可以輸入以下指令檢查是否已經裝好 Cargo：

```bash
$ cargo --version
```

在終端機內，如果你能看到版本號碼，那就太好了！
如果你看到錯誤訊息像是 `command not found`，則你應該要查閱你所安裝 Rust 的系統的相關文件，確定 Cargo 是否需要另外安裝。

### 轉換到 Cargo

讓我們開始轉換 Hello World 程式到 Cargo 上。
要將專案 Cargo 化的話，你需要做以下三件事：

1. 把你的原始碼檔案放到正確的目錄。
2. 去除舊的執行檔（在 Windows 是 `main.exe`，其他則是 `main`）， 並另外建一個新的。
3. 建立 Cargo 配置 (configuration) 檔。

讓我們開始吧！

#### 建立新的執行檔和原始碼目錄

首先，回到終端機，移到 *hello_world* 目錄，輸入以下指令：

```bash
$ mkdir src
$ mv main.rs src/main.rs
$ rm main  # or 'del main.exe' on Windows
```

Cargo 預期原始碼會放在 *src* 目錄內，所以我們先弄好這部分。
最上層的專案目錄（在此為 *hello_world*）被保留來放置 README 檔、授權資訊、及其他與程式碼無關的東西。
這樣可以讓你保持專案的整潔。
物有所屬，所有東西都有自己的位置。

然後，複製 *main.rs* 到 *src* 目錄，然後刪除 `rustc` 所編譯出的檔案。
一如往常地， 在 Windows 下用 `main.exe` 取代 `main`。

這個例子中保留 `main.rs` 作為原始碼檔名，是因為它可以建立執行檔。
如果你想要建立一個函式庫 (library)，你應該把檔案命名為 `lib.rs`。
Cargo 使用這樣的慣例去編譯專案，但如果你想要的話，你還是可以更改它。

#### 建立配置檔

接著，在 *hello_world* 目錄下建立一個新的檔案，命名為 `Cargo.toml`。

確保 `Cargo.toml` 的 `C` 是大寫，否則 Cargo 會無法處理這樣的配置檔。

這個檔案使用 *[TOML]* (Tom's Obvious, Minimal Language) 格式。
TOML 跟 INI 很類似，但是有一些額外的好處，且被用來作為 Cargo 的配置格式。

[TOML]: https://github.com/toml-lang/toml

在檔案內，輸入以下資訊：

```toml
[package]

name = "hello_world"
version = "0.0.1"
authors = [ "Your name <you@example.com>" ]
```

第一行，`[package]` 表示以下的陳述是用來配置一個套件 (package)。
當我們要加入更多資訊到這個檔案內，我們會增加其他的小節 (sections)，但是現在，我們只有套件的配置。

其他三行設定了三項 Cargo 在編譯程式時所需知道的配置：程式的名字、版本、和作者。

在 *Cargo.toml* 檔案內加入這些資訊後，存檔然後結束。

### 建立並執行 Cargo 專案

當 *Cargo.toml* 檔案被放在專案的根目錄後，你應該就能建立並執行 Hello World 程式了！
輸入以下指令：

```bash
$ cargo build
   Compiling hello_world v0.0.1 (file:///home/yourname/projects/hello_world)
$ ./target/debug/hello_world
Hello, world!
```

蹦！如果一切順利，`Hello, world!` 應該再次印在終端機上了。

你可以用 `cargo build` 建置專案、並透過 `./target/debug/hello_world` 執行它，但你其實可以直接用 `cargo run` 一次就執行兩者：

```bash
$ cargo run
     Running `target/debug/hello_world`
Hello, world!
```

請注意，這個範例中並沒有重新建置專案。
Cargo 判斷檔案沒有更動，所以它直接去執行二進位執行檔。
如果你有修改原始碼，Cargo 就會在執行前重新建置專案，然後你會看到：

```bash
$ cargo run
   Compiling hello_world v0.0.1 (file:///home/yourname/projects/hello_world)
     Running `target/debug/hello_world`
Hello, world!
```

Cargo 會檢查專案內的檔案是否有被更改，它只會在專案有更動的時候重新建置。

在簡單的專案中，Cargo 無法比 `rustc` 帶來更多好處，但是它在未來會越來越有用。
在用到許多 crates 的複雜專案中，使用 Cargo 去協調建置會比較簡單。
你只需要執行 `cargo build`，然後一切都會正確的運行。

#### 建立發行版

當你的專案最終準備要發行時，你應該使用 `cargo build --release` 來最佳化編譯你的專案。
這些最佳化讓你的 Rust 程式碼執行得更快，但你的程式編譯起來會多花點時間。
這也是為什麼會有兩種不同的 profiles，其中一個用於開發，另一個用於建置最終給使用者的程式。

#### 什麼是 `Cargo.lock`？

執行 `cargo build` 也會讓 Cargo 建立一個叫做 *Cargo.lock* 的檔案，內容看起來像這樣：

```toml
[root]
name = "hello_world"
version = "0.0.1"
```

Cargo 使用 *Cargo.lock* 去追蹤應用程式的 dependencies。這是 Hello World 專案的 *Cargo.lock* 檔。
因為此專案沒有任何 dependencies，所以檔案有點稀疏。
實際上，你不需要自己去碰這個檔案；你只要讓 Cargo 去處理就好了。

就這樣！如果你一路照著做到現在，你應該已經成功地以 Cargo 建置 `hello_world` 了。

即使這個專案很簡單，它也使用到許多在你之後的 Rust 生涯中會真實用到的工具。
事實上，你能預期幾乎所有的 Rust 專案都會透過類似下述指令開始：

```bash
$ git clone someurl.com/foo
$ cd foo
$ cargo build
```

### 簡單地開始一個 Cargo 專案

當你想要開始一個新專案時，你不需要每次都重新執行一遍前面的所有流程！
Cargo 可以快速的建立專案目錄骨架，然後你就可以開始開發。

用 Cargo 開始一個新專案，只要輸入 `cargo new`：

```bash
$ cargo new hello_world --bin
```

這個指令傳遞了 `--bin`，因為它的目的是建立一個可執行的應用程式，而不是函式庫。
執行檔也常被稱作 *二進位檔* (binaries)。

Cargo 會產生兩個檔案和一個目錄給我們：`Cargo.toml` 檔案，*src* 目錄、和 src 下的 *main.rs* 檔案。
它們看起來與前面手動建立的結構很像。

這些就是我們全部所需的東西。
首先，打開 `Cargo.toml`。
它應該看起來像這樣：

```toml
[package]

name = "hello_world"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
```

Cargo 會根據你給它的參數和你的 `git` 環境配置產生帶有預設值的 *Cargo.toml*。
而你也能注意到 Cargo 同時會把 `hello_world` 目錄初始化成 `git` 的 repository。

至於 `src/main.rs` 內應該會長的像這樣：

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo 會幫你產生 "Hello World!" 程式碼，接著你就可以開始寫程式了！

> 註：如果你想要閱覽更多 Cargo 的細節，可以查閱官方的 [Cargo 指南][Cargo guide]，其中包含有所有的功能。

[Cargo guide]: http://doc.crates.io/guide.html

## 結語

本章包含了在本書後述部分、及未來你的 Rust 時光的各種基礎。
現在你已經學會了工具，我們接著將更多地論及 Rust 本身。

你有兩個選擇：從 "[教學: 猜謎遊戲][guessinggame]" 深入一個專案，或從 "[語法及語意][syntax]" 由下而上開始。
有經驗的系統程式設計師可能會傾向從 "教學: 猜謎遊戲" 開始，而動態語言背景的人也許兩者都可以。
不同人有不同的學習方式！
請選擇適合自己的方式。

[guessinggame]: guessing-game.html
[syntax]: syntax-and-semantics.html


> *commit c3f6122*
