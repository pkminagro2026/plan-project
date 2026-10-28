# PROJECT INFO — Website Quisioner

---

## 1. Stack & Versi

| Komponen | Versi/Keterangan |
|---|---|
| PHP | 8.3 |
| Laravel | 13.x |
| Filament | 5.x (Livewire 3, Tailwind, Alpine) |
| Database | MySQL 8.0 (utf8mb4_unicode_ci) |
| Node | 22 + Vite |
| Redis | Opsional (queue/cache) |

---

## 2. Paket & Alasan

| Paket | Fungsi |
|---|---|
| filament/filament ^5 | Panel admin, resource, widget |
| bezhansalleh/filament-shield | UI permission + role (RBAC) |
| spatie/laravel-permission | Engine role/permission di DB |

> Catatan: export CSV/Looker memakai custom service (`WideResponseExport`/`LookerLongExport`); QR code memakai bawaan Filament (`chillerlan/php-qrcode`); slug via `Str::slug`. Tidak ada paket mail/verification (fitur email tidak dipakai).

### 2.1 Pemetaan Filament (Default vs Paket vs Custom)

| Bagian | Implementasi |
|---|---|
| Login | Panel auth Filament (default) |
| CRUD Kategori/Soal/Kuesioner/User + kelola Respons (view, edit terbatas, delete) | Resource + form/table Filament (default) |
| Satu entity Soal semua tipe | `Select` tipe + field kondisional (default) |
| Dashboard | Widget StatsOverview/Chart/Table (default) |
| Export CSV/Looker | `WideResponseExport` / `LookerLongExport` (custom) |
| Soft delete/restore/force | SoftDeletes + trashed filter (default) |
| Aktif/nonaktif user | Toggle + gate (default) |
| Wizard form publik | Komponen Filament `Wizard` (default) + halaman guest (custom) |
| Change password | Filament `Action` (custom kecil) |
| RBAC | filament-shield + spatie (paket) |
| QR | Bawaan Filament (chillerlan/php-qrcode) |
| Autosave/resume, snapshot, resolver, rata-rata, import | Custom logic |

---

## 3. Perintah Setup

```bash
composer create-project laravel/laravel inagro-kuesioner
cd inagro-kuesioner
composer require filament/filament:"^5"
php artisan filament:install --panels
composer require bezhansalleh/filament-shield
php artisan shield:install admin
php artisan migrate
php artisan shield:generate --all --panel=admin
php artisan db:seed
php artisan filament:assets
npm install && npm run build
```

---

## 4. Environment (.env)

```dotenv
APP_NAME="Website Quisioner"
APP_ENV=local
APP_URL=http://localhost:8000
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=inagro_kuesioner
DB_USERNAME=root
DB_PASSWORD=root
QUEUE_CONNECTION=database
# MAIL TIDAK DIPERLUKAN (tanpa forgot/reset/verifikasi email)
```

---

## 5. Struktur Direktori

```text
app/
  Models/ User, QuestionGroup, Question, Questionnaire, Response, Answer
  Enums/ QuestionType, SelectionMode, QuestionnaireStatus, ResponseStatus
  Filament/Resources/ QuestionGroupResource, QuestionResource,
    QuestionnaireResource, ResponseResource, UserResource, RoleResource
  Filament/Widgets/ StatsOverview, ResponsesPerMonthChart,
    OptionDistributionChart, LowestCriteriaTable
  Services/ QuestionnaireQuestionResolver, LookerLongExport, WideResponseExport
  Support/ ScoreNormalizer
  Actions/ PublishQuestionnaire
  Livewire/ PublicQuestionnaireWizard (komponen Filament Wizard)
  Imports/ HistoricalQuestionnaireImport
  Policies/ (auto via Shield)
database/ migrations, seeders, data/historis_2024.csv
routes/web.php -> /q/{token} publik
tests/ Feature, Unit
```

