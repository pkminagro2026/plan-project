# Wireframe ASCII — Dashboard HC INAGRO 2026 (6 halaman)

```
================== P0 · COVER / RINGKASAN ==================   [Filter: Tahun ▽]
+-------------------------------------------------------------------------------------------+
| [LOGO]   DASHBOARD HC INAGRO 2026                                               [Date ▾]  |
| [COVER] DEMOGRAFI | ATTRITION | TRAINING | REVIEW | BTK/SALES | QUESTIONAIRE   (navigasi) |
+-------------------------------------------------------------------------------------------+
| +---------+ +---------+ +---------+ +---------+ +----------+ +----------+                 |
| | TotalTK | |Turnover%| |Jam Train| |AvgRating| |Pengunjung| |Okupansi% |                 |
| |   219   | |  1,9%   | |   247   | |  4,53   | |   ...    | |   31%    |                 |
| +---------+ +---------+ +---------+ +---------+ +----------+ +----------+                 |
+--------------------------------------------------+---------------------------------------+
| Tren Pengunjung 12 bulan (line)                  | Headcount per Divisi (bar)            |
|                                                  |                                       |
+--------------------------------------------------+---------------------------------------+
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
