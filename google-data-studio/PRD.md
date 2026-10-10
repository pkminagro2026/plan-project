# FULL PREVIEW PLAN — Dashboard HC INAGRO 2026 (Looker Studio)

> Status: **RENCANA (belum dieksekusi)**. Dokumen ini = preview lengkap yang nanti dipecah jadi file `PRD.md`, `wireframes.md`, `rumus.md`.

---

## 1. Ringkasan & Tujuan
Membangun **1 dashboard Looker Studio** dari 1 spreadsheet Google (5 sheet) untuk HRD & Manajemen: demografi, attrition/turnover, training, rating Google Maps, dan penjualan/okupansi. Simple, tidak sesak, dipakai ulang (template).

**Perbaikan dari dashboard lama (mangkrak):** hapus tab COMPENSATION/VALUE/AGRIBISNIS dll, buang teks bertumpuk ("Tidak ada data" menumpuk), buang MARGIN, ganti metrik surplus jadi **Selisih Target** + **Pemasukan per Karyawan**.

---

## 2. Inventaris Sumber Data
Spreadsheet ID: `1U7012E0_HNmzVHipxTDKvYJU4N91JCNdeol2_LBRC7c`

| Domain | Sheet (gid) | Kolom inti | Kondisi saat init |
|---|---|---|---|
| Demografi | DATA TENAGA KERJA (`1978650831`) | NIK, NAMA, DEPARTEMEN, DIVISI, POSISI, JK, MARITAL, STATUS, PENDIDIKAN, TGL LAHIR, TGL JOIN | NAMA/TGL LAHIR kosong (template); USIA=127 rusak; ada `#REF!` |
| Attrition | DATA ATTRITION (`324423395`) | Bulan, Existing, Resign, Voluntary, Involuntary, Kary. Baru | 15 baris (2023–Mar 2024) |
| Training | TRAINING RECORD (`1229589266`) | NIK, NAMA, JABATAN, STATUS, DIVISI, DEPARTEMEN, NAMA TRAINING, TGL, JAM, PRE/POST/N-Gain | NAMA & DEPARTEMEN kosong; pre/post ~19 baris |
| Review | GOOGLE RIVIEW (`791906249`) | NO, NAMA, TANGGAL, RATING | Lengkap (5★160, 4★34, 3★5, 2★4, 1★11) |
| Sales | BTK TO SALES (`1035995509`) | BULAN, INCOME, TAX, TOTAL, MARGIN, TARGET, ANAK/GURU/DEWASA, OKUPANSI, okupansi wahana & room | INCOME/TAX/MARGIN kosong (template) |

**Questionnaire → halaman P6.**

---

## 3. Definisi Metrik (Final)
| Metrik | Rumus | Sumber |
|---|---|---|
| Headcount | `COUNT(NIK)` | Tenaga Kerja |
| Rata Usia | `AVG(Usia)` | Tenaga Kerja |
| Turnover Rate | `SUM(Resign)/AVG(Existing)` | Attrition |
| Target Turnover | `0.10` | konstanta |
| Total Jam Training | `SUM(JAM)` | Training |
| Peserta Unik | `COUNT_DISTINCT(NIK)` | Training |
| Rata N-Gain | `AVG(N GAIN SCORE)` | Training |
| Avg Rating | `AVG(RATING)` | Review |
| **Selisih Target** | `SUM(INCOME) − SUM(TARGET)` | BTK |
| **Pemasukan / Karyawan** | `SUM(INCOME) / AVG(ATTRITION.Existing)` | BTK × Attrition (blend by Bulan) |
| Pendapatan / Hari | `SUM(INCOME) / 20` | BTK (20 hari kerja) |
| % Achievement | `SUM(INCOME)/SUM(TARGET)` | BTK |
| % Okupansi | `AVG(PERSENTASE OKUPANSI)` | BTK |
| Responden | `COUNT_DISTINCT(response_id)` | Questionaire (long) |
| Rata Skor | `AVG(score)` | Questionaire (long) |
| Skor % | `AVG((score − scale_min)/(scale_max − scale_min))` | Questionaire (long) |

> **MARGIN dihapus total** dari semua halaman.

---