> Form publik memakai komponen `Filament\Schemas\Components\Wizard`. Export CSV/Looker memakai `WideResponseExport`/`LookerLongExport` dan mengikuti filter/pencarian yang sudah diterapkan.

---

## 6. Skema Database (Seluruh Tabel)

### 6.1 `users`

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| id | bigint unsigned | No | auto | PK |
| name | varchar(255) | No | - | |
| email | varchar(255) | No | - | UNIQUE |
| password | varchar(255) | No | - | Hash |
| is_active | boolean | No | true | Gate login |
| last_login_at | timestamp | Yes | null | |
| remember_token | varchar(100) | Yes | null | |
| created_at | timestamp | Yes | null | |
| updated_at | timestamp | Yes | null | |
| deleted_at | timestamp | Yes | null | SoftDeletes |

> `email_verified_at` sengaja tidak ada (tanpa verifikasi email).

### 6.2 `question_groups`

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| id | bigint unsigned | No | auto | PK |
| name | varchar(255) | No | - | |
| slug | varchar(255) | No | - | UNIQUE |
| description | text | Yes | null | |
| sort | int | No | 0 | Index |
| is_active | boolean | No | true | |
| created_at | timestamp | Yes | null | |
| updated_at | timestamp | Yes | null | |
| deleted_at | timestamp | Yes | null | SoftDeletes |

### 6.3 `questions`

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| id | bigint unsigned | No | auto | PK |
| question_group_id | bigint unsigned | Yes | null | FK → question_groups.id (nullOnDelete) |
| label | varchar(255) | No | - | |
| help_text | text | Yes | null | |
| type | enum(rating,text,textarea,select,radio,date,number) | No | - | |
| scale_min | tinyint | Yes | 1 | Untuk rating (dinamis 0..N) |
| scale_max | tinyint | Yes | 7 | Untuk rating (dinamis bisa >7) |
| options | json | Yes | null | Untuk select/radio |
| is_required | boolean | No | false | |
| sort | int | No | 0 | Index |
| is_active | boolean | No | true | |
| created_at | timestamp | Yes | null | |
| updated_at | timestamp | Yes | null | |
| deleted_at | timestamp | Yes | null | SoftDeletes |

> MVP implementasi: `rating` (radio skala 1–x) dan `radio` (kategorikal). `text`, `textarea`, `select`, `date`, `number` = arsitektur (tanpa implementasi).

### 6.4 `questionnaires`

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| id | bigint unsigned | No | auto | PK |
| title | varchar(255) | No | - | |
| slug | varchar(255) | No | - | UNIQUE |
| public_token | uuid | No | - | UNIQUE |
| description | text | Yes | null | |
| selection_mode | enum(questions,groups,custom) | No | custom | |
| layout | enum(wizard,tabs,single) | No | wizard | |
| status | enum(draft,published,closed) | No | draft | Index |
| is_frozen | boolean | No | false | Snapshot flag |
| frozen_at | timestamp | Yes | null | |
| opens_at | timestamp | Yes | null | Index |
| closes_at | timestamp | Yes | null | Index |
| created_by | bigint unsigned | Yes | null | FK → users.id (nullOnDelete) |
| identity_fields | json | Yes | null | Identitas level kuesioner — `enabled` & `required` per field (default tampil + wajib via model) |
| created_at | timestamp | Yes | null | |
| updated_at | timestamp | Yes | null | |
| deleted_at | timestamp | Yes | null | SoftDeletes |

> MVP implementasi: `layout` = `wizard`; `tabs`, `single` = arsitektur (tanpa implementasi). Field `layout` tidak diekspos di form (fixed `wizard`).

### 6.5 `questionnaire_question_group`

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| id | bigint unsigned | No | auto | PK |
| questionnaire_id | bigint unsigned | No | - | FK → questionnaires.id (cascade) |
| question_group_id | bigint unsigned | No | - | FK → question_groups.id (cascade) |
| sort | int | No | 0 | |
| created_at | timestamp | Yes | null | |
| updated_at | timestamp | Yes | null | |

