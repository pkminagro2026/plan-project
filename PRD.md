# PRD — Website Quisioner DUMI UNPAK

| Item | Nilai |
|---|---|
| Versi | 1.0 (Final) |
| Tanggal | 2026-10-07 |
| Status | Siap implementasi |
| Stack | Laravel 13 + Filament v5 + MySQL 8 |

---

## 1. Ringkasan Eksekutif
Website Quisioner menggantikan pencatatan kuesioner kunjungan manual di Google Sheets.
Fitur: pengelolaan kategori & soal, link publik + QR, pengisian via wizard tanpa login,
pengelolaan & ekspor hasil, dashboard agregat, serta manajemen user dengan RBAC dinamis.

---

## 2. Latar Belakang & Masalah

| Kode | Masalah |
|---|---|
| M1 | Identitas responden tidak pernah tercatat (NAMA 0/353, SEKOLAH 0/353) |
| M2 | Penomoran NO reset tiap bulan → sulit melacak responden unik |
| M3 | Skala 1–7 tanpa label makna |
| M4 | Kolom rata-rata dihitung manual → rawan salah (desimal koma) |
| M5 | 9 sel rating kosong (0,4%) tanpa penanganan |
| M6 | Tanpa validasi → 11 baris outlier (skor <3) |
| M7 | Tanpa kontrol akses/audit (pemegang link sheet bisa edit) |

---

## 3. Analisa Data Mentah (Baseline)
Sumber: tab **QUESTIONNAIRE** (gid `1695230368`) — 353 baris, Feb–Oct 2024.

| Kriteria | n | Rata² | Min | Max | Missing |
|---|---|---|---|---|---|
| PENGARAHAN TOUR GUIDE | 353 | 6.18 | 2 | 7 | 0 |
| PENJELASAN SALES | 352 | 6.01 | 1 | 7 | 1 |
| FASILITAS MEETING POIN | 351 | 6.00 | 2 | 7 | 2 |
| FASILITAS WARA WIRI | 351 | 5.77 | 2 | 7 | 2 |
| KERAGAMAN PERMAINAN EDUKASI | 353 | 5.65 | 1 | 7 | 0 |
| CITARASA KATERING | 349 | 5.52 | 2 | 7 | 4 |
| FASILITAS TOILET | 353 | 5.37 | 1 | 7 | 0 |

| Metrik | Nilai |
|---|---|
| Distribusi skor | 1=5, 2=11, 3=42, 4=183, 5=672, 6=841, 7=708 |
| Niat kembali | BELUM TAU=182 (51,6%), YA=170 (48,2%), TIDAK=1 (0,3%) |
| Sebaran bulan | Feb 53, Mar 29, Apr 15, Mei 92, Jun 67, Jul 7, Agu 9, Sep 25, Okt 56 |
| Temuan | Toilet & Katering titik lemah; mayoritas "belum tahu" |

---

## 4. Tujuan & Metrik

| Kode | Tujuan |
|---|---|
| G1 | Identitas/validasi 100% untuk periode baru |
| G2 | Buat link < 5 menit + QR otomatis |
| G3 | Completion rate > 80% (pantau draft vs submitted) |
| G4 | Agregat otomatis, nol perhitungan manual |
| G5 | Akses 100% terkontrol RBAC |

---

## 5. Ruang Lingkup

### In-scope
Auth (login only), RBAC dinamis, manajemen user (soft delete, aktif/nonaktif, update
password via admin), CRUD kategori & soal, builder kuesioner (questions/groups/custom), link
publik + QR + tenggat, wizard + autosave, manage respons (draft+submitted) + export Excel,
dashboard, import 353 data historis.

### Out-of-scope
Forgot/reset password, verifikasi email, multi-bahasa, notifikasi email/WA, analitik AI,
SSO, integrasi eksternal, API pihak ketiga.

---

## 6. Persona & Hak Akses

| Persona | Hak |
|---|---|
| Super Admin | Akses penuh, kelola role/permission/user & password |
| Admin | Kelola kategori, soal, kuesioner, respons, export, update password user |
| Operator | Lihat & koreksi identitas respons (Nama/Sekolah), export |
| Viewer | Read-only dashboard & respons |

> Akses tiap aksi (termasuk update password user) ditentukan permission.

---

## 7. Glosarium

