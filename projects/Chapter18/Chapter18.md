# Chapter18: パターンとマッチング

- データの型や値に対してマッチングを行い特定のコードを実行するか決める
    - match式
    - 関数の引数　等
- 18.1: Rustでパターンが使われるケースを紹介
- 18.2: 論駁可能性　パターンがいつもマッチすることが期待されているケースとそうでないケースを解説
- 18.3: 色々なパターン記法を紹介

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
    - 論駁可能: マッチしないものが存在
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

### リテラルにマッチ

```rs
let x = 1;

match x {
    1 => println!("one"),       // 1
    2 => println!("two"),       // 2
    3 => println!("three"),     // 3
    _ => println!("anything"),  // なんでも
}
```

### 名前付き変数にマッチ

- match式は新しいスコープを開始するので，その中のパターンの一部として宣言された変数は既に宣言された同名の変数を覆い隠す
    - パターンの表記も変数宣言と同じ扱いなんだなぁ
```rs
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        // 50だったよ
        Some(50) => println!("Got 50"),
        // マッチしたよ
        Some(y) => println!("Matched, y = {:?}", y), // ← 最初に宣言されたyとは違うmatch式で宣言されたy　y == 5
        // 既定のケース
        _ => println!("Default case, x = {:?}", x),
    }

    // 最後にはx = {}, y = {}
    println!("at the end: x = {:?}, y = {:?}", x, y); // ← match式のスコープを抜けたのでy == 10
}
```
- match式内部の`y`は新しい変数であり最初に宣言した`y`ではない
- この`y`は`x`の値に束縛されるのでその値は5になる（`x`が`None`の場合はパターンにマッチしない）
- match式のスコープを抜けると最初に宣言された`y`の値（10）になる

- match式の外で宣言された`x`と`y`を比較したい場合は**マッチガード条件式**を使用する（後述）

### 複数のパターン

- `|`はorを意味し，複数のパターンにマッチさせられる
```rs
let x = 1;

match x {
    // 1か2
    1 | 2 => println!("one or two"),
    // 3
    3 => println!("three"),
    // なんでも
    _ => println!("anything"),
}
```

### ...で値の範囲に合致させる

- 数値かchar値のみで有効
    - 数値: `1...5 → 1|2|3|4|5`
    - char: `'a'...'e' → 'a'|'b'|'c'|'d'|'e'`
```rs
let x = 5;

match x {
    // 1から5まで
    1 ... 5 => println!("one through five"),
    // それ以外
    _ => println!("something else"),
}
```

### 分配して値を分解する

#### 構造体を分配する

- すでに宣言した構造体のフィールド値と合致する変数をパターンマッチで生成可能
- 下の例ではフィールド`x`と`y`に合致する`a`と`b`を生成
```rs
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x: a, y: b } = p;
    assert_eq!(0, a);
    assert_eq!(7, b);
}
```
- フィールド名と同名の変数を生成したいときは変数名を省略可能
```rs
fn main() {
    let p = Point { x: 0, y: 7 };

    let Point { x, y } = p; // ← 変数名を省略
    assert_eq!(0, x);
    assert_eq!(7, y);
}
```

- パターンマッチの中で一部だけリテラル値にすることも可能
- 下の例は2番目にマッチ
```rs
fn main() {
    let p = Point { x: 0, y: 7 };

    match p {
        // x軸上の{}
        Point { x, y: 0 } => println!("On the x axis at {}", x),
        // y軸上の{}
        Point { x: 0, y } => println!("On the y axis at {}", y),
        // どちらの軸上でもない: ({}, {})
        Point { x, y } => println!("On neither axis: ({}, {})", x, y),
    }
}
```

#### enumを分配する

```rs
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let msg = Message::ChangeColor(0, 160, 255);

    match msg {
        Message::Quit => {
            // Quit列挙子には分配すべきデータがない
            println!("The Quit variant has no data to destructure.")
        },
        Message::Move { x, y } => { // ← 構造体の時と似たパターンを使用可能
            println!(
                // x方向に{}、y方向に{}だけ動く
                "Move in the x direction {} and in the y direction {}",
                x,
                y
            );
        }
        // テキストメッセージ: {}
        Message::Write(text) => println!("Text message: {}", text),
        Message::ChangeColor(r, g, b) => { 
            println!(
                // 色を赤{}, 緑{}, 青{}に変更
                "Change the color to red {}, green {}, and blue {}",
                r,
                g,
                b
            )
        }
    }
}
```

#### 参照を分配する

- パターンと待ちさせている値に参照が含まれている場合，値から参照を分配する必要がある
- `&`を指定する
```rs
let points = vec![
    Point { x: 0, y: 0 },
    Point { x: 1, y: 5 },
    Point { x: 10, y: -3 },
];

let sum_of_squares: i32 = points
    .iter()
    .map(|&Point { x, y }| x * x + y * y)
    .sum();
}
```
- `&`を指定しなければ，型不一致エラー
    - `iter`が実際の値ではなくベクタの要素への参照を走査する

#### 構造体とタプルを分配する

```rs
let ((feet, inches), Point {x, y}) = ((3, 10), Point { x: 3, y: -10 });
```

### パターンの値を無視する

#### `_`で値全体を無視する

- `_`はどんな値にも一致するが，値を束縛しないワイルドカード
- matchアームの最後以外にも使える
```rs
fn foo(_: i32, y: i32) {
    // このコードは、y引数を使うだけです: {}
    println!("This code only uses the y parameter: {}", y);
}

fn main() {
    foo(3, 4);
}
```
- 上記のような書き方は，関数の引数が必要なくなった場合に，未使用の引数が含まれないようにする場合に使う
- 例えば，トレイトの実装時に自分の実装では引数の一つが必要ない場合など
- こうしておけば引数が未使用なことについてコンパイラに怒られることもない

