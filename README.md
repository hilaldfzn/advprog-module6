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

### Commit 2 Reflection notes

Pada function `handle_connection` terdapat beberapa baris kode tambahan, seperti di bawah ini.

```rust
let status_line = "HTTP/1.1 200 OK";
let contents = fs::read_to_string("hello.html").unwrap();
let length = contents.len();

let response = format!(
"{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
);

stream.write_all(response.as_bytes()).unwrap();
```

Kode tambahan tersebut bertanggung jawab untuk mengirimkan HTTP response ke client. Berikut adalah penjelasan tentang apa yang dilakukan oleh setiap baris tambahan dari kode di atas berdasarkan *Rust documentation*.
- `let status_line = "HTTP/1.1 200 OK";` menetapkan status HTTP response yang menunjukkan bahwa permintaan telah berhasil diproses. Status `200 OK` merupakan kode status standar dalam protokol HTTP yang menandakan keberhasilan umum tanpa error.

- `let contents = fs::read_to_string("hello.html").unwrap();` menggunakan function `read_to_string` dari modul fs untuk membaca seluruh isi dari file `hello.html` ke dalam sebuah String. Penggunaan `unwrap()` di sini menunjukkan bahwa program diharapkan untuk panic jika terjadi kesalahan saat membaca file, seperti file tidak ditemukan.

- `let length = contents.len();` menghitung panjang dari string contents yang telah diisi dengan isi dari file `hello.html`. Nilai ini kemudian digunakan untuk menentukan nilai Content-Length dalam header respons, yang menunjukkan ukuran dari body response dalam byte.

- `format!("{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}");` menyusun HTTP response lengkap yang terdiri dari status line, header Content-Length yang menunjukkan panjang dari body, diikuti oleh sebuah baris kosong sebagai pemisah antara header dan body, dan akhirnya isi dari file `hello.html` sebagai body dari respons.

- `stream.write_all(response.as_bytes()).unwrap();` mengirimkan HTTP response yang telah disusun ke client. Method `write_all` dari TcpStream digunakan untuk menulis semua byte dari string response ke dalam stream.Response dikonversi ke array byte dengan `as_bytes()`. Penggunaan `unwrap()` di sini menandakan bahwa program akan panic jika terjadi kesalahan saat menulis ke stream, seperti jika koneksi telah terputus.

![Commit 2 screen capture](/assets/images/commit2.png)