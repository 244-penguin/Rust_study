# Chapter12: An I/O Project: Building a Command Line Program
- これまで学んできた知識を用いてRustによる`grep`コマンドを作成
    - `grep`コマンドの概要[TODO]
        - 指定したファイルから文字列を検索
        - エラーの出力
    - これまで学んできた知識[TODO]
        - コードの体系化（7章）
        - ベクタと文字列（8章）
        - エラー処理（9章）
        - 適切な箇所でトレイととライフタイムを使用（10章）
        - テストの記述（11章）

## 12.1 Accepting Command Line Arguments
- `grep`コマンドを作るには，引数としてファイル名を指定できるようにする必要がある
- とりあえず以下のように引数を受け取ってRustのプログラムを動かせるようにしたい
```
$ cargo run searchstring example-filename.txt
```

#### Reading the Argument Values
- コマンドライン引数を読み取るには，標準ライブラリ`std::env::args`を使用
    - コマンドライン引数のイテレータを返す
    - イテレータの詳細はChapter13
    - イテレータについてとりあえず知っておくこと
        - 一連の値を生成すること
        - イテレータを生成する要素全部を含むベクタなどのコレクションに変えられること
```rs
// src/main.rs
use std::env; //※1

fn main() {
    let args: Vec<String> = env::args().collect(); //※2
    println!("{:?}", args);
}
```
- ※1: `std::env::args`関数は2モジュール以上ネストされているので，親モジュールをスコープに入れて呼び出すのが一般的
    - `args`はよく使われる関数名なので，親モジュールから指定するのが良い
- ※2: `env::args()`が返したイテレータに対して`collect`を使用して，イテレータが生成する値すべてを含むベクタに変換
    - `collect`関数は多くの種類のコレクションを生成できるので，`Vec<String>`で明示的に文字列を返すよう指定する必要がある
