# WIREFRAME — Website Quisioner DUMI UNPAK

Prinsip: mobile-first. Format: ASCII + anotasi + state + interaksi.

---

## A. AUTH (login only — tanpa forgot/reset password)

### A1. Login

```text
+--------------------------------------------------+
|                    [ LOGO ]                      |
|               Website Quisioner                  |
|                                                  |
|  Email    [_________________________________]    |
|  Password [_________________________________] (o)|
|  [x] Ingat saya                                  |
|                                                  |
|  [                  MASUK                    ]   |
|                                                  |
|  Error: [!] Email atau password salah.           |
|  Error: [!] Akun dinonaktifkan, hubungi admin.   |
+--------------------------------------------------+
```

- **State**: idle | loading (spinner) | error.
- **Interaksi**: Enter = submit; 5× gagal → throttle.
- **Catatan**: tidak ada menu "Lupa password".

---

## B. DASHBOARD

```text
+--------+-----------------------------------------------+
| SIDEBAR|  Dashboard                                     |
|--------|  Filter: [Kuesioner v] [Periode v]  [Terapkan]  |
| Dash   |  +----------+ +---------+ +-------------+       |
| Ktgri  |  |Pertanyaan| |Kategori | |Responded    |       |
| Soal   |  |   128    | |    7    | | 353 submitted|      |
| Kuesio |  |          | |         | | +12 draft    |      |
| Respon |  +----------+ +---------+ +-------------+       |
| User   |  +---------------------+ +------------------+  |
| Role   |  | Skor %              | | Distribusi Opsi  |  |
|        |  | [##########----]    | | YA    170 (48%)  |  |
|        |  +---------------------+ | BLM   182 (52%)  |  |
|        |  | Respons / Bulan     | | TDK     1 (0%)   |  |
|        |  |  # ## ### ## #      | +------------------+  |
|        |  |  F M A M J J A S O  |                       |
|        |  +---------------------+                       |
|        |  Soal Terendah (Skor %): 1.Toilet 2.Katering        |
+--------+-----------------------------------------------+
```

- **State**: loading (skeleton) | empty ("Belum ada respons").
- **Filter**: berada di dalam widget chart (ikon filter), bukan bar filter di atas.

---

## C. KATEGORI SOAL

### C1. List

```text
+----------------------------------------------------------+
| Kategori Soal                          [+ Tambah Kategori]|
|                                                          |
| Fasilitas      3 soal   [aktif]  [edit][soft-del]        |
| Pelayanan      2 soal   [aktif]  [edit][soft-del]        |
| Produk         2 soal   [aktif]  [edit][soft-del]        |
| Kepuasan       1 soal   [aktif]  [edit][soft-del]        |
| (umum)         0 soal   -        [lihat soal]            |
|                                                          |
+----------------------------------------------------------+
```

- **Empty**: "Belum ada kategori. Tambah kategori pertama."

### C2. Form

```text
+----------------------------------------------------------+
| Nama* [____________]  Slug [auto________]  Urutan [10]    |
| Deskripsi [__________________________]   Aktif [x]        |
| [ SIMPAN ]  [ BATAL ]                                     |
+----------------------------------------------------------+
```

> **Deskripsi** = deskripsi kategori; tampil di atas soal pada step publik (line break dipertahankan).

---

## D. SOAL

### D1. List

```text
+------------------------------------------------------------------------+
| Soal       Filter: [Kategori: Fasilitas v]         [+ Tambah Soal]      |
|                                                                        |
| "Fasilitas Toilet"       Rating 1-7  [wajib] [aktif] [edit][del]       |
| "Fasilitas Meeting Poin" Rating 1-7  [     ] [aktif] [edit][del]       |
| "Fasilitas Wara Wiri"    Rating 1-7  [wajib] [aktif] [edit][del]       |
+------------------------------------------------------------------------+
```

### D2. Form

```text
+----------------------------------------------------------+
| Label*   [____________________________________]           |
| Kategori [ Fasilitas          v ] (opsional)              |
| Teks bantuan [________________________________]           |
|                                                          |
| Tipe*    (o)Rating ( )Radio                               |
|                                                          |
| -- Rating --  Skala min[0] max[7] (dinamis)  Label sesuai skala|
| -- Select/Radio -- Opsi: [ ..... ] [x] (+ tambah)         |
|                                                          |
| [x] Wajib dijawab                                        |
| Urutan [10]  Aktif [x]      [ SIMPAN ] [ BATAL ]         |
+----------------------------------------------------------+
```

- **Teks bantuan**: tampil di bawah field soal di form publik.
- **Tipe**: hanya `Rating` & `Radio` tersedia (tipe lain arsitektur, tidak tampil di form).
- **Skala rating**: divalidasi `min < max`.

---

## E. KUESIONER (LINK PUBLIK)