#### ネストされた`_`で値の一部を無視する

- 下記の例では二つの設定変数に何か値（Some(x)）が設定されていれば何もしないが，何も設定されていない場合は`new_setting_value`を`setting_value`にセットする
```rs
let mut setting_value = Some(5);
let new_setting_value = Some(10);

match (setting_value, new_setting_value) {
    (Some(_), Some(_)) => {
        // 既存の値の変更を上書きできません
        println!("Can't overwrite an existing customized value");
    }
    _ => {
        setting_value = new_setting_value;
    }
}

// 設定は{:?}です
println!("setting is {:?}", setting_value);
```

- 下記の例はタプル内の2番目と4番目の値は無視してmatch式を実行している
```rs
let numbers = (2, 4, 8, 16, 32);

match numbers {
    (first, _, third, _, fifth) => {
        // 何らかの数値: {}, {}, {}
        println!("Some numbers: {}, {}, {}", first, third, fifth)
    },
}
```

#### 名前を`_`で始めて未使用の変数を無視する

- 通常未使用の変数に対してはコンパイラが警告を出すが，`_`を変数の頭につければ警告を出されない
    - プロジェクトのはじめなどには未使用の変数を置いておきたい場合もある

```rs
fn main() {
    let _x = 5;
    let y = 10;
}
```

- `_`単体では値を束縛しないが，`_`をつけた変数は値を束縛するので注意
```rs
if let Some(_) = s { // ← sの値のムーブが起きない
    println!("found a string");
}

if let Some(_s) = s { // ← sの値のムーブが起きる
    println!("found a string");
}
```

#### ..で値の残りの部分を無視する

- `..`パターンは、 パターンの残りで明示的にマッチさせていない値のどんな部分も無視できる
```rs
struct Point {
    x: i32,
    y: i32,
    z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x), // ← x以外のy, zを無視
}
```
- 途中のものを無視するという使い方もできる
```rs
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (first, .., last) => { // ← 途中の2, 3, 4番目の要素を無視
            println!("Some numbers: {}, {}", first, last);
        },
    }
}
```

- 無視する部分の定義があいまいだとコンパイルエラー
```rs
fn main() {
    let numbers = (2, 4, 8, 16, 32);

    match numbers {
        (.., second, ..) => { // ← どの部分を無視するか曖昧
            println!("Some numbers: {}", second)
        },
    }
}
```

### refとref mutでパターンに参照を生成する

- パターンマッチの際，値の所有権はパターンの変数に束縛されてしまうため，参照を使用して回避したい
- 下記のコードでは`robot_name`の値の所有権がmatch式の`name`変数に移ってしまうためエラーになる
```rs
let robot_name = Some(String::from("Bors"));

match robot_name {
    // 名前が見つかりました: {}
    Some(name) => println!("Found a name: {}", name),
    None => (),
}

// robot_nameは: {:?}
println!("robot_name is: {:?}", robot_name);
```

- `Some(name)`を`Some(&name)`にしたらいいように思えるが，パターンに置ける`&`記法は参照を生成せず，値の既存の参照にマッチするという他の役割があるため，この目的では使用できない
- パターンでは`&`の代わりに`ref`を使う

```rs
let robot_name = Some(String::from("Bors"));

match robot_name {
    Some(ref name) => println!("Found a name: {}", name),
    None => (),
}

println!("robot_name is: {:?}", robot_name);
```

- パターンで合致した値を可変化できるように可変参照を生成するには、`&mut`の代わりに`ref mut`を使用
```rs
let mut robot_name = Some(String::from("Bors"));

match robot_name {
    // 別の名前
    Some(ref mut name) => *name = String::from("Another name"),
    None => (),
}

println!("robot_name is: {:?}", robot_name);
```

### マッチガードで追加の条件式

- **マッチガード**: matchアームのパターンの後に指定されるパターンマッチングとともに，そのアームが選択されるのにマッチしなければならない追加のif条件式
```rs
let num = Some(4);

match num {
    // 5未満です: {}
    Some(x) if x < 5 => println!("less than five: {}", x), // ← if x < 5 の部分がマッチガード
    Some(x) => println!("{}", x),
    None => (),
}
```

- マッチガードを使うことで，マッチパターンの中では外部の変数が使えない（シャドーイングされてしまう）問題を解決できる
    - マッチガードの`if n == y`はパターンではなく，新しい変数を導入しない
```rs
fn main() {
    let x = Some(5);
    let y = 10;

    match x {
        Some(50) => println!("Got 50"),
        Some(n) if n == y => println!("Matched, n = {:?}", n),
        _ => println!("Default case, x = {:?}", x),
    }

    println!("at the end: x = {:?}, y = {:?}", x, y);
}
```

- `|`を使ったパターンと併用する場合，下記のようにふるまう
```rs
4 | 5 | 6 if y => ...

↓

(4 | 5 | 6) if y => ...
```

### @束縛

- `@`演算子により，値を保持する変数を生成する能登東寺にその値がパターンに一致するかを調べることができる

- パターンマッチで使用した変数を，アームに紐づいたコードで使用したい場合
```rs
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3...7 } => {
        // 範囲内のidが見つかりました: {}
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10...12 } => {
        // 別の範囲内のidが見つかりました
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        // それ以外のidが見つかりました
        println!("Found some other id: {}", id)
    },
}
```
- 一番目のアームでは`id_variable`に値が格納されているのでアームのコード内で値を使うことができる
- 二番目のアームでは値を格納していないので，どの数値にマッチしたかが不明
