# Penjelasan Setiap Properti `fortify-sca-quickscan.properties`

> **Sumber:** [fortify-sca-quickscan.properties — Micro Focus Docs (SCA 23.2.0)](https://www.microfocus.com/documentation/fortify-static-code-analyzer-and-tools/2320/SCA_Help_23.2.0/Content/config-props/quick-scan-props.htm)

Dikelompokkan jadi 4 kategori:

---

## 🕐 Kategori 1: Batas Waktu Analisis (Time Limit)

### 1. `com.fortify.sca.CtrlflowMaxFunctionTime`

**Apa:** Batas waktu (milidetik) untuk analisis **Control Flow** pada satu fungsi.

- **Quick scan:** `30000` (30 detik per fungsi)
- **Normal:** `600000` (10 menit per fungsi)

**Cara kerja:** Kalau satu fungsi butuh waktu lebih dari batas ini, analyzer berhenti menganalisis fungsi itu dan lanjut ke fungsi berikutnya. Jadi fungsi kompleks bisa dianalisis sebagian saja.

- **Efek jika diperbesar:** Coverage analisis lebih baik, tapi scan bisa sangat lambat.
- **Efek jika diperkecil:** Scan cepat, tapi fungsi kompleks "dipotong" analisisnya → potensi false negative (bug terlewat).
- **Analogi:** Seperti guru yang memberi waktu 30 detik per soal ujian — soal susah ditinggal, soal gampang diselesaikan.

---

### 2. `com.fortify.sca.NullPtrMaxFunctionTime`

**Apa:** Sama seperti di atas, tapi khusus untuk analisis **Null Pointer** (bug null dereference).

- **Quick scan:** `10000` (10 detik)
- **Normal:** `300000` (5 menit)

Default normalnya 5 menit. Kalau diperkecil, waktu scan turun tapi bisa melewatkan null-pointer bug di fungsi besar.

**Contoh bug yang dicari:** `obj.method()` padahal `obj` bisa `null`.

---

## 🔧 Kategori 2: Kontrol Analyzer

### 3. `com.fortify.sca.DisableAnalyzers`

**Apa:** Daftar analyzer yang **dimatikan** saat scan. Bisa dipisahkan dengan koma atau titik dua.

Analyzer yang tersedia:

| Analyzer | Fungsi |
|---|---|
| `buffer` | Deteksi buffer overflow |
| `content` | Analisis konten (misal file/request) |
| `configuration` | Cek konfigurasi |
| `controlflow` | Alur eksekusi program |
| `dataflow` | Aliran tainted data (source → sink) |
| `nullptr` | Null pointer |
| `semantic` | Analisis makna/semantik kode |
| `structural` | Analisis struktur kode |

- **Quick scan:** `controlflow:buffer` → dua analyzer paling "mahal" dimatikan
- **Normal:** (tidak ada yang dimatikan)

**Ini adalah properti perubahan terbesar antara quick dan normal scan.** Karena itu quick scan bisa jauh lebih cepat — dua analyzer terberat tidak jalan.

⚠️ **Penting:** Dengan `controlflow` dan `buffer` mati, bug seperti buffer overflow & kelemahan alur eksekusi bisa **tidak terdeteksi sama sekali** di quick scan.

---

### 4. `com.fortify.sca.TrackPaths`

**Apa:** Mengontrol **path tracking** untuk analisis Control Flow.

Nilai yang mungkin:

- `None` → semua fungsi tidak di-track
- `NoJSP` → hanya JSP yang tidak di-track
- (kosong) → tracking penuh

- **Quick scan:** (tidak ada nilai / kosong)
- **Normal:** `NoJSP`

**Path tracking** = melacak jalur eksekusi detail dari awal sampai bug terjadi. Lebih detail = lebih lama.

Menarik: di quick scan, karena `controlflow` analyzer sudah dimatikan total, setelan ini jadi tidak relevan. Di normal scan, JSP dikecualikan supaya lebih cepat.

---

## 📊 Kategori 3: Limiter Dataflow (Kedalaman & Kompleksitas)

### 5. `com.fortify.sca.limiters.MaxChainDepth`

**Apa:** Kedalaman maksimum **rantai pemanggilan fungsi** yang dilacak Dataflow Analyzer saat mengikuti tainted data.

- **Quick scan:** `3`
- **Normal:** `5`

**Penting:** Ini bukan kedalaman dari `main()`, tapi jarak maksimum antara **taint source → sink**.

Contoh: `input user → fungsiA() → fungsiB() → fungsiC() → SQL query`.
Kalau depth-nya 3, jalur sampai fungsi C tidak akan terdeteksi.

**Jika diperbesar:** Coverage dataflow bertambah (bug lebih banyak ketemu), tapi scan **melambat drastis** (berpotensi eksponensial).

---

### 6. `com.fortify.sca.limiters.MaxFunctionVisits`

**Apa:** Berapa kali **taint propagation analyzer** boleh mengunjungi satu fungsi.

- **Quick scan:** `5`
- **Normal:** `50`

Semakin tinggi, semakin teliti aliran taint dari berbagai path diperhitungkan — tapi makin lambat.

---

### 7. `com.fortify.sca.limiters.MaxPaths`

**Apa:** Jumlah maksimum **path** yang dilaporkan untuk satu kerentanan dataflow.

- **Quick scan:** `1`
- **Normal:** `5`

**Catatan penting:** Mengubah nilai ini **tidak mengubah jumlah bug yang ditemukan** — hanya berapa banyak cara/path yang ditampilkan per bug. Jadi ini murni soal detail laporan.

Fortify menyarankan **jangan lebih dari 5**, karena bisa memperlambat scan.

**Analogi:** Bug-nya sama, cuma "petunjuk arah" yang ditampilkan 1 atau 5 rute.

---

### 8. `com.fortify.sca.limiters.MaxTaintDefForVar`

**Apa:** Batas kompleksitas untuk Dataflow Analyzer per variabel.

- **Quick scan:** `250`
- **Normal:** `1000`

Kalau sebuah fungsi melebihi metrik kompleksitas ini, Fortify **menurunkan presisi analisisnya** (analisis tetap jalan, tapi kurang detail/akurat).

**Efek diperbesar:** Analisis lebih presisi, scan lebih lambat.

---

### 9. `com.fortify.sca.limiters.MaxTaintDefForVarAbort`

**Apa:** Batas **keras** kompleksitas fungsi.

- **Quick scan:** `500`
- **Normal:** `4000`

Bedanya dengan di atas: kalau batas ini dilampaui **pada level presisi paling rendah sekalipun**, fungsi tersebut **dilewati sama sekali** (skip total).

**Jadi urutannya:**

1. Fungsi normal → dianalisis penuh
2. Kompleks (> `MaxTaintDefForVar`) → presisi diturunkan
3. Sangat kompleks (> `MaxTaintDefForVarAbort`) → **tidak dianalisis sama sekali**

---

### 10. `com.fortify.sca.limiters.ConstraintPredicateSize`

**Apa:** Batas ukuran kalkulasi kompleks di **Buffer Analyzer**.

- **Quick scan:** `10000`
- **Normal:** `500000`

Kalau ada kalkulasi (misal operasi aritmatika panjang untuk menentukan ukuran buffer) yang melebihi batas ini, **perhitungan dilewati** demi kecepatan.

**Implikasi:** Bug buffer yang butuh perhitungan kompleks bisa tidak terdeteksi di quick scan.

---

## 📁 Kategori 4: Output & FPR (Hasil Scan)

### 11. `com.fortify.sca.FilterSet`

**Apa:** Set filter yang diterapkan **saat scan** (bukan setelah scan).

- **Quick scan:** `Quick View`
- **Normal:** (tidak ada)

`Quick View` = hanya rule dengan **impact tinggi**:

- High impact + kemungkinan besar terjadi
- High impact + kemungkinan kecil terjadi

**Keuntungan penting:** Isu yang terfilter **tidak ditulis ke FPR** → ukuran file FPR lebih kecil.

Bisa dikombinasikan dengan issue template via `com.fortify.sca.ProjectTemplate`.

**Analogi:** Seperti filter email — hanya yang penting yang masuk inbox, sisanya dibuang (bukan cuma diarsip).

---

### 12. `com.fortify.sca.FPRDisableMetatable`

**Apa:** Menonaktifkan pembuatan **metatable** — data yang mendukung fitur **Function view** di Fortify Audit Workbench (klik kanan variabel → lihat deklarasi).

- **Quick scan:** `true` (tidak dibuat)
- **Normal:** `false` (dibuat)
- **Opsi CLI:** `-disable-metatable`

**Poin penting:** Untuk scan **C/C++**, mengaktifkan properti ini (nilai `true`) bisa **menghemat waktu berjam-jam**.

**Trade-off:** Fitur "klik variabel → lihat deklarasi" di Audit Workbench tidak berfungsi.

---

### 13. `com.fortify.sca.FPRDisableSourceBundling`

**Apa:** Menonaktifkan **penyertaan source code** ke dalam file FPR. Fortify tidak membuat file source yang sudah di-markup.

- **Quick scan:** `true` (source tidak dibundel)
- **Normal:** `false` (source dibundel)
- **Opsi CLI:** `-disable-source-bundling`

⚠️ **Kritikal:** Kalau kamu berencana **upload FPR dari quick scan ke Fortify SSC**, kamu **WAJIB** set properti ini ke `false` — kalau tidak, FPR tanpa source code tidak akan tampil optimal di server (tidak bisa lihat code di UI).

---

## Ringkasan Pola yang Bisa Dilihat

| Strategi Quick Scan | Properti yang Berperan |
|---|---|
| Matikan analisis berat | `DisableAnalyzers`, `TrackPaths` |
| Perpendek timeout | `CtrlflowMaxFunctionTime`, `NullPtrMaxFunctionTime` |
| Perdangkal pelacakan data | `MaxChainDepth`, `MaxFunctionVisits` |
| Kurangi detail output | `MaxPaths` |
| Skip fungsi terlalu kompleks | `MaxTaintDefForVar`, `MaxTaintDefForVarAbort`, `ConstraintPredicateSize` |
| Filter hasil | `FilterSet` |
| Kecilkan FPR | `FPRDisableMetatable`, `FPRDisableSourceBundling` |

**Intinya:** Quick scan = kompromi antara **kecepatan** dan **kedalaman**. Setiap properti di atas adalah "tombol" yang diturunkan untuk mengejar kecepatan, dengan konsekuensi risiko false negative yang lebih tinggi — terutama untuk buffer overflow, control flow, dan dataflow lintas fungsi yang dalam.
