# Fortify SCA Quick Scan — Ringkasan Sederhana

> **Sumber:** [fortify-sca-quickscan.properties — Micro Focus Docs (SCA 23.2.0)](https://www.microfocus.com/documentation/fortify-static-code-analyzer-and-tools/2320/SCA_Help_23.2.0/Content/config-props/quick-scan-props.htm)

---

## Apa itu Quick Scan?

Mode scan **lebih cepat tapi kurang mendalam**. Aktifkan dengan opsi `-quick` di command line. Secara default, quick scan:

- Mengurangi kedalaman analisis (control flow & buffer analyzer dibatasi/dimatikan)
- Menerapkan filter **Quick View** → hanya menampilkan isu **Critical & High**

> ⚠️ Properti di file `fortify-sca-quickscan.properties` **hanya dipakai kalau kamu pakai opsi `-quick`**.

---

## Tabel Properti (Quick Scan vs Normal)

| Properti | Fungsi singkat | Quick | Normal |
|---|---|---|---|
| `CtrlflowMaxFunctionTime` | Batas waktu analisis Control Flow per fungsi (ms) | **30.000** | 600.000 |
| `DisableAnalyzers` | Analyzer yang dimatikan | **controlflow:buffer** | (tidak ada) |
| `FilterSet` | Filter isu saat scan | **Quick View** | (tidak ada) |
| `FPRDisableMetatable` | Matikan metatable (info Function view di Audit Workbench) | **true** | false |
| `FPRDisableSourceBundling` | Jangan sertakan source code ke dalam FPR | **true** | false |
| `NullPtrMaxFunctionTime` | Batas waktu analisis Null Pointer per fungsi (ms) | **10.000** | 300.000 |
| `TrackPaths` | Pelacakan path untuk Control Flow | **(tidak ada)** | NoJSP |
| `limiters.ConstraintPredicateSize` | Batas kompleksitas kalkulasi Buffer Analyzer | **10.000** | 500.000 |
| `limiters.MaxChainDepth` | Kedalaman maksimum pelacakan tainted data (dataflow) | **3** | 5 |
| `limiters.MaxFunctionVisits` | Berapa kali analyzer mengunjungi satu fungsi | **5** | 50 |
| `limiters.MaxPaths` | Jumlah path yang dilaporkan per isu dataflow | **1** | 5 |
| `limiters.MaxTaintDefForVar` | Batas kompleksitas sebelum presisi diturunkan | **250** | 1.000 |
| `limiters.MaxTaintDefForVarAbort` | Batas keras — fungsi dilewati analisis jika melebihi ini | **500** | 4.000 |

---

## Poin Penting yang Perlu Diingat

1. **Yang dipotong utama:** Control Flow + Buffer analyzer dimatikan (`controlflow:buffer`), waktu analisis per fungsi dipersingkat (30s vs 10 menit).
2. **Hasil lebih sedikit:** Hanya isu Critical/High yang masuk ke FPR (karena filter Quick View).
3. **FPR lebih kecil & lebih cepat:** Source code tidak dibundel, metatable tidak dibuat.
4. **Kalau mau upload FPR ke SSC:** Set `FPRDisableSourceBundling=false` — kalau tidak, FPR tanpa source code mungkin tidak bisa dipakai penuh di server.
5. **Dataflow lebih dangkal:** Chain depth 3 (vs 5), path yang dilaporkan hanya 1 (vs 5), fungsi dikunjungi 5x (vs 50x).
6. **Untuk scan C/C++ yang sangat lambat:** `FPRDisableMetatable=true` bisa menghemat waktu berjam-jam (tapi ini sudah default di quick scan).

---

## Perbandingan Kilat

| Aspek | Quick Scan | Normal Scan |
|---|---|---|
| Kecepatan | ⚡ Jauh lebih cepat | 🐢 Lebih lambat |
| Kedalaman | Dangkal | Mendalam |
| Isu yang dilaporkan | Critical + High saja | Semua severity |
| Ukuran FPR | Kecil | Besar (ada source code) |
| Cocok untuk | Cek cepat, CI/CD awal, dev loop | Audit menyeluruh, sebelum rilis |
