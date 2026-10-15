# Rumus — Skema Database Looker, Upload Append & Calculated Field

## A. Calculated Field (Looker Studio)

```
--- Demografi ---
Headcount        = COUNT(NIK)
Rata Usia        = AVG(Usia)
Gender           = CASE WHEN JENIS KELAMIN="L" THEN "Laki-laki" ELSE "Perempuan" END

--- Attrition ---
Turnover Rate    = SUM(Jumlah Karyawan Resign) / AVG(Jumlah karyawan Existing)
Voluntary Rate   = SUM(Jumlah Voluntary Resign) / AVG(Jumlah karyawan Existing)
Target Turnover  = 0.10
Bulan (date)     = TODATE(Bulan, "MONTH")

--- Training ---
Total Jam        = SUM(TOTAL JAM TRAINING (JAM))
Peserta Unik     = COUNT_DISTINCT(NIK)
Rata N-Gain      = AVG(N GAIN SCORE)
Bulan Training   = TODATE(TANGGAL, "MONTH")

--- Review ---
Avg Rating       = AVG(RATING)
Total Ulasan     = COUNT(RATING)
Bintang          = CASE WHEN RATING=5 THEN "5★" WHEN RATING=4 THEN "4★"
                        WHEN RATING=3 THEN "3★" WHEN RATING=2 THEN "2★" ELSE "1★" END

--- BTK / Sales ---
Income           = SUM(INCOME)
Target           = SUM(TARGET)
Achievement %    = SUM(INCOME) / SUM(TARGET)
Selisih Target   = SUM(INCOME) - SUM(TARGET)
Pendapatan/Hari  = SUM(INCOME) / 20
Okupansi %       = AVG(PERSENTASE OKUPANSI)
Pemasukan/Karyawan = SUM(INCOME) / AVG(DATA ATTRITION.Jumlah karyawan Existing)  // blend by Bulan

--- Questionaire (long) ---
Responden        = COUNT_DISTINCT(response_id)
Rata Skor        = AVG(score)
Skor %           = AVG((score - scale_min)/(scale_max - scale_min))
Jumlah Jawaban   = COUNT(response_id)
Skala            = CONCAT(CAST(scale_min AS TEXT), "–", CAST(scale_max AS TEXT))
Bln              = TODATE(visit_date, "MONTH")
```

---

## B. Database Looker (Template Utama)

Database = **spreadsheet ini sendiri** (1 konektor Google Sheets). **6 tab, urutan tetap (jangan digeser)**:
1. GOOGLE RIVIEW
2. QUESTIONNAIRE — **long template**
3. TRAINING RECORD
4. BTK TO SALES
5. DATA ATTRITION
6. DATA TENAGA KERJA

Aturan: **header apa adanya** (tanpa rename); **perbaikan format = hanya nilai/tipe**, bukan kolom; `MARGIN` & kolom kosong **diabaikan di Looker, tidak dihapus**.

**Tab 1 · GOOGLE RIVIEW**
```
NO FMT(number; unik nanti) · NAMA FMT(trim) · TANGGAL FMT(date) · RATING number · kolom 5–26 ABAIKAN
```

**Tab 2 · QUESTIONNAIRE — LONG TEMPLATE (15 kolom)**
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

> `response_id` unik per responden (`r1`…`r353`); `respondent_name` = `Responden Historis #n`; `respondent_school` = `-`.

**Tab 3 · TRAINING RECORD**
```
NIK key · NAMA PESERTA TRAINING · JABATAN · STATUS · DIVISI · " DEPARTEMEN " (apa adanya) · NAMA TRAINING · TANGGAL AWAL PELAKSANAAN FMT(date) · NAMA PEMATERI TRAINING · TOTAL JAM TRAINING (JAM) FMT(number) · PRE TEST FMT(number) · POST TEST FMT(number) · POST - PRE FMT(number) · SCORE IDEAL FMT(number) · N GAIN SCORE FMT(number) · kolom 16–29 ABAIKAN
```