| Istilah | Definisi |
|---|---|
| Kategori (Group) | Kelompok soal = satu langkah wizard |
| Soal (Question) | Unit pertanyaan (tipe, skala, required); boleh tanpa kategori |
| Kuesioner | Entitas link publik berisi resep soal + tenggat |
| Snapshot | Pembekuan daftar soal saat publish |
| Draft | Respons terisi sebagian (autosave) |
| Submitted | Respons final |
| Identitas | Metadata responden (Nama, Sekolah, Tgl Kunjungan) di level kuesioner; tiap field punya `enabled`/`required` (default tampil + wajib); disimpan per respons |

---

## 8. Spesifikasi Fitur

### F1 — Autentikasi (Login Only)

| Aspek | Detail |
|---|---|
| Deskripsi | Login email & password |
| Field | email (required, email); password (required) |
| Fitur | Rate limit 5 percobaan/menit |
| Tidak ada | Forgot password, reset password, verifikasi email |
| Aturan | `is_active=false` tidak bisa login ("Akun dinonaktifkan, hubungi admin"); password diatur/diubah admin (tanpa email) |
| Edge case | Akun soft-deleted tidak bisa login |
| Acceptance | Login sukses → dashboard; 5× gagal → throttle; user nonaktif ditolak |

### F2 — RBAC Dinamis

| Aspek | Detail |
|---|---|
| Deskripsi | Role & permission tersimpan di DB (`filament-shield` + `spatie/laravel-permission`); admin membuat role, memilih permission, menempel ke user A/B/C |
| Entitas | roles, permissions, role_has_permissions, model_has_roles |
| Permission | Granular per resource/aksi (Shield) + custom: `publish_questionnaire`, `export_response`, `update_password_user`, `assign_role` |
| Aturan | Super Admin bypass (`Gate::before`) |
| Acceptance | User tanpa permission tidak melihat menu & 403 saat akses langsung |

### F3 — Kategori Soal

| Aspek | Detail |
|---|---|
| Field | name (required, uniq), slug (auto), description (nullable), sort, is_active |
| Aksi | Create / edit / delete (soft) / toggle |
| Render publik | `description` tampil sebagai teks kecil abu-abu di atas soal pada step kategori; line break dipertahankan; null/kosong tidak dirender |
| Aturan | Kategori nonaktif tidak muncul di builder/publik |
| Edge case | Kategori dihapus → soal jadi tanpa kategori (`question_group_id=null` → "Umum") |
| Acceptance | Urutan kategori = urutan langkah wizard |

### F4 — Soal

| Aspek | Detail |
|---|---|
| Field | question_group_id (nullable FK), label (required), help_text (nullable), type (enum), scale_min/scale_max (default 1/7, dinamis 0..N bisa >7), options (json), is_required, sort, is_active |
| Tipe | rating, text, textarea, select, radio, date, number |
| Validasi | rating integer min..max; text max 255; textarea max 2000; select/radio in(options); date Y-m-d; number numeric; rating scale_min < scale_max |
| Render | Tipe menentukan widget di dalam step kategori (bukan step tersendiri): `rating` = radio skala min–max soal (numerik); `radio` = pilihan kategorikal berlabel teks; tipe lain reserved (future) |
| Render publik | `help_text` tampil sebagai helper text di bawah field; line break dipertahankan; null/kosong tidak dirender |
| MVP | Tipe diimplementasikan: `rating`, `radio`; lainnya arsitektur saja. Form hanya menampilkan pilihan `rating` & `radio`. |
| Edge case | Ubah tipe soal yang dipakai kuesioner published dilarang untuk snapshot |
| Acceptance | Soal rating menampilkan 1..7 + label makna |

### F5 — Builder Kuesioner & Link Publik

| Aspek | Detail |
|---|---|
| Field | title (required), slug, public_token (uuid auto), description, selection_mode (questions/groups/custom), layout (fixed wizard; tidak diekspos di form), status (draft/published/closed), opens_at, closes_at, is_frozen, frozen_at, created_by, identity_fields |
| Identitas | Field identitas (Nama, Sekolah, Tgl Kunjungan) level kuesioner; `enabled`/`required` per field (default tampil + wajib); perubahan berlaku langsung untuk responden baru; disimpan di `responses` |
| Mode | `questions`=soal individual saja; `groups`=soal kategori terpilih; `custom`=union(kategori, soal individual), dedup, urut group.sort lalu question.sort |
| Aksi | Simpan draft (resep); publish → resolusi + materialize + `is_frozen=1` + `status=published`; generate QR (SVG) + tampilkan link publik; tutup manual |
| Aturan | Publik bila `status=published` DAN `now` in [opens_at, closes_at] |
| Edge case | Master soal diubah setelah publish → kuesioner tidak berubah (frozen); publish tanpa soal ditolak; `opens_at >= closes_at` ditolak |
| Acceptance | QR scan membuka `/q/{token}`; di luar tenggat → "Ditutup" |