> UNIQUE (`questionnaire_id`, `question_group_id`) — resep kategori.

### 6.6 `questionnaire_question`

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| id | bigint unsigned | No | auto | PK |
| questionnaire_id | bigint unsigned | No | - | FK → questionnaires.id (cascade) |
| question_id | bigint unsigned | No | - | FK → questions.id (cascade) |
| sort | int | No | 0 | |
| created_at | timestamp | Yes | null | |
| updated_at | timestamp | Yes | null | |

> UNIQUE (`questionnaire_id`, `question_id`) — soal individual + snapshot render.

### 6.7 `responses`

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| id | bigint unsigned | No | auto | PK |
| questionnaire_id | bigint unsigned | No | - | FK → questionnaires.id (cascade) |
| respondent_name | varchar(255) | Yes | null | Wajib di form publik |
| respondent_school | varchar(255) | Yes | null | Wajib di form publik |
| visit_date | date | Yes | null | |
| status | enum(draft,submitted) | No | draft | Index |
| draft_token | uuid | No | - | UNIQUE |
| last_step | int | No | 0 | |
| submitted_at | timestamp | Yes | null | |
| ip | varchar(45) | Yes | null | |
| user_agent | text | Yes | null | |
| meta | json | Yes | null | no asli, dll |
| created_at | timestamp | Yes | null | |
| updated_at | timestamp | Yes | null | |
| deleted_at | timestamp | Yes | null | SoftDeletes |

### 6.8 `answers`

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| id | bigint unsigned | No | auto | PK |
| response_id | bigint unsigned | No | - | FK → responses.id (cascade) |
| question_id | bigint unsigned | No | - | FK → questions.id (restrict) |
| value_number | decimal(6,2) | Yes | null | Rating/number |
| value_text | text | Yes | null | Text/textarea |
| value_option | varchar(255) | Yes | null | Select/radio |
| created_at | timestamp | Yes | null | |
| updated_at | timestamp | Yes | null | |

> UNIQUE (`response_id`, `question_id`) — idempotent autosave upsert.

> Agregasi: hanya `rating` dihitung; `radio` = distribusi; tipe lain & nilai kosong dilewati. Rating dihitung ternormalisasi (Skor %) untuk perbandingan lintas soal.

### 6.9 `roles` (Spatie)

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| id | bigint unsigned | No | auto | PK |
| name | varchar(255) | No | - | |
| guard_name | varchar(255) | No | - | |
| created_at | timestamp | Yes | null | |
| updated_at | timestamp | Yes | null | |

> UNIQUE (`name`, `guard_name`).

### 6.10 `permissions` (Spatie)

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| id | bigint unsigned | No | auto | PK |
| name | varchar(255) | No | - | |
| guard_name | varchar(255) | No | - | |
| created_at | timestamp | Yes | null | |
| updated_at | timestamp | Yes | null | |

> UNIQUE (`name`, `guard_name`).

### 6.11 `role_has_permissions` (Spatie)

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| permission_id | bigint unsigned | No | - | FK → permissions.id (cascade) |
| role_id | bigint unsigned | No | - | FK → roles.id (cascade) |

> PRIMARY KEY (`permission_id`, `role_id`).

### 6.12 `model_has_roles` (Spatie)

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| role_id | bigint unsigned | No | - | FK → roles.id (cascade) |
| model_type | varchar(255) | No | - | Polymorphic |
| model_id | bigint unsigned | No | - | Polymorphic |

> PRIMARY KEY (`role_id`, `model_id`, `model_type`); INDEX (`model_type`, `model_id`).

### 6.13 `model_has_permissions` (Spatie)

| Kolom | Tipe | Null | Default | Keterangan |
|---|---|---|---|---|
| permission_id | bigint unsigned | No | - | FK → permissions.id (cascade) |
| model_type | varchar(255) | No | - | Polymorphic |
| model_id | bigint unsigned | No | - | Polymorphic |

> PRIMARY KEY (`permission_id`, `model_id`, `model_type`); INDEX (`model_type`, `model_id`).

