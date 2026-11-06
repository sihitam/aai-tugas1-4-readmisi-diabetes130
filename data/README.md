# Data

Sumber: UCI Machine Learning Repository, *Diabetes 130-US Hospitals for Years 1999-2008*.
DOI: https://doi.org/10.24432/C5230J
Lisensi: CC BY 4.0. Sitasi: Clore, J., Cios, K., DeShazo, J., & Strack, B. (2014).

Berkas di `raw/` adalah salinan asli dari UCI dan tidak diubah:

| Berkas | Ukuran (byte) | SHA-256 |
|---|---|---|
| diabetic_data.csv | 19.159.383 | 0689e7ec031237dc63031b938805c48377748761a3b26acab621567afa24df97 |
| IDS_mapping.csv | 2.547 | f1bb82b471cb34649352597572c9b1fb00bd27f77b9f5a22a03dc3eb1039749e |

Cek di terminal: `sha256sum data/raw/*` (Linux, macOS) atau `Get-FileHash data\raw\* -Algorithm SHA256` (PowerShell).

Hasil praproses ditulis notebook ke folder `hasil_praproses/` dan tidak disimpan di repo karena besar. Jalankan ulang notebook 01 untuk membuatnya.