### E1. List

```text
+------------------------------------------------------------------------+
| Kuesioner                                        [+ Buat Kuesioner]     |
|                                                                        |
| Penilaian Kunjungan Q2   7 soal   [published]  [QR][edit][tutup]        |
| Survei Kepuasan Juli     12 soal  [draft]      [--][edit][del]          |
+------------------------------------------------------------------------+
```

### E2a. Builder — Mode "Soal Only"

```text
+-----------------------------------------------------------------+
| Judul* [ Penilaian Kunjungan Q2 ]                               |
| Mode soal: (o) Soal Only  ( ) Kategori  ( ) Custom / Mix        |
|-----------------------------------------------------------------|
| IDENTITAS KUESIONER                                      |
|  Nama          [Tampilkan x] [Wajib x]                   |
|  Sekolah       [Tampilkan x] [Wajib x]                   |
|  Tgl Kunjungan [Tampilkan x] [Wajib x]                   |
|-----------------------------------------------------------------|
| PILIH SOAL (tanpa kategori)                                     |
|  [ cari soal.. v ]  [+ Tambah]                                  |
|   - Fasilitas Toilet                     [x]  [hapus]           |
|   - Penjelasan Sales                     [x]  [hapus]           |
|   - Niat Kembali                         [x]  [hapus]           |
| Terpilih: 3 soal (dalam 1 step flat)                            |
|                                                                 |
| Buka [ 2026-07-01 08:00 ]  Tutup [ 2026-07-31 23:59 ]           |
|                                                                 |
| [ SIMPAN DRAFT ]        [ PUBLISH + QR ]                        |
+-----------------------------------------------------------------+
```

### E2b. Builder — Mode "Kategori"

```text
+-----------------------------------------------------------------+
| Judul* [ Penilaian Kunjungan Q2 ]                               |
| Mode soal: ( ) Soal Only  (o) Kategori  ( ) Custom / Mix        |
|-----------------------------------------------------------------|
| IDENTITAS KUESIONER                                      |
|  Nama          [Tampilkan x] [Wajib x]                   |
|  Sekolah       [Tampilkan x] [Wajib x]                   |
|  Tgl Kunjungan [Tampilkan x] [Wajib x]                   |
|-----------------------------------------------------------------|
| PILIH KATEGORI (centang = ambil semua soal di dalamnya)         |
|  [x] v Fasilitas   (3)  -> [x]Toilet [x]Meeting [x]Wara          |
|  [x] v Pelayanan   (2)  -> [x]Sales  [x]Tour Guide              |
|  [ ] > Produk      (2)                                          |
|  [x] > Kepuasan    (1)                                          |
| Terpilih: 6 soal / 3 kategori                                   |
|                                                                 |
| Buka [ 2026-07-01 08:00 ]  Tutup [ 2026-07-31 23:59 ]           |
|                                                                 |
| [ SIMPAN DRAFT ]        [ PUBLISH + QR ]                        |
+-----------------------------------------------------------------+
```

### E2c. Builder — Mode "Custom / Mix"

```text
+-----------------------------------------------------------------+
| Judul* [ Penilaian Kunjungan Q2 ]                               |
| Mode soal: ( ) Soal Only  ( ) Kategori  (o) Custom / Mix        |
|-----------------------------------------------------------------|
| IDENTITAS KUESIONER                                      |
|  Nama          [Tampilkan x] [Wajib x]                   |
|  Sekolah       [Tampilkan x] [Wajib x]                   |
|  Tgl Kunjungan [Tampilkan x] [Wajib x]                   |
|-----------------------------------------------------------------|
| KATEGORI (ambil borongan)                                       |
|  [x] v Fasilitas   (3)  -> [x]Toilet [x]Meeting [x]Wara          |
|  [ ] > Pelayanan   (2)                                          |
|-----------------------------------------------------------------|
| SOAL INDIVIDUAL DITAMBAH                                        |
|  [ cari soal.. v ]  [+ Tambah]                                  |
|   - Pengarahan Tour Guide               [x]  [hapus]            |
|   - Produk: Citarasa Katering           [x]  [hapus]            |
|-----------------------------------------------------------------|
| Terpilih: 5 soal (3 kategori + 2 individual)                    |
|                                                                 |
| Buka [ 2026-07-01 08:00 ]  Tutup [ 2026-07-31 23:59 ]           |
|                                                                 |
| [ SIMPAN DRAFT ]        [ PUBLISH + QR ]                        |
+-----------------------------------------------------------------+
```

- **Publish**: konfirmasi "Soal, group, dan identitas akan dibekukan. Setelah publish tidak bisa diubah; buat kuesioner baru untuk perubahan. Lanjut?"
- **Identitas**: dapat diubah saat draft; terkunci setelah publish.
- **Error**: "Minimal 1 soal", "Tutup harus setelah Buka".

