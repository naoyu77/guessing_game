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

### 関連関数（Associated Function）

- `型名::関数名()` の形で呼び出す（例: `String::new()`）
- インスタンスなしで呼べる。他言語の静的メソッドに近い
- メソッドとは異なり `self` を取らない

### Result型によるエラーハンドリング

- `Result<T, E>` は `Ok(成功値)` か `Err(エラー)` のどちらかを持つ列挙型
- Rustでは `Result` を処理しないとコンパイラが警告を出す
- `.expect("メッセージ")` でエラー時にパニック終了
- `match` で `Ok` と `Err` を個別に処理できる

```rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```

### シャドーイング（Shadowing）

- 同じ名前で新しい変数を宣言し、前の変数を隠す
- 型を変換しつつ同じ名前を使い続けられる

```rust
let mut guess = String::new();          // String型
let guess: u32 = guess.trim().parse()...; // u32型に変換（シャドーイング）
```

### crateの使い方

- `Cargo.toml` の `[dependencies]` にcrate名とバージョンを記載
- `cargo add クレート名` で最新安定版を自動追加できる
- `cargo search` や `cargo info` でバージョン情報を確認できる
- `Cargo.lock` にビルド時のバージョンが固定される

### セマンティックバージョニング

- `"0.9.2"` → `>= 0.9.2` かつ `< 0.10.0`
- `"0.9"` → `>= 0.9.0` かつ `< 0.10.0`
- `cargo build` で範囲内の最新版が自動選択される
- `cargo update` で範囲内の最新版に更新

### randクレート（0.9）

- `rand::rng()` でRNG（乱数生成器）を取得
- `.random_range(1..=100)` で範囲指定の乱数を生成
- ※ 0.8では `thread_rng()` と `gen_range()` だったがリネームされた

### ループと制御フロー

- `loop` で無限ループ
- `break` でループを抜ける
- `continue` でループの先頭に戻る（ループ内でのみ使用可能）