## 4. Data Prep (helper column di spreadsheet)
**DATA TENAGA KERJA** (P=TGL LAHIR, F=TGL JOIN, Z=STATUS, V=JK, AI=USIA, AH=MASA KERJA):
```gs
AI2 (Usia)         =IFERROR(DATEDIF(P2,TODAY(),"Y"),"")
AH2 (Masa Kerja)   =IFERROR(DATEDIF(F2,TODAY(),"Y"),"")
AK2 (Rentang Usia) =IF(AI2="","",IFS(AI2<20,"<20",AI2<=40,"20-40",AI2<=58,"41-58",TRUE,">58"))
AL2 (Status bersih)=IFERROR(PROPER(TRIM(Z2)),"")
AM2 (Gender)       =IF(V2="L","Laki-laki",IF(V2="P","Perempuan",""))
```
**TRAINING RECORD** (F=DEPARTEMEN ← turunkan dari C=JABATAN):
```gs
F2 = IFS(
  REGEXMATCH(C2,"(?i)housekeep"),"HOUSEKEEPING",
  REGEXMATCH(C2,"(?i)security"),"SECURITY",
  REGEXMATCH(C2,"(?i)kebun"),"KEBUN",
  REGEXMATCH(C2,"(?i)toko|cafe|barista"),"TOKO & CAFE",
  REGEXMATCH(C2,"(?i)kuljar|kultur"),"KULTUR JARINGAN",
  REGEXMATCH(C2,"(?i)ternak|fauna|mini zoo"),"FAUNA",
  REGEXMATCH(C2,"(?i)edutainment"),"EDUTAINMENT",
  REGEXMATCH(C2,"(?i)markom|marcom"),"MARKETING COMMUNICATION",
  REGEXMATCH(C2,"(?i)keuangan|accounting|fap"),"FINANCE ACCOUNTING & PURCHASING",
  REGEXMATCH(C2,"(?i)driver"),"DRIVER",
  TRUE,"LAINNYA")
P2 (Bulan) =IFERROR(TEXT(H2,"mmm yyyy"),"")
```
**BTK TO SALES** (B=INCOME, C=TAX, D=TOTAL, F=TARGET):
```gs
C2 (Tax 10%)        =IFERROR(B2*0.1,"")
D2 (Net Income)     =IFERROR(B2-C2,"")
-   (Selisih dihitung di Looker, bukan di sheet)
-   (Pendapatan/Hari & Pemasukan/Karyawan dihitung di Looker)
```
**DATA ATTRITION** (B=Existing, C=Resign, D=Voluntary, F=Baru):
```gs
Turnover  =IFERROR(C2/B2,"")
Voluntary =IFERROR(D2/B2,"")
Net Growth=IFERROR((F2-C2)/B2,"")
```

---

## 5. PRD (Product Requirements)
**Pengguna:** HRD (operasional harian), Manajemen (ringkasan).
**Scope:** 6 halaman, 1 konektor Google Sheets, filter global + per halaman.
**Non-fungsional:** load < 5 s, responsif (desktop/mobile), aman saat data kosong (IFERROR/parameter), label tidak tumpang-tindih, angka 1 baris.
**Fungsional per halaman:**
- **P0 Cover:** 6 kartu KPI + tren pengunjung + headcount per divisi; filter Tahun.
- **P1 Demografi:** headcount per departemen (stack status), gender donut, rentang usia, posisi, pendidikan, tabel rincian; filter Divisi/Dept/Status/Posisi/Gender/Usia.
- **P2 Attrition:** kartu Existing/Resign/Voluntary/Baru, line gabungan + target 10%, turnover per bulan, donut vol/invol.
- **P3 Training:** kartu jam/peserta/N-Gain, bar peserta per training, bar jam per divisi, line bulanan, tabel peserta.
- **P4 Review:** avg rating, total ulasan, distribusi bintang, tren bulanan, tabel ulasan.
- **P5 BTK/Sales:** kartu Income/Target/%Achieve/Selisih Target/Pemasukan per Karyawan/Pendapatan per Hari; bar Income vs Target vs Selisih; line okupansi + target 70%; donut anak/guru/dewasa; bar okupansi per wahana & per room.
- **P6 Questionaire:** scorecards Responden/Rata Skor/opsi teratas; bar rata skor per soal (+target 6,0); donut opsi radio; line tren per bulan; tabel detail; filter Kuesioner/Kategori/Tanggal.
**Out of scope:** tab COMPENSATION/VALUE/dll, MARGIN.

**Strategi Blend (Looker):**
- `BTK × DATA ATTRITION` pada kunci **Bulan** → untuk Pemasukan per Karyawan.
- `TRAINING × DATA TENAGA KERJA` pada kunci **NIK** → untuk melengkapi Nama/Departemen.
- Sisanya single-source (no blend) agar cepat.

