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

### Commit 3 Reflection notes

Menambahkan file baru `404.html` yang akan ditampilkan ketika pengguna mengakses path yang tidak dikenal atau tidak tersedia di server.
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Hello!</title>
  </head>
  <body>
    <h1>Oops!</h1>
    <p>Sorry, I don't know what you're asking for.</p>
  </body>
</html>
```

Terdapat perubahan pada function `handle_connection` sebagai berikut.
```rust
let buf_reader = BufReader::new(&mut stream);
let request_line = buf_reader.lines().next().unwrap().unwrap();

if request_line == "GET / HTTP/1.1" {
    let status_line = "HTTP/1.1 200 OK";
    let contents = fs::read_to_string("hello.html").unwrap();
    let length = contents.len();

    let response = format!(
        "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
    );

    stream.write_all(response.as_bytes()).unwrap();
} else {
    let status_line = "HTTP/1.1 404 NOT FOUND";
    let contents = fs::read_to_string("404.html").unwrap();
    let length = contents.len();

    let response = format!(
        "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
    );

    stream.write_all(response.as_bytes()).unwrap();
}
```
`let request_line = buf_reader.lines().next().unwrap().unwrap();` berfungsi untuk membaca baris pertama dari HTTP request, yang biasanya berisi informasi tentang method HTTP dan path yang diminta. Penggunaan method `next()` digunakan untuk mengambil item pertama dari iterator yang merupakan baris pertama dari request, `unwrap()` yang pertama digunakan untuk mengurai Result dari Option, `unwrap()` yang kedua digunakan untuk mendapatkan String dari Result. Penggunaan *if-else statement* untuk memeriksa apakah path yang diminta valid. Jika pengguna hanya memasukkan url `http://127.0.0.1:7878/` dan `request_line` sesuai dengan `"GET / HTTP/1.1"`, maka server akan merespons dengan web page dari `hello.html`. Jika tidak, server akan merespons dengan web page baru dari `404.html` yang berupa halaman error.

Selanjutnya, kode tersebut di-*refactor* untuk mengurangi duplikasi dengan menggunakan pendekatan conditional untuk menentukan `status_line` dan `filename` dalam bentuk tuple berdasarkan `request_line`:
```rust
let (status_line, filename) = if request_line == "GET / HTTP/1.1" {
    ("HTTP/1.1 200 OK", "hello.html")
} else {
    ("HTTP/1.1 404 NOT FOUND", "404.html")
};

let contents = fs::read_to_string(filename).unwrap();
let length = contents.len();

let response = format!(
    "{status_line}\r\nContent-Length: {length}\r\n\r\n{contents}"
);

stream.write_all(response.as_bytes()).unwrap();
```

![Commit 3 screen capture](/assets/images/commit3.png)

### Commit 4 Reflection notes

Pada kode yang sekarang, penggunaan `match` digunakan untuk mencocokkan nilai dari `request_line` dengan beberapa pola yang telah ditentukan. Pola ini merepresentasikan berbagai path request yang mungkin diterima oleh server. Dalam konteks ini, `match` digunakan untuk menentukan response yang akan dikirimkan oleh server berdasarkan path yang diminta oleh pengguna.Selain itu, terdapat perubahan berupa penambahan *endpoint* `/sleep` yang menunjukkan bagaimana server yang bersifat *single-threaded* dapat menangani permintaan yang memerlukan waktu lebih lama untuk diproses. Ini dilakukan dengan memanggil path request baru, yaitu `"GET /sleep HTTP/1.1"` yang akan menjalankan `thread::sleep(Duration::from_secs(10));` untuk menunda pemrosesan selama 10 detik.

Dalam simulasi *slow request* ini, ketika pengguna membuka dua *browser windows* kemudian mencoba mengakses `127.0.0.1/sleep` dan `127.0.0.1` pada *windows* yang lain, pengguna akan menyadari bahwa server menangani request secara berurutan karena sifatnya yang *single-threaded*. Ketika request ke `/sleep` diproses, server tidak dapat menangani request lain hingga pemrosesan selesai, yang dalam kasus ini termasuk delay selama 10 detik. Hal ini mengilustrasikan bagaimana request yang memakan waktu dapat memblokir pemrosesan request lain pada server *single-threaded*.

### Commit 5 Reflection notes

Untuk menciptakan sistem yang menggunakan *multithreading*, perlu dibangun sebuah *ThreadPool* agar dapat mengelola sejumlah besar request secara simultan. Dalam prosesnya, pembuatan *Worker* juga diperlukan agar setiap request dapat dikirimkan ke *job* yang sesuai untuk tiap request. Oleh karena itu, *sender* perlu dibuat pada *ThreadPool* dan *receiver* pada setiap *Worker*, sehingga ketika request dieksekusi pada *ThreadPool*, sinyal dapat dikirim melalui *sender* ke *receiver* yang telah di-*assign* pada *Worker* untuk memproses *job*nya. Setiap *Worker* memiliki satu *thread* yang terus menunggu data. Ketika data diterima, *thread* tersebut akan *lock receiver*-nya dan memproses data hingga selesai, kemudian melepas kembali *lock*nya untuk memberi kesempatan *Worker* lain mendapatkan informasi dari *receiver*. Implementasi ini memastikan bahwa penggunaan *thread* menjadi lebih efisien dan sistem dapat menangani beban kerja yang lebih besar dengan lebih baik.