### E3. QR Modal

```text
+--------------------------------------------------+
| QR +----------------+  https://app.id/q/a1b2c3d4 |
|    | ## qr ## ## ## |  [Download QR SVG] [link]  |
|    +----------------+                            |
+--------------------------------------------------+
```

---

## F. FORM PUBLIK (WIZARD)

### F1. Identitas

```text
+--------------------------------------------------+
|  Penilaian Kunjungan Q2                          |
|  Deskripsi singkat kuesioner...                  |
|                                                  |
|  Nama*[____] Sekolah*[____] Tgl Kunjungan*[date] |
|  (Sumber: identity_fields; hidden/optional OK)   |
|                                                  |
+--------------------------------------------------+
```

> Draft dibuat saat interaksi pertama (blur/pilih opsi/Next); membuka halaman tidak membuat draft.

### F2a. Step — Mode "Custom / Mix"

```text
+----------------------------------------------------------+
| Langkah N / N  [=====================]                   |
| Kategori: Umum (soal individual)                         |
|                                                          |
| N.1 Niat Kembali *       ( )YA ( )BELUM TAU ( )TIDAK      |  <- tipe radio kategorikal
| N.2 <soal individual> *  (1)(2)(3)(4)(5)(6)(7)            |  <- tipe rating (radio skala)
|                                                          |
| [!] Soal bertanda * wajib dijawab.                       |
| [ < Sebelumnya ]        [ KIRIM ]                        |
| Autosave aktif — "Tersimpan ✓"                           |
+----------------------------------------------------------+
```

### F2b. Step — Mode "Kategori"

```text
+----------------------------------------------------------+
| Langkah 1 / N  [=====>........]                          |
| Kategori: Fasilitas                                      |
|                                                          |
| 1.1 Fasilitas Toilet *       (1)(2)(3)(4)(5)(6)(7)        |  <- tipe rating (radio skala)
| 1.2 Fasilitas Meeting Poin * (1)(2)(3)(4)(5)(6)(7)        |
| 1.3 Fasilitas Wara Wiri *    (1)(2)(3)(4)(5)(6)(7)        |
|                                                          |
| [!] Soal bertanda * wajib dijawab.                       |
| [ < Sebelumnya ]        [ Berikutnya > ]                 |
| Autosave aktif — "Tersimpan ✓"                           |
+----------------------------------------------------------+
```

### F2c. Step — Mode "Soal Only"

```text
+----------------------------------------------------------+
| Langkah 1 / 1  [=====================]                   |
| (Soal Only — tanpa kategori, 1 step flat)                 |
|                                                          |
| 1. Fasilitas Toilet *       (1)(2)(3)(4)(5)(6)(7)         |
| 2. Penjelasan Sales *       (1)(2)(3)(4)(5)(6)(7)         |
| 3. Niat Kembali *      ( )YA ( )BELUM TAU ( )TIDAK        |
|                                                          |
| [!] Soal bertanda * wajib dijawab.                       |
| [ Berikutnya > ]                                         |
| Autosave aktif — "Tersimpan ✓"                           |
+----------------------------------------------------------+
```

### F2d. Step — Deskripsi Grup + Teks Bantuan

```text
+------------------------------------------------------------------+
|  [1 Identitas]  [2 Fasilitas]  [3 Pelayanan]  [4 Review]         |
|  Langkah 2 / 4   [========>.................]                     |
+------------------------------------------------------------------+
|  Mohon nilai fasilitas berdasarkan pengalaman Anda.              |
|  Skala 1 = sangat buruk, 7 = sangat baik.                         |
|                                                                  |
|  1.1  Fasilitas Toilet *                                         |
|       (1) (2) (3) (4) (5) (6) (7)                                |
|       Kebersihan, air, sabun, dan aroma toilet.                  |
|                                                                  |
|  1.2  Fasilitas Meeting Poin *                                   |
|       (1) (2) (3) (4) (5) (6) (7)                                |
|                                                                  |
|  [!] Soal bertanda * wajib dijawab.                              |
|  [ < Sebelumnya ]                             [ Berikutnya > ]   |
+------------------------------------------------------------------+
```

- Deskripsi grup & teks bantuan opsional; di-escape; line break dipertahankan.
- Tipe soal = widget render di dalam step; bukan step tersendiri.
- Next/Prev → autosave. Required gagal → field merah, nilai lain tetap tersimpan.
- Skala `rating` = radio button min..max (bisa 0, >7); `radio` kategorikal (mis. Niat Kembali) = pilihan teks.
- Widget `rating` merender skala min..max soal (contoh `(1)…(7)` hanya ilustrasi).

### F3. Review

```text
| Periksa jawaban  [ Ubah > ]  ...  [ KIRIM ]
```

### F4. Sukses

