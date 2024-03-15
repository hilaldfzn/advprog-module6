### Commit 1 Reflection notes

What is inside the handle connection?

Sebelum membuat implementasi function dan membahas isinya, kita perlu import modul, seperti `io::prelude` dan `io::BufReader`. Function `handle_connection` dalam konteks server HTTP sederhana berfungsi untuk menangani koneksi TCP dari klien. Berikut adalah penjelasan lebih lanjut mengenai isi dari function tersebut berdasarkan *Rust documentation*.

```rust
let buf_reader = BufReader::new(&mut stream);
```

Pada baris kode di atas, kita membuat instansiasi objek `BufReader` menggunakan stream sebagai parameter. `BufReader` adalah struktur yang menyediakan kemampuan buffer untuk operasi baca yang sangat berguna dalam membaca data dari stream secara efisien, seperti `TcpStream` dalam kasus ini. Penggunaan `&mut stream` menunjukkan bahwa kita meminjam stream secara mutable, sehingga `BufReader` bisa membaca dari stream tersebut.

```rust
let http_request: Vec<_> = buf_reader
        .lines()
        .map(|result| result.unwrap())
        .take_while(|line| !line.is_empty())
        .collect();
```

Kode di atas melakukan beberapa langkah penting dalam proses pembacaan dan parsing HTTP request:

- `.lines()`: Berfungsi untuk memecah data stream menjadi iterator dari setiap baris. Hal ini berguna dalam konteks HTTP di mana request dan header diorganisir dalam baris-baris teks. Setiap kali newline (\n) ditemukan, sebuah baris baru diidentifikasi.

- `.map(|result| result.unwrap())`: Karena `.lines()` menghasilkan iterator dari Result, `.map` digunakan untuk mengonversi setiap Result menjadi String sebenarnya. `.unwrap()` dipanggil pada setiap Result, yang akan panic jika terdapat error. Hal ini merupakan pendekatan sederhana untuk menangani error.

- `.take_while(|line| !line.is_empty())`: Digunakan untuk terus membaca baris sampai menemukan baris kosong, yang menandakan akhir dari header HTTP request. Dalam protokol HTTP, header dan body dipisahkan oleh baris kosong, sehingga cara ini efektif untuk membaca hanya bagian header dari request.

- `collect()`: Mengumpulkan semua baris yang telah dibaca dan di-parse menjadi sebuah `Vec<String>`. Hasil akhirnya adalah semua baris header HTTP request yang telah dibaca.