**Tab 4 · BTK TO SALES**
```
BULAN FMT(date yyyy-mm) · INCOME FMT(number) · TAX 10% FMT(number) · TOTAL number · MARGIN ABAIKAN · TARGET FMT(number) · ANAK FMT(number) · GURU FMT(number) · DEWASA FMT(number) · TOTAL CUSTOMER FMT(number) · OKUPANSI FMT(number) · PERSENTASE OKUPANSI FMT(number) · % ANAK FMT(number) · % GURU FMT(number) · % DEWASA FMT(number) · (label wahana) ABAIKAN · (OKUPANSI MINA PADI … OKUPANSI GLAMPING) FMT(number)
```

**Tab 5 · DATA ATTRITION**
```
Bulan FMT(date yyyy-mm) · Jumlah karyawan Existing · Jumlah Karyawan Resign · Jumlah Voluntary Resign · Jumlah Involuntary Resign · Jumlah karyawan baru · Jumlah Karyawan Baru Resign (semua number)
```

**Tab 6 · DATA TENAGA KERJA**
```
NIK key · NO. KTP · NO. KK · NAMA · INISIAL · TGL JOIN FMT(date) · GRUP ABSEN · COMPANY · DEPARTEMEN · SECTION · DIVISI · POSISI · JABATAN · DESA · TPT LAHIR · TGL LAHIR FMT(date) · ALAMAT · NO HP teks · EMERGENCY CONTACT · EMAIL · AGAMA · JENIS KELAMIN · MARITAL STATUS · MOTHER NAME · BLOOD TYPE · STATUS KARYAWAN · KONTRAK KE FMT(number) · PKWT PERTAMA/AWAL/AKHIR FMT(date) · PENDIDIKAN (KHS) · PENDIDIKAN · SISA KONTRAK · MASA KERJA FMT(number,127) · USIA FMT(number,127) · (kosong) ABAIKAN · (kosong) ABAIKAN
```

---

## C. Upload Append (perbaruan tab)

**Aturan:** file upload ber-**header identik** dengan tab; **append** baris di bawah; jangan tambah/kurangi/rename/geser kolom; duplikat dibiarkan; boleh parsial (isi tab yang diperlukan saja).

**Langkah:**
1. Siapkan file (CSV/XLSX) header sama.
2. Buka spreadsheet → tab tujuan.
3. `File ▸ Import ▸ Upload` → **Append rows to current sheet** (atau copas di baris kosong berikutnya).
4. Cek kolom & posisi tab tidak bergeser.
5. Looker → **Refresh**.

> Tab kosong → chart "No data" (bukan error); hindari chart blend bila salah satu tab belum diisi.

---

## D. Setelan Looker Studio

- **Connector:** Google Sheets → spreadsheet ini → pilih semua tab.
- **Field type:** set sesuai §B; sembunyikan kolom ABAIKAN.
- **Calculated field** (contoh):
```
Headcount          = COUNT(NIK)
Total Jam          = SUM(TOTAL JAM TRAINING (JAM))
Rata N-Gain        = AVG(N GAIN SCORE)
Avg Rating         = AVG(RATING)
Turnover           = SUM(Jumlah Karyawan Resign) / AVG(Jumlah karyawan Existing)
Income             = SUM(INCOME)
%Achieve           = SUM(INCOME) / SUM(TARGET)
Okupansi %         = AVG(PERSENTASE OKUPANSI)
Responden          = COUNT_DISTINCT(response_id)
Rata Skor          = AVG(score)
Skor %             = AVG((score - scale_min)/(scale_max - scale_min))
Jumlah Jawaban     = COUNT(response_id)
Skala              = CONCAT(CAST(scale_min AS TEXT), "–", CAST(scale_max AS TEXT))
Bln                = TODATE(visit_date, "MONTH")
```

> `Skor %` = metrik utama lintas soal (0–100%); **tanpa target**.