### F6 — Form Publik (Wizard) + Autosave

| Aspek | Detail |
|---|---|
| Alur | Buka token → validasi → set draft_token (tanpa membuat respons) → Intro + Identitas (opsional; step dilewati bila semua field hidden) → langkah sesuai mode (lihat Render mode) → review → submit; draft dibuat saat interaksi pertama |
| Render mode | `questions`=1 step flat (tanpa kategori); `groups`=1 step per kategori; `custom`=1 step per kategori + step "Umum" untuk soal individual |
| Komponen | Form publik memakai komponen `Wizard` Filament di halaman Livewire publik |
| Teks pendukung | Deskripsi grup tampil di atas soal pada step kategori; `help_text` soal tampil di bawah field; keduanya optional, di-escape, line break dipertahankan |
| Autosave | Saat field berubah (blur/pilih opsi) & Next → upsert answers; draft dibuat saat interaksi pertama (bukan saat membuka halaman) |
| Validasi | Next memvalidasi required langkah; gagal → tetap simpan nilai terisi, tidak maju |
| Submit | Validasi semua required server-side → `status=submitted` + `submitted_at` |
| Resume | draft_token dapat lanjut lintas device; anti-duplikat per token |
| Anti-spam | Rate limit per IP + honeypot (opsional captcha) |
| Draft | Tanpa batas waktu |
| Edge case | Token invalid/expired/closed → halaman error; sudah submit → "Terima kasih" |
| Acceptance | Tutup browser di tengah → buka lagi → data terisi kembali (resume) |

### F7 — Manage Respons

| Aspek | Detail |
|---|---|
| Tabel | Default draft + submitted dengan badge status |
| Kolom | ID, Nama, Sekolah, Tgl Kunjungan, Rata-rata (Skor %), Opsi terpilih, Status, Waktu Submit |
| Filter | Kuesioner, periode (`visit_date`), sekolah, status, opsi (termasuk Niat Kembali) |
| Aksi | View (per kategori), Edit terbatas (Nama & Sekolah; field integritas — Questionnaire, Status, Draft token, Waktu Submit, Tgl Kunjungan — terkunci), export Excel (XLSX, wide format = kolom asli) via `WideResponseExport`, soft delete, restore, force delete (super admin). Create respons dihapus; respons hanya dari wizard publik. |
| Aturan | Export submitted-only; soft-deleted selalu dikecualikan; filter Trashed tidak memengaruhi export; filter/pencarian yang sudah diterapkan ikut; kolom soal mengikuti filter Kuesioner; urutan `id DESC`; XLSX dengan header styled, freeze, autofilter, lebar kolom, format tanggal/angka; rata-rata hanya dari `rating`, kosong dilewati |
| Acceptance | XLSX identik struktur kolom sheet asli |

### F8 — Dashboard

| Aspek | Detail |
|---|---|
| Widget | Jumlah Pertanyaan; Jumlah Kategori; Jumlah Responded (submitted; draft dipisah); Rata-rata Skor % (dari `rating`); Distribusi opsi per soal (`radio`/`select`); Chart Respons/Bulan; Soal Terendah berdasarkan Skor % (dari `rating`); Trend completion |
| Filter | Periode & kuesioner |
| Acceptance | Angka konsisten dengan tabel respons |

### F9 — Manage User

| Aspek | Detail |
|---|---|
| Field | name, email (uniq), password, is_active, roles |
| Aksi | Create / edit / soft delete / restore / force delete / aktif-nonaktif (gate login) / assign role / update password |
| Update Password | Aksi Filament di Manage User, digerbangi permission `update_password_user`; admin isi password baru (hash) tanpa verifikasi email; tidak ada halaman ganti password mandiri; Profile/Account page dinonaktifkan |
| Aturan | Tidak bisa nonaktif/hapus diri sendiri; minimal 1 super admin aktif |
| Acceptance | User nonaktif gagal login; password baru (di-set admin) langsung berlaku |

### F10 — Export Long-Format (Append Sesuai Template)

