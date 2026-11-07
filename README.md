# Advanced AI, Tugas 1 sampai 4: Prediksi Readmisi 30 Hari Pasien Diabetes

Tugas mata kuliah Advanced AI, S2 PJJ Informatika, Universitas AMIKOM Yogyakarta, semester 1 tahun ajaran 2026/2027.

Keempat tugas memakai satu kasus dan satu dataset: memprediksi apakah pasien diabetes dirawat ulang di rumah sakit dalam 30 hari setelah pulang (readmisi 30 hari). Data yang dipakai adalah UCI *Diabetes 130-US Hospitals for Years 1999-2008*.

## Status tiap tugas

| Tugas | Isi | Berkas | Status |
|---|---|---|---|
| 1 | Knowledge-based system: semantic network, aturan IF-THEN, query dengan pytholog | `notebooks/01_tugas1_kbs_readmisi_pytholog.ipynb` | Ada |
| 2 | Praproses dataset | `notebooks/02_tugas2_praproses_diabetes130.ipynb` | Ada |
| 3 | Pengolahan dataset dengan model yang sesuai dan evaluasinya | `notebooks/03_...` | Belum dikerjakan |
| 4 | Presentasi coding, model terbaik dijadikan knowledge-based system | `notebooks/04_...` | Belum dikerjakan |

Isi dan tenggat Tugas 3 dan 4 mengikuti instruksi di Waskita. Tabel ini diperbarui saat tugasnya dikerjakan.

## Buka di Google Colab

