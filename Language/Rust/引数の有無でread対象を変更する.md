# [Rust] 引数の有無で read 対象をファイル・標準入力に変更する

## 概要

Rustで作成したプログラムの動作を、以下のようにしたかった。

* 引数にファイルが指定されている場合、ファイルの内容を読み込む。
* 引数にファイルが指定されていない場合、標準入力の内容を読み込む。

上記の動作を実現した方法を記載する。  

## 使用するプログラム  

説明では、ファイルの内容を表示し、行頭に行数を表示するプログラムを使用します。  
説明で使用するプログラムのソースコードは、以下にあります。  

プログラムの引数にファイル名が指定されている場合、ファイルの内容を表示します。  

```shell
$ cargo run test.txt
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/bufread_sample test.txt`
1: aaa
2: bbb
3: ccc
4: ddd
5: eee
6: fff
```

プログラムの引数に何も指定されていない場合は、標準入力の内容を表示します。  

```shell
$ echo -e "aaa\nbbb\nccc\nddd\neee\nfff" | cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/bufread_sample`
1: aaa
2: bbb
3: ccc
4: ddd
5: eee
6: fff
```

## 実装内容  

引数を受け取り、引数の数が  

* 1 より大きい場合、ファイルから読み込む。  
* 1 以下の場合、標準入力から読み込む。

とすることで、概要にて記載した動作を実現した。  

```rust
use std::{
    env::args,
    fs::File,
    io::{BufRead, BufReader, Stdin, stdin},
};

fn main() {
    // 引数を受け取る
    let args: Vec<String> = args().collect();

    if args.len() > 1 {
        // ファイルを read する処理
        let file: File = match File::open(&args[1]) {
            Ok(f) => f,
            Err(e) => panic!("{e}")
        };
        let br: BufReader<File> = BufReader::new(file);
        read_line(br);
    } else {
        // 標準入力を read する処理
        let stdin: Stdin = stdin();
        let br: BufReader<Stdin> = BufReader::new(stdin);
        read_line(br);
    }
}

/// ファイル・標準入力の内容を、行数ともに表示する関数。
fn read_line<T: BufRead>(br: T) {
    for (i, result) in br.lines().enumerate() {
        match result {
            Ok(line) => println!("{}: {}", i+1, line),
            Err(e) => panic!("{e}")
        }
    }
}
```

## 苦戦したところ

`ファイルから read する処理`と`標準入力から read する処理`の共通部分を関数にまとめるのに苦戦しました。  
上記の処理は、それぞれ以下のように実装していました。  

* ファイルから read する処理

    ```Rust
            let file: File = match File::open(&args[1]){
                Ok(f) => f,
                Err(e) => panic!("{e}")
            };

            let br: BufReader<File> = BufReader::new(file);

            for (i, result) in br.lines().enumerate() {
                match result {
                    Ok(line) => println!("{}: {}", i+1, line),
                    Err(e) => panic!("{e}")
                }
            }
    ```

* 標準入力から read する処理

    ```rust
            let stdin: Stdin = stdin();
            let br: BufReader<Stdin> = BufReader::new(stdin);

            for (i, result) in br.lines().enumerate() {
                match result {
                    Ok(line) => println!("{}: {}", i+1, line),
                    Err(e) => panic!("{e}")
                }
            }
    ```

ファイル・標準入力のどちらの場合でも、以下の処理は共通しています。  
これでは処理内容が重複しているため、[DRY原則](https://ja.wikipedia.org/wiki/Don%27t_repeat_yourself)に反しています。  

```rust
    for (i, result) in br.lines().enumerate() {
        match result {
            Ok(line) => println!("{}: {}", i+1, line),
            Err(e) => panic!("{e}")
        }
    }
```

重複している処理を関数にしたかったです。  
ファイル・標準入力のどちらの処理でも、`BufReader`を使用しているため、  
以下のように、引数に`BufReader`をとる関数を考えましたが、コンパイルエラーになりました。  

```rust
fn read_line(br: BufReader) {
    for (i, result) in br.lines().enumerate() {
        match result {
            Ok(line) => println!("{}: {}", i+1, line),
            Err(e) => panic!("{e}")
        }
    }
}
```

上記のように関数を定義する場合、引数に`BufReader<T>`と型を指定する必要があります。  
以下のように`BufReader<File>`とすることでコンパイルエラーは解消できます。  

```rust
fn read_line(br: BufReader<File>) {
    for (i, result) in br.lines().enumerate() {
        match result {
            Ok(line) => println!("{}: {}", i+1, line),
            Err(e) => panic!("{e}")
        }
    }
}
```

しかし、引数を`BufReader<File>`とすると`BufReader<Stdin>`を引数に受け取ることができません。（逆も然りです）  
そのため、`BufReader<File>`, `BufReader<Stdin>`のどちらにも対応できる関数にはなりません。  

`BufReader`は、`BufRead`トレイトを実装しているため、ジェネリクスを使って、以下のように定義することで実現できました。

```rust
fn read_line<T: BufRead>(br: T) {
    for (i, result) in br.lines().enumerate() {
        match result {
            Ok(line) => println!("{}: {}", i+1, line),
            Err(e) => panic!("{e}")
        }
    }
}
```
