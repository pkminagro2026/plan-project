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
