# Chapter02
- ランダム生成された数字の推測ゲーム作り
- Rustでは変数はデフォルトでimmutable
    - 変数宣言時に`mut`をつけるとmutableになる
```
let foo = 5; // immutable
let mut bar = 5; // mutable
```
- `&`を変数の頭につけることで参照を表す
    - メモリの番地に入っている変数情報を書き換えられるようになる？  
    - 教科書には「コピーせずに変数にアクセスできるようになる」と書いてある
- これもmutableにするには`&mut guess`のように`mut`をつけないといけない
- `read_line`は`io::Result`を返す
    - `Result`は決まった値の集合（enumerations)
    - 関数が成功した場合は`Ok`、失敗した場合は`Err`を返す
    - `Err`にはエラー情報が含まれる
    - `Err`の場合、プログラムはクラッシュして`.expect`に設定されたエラーメッセージが表示される
    - `.expect`を書かなかった場合、コンパイルは通るがwarningが出る
    - `?`というのも存在する　（新しめの言語ではよくあるらしい）
        - https://qiita.com/kanna/items/a0c10a0563573d5b2ed0
            - https://doc.rust-lang.org/edition-guide/rust-2018/error-handling-and-panics/the-question-mark-operator-for-easier-error-handling.html
- `println!`の中に変数情報を入れる方法はpythonのフォーマット文に似ている
    - `println!("x = {} and y = {}", x, y);`
- 必要な依存関係は`Cargo.toml`に書く
    - バージョンを`^0.8.3`と書くと、「0.8.3以上0.9.0より下」という意味になる
    - パッケージは`Crates.io`というオープンソースのRustプロジェクトから取ってきている
    - 今回はrandだけ指定しているが、ビルド時にはrandと依存関係にあるパッケージもインストールされる
- 依存関係にあるクレートのアップデートは`cargo update`
    - `Cargo.toml`に書いてあるクレートのアップデートができる
    - 自分のやつはなぜかアップデートされなかった
        - 0.8.3のまま

- armとは？
- cmp関数はいつ実行されるの？
    - cmp関数が実行されてその結果をmatch関数が受け取って処理してる感じ
    - cmp関数は大きいとか小さいとかいう結果を返す
- 同じ変数名で型を変えたものを作ることができる
    - シャドウィングと呼んでる
    - 別の変数に同じ名前をつけている
    - 変数のスコープ内では元の変数（関数）を同じ名前では呼び出せなくなる
        - https://en.wikipedia.org/wiki/Variable_shadowing