- プログラムは実行すると，`[プログラムのバイナリ名]，[引数1]，[引数2]，，，`の形式で表示される
```
$ cargo run needle haystack
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/minigrep needle haystack`
["target/debug/minigrep", "needle", "haystack"] ←　ココが出力部分
```
#### Saving the Argument Values in Variables
- 引数の値を変数に保存する
```rs
// src/main.rs
use std::env; 

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1]; // ※1
    let filename = &args[2];

    // {}を探しています
    println!("Searching for {}", query);
    // {}というファイルの中
    println!("In file {}", filename);
}
```
- ※1 args[0]にはプログラムのバイナリ名が入っているので，添え字は1から
- 実行すると以下のように引数を取り出して表示できているのがわかる
```
$ cargo run test sample.txt
   Compiling minigrep v0.1.0 (/home/mizouchi/Rust_study/projects/Chapter12/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.39s
     Running `target/debug/minigrep test sample.txt`
Searching for test ← 検索文字列
In file sample.txt ← 検索対象のファイル
```
## 12.2 Reading a File
- 詩が書かれたテキストファイルを用意して，プログラムで読み込む
- 詩の内容は以下（Emily Dickinson作）
```
// porm.txt
I'm nobody! Who are you?
Are you nobody, too?
Then there's a pair of us - don't tell!
They'd banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

- ファイルの中身を読み込むプログラム
```rs
// src/main.rs
use std::env;
use std::fs::File; //ファイル操作のライブラリ
use std::io::prelude::*; //ファイル入出力処理のためのトレイトを含むライブラリ

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let filename = &args[2];

    // {}を探しています
    println!("Searching for {}", query);
    // {}というファイルの中
    println!("In file {}", filename);

    // ファイルが見つかりませんでした
    let mut f = File::open(filename).expect("file not found");

    let mut contents = String::new();
    f.read_to_string(&mut contents)
        // ファイルの読み込み中に問題がありました
        .expect("something went wrong reading the file");

    // テキストは\n{}です
    println!("With text:\n{}", contents);
}
```
- 実行すると，ファイルの中身が読み込まれているのがわかる
```
$ cargo run the poem.txt
   Compiling minigrep v0.1.0 (/home/mizouchi/Rust_study/projects/Chapter12/minigrep)
    Finished dev [unoptimized + debuginfo] target(s) in 0.44s
     Running `target/debug/minigrep the poem.txt`
Searching for the
In file poem.txt
With text:
I'm nobody! Who are you?
Are you nobody, too?
Then there's a pair of us - don't tell!
They'd banish us, you know.

How dreary to be somebody!
How public, like a frog
To tell your name the livelong day
To an admiring bog!
```

- 現状のコードの問題点
    - main関数が複数の機能を持ってしまっている
    - エラー処理がきちんと書かれていない
- 次はリファクタリング！

## 12.3 Refactoring to Improve Modularity and Error Handling
- 現状のプログラムの4つの問題
    1. main関数に複数の機能がある
    2. 
    3. ファイルを開くときのエラーメッセージが適切でない
        - ファイルが開けなかった理由がわからない
    4. 

#### Separation of Concerns for Binary Projects
- main関数の機能を分割させる手順
    1. プログラムをmain.rsとlib.rsに分け、ロジックをlib.rsに移動
        - main.rsは極力lib.rsに書かれたロジックを呼び出すだけにしたい
        - コマンドライン引数の解析ロジックが小規模な限り、main.rsに置いても良い
    2. コマンドライン引数の解析ロジックが複雑化してきたら、main.rsから抽出してlib.rsに移動

- まず，main.rsの中でmain関数からロジックを分離させる
    - 引数を取得して，`query`と`filename`を抽出する部分(下のコードの※1，2の部分)
    ```rs
    // src/main.rs

    // --snip--
    fn main() {
        let args: Vec<String> = env::args().collect();

        let query = &args[1];    // ※1
        let filename = &args[2]; // ※2 

    // --snip--
    }
    ```
    - 変更後のコードは以下
    ```rs
    // src/main.rs
    fn main() {
        let args: Vec<String> = env::args().collect();

        let config = Config::new(&args);

        // --snip--
    }

    struct Config {
        query: String,
        filename: String,
    }

    impl Config {
        fn new(args: &[String]) -> Config {
            let query = args[1].clone();
            let filename = args[2].clone();

            Config { query, filename }
        }
    }
    ```
    - ポイント
        - `query`と`filename`を`Config`構造体にまとめる
        - 変数に値を入れる処理は構造体のコンストラクタにさせる
        - 構造体に渡すデータは`clone`メソッドでデータのコピーを生成している

#### Fixing the Error Handling
- 引数の数が2より少ないとき，args[2]にアクセスしようとすると以下のようなエラーが起こる
```
thread 'main' panicked at 'index out of bounds: the len is 1 but the index is 1'
```
- しかし，プログラマではない人が見ても意味がわかりにくいので，わかりやすいメッセージを出すようにする
    - メッセージを"not enough arguments"にする
    - `panic!`を使うといらない情報（デバッグのための情報）まで与えてしまうので，使わないようにする
        - `Config`の`new`からは，`Result`を使って，成功時には`Config`インスタンス，エラー時にはエラーメッセージを返すようにして，main関数側でエラーが出たらエラーメッセージを表示してプログラムを終了する
        ```rs
        // src/main.rs
        //--snip--
        use std::process;

        fn main() {
            let args: Vec<String> = env::args().collect();

            let config = Config::new(&args).unwrap_or_else(|err| {
                // 引数解析時に問題
                println!("Problem parsing arguments: {}", err);
                process::exit(1);
            });
            //--snip--
        }
        //--snip--

        impl Config {
            fn new(args: &[String]) -> Result<Config, &'static str> {
                if args.len() < 3 {
                    return Err("not enough arguments");
                }

                let query = args[1].clone();
                let filename = args[2].clone();

                Ok(Config { query, filename })
            }
        }
        ```
        - ポイント
            - `Config`
                - コンストラクタは単に`Config`型を返すのではなく，成功時に`Config`型を，エラー時にエラーメッセージ（`&'static str`）を返す
            - main関数
                - `Config`のコンストラクタ呼び出し時に`unwrap_or_else`メソッドを使用
                    - `Resutl<T, E>`を受けとって，OkならOk時の値を返し，エラー時は設定した動作をさせることができる
                    - err変数で，`Resutl<T, E>`から受け取ったエラーメッセージを使用することもできる
        ```
        $ cargo run
        Compiling minigrep v0.1.0 (/home/mizouchi/Rust_study/projects/Chapter12/minigrep)
        --- 略 ---
            Finished dev [unoptimized + debuginfo] target(s) in 0.45s
            Running `target/debug/minigrep`
        Problem parsing arguments: not enough arguments
        ```
        - エラーメッセージがスッキリする

#### Extracting Logic from main


## 12.4 Developing the Library’s Functionality with Test-Driven Development

## 12.5 Working with Environment Variables

## 12.6 Writing Error Messages to Standard Error Instead of Standard Output

## Summary