### 6.14 Tabel Framework Laravel (bawaan)

| Tabel | Keterangan |
|---|---|
| migrations | Tracking migrasi |
| cache | Cache store |
| cache_locks | Lock cache |
| sessions | Session driver database |
| jobs | Queue jobs |
| job_batches | Job batching |
| failed_jobs | Job gagal |

> `password_reset_tokens` dibuat bawaan tetapi tidak dipakai (fitur reset password dihapus).

---

## 7. ERD

```text
users 1───* questionnaires (created_by)
question_groups 1───* questions (nullable)
questionnaires *───* question_groups   [questionnaire_question_group]
questionnaires *───* questions         [questionnaire_question]
questionnaires 1───* responses
responses 1───* answers
questions 1───* answers
users *───* roles                      [model_has_roles]
```

---

## 8. Resolusi Soal — `QuestionnaireQuestionResolver`

| Mode | Hasil |
|---|---|
| `questions` | Soal individual terpilih saja (tanpa kategori) |
| `groups` | Soal dari kategori pivot terpilih |
| `custom` | union(kategori pivot, soal pivot individual), dedup, urut |

> Render wizard per mode: `questions` = 1 step flat; `groups` = 1 step per kategori; `custom` = 1 step per kategori + step "Umum" (soal individual).

> Soal tanpa kategori → kelompok virtual "Umum" saat render (selalu di akhir).

> Step wizard = kategori; tipe soal hanya menentukan widget render di dalamnya, bukan jumlah/urutan step.

> Teks pendukung: `question_groups.description` dirender di atas soal pada step; `questions.help_text` dirender sebagai helper text di bawah field. Keduanya optional, di-escape, line break dipertahankan.

---

## 9. Snapshot saat Publish — `PublishQuestionnaire`

| Step | Aksi |
|---|---|
| 1 | Validasi minimal 1 soal + `opens_at < closes_at` |
| 2 | Resolve daftar soal |
| 3 | Hapus + insert ulang `questionnaire_question` (materialize) |
| 4 | `is_frozen=1`, `frozen_at=now()`, `status=published` |
| 5 | Generate `public_token` bila belum ada |
| 6 | (opsional) generate artefak QR |

---

## 10. Alur Autosave Publik

| Step | Aksi |
|---|---|
| 1 | `GET /q/{token}` → validasi (published & dalam rentang) → tampilkan Intro + Identitas (`identity_fields`) |
| 2 | Find `responses` by `draft_token`; **buat saat interaksi pertama** (blur/pilih opsi/Next), `status=draft` |
| 3 | Autosave saat field berubah (blur/pilih opsi) & Next → upsert `answers`. Next: validasi required; gagal → simpan, tidak maju. Prev: client-side (tanpa request) |
| 4 | Submit → validasi semua required → `status=submitted`, `submitted_at=now()` |

> Idempotent via UNIQUE (`response_id`, `question_id`).

---

## 11. Routing

| Route | Handler |
|---|---|
| `GET /` | Dashboard panel admin (Filament) |
| `GET /login` | Login admin |
| `GET /questionnaires`, `/responses`, dst | Resource panel admin (root) |
| `GET /q/{token}` | PublicQuestionnaireWizard (intro + identitas + step + review) |
| LIVEWIRE | autosave / next / prev / submit |

- **Panel admin di root** (`/`) — bukan `/admin`.
- **Guard publik**: token valid, status published, `now` in [opens_at, closes_at], throttle, honeypot.
- Tidak ada route profile/forgot/reset password.

---

## 12. Matriks Permission

| Resource | view | create | update | delete | restore | force | custom |
|---|---|---|---|---|---|---|---|
| Kategori | SA,A,O,V | SA,A | SA,A | SA,A | SA | SA | |
| Soal | SA,A,O,V | SA,A | SA,A | SA,A | SA | SA | |
| Kuesioner | SA,A,O,V | SA,A | SA,A | SA,A | SA | SA | publish (SA,A) |
| Respons | SA,A,O,V | — | SA,A,O | SA,A | SA,A | SA | export (SA,A,O) |
| User | SA,A | SA,A | SA,A | SA,A | SA | SA | update_password_user (SA,A) |
| Role | SA | SA | SA | SA | SA | SA | assign_role (SA) |