```text
| [v] Terima kasih! Jawaban Anda telah tersimpan.
```

### F5. Ditutup / Expired / Invalid / Sudah Submit

```text
| Kuesioner sudah ditutup / tautan tidak valid / sudah dikirim.
```

---

## G. MANAGE RESPONS

### G1. List

```text
+----------------------------------------------------------------------+
| Respons                                                              |
| Filter: [Kuesioner v][Periode v][Sekolah..][Status v][Opsi v][Reset]  |
| [ Export Excel ] [ Export Looker ]                                    |
|                                                                      |
| ID  | Nama  | Sekolah | Tgl      | Avg  | Opsi | Status        | Waktu |
| 351 | Ani   | SMPN 1  |12/10/24  | 6.14 | YA   | (v)submitted  | ...   |
| 352 | Budi  | SMAN 2  |12/10/24  | 5.43 | BLM  | [!]draft      | ...   |
+----------------------------------------------------------------------+
```

- **Badge**: submitted=hijau, draft=kuning [!]. **Empty**: "Belum ada respons".
- **Export Excel** = wide XLSX (kolom asli, F7). **Export Looker** = long 15 kolom XLSX (F10).
- Kedua export XLSX mengikuti filter & pencarian yang sudah diterapkan; submitted-only; soft-deleted tidak pernah ikut.

### G2. Detail

```text
| Respons #351 — Ani — SMPN 1 — 12/10/2024 | submitted
| Identitas: Ani / SMPN 1 / 12-10-2024
| Fasilitas 5.67: Toilet 5 | Meeting 6 | Wara 6
| Pelayanan 6.50: Sales 6 | Tour 7
| Produk    5.50: Permainan 5 | Katering 6
| Kepuasan  -   : Opsi = YA
| [ Kembali ] [ Export baris ini ]
```

- **Tanpa Create**; **Edit terbatas**: Nama & Sekolah dapat diubah, field lain disabled (Questionnaire, Status, Draft token, Waktu Submit, Tgl Kunjungan).

---

## H. USER & ROLE

### H1. User List

```text
+---------------------------------------------------------------+
| User                                       [+ Tambah User]     |
|                                                               |
| Arya <arya@..> [Super Admin] [Aktif v] [ubah pw][edit]         |
| Dedi <dedi@..> [Operator]    [Aktif v] [ubah pw][edit]         |
|                                                               |
| [Nonaktifkan][Soft Delete]   Toggle: [Lihat User Terhapus]     |
+---------------------------------------------------------------+
```

- Tombol "ubah pw" hanya tampil bila punya permission `update_password_user`.

### H2. User Form

```text
| Nama*[___] Email*[___] Password*[___] Aktif[x] Role*[ ... v ] [ SIMPAN ]
```

### H3. Update Password (oleh Admin)

```text
+--------------------------------------------------+
| User: Dedi <dedi@..>                             |
| Password baru [__________________]  (o)          |
| Konfirmasi    [__________________]               |
| [ SIMPAN PASSWORD ]  (tanpa verifikasi email)    |
+--------------------------------------------------+
```

### H4. Role List

```text
| Super Admin (semua) [edit] | Admin [edit] | Operator [edit] | Viewer [edit]
```

### H5. Role Form (Permission Matrix)

```text
+-----------------------------------------------------------------+
| Nama Role* [ Operator ]                                         |
|-----------------------------------------------------------------|
| RESOURCE  | View | Create | Update | Delete | Restore | Force    |
| Kategori  | [x]  | [ ]    | [ ]    | [ ]    | [ ]     | [ ]      |
| Soal      | [x]  | [ ]    | [ ]    | [ ]    | [ ]     | [ ]      |
| Kuesioner | [x]  | [ ]    | [ ]    | [ ]    | [ ]     | [ ]      |
| Respons   | [x]  | [ ]    | [x]    | [ ]    | [ ]     | [ ]      |
| User      | [ ]  | [ ]    | [ ]    | [ ]    | [ ]     | [ ]      |
| Role      | [ ]  | [ ]    | [ ]    | [ ]    | [ ]     | [ ]      |
|-----------------------------------------------------------------|
| CUSTOM: [x]publish_questionnaire [x]export_response             |
|         [x]update_password_user  [ ]assign_role                 |
| [ SIMPAN ]                                                      |
+-----------------------------------------------------------------+
```

- Respons: Create dihapus; Update hanya Nama/Sekolah (field lain terkunci).

---

## I. RESPONSIF

| Breakpoint | Perilaku |
|---|---|
| ≥1024 | Sidebar + 2 kolom |
| 768–1023 | Sidebar collapsible + 2 kolom |
| <768 | Sidebar drawer; wizard full-width; tabel → kartu per baris |

> Halaman Profile/Account user dinonaktifkan (tidak ada ganti password mandiri).