---

## 6. Design System
- **Grid:** 12 kolom, gutter 16px; maks 6 chart/halaman.
- **Warna INAGRO:** Navy `#1A237E` (header), Hijau `#2E7D32`, Oranye `#EF6C00`, abu `#ECEFF1`, putih.
- **Tipografi:** sans (Roboto), judul 20–24, label 11–12, angka KPI 28–32.
- **Konsisten:** semua kartu KPI tinggi seragam; garis target selalu putus-putus merah.

---

## 7. Wireframe ASCII (final, 6 halaman)

```
================== P0 · COVER / RINGKASAN ==================   [Filter: Tahun ▽]
+--------------------------------------------------------------------------------+
| [LOGO]   DASHBOARD HC INAGRO 2026                                    [Date ▾]  |
| [COVER] DEMOGRAFI | ATTRITION | TRAINING | REVIEW | BTK/SALES   (navigasi)      |
+--------------------------------------------------------------------------------+
| +---------+ +---------+ +---------+ +---------+ +----------+ +----------+       |
| | TotalTK | |Turnover%| |Jam Train| |AvgRating| |Pengunjung| |Okupansi% |       |
| |   219   | |  1,9%   | |   247   | |  4,53   | |   ...    | |   31%    |       |
| +---------+ +---------+ +---------+ +---------+ +----------+ +----------+       |
+------------------------------------------+-------------------------------------+
| Tren Pengunjung 12 bulan (line)          | Headcount per Divisi (bar)          |
|                                          |                                     |
+------------------------------------------+-------------------------------------+
```

```
================ P1 · DEMOGRAFI TENAGA KERJA ================
[Filter: Divisi | Departemen | Status | Posisi | Gender | Rentang Usia]
+------------------------+------------------------+-------------------------------+
| Headcount per          | Distribusi Gender      | Rata Usia & Masa Kerja        |
| Departemen (bar)       | (donut L/P: 194/25)    | (2 kartu angka)               |
+------------------------+------------------------+-------------------------------+
| Headcount per Departemen (bar, stack Status: TKHL / PKWT / PKWTT / OS)          |
+------------------------+------------------------+-------------------------------+
| Rentang Usia (bar)     | Posisi (bar)           | Pendidikan (bar)              |
+------------------------+------------------------+-------------------------------+
| TABEL RINCIAN: NIK | Nama | Departemen | Posisi | Status | Gender | Usia | MK    |
+--------------------------------------------------------------------------------+
```

```
=================== P2 · ATTRITION & TURNOVER ===================   [Tahun | Bulan]
+----------------+----------------+----------------+----------------+
| Existing       | Resign         | Voluntary      | Karyawan Baru  |
+----------------+----------------+----------------+----------------+
| Line: Existing vs Resign vs New Hire   - - - target Turnover 10% - - -          |
+------------------------------------------+--------------------------------------+
| Turnover Rate per Bulan (bar + target)   | Voluntary vs Involuntary (donut)     |
+------------------------------------------+--------------------------------------+
```

```
===================== P3 · TRAINING RECORD =====================
[Filter: Departemen | Status | Nama Training | Rentang Tanggal]
+------------------+------------------+------------------+
| Total Jam        | Peserta Unik     | Rata N-Gain      |
+------------------+------------------+------------------+
| Bar: Peserta per Nama Training        | Bar: Jam per Divisi (stack Status)     |
+---------------------------------------+----------------------------------------+
| Line: Jam Training per Bulan                                                   |
+--------------------------------------------------------------------------------+
| TABEL: NIK | Nama | Jabatan | Dept | Status | Nama Training | Tgl | Jam | NGain  |
+--------------------------------------------------------------------------------+
```

```
======================= P4 · GOOGLE REVIEW =======================   [Tgl | Rating]
+----------------+----------------+---------------------------------------------+
| Avg Rating     | Total Ulasan   | Distribusi Bintang (bar 5..1)               |
|    4,53        |     214        | 5★160 · 4★34 · 3★5 · 2★4 · 1★11             |
+----------------+----------------+---------------------------------------------+
| Line: Rata-rata Rating per Bulan                                                |
+--------------------------------------------------------------------------------+
| TABEL: NO | Nama Ulasan | Tanggal | Rating | Bintang                             |
+--------------------------------------------------------------------------------+
```

