# Writeup LKS Cyber Security — Provinsi Banten 2026

Writeup penyelesaian soal **LKS (Lomba Kompetensi Siswa) bidang Cyber Security tingkat Provinsi Banten 2026**, format Jeopardy + Blue Team Hardening.

- **Peserta:** Nevan Kaindra Rusdi & Luthfi
- **Sekolah:** SMK Bina Informatika
- **Kategori:** Red Team (Jeopardy) & Blue Team (OS Hardening)

> Catatan: semua soal dikerjakan di lingkungan lab tertutup milik panitia lomba. Alamat IP, binary, dan flag di sini hanya berlaku untuk server lomba saat itu dan tidak bisa dipakai di luar konteks lomba.

## Daftar Isi

**BAB I — Red Team (Jeopardy)**
1. [Sanity Check — Binary Exploitation (PWN)](#1-sanity-check--pwn)
2. [No Moni — Web Exploitation](#2-no-moni--web-exploitation)
3. [Real Sanity Check — Welcome](#3-real-sanity-check)
4. [Baby Rev — Reverse Engineering](#4-baby-rev--reverse-engineering)
5. [Pramuka Cipher — Cryptography](#5-pramuka-cipher--cryptography)
6. [Among Us — Digital Forensics](#6-among-us--digital-forensics)
7. [qyuuurissss — Digital Forensics](#7-qyuuurissss--digital-forensics)

**BAB II — Blue Team (OS Hardening Linux)**
8. [Password Policy](#8-password-policy)
9. [Menu Bootloader GRUB](#9-menu-bootloader-grub)
10. [Pembatasan `su` (pam_wheel)](#10-pembatasan-su)
11. [SSH Hardening](#11-ssh-hardening)
12. [Parameter noexec pada /dev/shm](#12-parameter-noexec-devshm)

---

# BAB I — Red Team (Jeopardy)

## 1. Sanity Check — PWN

Kategori: **Binary Exploitation**. Binary dianalisis dengan Ghidra, dan ditemukan fungsi `win()` yang membaca `flag.txt` lalu mencetaknya — jadi ini tipe **ret2win**.

![Analisa Ghidra](images/page-03.png)

Di dalam `win()` terlihat buffer `local_78` berukuran **104 byte**, sementara input dibaca melebihi batas itu (buffer overflow).

![Fungsi win()](images/page-04.png)

Offset menuju return address dicari dengan pola **cyclic** di GDB (`cyclic 200`), dan didapat offset **120**.

![Cyclic di GDB](images/page-05.png)

Setelah itu dikumpulkan alamat fungsi `win` dan sebuah gadget `ret` (untuk menjaga stack alignment).

![Alamat win & gadget ret](images/page-06.png)

Payload disusun menggunakan **pwntools**: padding sepanjang offset, lalu gadget `ret`, lalu alamat `win`.

![Script payload bagian 1](images/page-07.png)
![Script payload bagian 2](images/page-08.png)

Setelah payload dikirim, server mengembalikan flag.

![Flag Sanity Check](images/page-09.png)

**Flag:** `LKS{don't-think-too-hard-la-keep-chill-aja-ini-baby-kok}`

---

## 2. No Moni — Web Exploitation

Kategori: **Web Exploitation**. Toko online "Belanja Seru" dengan fitur beli.

![Tampilan website](images/page-10.png)

Request fitur beli ditangkap dengan **Burp Suite**, dan responsnya diperiksa.

![Response di Burp](images/page-11.png)

Pada request `POST /api/checkout.php`, body JSON ternyata menyertakan field `price` yang dikirim dari sisi client — artinya harga bisa dimanipulasi (**client-side price tampering**).

![Request checkout](images/page-12.png)

Dengan mengubah `price` menjadi nilai yang sesuai, proses checkout berhasil dan flag muncul.

**Flag:** `LKS{bisa_gasi_b3s0k-b3s0k_kalo_beli_g0r3ngan_g4_amb1l_4_bilangnya_1_terus_Minta_k3mbal1An}`

> Pelajaran: validasi harga harus dilakukan di server, jangan pernah percaya harga yang dikirim client.

---

## 3. Real Sanity Check

Soal welcome (pemanasan).

![Welcome](images/page-14.png)

**Flag:** `LKS{welcome_to_LKS_Banten_2026}`

---

## 4. Baby Rev — Reverse Engineering

Kategori: **Reverse Engineering**. Dengan perintah `strings` pada binary, ditemukan data mencurigakan berupa token.

![Strings](images/page-15.png)
![Flag Baby Rev](images/page-16.png)

**Flag:** `LKS{tuh-kan-baby-kan}`

---

## 5. Pramuka Cipher — Cryptography

Kategori: **Cryptography**. Diberikan **kode Morse** yang perlu di-decode.

![Pramuka Cipher](images/page-17.png)

Hasil decode: `PR4MUKA-PR4MUKA-PR4J4-MUD4-K4R4NA`. Jika langsung disubmit salah karena masih ada spasi, jadi dirapikan dulu.

![Hasil rapi](images/page-18.png)

**Flag:** `LKS{PR4MUKA-PR4MUKA-PR4J4-MUD4-K4R4NA}`

---

## 6. Among Us — Digital Forensics

Kategori: **Digital Forensics**. Diberikan file `.png`. Setelah dicek format aslinya benar PNG, file di-scan untuk mencari pola atau data tersembunyi di tiap channel (**steganografi**).

![Analisa PNG](images/page-19.png)
![Flag Among Us](images/page-20.png)

**Flag:** `LKS{sussy_b4k4_4hh_ch4llenge}`

---

## 7. qyuuurissss — Digital Forensics

Kategori: **Digital Forensics**. Diberikan gambar QRIS yang salah satu pola sudutnya (finder pattern) hilang sehingga tidak bisa discan.

![QR rusak](images/page-21.png)

Solusinya: menyalin salah satu kotak sudut yang masih utuh (dari sudut lain), lalu menempelkannya di posisi sudut yang rusak dan diputar sesuai orientasinya. Setelah diperbaiki dan difoto ulang, QR bisa discan.

![QR diperbaiki](images/page-22.png)

**Flag:** `LKS{W0w_sebetulnya_1n1_id3_dari_tem4nku_y4ng_tuk4ng_fot0kop1_W}`

---

# BAB II — Blue Team (OS Hardening Linux)

Rangkuman langkah hardening sistem Linux pada sesi Blue Team.

## 8. Password Policy

Sistem harus menolak password yang terlalu pendek. Aktifkan `pam_pwquality` dan atur panjang minimum.

- Pasang `libpam-pwquality` bila belum ada.
- Edit `/etc/pam.d/common-password` → tambahkan/ubah baris `pam_pwquality` (mis. `minlen`).
- Atur juga `/etc/login.defs` (`PASS_MIN_LEN`, `PASS_MAX_DAYS`, dll).
- Uji dengan `passwd` ke user.

![Password Policy](images/page-23.png)

## 9. Menu Bootloader GRUB

Menu GRUB harus meminta autentikasi sebelum entri boot bisa diedit.

- Buat hash password GRUB dengan `grub-mkpasswd-pbkdf2`.
- Tambahkan superuser dan hash ke `/etc/grub.d/40_custom`.
- Jalankan `update-grub` agar konfigurasi diterapkan.

![GRUB](images/page-24.png)

## 10. Pembatasan `su`

User biasa harus mendapat *Permission Denied* saat menjalankan `su`.

- Edit `/etc/pam.d/su`, aktifkan baris `pam_wheel.so`.
- Pastikan hanya user di grup `sudo`/`wheel` yang boleh pakai `su`.

![pam_wheel](images/page-25.png)

## 11. SSH Hardening

SSH harus menolak login `root` dari luar.

- Edit `/etc/ssh/sshd_config` (`PermitRootLogin no`).
- Restart service SSH agar berlaku.
- Jika butuh SSH, gunakan `PasswordAuthentication yes` + `ssh-keygen` untuk perangkat yang diizinkan.

![SSH](images/page-26.png)

## 12. Parameter noexec /dev/shm

Partisi shared memory (`/dev/shm`) harus di-mount dengan opsi `noexec` permanen supaya file tidak bisa dieksekusi dari sana (mencegah malware yang dijalankan via `/dev/shm`).

- Edit `/etc/fstab`, tambahkan opsi mount `noexec` untuk `/dev/shm`.
- Remount agar langsung berlaku: `mount -o remount /dev/shm`.
- Uji dengan membuat file eksekusi di `/dev/shm` dan menjalankannya (harus gagal).

![noexec](images/page-27.png)

---

*Writeup ini dibuat untuk dokumentasi dan pembelajaran pribadi. Seluruh soal dikerjakan dalam lingkungan resmi lomba LKS Provinsi Banten 2026.*