- Tugas 1 (knowledge-based system): [![Buka di Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/sihitam/aai-tugas1-4-readmisi-diabetes130/blob/main/notebooks/01_tugas1_kbs_readmisi_pytholog.ipynb)
- Tugas 2 (praproses): [![Buka di Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/sihitam/aai-tugas1-4-readmisi-diabetes130/blob/main/notebooks/02_tugas2_praproses_diabetes130.ipynb)

## Latar belakang kasus

Data berasal dari database Health Facts (Cerner) dan mencakup kunjungan rawat inap pasien diabetes di 130 rumah sakit Amerika Serikat selama 10 tahun. Strack dkk. (2014) memakainya untuk menguji hubungan antara pemeriksaan HbA1c dan readmisi dini. Kriteria kunjungan yang masuk: rawat inap, diagnosis diabetes, lama rawat 1 sampai 14 hari, ada tes laboratorium, dan ada obat yang diberikan.

Data mentah berisi 101.766 kunjungan dan 50 kolom. Kolom `readmitted` punya tiga nilai: `NO` (54.864), `>30` (35.545), dan `<30` (11.357). Pada label biner yang dipakai di sini, `<30` menjadi kelas positif, sehingga kelasnya tidak seimbang (sekitar 9% setelah pembersihan).

Data ini nyata (bukan sintetik), sesuai syarat dosen untuk tugas Advanced AI.

## Pilihan label: biner

Kolom `readmitted` punya tiga nilai. Repo ini memakai label biner: `<30` hari = 1 (dirawat ulang dalam 30 hari), `>30` dan `NO` = 0. Setelah pembersihan (69.979 pasien), pembagiannya:

| Nilai `readmitted` | Biner | 3 kelas | Jumlah | Porsi |
|---|---|---|---|---|
| `<30` | 1 | 2 | 6.283 | 9,0% |
| `>30` | 0 | 1 | 22.224 | 31,8% |
| `NO` | 0 | 0 | 41.472 | 59,3% |

Alasan memilih biner:

1. Sumber datanya (Strack dkk., 2014) membagi hasil dengan cara yang sama: dirawat ulang dalam 30 hari atau "selain itu", yang mencakup readmisi setelah 30 hari dan tidak ada readmisi.
2. Indeks LACE, dasar knowledge-based system di Tugas 1, memprediksi kematian atau readmisi tak terencana dalam 30 hari (van Walraven dkk., 2010). Kasusnya memang berpusat pada batas 30 hari.
3. Dua kelas membuat evaluasi di Tugas 3 lebih jelas: confusion matrix 2x2, serta recall dan presisi untuk satu kelas yang dicari.
4. Label 3 kelas menambah kelas `>30` (31,8%) yang bukan fokus kasus ini, dan menuntut metrik rata-rata antar kelas serta perubahan KBS di Tugas 1.

Harga dari pilihan ini: kelas positif hanya 9,0%, jadi kelasnya tidak seimbang. Akurasi saja menyesatkan (menebak "tidak" untuk semua pasien sudah 91%), sehingga evaluasi memakai recall, F1, dan AUC, dan ketidakseimbangan ditangani di praproses (langkah 8). Bila memakai 3 kelas, ubah `LABEL_MODE` di notebook Tugas 2 dan jalankan ulang.

## Isi repo

```
.
├── README.md
├── requirements.txt
├── data/
│   ├── README.md                            sumber, lisensi, dan checksum data
│   └── raw/
│       ├── diabetic_data.csv                data asli UCI, tidak diubah
│       └── IDS_mapping.csv                  arti kode admisi dan pulang
└── notebooks/
    ├── 01_tugas1_kbs_readmisi_pytholog.ipynb
    ├── 02_tugas2_praproses_diabetes130.ipynb
    └── semantic_network_readmisi.png        semantic network dari notebook Tugas 1
```

## Tugas 1: knowledge-based system

Notebook ini membangun prototipe knowledge-based system untuk kasus readmisi: semantic network, sembilan aturan IF-THEN, dan query dengan pytholog. Indikator risikonya mengacu pada indeks LACE (van Walraven dkk., 2010). Pada seluruh data uji AUC-nya sekitar 0,60, jadi kemampuannya membedakan risiko hanya sedang. Notebook ini membersihkan data secara minimal; praproses lengkap ada di notebook Tugas 2.

## Tugas 2: praproses

| Langkah | Isi |
|---|---|
| 1 | Muat data dan profil singkat. Missing value ditulis `?`, bukan kosong. |
| 2 | Tangani missing value: `weight` (96,9% kosong) dibuang, `race`, `payer_code`, dan `medical_specialty` diisi "Unknown", baris dengan `diag_1` kosong dibuang, `diag_2` dan `diag_3` kosong diisi "Tidak ada". |
| 3 | Bersihkan baris: buang pasien meninggal atau hospice, gender tidak valid, dan kunjungan berulang (satu kunjungan per pasien). |
| 4 | Buang kolom: pengenal (`encounter_id`, `patient_nbr`), kolom konstan, dan kolom obat yang 99% atau lebih bernilai `No`. |
| 5 | Transformasi: usia ke titik tengah rentang, kode ICD-9 ke kelompok diagnosis, kode admisi dan pulang ke kategori, label ke biner. |
| 6 | Bagi data latih dan uji (80/20, stratified, `random_state=42`). |
| 7 | Outlier (batas persentil ke-99 dari data latih, lalu `log1p`), one-hot encoding, dan standardisasi. Semuanya dipelajari dari data latih saja. |
| 8 | Seimbangkan kelas dengan oversampling acak, hanya pada data latih. Data uji tidak diubah. |
| 9 | Analisis kepentingan fitur pada data latih: Random Forest (embedded), mutual information (filter), dan cek korelasi antar fitur numerik. Tidak ada fitur dibuang; hasilnya jadi bahan Tugas 3 dan 4. |
| 10 | Tabel sebelum dan sesudah, ditambah pemeriksaan otomatis. |
| 11 | Simpan hasil ke `hasil_praproses/` (tidak ikut repo). |

Hasil akhir dari satu kali jalan penuh:

| Tahap | Baris | Kolom |
|---|---|---|
| Data mentah | 101.766 | 50 |
| Setelah missing value | 101.745 | 49 |
| Setelah bersihkan baris | 69.979 | 49 |
| Setelah buang kolom dan transformasi | 69.979 | 32 (31 fitur + label) |

Data latih berisi 55.983 baris dan data uji 13.996 baris, masing-masing 191 fitur setelah encoding. Proporsi kelas positif sekitar 9% di keduanya. Setelah oversampling, data latih menjadi 101.914 baris dengan kelas yang seimbang (50.957 per kelas). Pemeriksaan akhir memastikan tidak ada missing value, kolom data latih sama dengan data uji, dan tidak ada baris yang muncul di keduanya.

Parameter di bagian awal notebook: label biner (`<30` hari = 1), kolom `medical_specialty` dan `payer_code` dipertahankan sebagai kategori "Unknown", ambang kolom obat hampir konstan 0,99, dan `RANDOM_STATE = 42`.

Asumsi yang perlu disebut:

1. `?` pada `diag_2` dan `diag_3` dianggap berarti pasien tidak punya diagnosis tambahan.
2. Urutan `encounter_id` dianggap urutan waktu saat mengambil kunjungan pertama tiap pasien. Dokumentasi UCI tidak menjamin hal ini.
3. Kode admisi dan pulang dikelompokkan menurut `IDS_mapping.csv`, dengan beberapa kode digabung ke "Lainnya".
4. Pengelompokan ICD-9 mengikuti pengelompokan yang lazim dipakai pada data ini. Rentang kodenya perlu dicek ke sumber sebelum dikutip.

## Cara menjalankan

Di Google Colab: klik badge "Buka di Colab" di atas (atau di bagian paling atas tiap notebook), lalu pilih Runtime, Run all. Di Colab folder `data` tidak ikut tersalin, jadi notebook mengunduh `diabetic_data.csv` dari repo ini, dan dari UCI bila unduhan itu gagal.

Di komputer sendiri (Python 3.10 atau lebih baru):

```bash
pip install -r requirements.txt
jupyter notebook notebooks/02_tugas2_praproses_diabetes130.ipynb
```

Pilih Run All. Notebook mencari `diabetic_data.csv` mulai dari folder kerja dan folder induknya, jadi tidak perlu mengubah path.

## Sitasi dan lisensi data

- Clore, J., Cios, K., DeShazo, J., & Strack, B. (2014). *Diabetes 130-US Hospitals for Years 1999-2008* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5230J (CC BY 4.0)
- Strack, B., DeShazo, J. P., Gennings, C., Olmo, J. L., Ventura, S., Cios, K. J., & Clore, J. N. (2014). Impact of HbA1c measurement on hospital readmission rates: analysis of 70,000 clinical database patient records. *BioMed Research International*, 2014, Article 781670. https://doi.org/10.1155/2014/781670
- van Walraven, C., Dhalla, I. A., Bell, C., Etchells, E., Stiell, I. G., Zarnke, K., Austin, P. C., & Forster, A. J. (2010). Derivation and validation of an index to predict early death or unplanned readmission after discharge from hospital to the community. *CMAJ*, 182(6), 551-557. https://doi.org/10.1503/cmaj.091117

Data asli tidak diubah dan dilindungi lisensi CC BY 4.0. Detail dan checksum ada di `data/README.md`.
