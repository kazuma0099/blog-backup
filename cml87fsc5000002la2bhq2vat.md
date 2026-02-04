---
title: "Shadowing variable di Rust"
datePublished: Wed Feb 04 2026 15:51:02 GMT+0000 (Coordinated Universal Time)
cuid: cml87fsc5000002la2bhq2vat
slug: shadowing-variable-di-rust
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/0eqgB57xMeA/upload/574f7fefb2299d04f729154b9b7bb0c8.jpeg
tags: rust, variable-shadowing

---

Apa itu shadowing variable, shadowing variable adalah di mana variabel yang dideklarasikan dalam lingkup (scope) lokal memiliki nama yang sama dengan variabel di lingkup global, untuk saya yang lebih sering menggunakan Java shadowing variable ini seolah - olah, jarang kelihatan karena oleh linter ketika kita membuat variable yang sama pada scope yang sama maka akan di highlight merah dan compiler akan memberitahu kalau variable ini sudah di declare sebagai contoh, ini bukan shadowing ini

```java
private void sayName(String name){
    // compiler akan memberitahu jika variable ini sudah di deklaraskikan
    String name = "Hello world";
    System.out.println(name)
}
```

contoh lain shadowing variable adalah variable antara inner class dan outerclass

```java
class OuterClass {
    String name = "outer name";

    Class InnerClass {
        String name = "inner name"
        
        private void print(String name){
            System.out.println(name)
            System.out.println(this.name);
            System.out.println(OutterClass.this.name);
        }
    }
}
```

Jika kita lihat disana sebenarnya pada Java ada shadowing variable tapi untuk menghilangkan ke ambiguan ini pada Java ada keyword `this` , yang mana ini membuat kita lebih mudah membaca program di Java, terus bagaimana shadowing variable di Rust?  
  
Shadowing variable pada Rust kurang lebih agak mirip-mirip dengan shadowing variable di JavaScript, jadi kita bisa declare variable dengan nama yang sama untuk tipe data juga boleh beda, yang perlu di ingat adalah value terakhir yang akan digunakan adalah value terakhir yang di deklarasikan contoh

```rust
fn main(){
    let a = 10;
    let a = String::from("hello");
    println!("{}", a);
}
```

Mungkin kita akan mikir ulang, ini kenapa perlu ada shadowing karena kalau keseringan dipakai bikin pusing bacanya, yup pertanyaan ini benar jika di implemntasikan di Java tapi jika di Rust mungkin akan menjadi keunggulan kalau dipikir, contohnya

```rust
fn increment(num: i32) -> i32 {
    return num + 1;
}

fn main() {
    let input_str = "42";
    let input_parsed = input_str.parse::<i32>().unwrap();
    let input_incremented = increment(input_parsed);
    let input_final = input_incremented + 3;
    println!("{}", input_final);
}
```

kalau dilihat diatas kita bakalan sibuk mikirin nama variable untuk setiap proses, coba kita lihat ketika kita meleverage manfaat shadowing

```rust
fn increment(num: i32) -> i32 {
    return num + 1;
}

fn main() {
    let input = "42";

    let input = input.parse::<i32>().unwrap();
    let input = increment(input);
    let input = input + 3;

    println!("{}", input);
}
```

lebih clean bukan dan kita bisa lebih fokus ke proses apa yang sebenarnya terjadi, namum perlu di note shadowing ini kaya pedang bermata dua kalau tidak wise menggunakannya ya pusing juga.