+++
author = "Sammi"
title = "Rust ~ Ownership, Borrowing & Lifetimes"
date = "2021-11-13"
description = "Membahas Ownership, Borrowing & Lifetimes di Bahasa Pemrograman Rust"
tags = [
    "Rust",
    "Ownership",
    "Rustacean",
]
+++

### Preface

*Ownership, Borrowing & Lifetimes* adalah *The most of distinct and compelling features in Rust* artinya ketiga hal tersebut adalah hal paling mendasar & penting untuk mulai mempelajari Bahasa Pemrograman Rust.

Rust adalah bahasa pemrograman yang fokus pada keamanan (*safety*) dan kecepatan (*speed*) dengan fokus tersebut, Rust dapat memenuhi tujuan *zero-cost abstractions* artinya bahwa dengan bahasa Rust kita dapat membuat aplikasi dengan biaya yang sedikit mungkin, biaya yang dimaksudkan adalah biaya alokasi memori. 

*`Ownership`* adalah contoh utama dari *zero-cost abstractions*. Untuk mempelajari Rust memang harus lebih disiplin dan jika anda mempunyai pengalaman dalam bahasa pemrograman lainnya maka anda perlu belajar kembali tentang kedisiplinan penulisan maupun penyusunan kode program karena dalam hal ini Rust adalah bahasa yang jauh berbeda dengan bahasa lainnya.

### Ownership

Rust hanya memperbolehkan satu *variable* hanya memiliki satu pemilik saja atau apapun yang di ikatkan (*binding*) dengan *variable* tersebut, ini berarti ketika *binding* tersebut keluar dari ruang lingkupnya (*scope*) maka, dengan otomatis Rust akan mengosongkan sumber daya terkait (*will free the bound of resources*), contohnya gini:

```Rust
fn hello() {
    let msg = "Hello Universe";
}
```
Ketika *function* **hello()** di panggil dan disitu terdapat variable *msg* maka string baru akan di buat kedalam *stack*, dan akan di alokasikan ruang memori kedalam **heap** untuk variable *msg*. Tapi ketika function **hello()** sudah selesai melaksanakan tugasnya dan variable *msg* selesai dieksekusi maka Rust akan membersihkan ruang yang tadinya dipakai oleh *variable msg* dan semua yang berhubungan dengan alokasi ruang yang dipakai dalam *function* tersebut. Perlu diketahui bahwa ruang lingkup (*scope*) diawali dengan **{** dan di akhiri **}**.

Rust selalu memastikan bahwa hanya ada satu pemilik untuk satu *binding* (*variable*) untuk semua *resource* yang diberikan. Contoh:

```Rust
let students = vec!["Sammi", "Faren", "Chiko"];
let students2 = students;
println!("The students {}", students[0]);
```

Kita mempunyai satu *variable students* dan *variable* tersebut sudah di berikan ke *variable* lainnya yaitu *students2*, jika kita panggil kembali variable *students* maka kita akan mendapatan *error* seperti ini :

```
use of moved value: `students`
let students2 = students;
    ----- value moved here
println!("The students {}", students[0]);
                           ^^^^ value used here after move
```

Itu artinya, kita tidak boleh menggunakan *variable* yang sudah di pindahkan ke *variable* atau *binding* lainnya.

Ketika kita mempunyai kode seperti ini :

```Rust
let age = 20;
```

Maka rust akan mengalokasikan memori dengan tipe data *integer i32* ke dalam *stack* lalu menyalin alamat memori (*bit pattern*) dan menghubungkan ke variable *age* tadi.

Mari kita bahas lebih dalam lagi tentang *Ownership*. Jika kita mempunyai kode seperti ini:

```Rust
let students = vec!["Sammi", "Faren", "Chiko"];
let mut students2 = students;
```
Pada baris pertama, Rust mengalokasikan memori untuk objek vektor **students** kedalam *stack*, tapi selain itu Rust juga mengalokasikan beberapa memori di *heap* untuk data aktual `(["Sammi", "Faren", "Chiko"])` lalu Rust menyalin alamat memori di *heap* tadi kedalam *pointer internal* yang merupakan bagian dari objek vektor tadi yang ditempatkan di *stack* (ini disebut data *pointer*). Kedua bagian dari vektor (satu di *stack* dan satu lagi di *heap*) panjang (*length*), *capacity* (kapasitas) dan lain-lain adalah sama nilainya.

Ketika kita memindahkan **students** ke **students2**, Sebenarnya Rust menyalin *bitwise* dari objek vektor **students** ke dalam alokasi *stack* untuk **students2** tapi salinan ini tidak membuat kopian dari alokasi di *heap* yang berisi data aktual. Artinya bahwa akan ada dua pointer untuk kedua objek vector yang mengarah ke *heap* yang sama.

Lalu bagaimana jika kedua objek tersebut diakses secara bersamaan? *For example*, jika kita menghapus elemen ke tiga dari objek **students2** :

```Rust
students2.truncate(2);
```

Sedangkan objek **students** masih diakses maka kita akan mendapatkan objek vektor yang tidak lagi valid karena objek **students** tidak akan tahu bahwa salah satu elemen telah dihapuskan. Sekarang objek **students** pada stack tidak lagi sama dengan di *heap*, ini adalah kesalahan yang *risk*-an, karena menyebabkan kesalahan segmentasi atau lebih buruknya lagi memunginkan pengguna yang tidak sah untuk membaca alamat memori yang sudah tidak lagi memiliki akses. Inilah sebabnya mengapa Rust melarang untuk menggunakan objek yang sebelumnya sudah dipindahkan/dipergunakan.

#### *Copy types*

Rust sudah menetapkan bahwa ketika *ownership* sudah di transfer ke *binding* yang lain maka kita tidak dapat lagi menggunakan *binding* yang sebelumnya (asli). Namun kita bisa menggunakan fitur dari *trait* untuk menangani masalah ini yaitu **Copy**, Contohnya gini:

```Rust
let a = 10;
let aa = a;
println!("A is: {}", a);
```

*In this case*, tipe a adalah **i32**, yang mengimplementasikan **trait Copy**. Ketika kita meng-assign a ke aa maka duplikat dari a akan dibuat tapi tidak dipindahkan dan kita tetap dapat mengakses a. Karena **i32** memiliki *pointer* ke data yang lain.

Semua *Primitive Types* adalah implementasi dari **trait Copy** dan kepemilikan nya tidak dipindahkan seperti yang diasumsikan mengikuti `aturan ownership`. Untuk mengilustrasikannya, 2 potongan kode dibawah ini hanya mengkompilasi tipe data **i32** dan **boolean**.

```Rust
fn main() {
    let var_x = 5;
    let _y = double(var_x);
    println!("{}", var_x);
}
fn double(x: i32) -> i32 {
    x * 2
}
```

```Rust
fn main() {
    let var_x = true;
    let _y = change_truth(var_x);
    println!("{}", var_x);
}
fn change_truth(x: bool) -> bool {
    !x
}
```
Jika kita menggunakan tipe data yang tidak mengimplementasikan **trait Copy**, maka pada saat kompilasi kita akan mendapatkan pesan *error* karena kita mencoba untuk memindahkan nilai dari *variable* diatas.

```
error: use of moved value: `var_x`
println!("{}", var_x);
               ^
```

[Referensi](https://doc.rust-lang.org/stable/book)

### Borrowing
....

### Lifetimes
....
