# 数当てゲーム (Guessing Game)

Rust Bookのチュートリアルで学んだ内容のまとめ。

## 学んだこと

### 変数とミュータビリティ

- `let` で変数を宣言。デフォルトはイミュータブル（変更不可）
- `let mut` でミュータブル（変更可能）にする

```rust
let mut guess = String::new(); // 変更可能な空のStringを作成
```

### ヒープとスタック

- `&str`（文字列スライス）はスタックに置かれる。固定長で変更不可
- `String` はヒープに確保される。実行時にサイズが変わる

```
スタック (stack)           ヒープ (heap)
┌──────────────┐          ┌───────────┐
│ guess        │          │           │
│  ptr ────────┼────────→ │ (空のデータ) │
│  len: 0      │          └───────────┘
│  capacity: 0 │
└──────────────┘
```

`read_line` で入力を受け取るとヒープ上のデータが伸びる:

```
スタック (stack)           ヒープ (heap)
┌──────────────┐          ┌───────────────┐
│ guess        │          │ '4', '2', '\n'│
│  ptr ────────┼────────→ │               │
│  len: 3      │          └───────────────┘
│  capacity: 8 │
└──────────────┘
```

### 関連関数（Associated Function）

- `型名::関数名()` の形で呼び出す（例: `String::new()`）
- インスタンスなしで呼べる。他言語の静的メソッド（static method）に近い
- メソッドとは異なり `self` を取らない

```rust
// 関連関数 — 型に対して呼ぶ（インスタンス不要）
let s = String::new();
let s = String::from("hello");

// メソッド — インスタンスに対して呼ぶ
s.len();
s.push_str(" world");
```

### 標準入力の読み取り

```rust
io::stdin()
    .read_line(&mut guess)
    .expect("Failed to read line");
```

- `io::stdin()` — 標準入力（キーボード入力）のハンドルを取得
- `.read_line(&mut guess)` — 入力を `guess` に格納。`&mut` でミュータブルな参照を渡す
- 戻り値は `Result<usize>` 型（成功 or 失敗）
- `.expect()` — 失敗時にメッセージ付きでパニック終了

### Result型によるエラーハンドリング

`Result<T, E>` は `Ok(成功値)` か `Err(エラー)` のどちらかを持つ列挙型:

```
Result<T, E>
├── Ok(T)   ← 成功パターン
└── Err(E)  ← 失敗パターン
```

- Rustでは `Result` を処理しないとコンパイラが警告を出す（エラーの見落としを防ぐ設計）
- 他の言語の例外（try/catch）とは異なり、無視できない

扱い方は3通り:

```rust
// 1. expect — 失敗したらメッセージ付きでパニック終了
let n = result.expect("失敗しました");

// 2. unwrap — 失敗したらパニック（メッセージなし）
let n = result.unwrap();

// 3. match — 成功と失敗を自分で処理する（一番丁寧）
match result {
    Ok(n)  => println!("{}バイト読めた", n),
    Err(e) => println!("エラー: {}", e),
}
```

### matchによるエラー処理 vs expect

```rust
// expect版 — エラーでプログラムが強制終了する
let guess: u32 = guess.trim().parse().expect("Please type a number!");

// match版 — エラーでもプログラムが続く（再入力を求める）
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,  // ループの先頭に戻る
};
```

- `expect` はエラーで即終了
- `match` ならエラーを優雅に処理して再チャンスを与えられる
- `Err(_)` の `_` は「エラーの中身を無視する」という意味

### 文字列から数値への変換

```rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```

処理の流れ:

```
"42\n" → trim → "42"  → parse → Ok(42)  → guess = 42（続行）
"abc\n" → trim → "abc" → parse → Err(_)  → continue（ループの先頭へ戻る）
```

- `.trim()` — 前後の空白・改行を除去（`read_line` は `\n` を含むため必要）
- `.parse()` — 文字列を型注釈 `: u32` で指定した型に変換。`Result` を返す

### シャドーイング（Shadowing）

同じ名前で新しい変数を宣言し、前の変数を隠す機能:

```rust
let mut guess = String::new();            // 1つ目: String型（mut）
let guess: u32 = guess.trim().parse()...; // 2つ目: u32型（immutable）
```

- 型を変換しつつ同じ名前を使い続けられる
- 「ユーザーの予想」という意味は同じなので、名前を分ける必要がない
- 2つ目は `mut` が不要なのでイミュータブルに戻せる

### crateの使い方

- `Cargo.toml` の `[dependencies]` にcrate名とバージョンを記載
- `cargo add クレート名` — 最新安定版を自動追加（一番簡単）
- `cargo search クレート名` — ターミナルでバージョン情報を確認
- `cargo info クレート名` — 詳細情報を確認
- crates.io のWebサイトでも調べられる
- `Cargo.lock` にビルド時のバージョンが固定される

### セマンティックバージョニング

- `"0.9.2"` → `>= 0.9.2` かつ `< 0.10.0`
- `"0.9"` → `>= 0.9.0` かつ `< 0.10.0`
- 初回 `cargo build` で範囲内の最新版が自動選択され、`Cargo.lock` に固定
- `cargo update` で範囲内の最新版に更新

### randクレート（0.9）

```rust
use rand::Rng;
let secret_number = rand::rng().random_range(1..=100);
```

- `rand::rng()` — RNG（乱数生成器）を取得
- `.random_range(1..=100)` — 1から100の範囲で乱数を生成
- `1..=100` は「1以上100以下」を意味する範囲式（range inclusive）
- ※ 0.8では `thread_rng()` と `gen_range()` だったが0.9でリネームされた

### ループと制御フロー

```rust
loop {
    // 無限ループ
    // ...
    Err(_) => continue,  // ループの先頭に戻る
    // ...
    break;               // ループを抜ける
}
```

- `loop` — 無限ループ
- `break` — ループを抜ける
- `continue` — ループの先頭に戻る（ループの中でのみ使用可能）
- `continue` をループの外で書くとコンパイルエラーになる
