Nama : Rizky Maulana
Kelas : RM 24A
Nim : 24552011345


# Tugas Hands-On 3: Pandas Lanjutan

**Kasus dipilih:** 01 – Credit Risk Prediction (Fintech), `case1_credit_risk.csv`, 2.500 baris × 15 kolom, klasifikasi biner (target `status_kredit`).

## Ringkasan kualitas data

| | Sebelum | Sesudah |
| --- | --- | --- |
| Baris | 2.500 | 2.500 |
| Duplikat (`customer_id`) | 0 | 0 |
| Missing value (total sel) | 972 | 739 |
| Outlier `credit_score` (IQR per `tujuan_pinjaman`) | 3 | 0 |

Masalah yang ditemukan dan keputusan pembersihan:

- **Missing value** di 9 kolom. Kategorikal (`jenis_kelamin`, `pendidikan`, `status_pekerjaan`) diisi `Tidak Diketahui`. Numerik **sengaja tidak diisi** saat pembersihan; imputasi median dilakukan di `ColumnTransformer` setelah split agar tidak leakage. Penanda `nilai_agunan_idr_kosong` ditambahkan (± 14% kosong).
- **`jenis_kelamin` tidak konsisten** (8 variasi, termasuk simbol `@`) dinormalkan menjadi `Laki-laki` / `Perempuan`.
- **`credit_score` negatif** (3 baris, di luar skala 300–850) diubah menjadi NaN, tidak di-clip, supaya tidak mengarang nilai.
- Kolom numerik lain tidak memiliki outlier IQR per grup. Tipe data dirapikan (`Int64`, `category`).
- Time series dan RFM dilewati: tidak ada kolom tanggal dan tidak ada riwayat transaksi per pelanggan.

## Tiga temuan

1. **Skor kredit adalah pemisah risiko terkuat.** Tingkat gagal bayar pada kuartil skor terendah (≤ 448) adalah **68,2%**, sedangkan pada kuartil tertinggi (> 719) hanya **26,1%** (rata-rata keseluruhan 43,4%). *Tindak lanjut:* jadikan skor < 450 sebagai pemicu tinjauan manual atau plafon pinjaman yang lebih rendah.
2. **Rasio utang/pendapatan di atas 0,69 melipatgandakan risiko, dan efeknya menumpuk dengan skor rendah.** Default 62,3% pada kuartil DTI tertinggi versus 33,2% pada kuartil terendah. Kelompok dengan skor terendah **dan** DTI tertinggi (141 pelanggan, ± 5,6% data) gagal bayar **85,1%**. *Tindak lanjut:* terapkan batas DTI maksimum sekitar 0,69 dan tolak otomatis atau minta jaminan tambahan untuk kombinasi skor rendah + DTI tinggi.
3. **Perilaku bayar lebih menentukan daripada profil demografis.** Pelanggan dengan ≥ 6 keterlambatan gagal bayar sekitar 50%, versus 32,0% untuk 0–2 keterlambatan. Sebaliknya, tingkat pendidikan, status pekerjaan, dan tujuan pinjaman hanya berselisih beberapa poin persentase (pendidikan 2,4 poin). *Tindak lanjut:* jangan jadikan pendidikan atau tujuan pinjaman dasar kebijakan risiko; fokuskan penilaian pada riwayat keterlambatan, skor, dan DTI.

## Persiapan model

Split 80/20 dengan `stratify=y` dan `random_state=42`, lalu `ColumnTransformer` (imputer median + `StandardScaler` untuk numerik, imputer modus + `OneHotEncoder(drop='first')` untuk kategorikal), semuanya di-`fit` hanya pada data train. Model dasar regresi logistik memberi ROC-AUC ± 0,72 pada data uji (hanya untuk memastikan pipeline berfungsi).

## Cara menjalankan

1. Buka `notebook.ipynb` di Google Colab (File → Upload notebook).
2. Pilih *Runtime → Run all*. Bila diminta, unggah `data/case1_credit_risk.csv`.
3. Untuk pengecekan akhir, gunakan *Restart session and run all*.

## Struktur

```text
tugas-hands-on-3/
├── README.md
├── notebook.ipynb
├── data/case1_credit_risk.csv
├── output/data_bersih.csv
├── output/kamus_data.csv
└── laporan_kualitas.csv
```