```
============= P5 · BTK TO SALES / OKUPANSI =============   (20 hari kerja)  [Bulan]
+---------+---------+----------+--------------+--------------+--------------+
| Income  | Target  | %Achieve | Selisih      | Pemasukan    | Pendapatan   |
|         |         |          | Target       | / Karyawan   | per Hari     |
+---------+---------+----------+--------------+--------------+--------------+
| Bar: Income vs Target vs Selisih Target per Bulan     | Donut Anak/Guru/Dewasa |
+-------------------------------------------------------+-------------------------+
| Line: % Okupansi + garis target 70%                                            |
+---------------------------------------+----------------------------------------+
| Bar: Okupansi per Wahana              | Bar: Okupansi per Tipe Room            |
+---------------------------------------+---------------------------------------+
```

```
======= P6 · QUESTIONAIRE / KEPUASAN PENGUNJUNG =======   [Kuesioner▾][Kategori▾][Tgl▾]
+---------+----------+----------------+-----------------+
|Responden| Rata Skor| Opsi Teratas   | Skor Terendah   |
|  352    |  5,79    | BELUM TAU 51.6%| Toilet 5,37     |
+---------+----------+----------------+-----------------+
| Bar: Rata Skor per Soal (sort naik) + target 6,0 | Donut Opsi Radio |
+--------------------------------------------------+------------------+
| Line: Tren Rata Skor per Bulan                                         |
+------------------------------------------------------------------------+
| TABEL: ResponseID | Nama | Sekolah | Soal | Skor/Opsi   | Kuesioner     |
+------------------------------------------------------------------------+
```

---

## 8. Katalog Rumus Looker Studio (Calculated Fields)
```
--- Demografi ---
Headcount        = COUNT(NIK)
Rata Usia        = AVG(Usia)
Gender           = CASE WHEN JENIS KELAMIN="L" THEN "Laki-laki" ELSE "Perempuan" END

--- Attrition ---
Turnover Rate    = SUM(Jumlah Karyawan Resign) / AVG(Jumlah karyawan Existing)
Voluntary Rate   = SUM(Jumlah Voluntary Resign) / AVG(Jumlah karyawan Existing)
Target Turnover  = 0.10
Bulan (date)     = PARSE_DATE("%m/%d/%Y", Bulan)

--- Training ---
Total Jam        = SUM(TOTAL JAM TRAINING (JAM))
Peserta Unik     = COUNT_DISTINCT(NIK)
Rata N-Gain      = AVG(N GAIN SCORE)
Bulan Training   = TODATE(PARSE_DATE("%m/%d/%Y", TANGGAL), "MONTH")

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
Bln              = TODATE(PARSE_DATE("%m/%d/%y", visit_date), "MONTH")
```

---

## 9. Langkah Eksekusi (saat disetujui)
1. **Prep sheet:** tambah helper column (§4) di DATA TENAGA KERJA, TRAINING RECORD, BTK, ATTRITION.
2. **Looker Studio:** buat 1 data source Google Sheets (5 sheet/tab, "Include all") → lalu buat 3 blend (§5).
3. **Bangun 6 halaman** sesuai wireframe (§7); pasang filter & calculated field (§8).
4. **Poles:** warna/label konsisten, cek tidak ada chart bertumpuk, uji data kosong (IFERROR/target).
5. **Uji terima** (§10) lalu bagikan.

## 10. Acceptance Criteria
- [ ] 6 halaman tampil, navigasi antar halaman jalan.
- [ ] Filter global & per-halaman memengaruhi semua chart.
- [ ] Semua rumus aman saat sheet masih kosong (tidak menampilkan error mentah).
- [ ] Angka KPI cocok dengan hitung manual pada data sampel.
- [ ] Tidak ada label/teks bertumpuk; layout rapi di desktop & mobile.
- [ ] MARGIN tidak ada; "Surplus" digantikan Selisih Target + Pemasukan/Karyawan.

## 11. Deliverable (dibuat saat eksekusi)
- `PRD.md` — dokumen PRD (§1–§10).
- `wireframes.md` — 6 wireframe ASCII (§7).
- `rumus.md` — helper Sheets + calculated field Looker (§4, §8).

## 12. Risiko / Loose Ends
- Nama & TGL LAHIR kosong (template) → tabel nama & usia baru bermakna setelah data terisi.
- `USIA=127` dan `#REF!` harus dibersihkan (langkah §4).
- Attrition hanya sampai Mar 2024 → tren terbatas sampai data diupdate.
- INCOME/MARGIN kosong → metrik sales baru bermakna setelah income diisi.
- Respons baru masuk lewat app, bukan sheet → wajib Export Looker agar P6 sinkron.
