# Rumus — Helper Google Sheets & Calculated Field Looker Studio

## A. Helper Column (Google Sheets)

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

## B. Calculated Field (Looker Studio)

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
```

---

## C. Questionaire — Long Format (untuk Looker)

### C1. Tab HISTORIS — unpivot QUESTIONNAIRE (paste sekali, lalu Values-only)
```
=QUERY(VSTACK(
  {"response_id","submitted_at","visit_date","respondent_name","respondent_school",
   "questionnaire","questionnaire_slug","public_token","category","question","question_type",
   "score","option","scale_min","scale_max"};
  IF(QUESTIONNAIRE!A2:A="","",HSTACK(QUESTIONNAIRE!A2:A,"-",TEXT(QUESTIONNAIRE!D2:D,"dd/mm/yyyy"),QUESTIONNAIRE!B2:B,QUESTIONNAIRE!C2:C,"Historis 2024","historis-2024","-","Fasilitas","Fasilitas Toilet","rating",QUESTIONNAIRE!F2:F,"",1,7));
  IF(QUESTIONNAIRE!A2:A="","",HSTACK(QUESTIONNAIRE!A2:A,"-",TEXT(QUESTIONNAIRE!D2:D,"dd/mm/yyyy"),QUESTIONNAIRE!B2:B,QUESTIONNAIRE!C2:C,"Historis 2024","historis-2024","-","Fasilitas","Fasilitas Meeting Poin","rating",QUESTIONNAIRE!G2:G,"",1,7));
  IF(QUESTIONNAIRE!A2:A="","",HSTACK(QUESTIONNAIRE!A2:A,"-",TEXT(QUESTIONNAIRE!D2:D,"dd/mm/yyyy"),QUESTIONNAIRE!B2:B,QUESTIONNAIRE!C2:C,"Historis 2024","historis-2024","-","Fasilitas","Fasilitas Wara Wiri","rating",QUESTIONNAIRE!H2:H,"",1,7));
  IF(QUESTIONNAIRE!A2:A="","",HSTACK(QUESTIONNAIRE!A2:A,"-",TEXT(QUESTIONNAIRE!D2:D,"dd/mm/yyyy"),QUESTIONNAIRE!B2:B,QUESTIONNAIRE!C2:C,"Historis 2024","historis-2024","-","Pelayanan","Penjelasan Sales","rating",QUESTIONNAIRE!E2:E,"",1,7));
  IF(QUESTIONNAIRE!A2:A="","",HSTACK(QUESTIONNAIRE!A2:A,"-",TEXT(QUESTIONNAIRE!D2:D,"dd/mm/yyyy"),QUESTIONNAIRE!B2:B,QUESTIONNAIRE!C2:C,"Historis 2024","historis-2024","-","Pelayanan","Pengarahan Tour Guide","rating",QUESTIONNAIRE!I2:I,"",1,7));
  IF(QUESTIONNAIRE!A2:A="","",HSTACK(QUESTIONNAIRE!A2:A,"-",TEXT(QUESTIONNAIRE!D2:D,"dd/mm/yyyy"),QUESTIONNAIRE!B2:B,QUESTIONNAIRE!C2:C,"Historis 2024","historis-2024","-","Produk","Keragaman Permainan Edukasi","rating",QUESTIONNAIRE!J2:J,"",1,7));
  IF(QUESTIONNAIRE!A2:A="","",HSTACK(QUESTIONNAIRE!A2:A,"-",TEXT(QUESTIONNAIRE!D2:D,"dd/mm/yyyy"),QUESTIONNAIRE!B2:B,QUESTIONNAIRE!C2:C,"Historis 2024","historis-2024","-","Produk","Citarasa Katering","rating",QUESTIONNAIRE!K2:K,"",1,7));
  IF(QUESTIONNAIRE!A2:A="","",HSTACK(QUESTIONNAIRE!A2:A,"-",TEXT(QUESTIONNAIRE!D2:D,"dd/mm/yyyy"),QUESTIONNAIRE!B2:B,QUESTIONNAIRE!C2:C,"Historis 2024","historis-2024","-","Kepuasan","Niat Kembali","radio","",QUESTIONNAIRE!L2:L,1,7))
),"select * where Col1 <> ''",0)
```

### C2. Tab APP-YYYY-MM — hasil Export Looker
```
(Import CSV → Replace sheet; tanpa kolom bantu)
```

### C3. Tab LOOKER_ALL — sumber tunggal Looker
```
={HISTORIS!A2:P; APP-2026-01!A2:P; APP-2026-02!A2:P}
# "1 tab = 1 periode". Menambah bulan = tambahkan APP-YYYY-MM ke union.
```

### C4. Calculated Field (Looker Studio)
```
Responden = COUNT_DISTINCT(response_id)
Rata Skor = AVG(score)
Skor %    = AVG((score - scale_min)/(scale_max - scale_min))
Bln       = TODATE(PARSE_DATE("%m/%d/%y", visit_date), "MONTH")
```
