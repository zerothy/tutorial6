# Reflection

## Reflection 1
Fungsi `handle_connection` adalah fungsi yang bertujuan untuk memproses koneksi TCP yang masuk dari klien dengan menggunakan `TcpStream`. Sesuai dengan dokumentasi Rust yang saya baca, `TcpStream` merepresentasikan koneksi socket antara server dan client. Fungsi ini menggunakan `BufReader` Untuk membaca data secara efisien, yang mana sesuai dokumentasi, `BufReader` adalah wrapper yang membuffer operasi I/O, mengurangi overhead pembacaan langsung dari stream. Dengan method `.lines()` yang digunakan pada `BufReader`, kita dapat membaca data dari stream per baris sebagai iterator yang menghasilkan `Result<String>`, dimana setiap baris di-`unwrap()` dan di-`match` untuk mendapatkan data yang diinginkan.

Pembacaan dihentikan ketika bertemu baris kosong dengan menggunakan `.take_while(|line| !line.is_empty())`, mengikuti konvensi HTTP dimana header diakhiri dengan baris kosong `\r\n\r\n`. Tapi, fungsi ini belum sepenuhnya benar. Method `.lines()` pada Rust memisahkan baris pada `\n`, akan tetapi HTTP header dipisahkan dengan `\r\n`. Hingga bisa saja baris masih mengandung `\r` sehingga pemeriksaan `line.is_empty()` tidak akan benar. Seharusnya kita menggunakan `line.trim().is_empty()` agar baris yang mengandung `\r` juga dianggap kosong. Setelah membaca header HTTP, fungsi akan mengeprint hasilnya ke konsol tanpa melakukan apapun ke klien.

Secara keseluruhan, fungsi ini merupakan contoh dasar bagaimana cara membaca TcpStream dan mengolah data yang diterima. Namun, fungsi ini belum sepenuhnya benar karena masih ada kekurangan dalam mengecek baris kosong pada header HTTP.

## Reflection 2
Setelah membaca dan mempelajari kode yang baru, saya mengetahui bahwa pembuatan response HTTP adalah suatu poin yang penting. Response yang digunakan adalah `200 OK` dan membaca isi file `hello.html`. Lalu ada penambahan header `Content-Length`, yang mana merupakan panjang dari isi file `hello.html`. Hal ini penting karena header tersebut memberitahu klien berapa panjang data yang akan diterima. Jika tidak ada header tersebut, klien tidak akan tahu kapan data sudah selesai diterima. Format response juga harus mengikuti standar HTTP, dimana setiap header dipisahkan dengan `\r\n` dan diakhiri dengan `\r\n\r\n`.

Selain itu, penggunaan `unwrap()` pada `read_to_string` dan `write_all` juga harus diperhatikan. `unwrap()` akan menghasilkan panic jika terjadi error. Hal ini memperlihatkan bahwa kode ini masih bersifat sederhana dan belum menangani error secara robust. Melalui kode baru ini, saya mempelajari struktur data server HTTP, header-header seperti `Content-Length`, dan cara Rust mengelola I/O jaringan.

Berikut screenshot hasil dari commit kedua:

![Commit 2 screen capture](/assets/images/commit2.png)

## Reflection 3
Refactoring dilakukan untuk meningkatkan maintainability dan readability dari kode. Sebelum di refactor, logic pembuatan response HTTP diulang dua kali untuk kasus response 200 dan 404. Hal ini dapat membuat terjadinya inkonsistensi jika ada perubahan yang akan dilakukan nanti. Dengan memisahkan status line dan nama file ke `if else`, logic dari routingnya akan terlihat lebih jelas. Dan jika ada perubahan di masa depan, maka hanya akan dilakukan sekali saja di tempat tersebut.

Untuk lebih jelas, perubahannya terlihat pada tuple `status_line` dan `file_name`, serta penghapusan duplikasi kode untuk membaca file dan mengirim response. Berikut screenshotnya:

![Commit 3 screen capture](/assets/images/commit3.png)

## Reflection 4
Kode ini menggunakan thread blocking untuk mensimulasikan proses yang berat. Ketika mengakses `/sleep`, server akan tidur selama 10 detik, lalu akan mengirim response. Karena kode masih berjalan dalam single thread, semua request akan dilakukan secara berurutan. Jika 2 browser mengakses hal yang sama, satu `/sleep` dan satunya lagi `/`, maka `/` akan muncul setelah 10 detik. Pada real life situation, hal ini akan menyebabkan bottleneck kalau banyak pengguna mengakses endpoint yang membutuhkan waktu lama. Solusi yang banyak digunakan adalah menggunakan multithreading atau async programming.