> SA=Super Admin (bypass `Gate::before`), A=Admin, O=Operator, V=Viewer.
> Respons: Create dihapus; Update terbatas Nama/Sekolah (field lain terkunci).

---

## 13. Seeder Master

| Kategori | Soal | Tipe |
|---|---|---|
| Fasilitas | Fasilitas Toilet, Fasilitas Meeting Poin, Fasilitas Wara Wiri | rating 1–7 |
| Pelayanan | Penjelasan Sales, Pengarahan Tour Guide | rating 1–7 |
| Produk | Keragaman Permainan Edukasi, Citarasa Katering | rating 1–7 |
| Kepuasan | Niat Kembali | radio (YA/BELUM TAU/TIDAK) |
| Umum | (kosong) | untuk soal lepas |

> Seeder hanya sampel; skala & opsi **dinamis** (skala min–max bisa 0..N, opsi bebas).

---

## 14. Import Historis (353 baris, Feb–Oct 2024)

| Kolom Sheet | Target |
|---|---|
| NO | `responses.meta.original_no` |
| NAMA | `respondent_name` (placeholder "Responden Historis #n") |
| SEKOLAH | `respondent_school` |
| TGL KUNJUNGAN | `visit_date` (dd/mm/yyyy → Y-m-d) |
| PENJELASAN SALES | answer "Penjelasan Sales" |
| FASILITAS TOILET | answer "Fasilitas Toilet" |
| FASILITAS MEETING POIN | answer "Fasilitas Meeting Poin" |
| FASILITAS WARA WIRI | answer "Fasilitas Wara Wiri" |
| PENGARAHAN TOUR GUIDE | answer "Pengarahan Tour Guide" |
| KERAGAMAN PERMAINAN EDUKASI | answer "Keragaman Permainan Edukasi" |
| CITARASA KATERING | answer "Citarasa Katering" |
| KEINGINAN KEMBALI BERKUNJUNG | answer "Niat Kembali" (`value_option`) |
| RATA RATA BAGIAN GA | DIABAIKAN (dihitung sistem) |
| RATA RATA ALL | DIABAIKAN (dihitung sistem) |

- Konteks: 1 questionnaire "Impor Historis 2024" (`status=closed`, `is_frozen=1`).
- Penanganan: koma → float; sel kosong → null; skor <3 disimpan + flag review.

---

## 15. Testing

| Jenis | Cakupan |
|---|---|
| Unit | Resolver (questions/groups/custom), rata-rata, mapping import |
| Feature | RBAC (403 tanpa permission), publik (tenggat/status), autosave & resume, required memblokir Next, snapshot tidak berubah, export CSV/Looker mengikuti filter, update password gated |
| Manual | Responsif mobile, QR scan, resume lintas device |

---

## 16. Deployment

```bash
php artisan migrate --force
php artisan shield:generate --all
php artisan db:seed
php artisan filament:assets
php artisan optimize
# Tanpa scheduler prune draft (draft tanpa batas waktu).
```

---

## 17. Integrasi Looker — Append Sesuai Template (manual)

Update data ke Google Sheets via **upload append** mengikuti **template utama** (header apa adanya); duplikat dibiarkan.

Alur: siapkan file header sama → tab domain tujuan → `File ▸ Import ▸ Upload ▸ Append rows to current sheet` → Looker Refresh. Boleh parsial (isi tab yang diperlukan). Guard submit ganda tetap di app.

### Mirror Long Template Questionaire (15 kolom)

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

> Kolom 3–5 (`visit_date`, `respondent_name`, `respondent_school`) dapat kosong bila field terkait di-hide. Header/urutan 15 kolom tetap; append tidak berubah.
