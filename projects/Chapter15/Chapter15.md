# Chapter18: パターンとマッチング

## 18.1パターンが使用されることのある箇所全部

### matchアーム
    - match式の中でパターンと右矢印，パターンにマッチした時に実行される式の部分
    ```rs
    match VALUE {
        PATTERN => EXPRESSION, // ←matchアーム
        PATTERN => EXPRESSION,
        PATTERN => EXPRESSION,
    }
    ```
- match式はすべてのパターンを網羅しておく必要がある
- `_`はすべてのパターンにマッチ
```rs
match some_u8_value {
    1 => println!("one"),
    3 => println!("three"),
    5 => println!("five"),
    7 => println!("seven"),
    _ => (),
}
```
### if let
- `.parse()`は文字列を数値に変換する
- `Ok()`はオペレーションが成功したことを意味する
- `Option<T>`は値が 存在しない 可能性を暗示する列挙型
- `Result<T,E>`はエラーになる可能性 を暗示する列挙型
- match式ではコンパイラがパターンの網羅性を確認してくれる
    - `if let`でやってくれない

### while let

- パターンが合致し続ける限り、whileループを走らせる
- `stack.pop()`が`None`を返すまでpopをくり返す
```rs
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
    println!("{}", top);
}
```

### forループ

```rs
let v = vec!['a', 'b', 'c'];

for (index, value) in v.iter().enumerate() {
    println!("{} is at index {}", value, index);
}
```

### let文

- `let x = 5`のような単純な変数宣言も実はパターン
```rs
let PATTERN = EXPRESSION;
```
- 値は何であれ変数xにマッチするものを代入する
- 以下の例がわかりやすい
```rs
let (x, y) = (1, 2, 3);
```
- 左辺と右辺のパターンが違うのでコンパイルエラーになる

### 関数の引数

- 下記の例では，xはi32の値一つにマッチする
```rs
fn foo(x: i32) {
    // コードがここに来る
    // code goes here
}
```

## 論駁可能性: パターンが合致しないかどうか

- 論駁: 相手の説に反対して、論じ攻撃すること
- パターンには論駁可能なものと不可能なものがある
    - 論駁可能: マッチしないモノが存在
        - if let
        - while let等
    - 論駁不可能: すべてのパターンにマッチ
        - let
        - for等
```rs
let Some(x) = some_option_value;
```
- ↑は`some_option_value`が`Some()`以外の値（None）をとる可能性がある
    - letは論駁不可能なはずなのでコンパイルエラー
- if letを使う
```rs
if let Some(x) = some_option_value {
    println!("{}", x);
}
```

- 逆にいつも成り立つ式をif letに与えると，これもエラーになる
    - 論駁可能なものには論駁可能な式しか与えられない
    - 厳しい

## パターン記法