| Aspek | Detail |
|---|---|
| Deskripsi | Hasilkan XLSX long sesuai **Mirror Long Template Questionaire** (di bawah) |
| Cakupan | Semua respons submitted yang lolos filter/pencarian yang sudah diterapkan; semua kuesioner bila tanpa filter |
| Format | 1 baris = 1 jawaban (rating→score; radio→option) |
| Aksi | Generate XLSX 15 kolom `looker_answers_YYYYMMDD_HHmm.xlsx` |
| Aturan | Update ke Google Sheets via **append** (bukan replace); header apa adanya; duplikat dibiarkan; soft-deleted dikecualikan; filter Trashed diabaikan; urutan `id DESC`; filter yang belum diterapkan tidak ikut; append dengan filter aktif hanya menambah subset |
| Anti-spam | Cegah submit ganda (draft_token/ip+visit_date; cek ip+visit_date hanya bila Tgl Kunjungan aktif & terisi) |
| Acceptance | XLSX cocok dengan Mirror Long Template Questionaire |

#### Mirror Long Template Questionaire (15 kolom)

| # | Kolom | Tipe |
|---|---|---|
| 1 | response_id | Text (kunci) |
| 2 | submitted_at | DateTime |
| 3 | visit_date | Date `yyyy-mm-dd` |
| 4 | respondent_name | Text |
| 5 | respondent_school | Text |
| 6 | questionnaire | Text |
| 7 | questionnaire_slug | Text |
| 8 | public_token | Text |
| 9 | category | Text |
| 10 | question | Text |
| 11 | question_type | Text (rating/radio) |
| 12 | score | Number |
| 13 | option | Text |
| 14 | scale_min | Number |
| 15 | scale_max | Number |

> Kolom 3–5 (`visit_date`, `respondent_name`, `respondent_school`) dapat kosong bila field identitas terkait di-hide. Struktur 15 kolom & urutan tidak berubah.

---

## 9. Aturan Bisnis Global

| Kode | Aturan |
|---|---|
| BR1 | Skala rating dinamis min–max (mis. 0–10, 1–5, 1–7); label makna per skala |
| BR2 | Identitas kuesioner (Nama, Sekolah, Tgl Kunjungan) dapat ditampilkan/disembunyikan & wajib/tidak wajib per kuesioner (default tampil + wajib); kolom DB nullable (untuk import historis) |
| BR3 | Soal boleh tanpa kategori → section "Umum" (selalu di akhir) |
| BR4 | Snapshot saat publish |
| BR5 | Draft tidak kedaluwarsa (tanpa auto-prune) |
| BR6 | Tanggal & angka tersimpan tipe asli; tampilan locale id (koma desimal) |
| BR7 | Rata-rata dihitung server-side |
| BR8 | Tidak ada alur email (forgot/reset/verify); password hanya lewat admin |

---

## 10. State Machine

| Entitas | Transisi |
|---|---|
| Kuesioner | draft → published → closed (published → draft hanya bila belum ada respons) |
| Respons | draft → submitted (soft-delete sebagai flag terpisah) |

---

## 11. Non-Functional

| Kode | Requirement |
|---|---|
| NFR1 | Mobile-first (360/768/1024) |
| NFR2 | Halaman publik < 2s; tabel ter-paginate |
| NFR3 | CSRF, validasi server-side, rate limit, akses publik via token |
| NFR4 | Autosave idempotent (unique response_id+question_id) |
| NFR5 | Audit: timestamps + soft delete |
| NFR6 | Aksesibilitas: label form, kontras, navigasi keyboard |

---

## 12. Asumsi & Dependensi

| Kode | Asumsi |
|---|---|
| A1 | QR digenerate lokal (tidak butuh internet) |
| A2 | Tidak ada dependensi email |
| A3 | Import historis pakai placeholder nama ("Responden Historis #n") |

---

## 13. Milestone

| Kode | Milestone |
|---|---|
| M0 | Setup |
| M1 | Auth + RBAC + User |
| M2 | Kategori & Soal |
| M3 | Kuesioner + QR |
| M4 | Form publik |
| M5 | Manage respons + export |
| M6 | Dashboard |
| M7 | Seeder + import 353 |
| M8 | QA |

---

## 14. Risiko & Mitigasi

| Kode | Risiko | Mitigasi |
|---|---|---|
| R1 | Perubahan master soal merusak data lama | Snapshot |
| R2 | Spam link publik | Rate limit + honeypot/captcha |
| R3 | Data historis kotor | Validasi import + log baris gagal |
| R4 | Drop-off tinggi | Analisa per langkah + resume draft |
