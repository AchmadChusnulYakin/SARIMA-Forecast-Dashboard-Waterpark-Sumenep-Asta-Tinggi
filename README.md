# 📊 SARIMA Forecast Dashboard — Waterpark Sumenep

Dashboard interaktif berbasis **Streamlit** untuk menampilkan hasil pemodelan dan peramalan jumlah kunjungan wisatawan pada **Waterpark Sumenep** menggunakan model **SARIMA**.

Project ini merupakan implementasi **baseline forecasting** dalam kegiatan **MBKM Skema Penelitian/Riset** dan digunakan untuk menyajikan hasil pengolahan data, evaluasi model, skenario windowing, serta forecast masa depan.

---

## 📌 Ringkasan Penelitian

Dataset yang digunakan terdiri dari **108 observasi** jumlah kunjungan wisatawan dengan periode data **Januari 2015 sampai Desember 2024**. Data berbentuk runtun waktu bulanan dengan variabel utama `Tanggal` dan `Jumlah_Kunjungan`.

Pada tahap preprocessing ditemukan **3 nilai nol** pada jumlah kunjungan. Sesuai arahan pembimbing, nilai tersebut diperlakukan sebagai nilai yang perlu ditangani, kemudian diubah menjadi `NaN` dan diestimasi menggunakan **Interpolasi Linear**. Hasil interpolasi menggantikan tiga nilai nol pada data.
**Tahun 2020 tidak tersedia pada dataset mentah dan tidak dibuat atau diisi secara sintetis.** Oleh karena itu, penelitian tetap menggunakan **108 observasi aktual**. Ketidaktersediaan tahun tersebut menyebabkan adanya gap kalender pada visualisasi, tetapi tidak dilakukan pembuatan data sintetis untuk mengisinya.

### Konfigurasi utama

| Komponen                 |                   Nilai |
| ------------------------ | ----------------------: |
| Jumlah observasi         |                     108 |
| Nilai nol yang ditangani |                       3 |
| Training                 |      75 observasi (70%) |
| Testing                  |      33 observasi (30%) |
| Window 6 → 1             |           102 observasi |
| Window 12 → 1            |            96 observasi |
| Model                    | SARIMA(1,0,1)(0,0,0,12) |
| Periode musiman          |                12 bulan |
| Evaluasi                 |     MAE, RMSE, MAPE, R² |

Data training dan testing dibagi secara kronologis dengan proporsi 70:30, menghasilkan **75 observasi training** dan **33 observasi testing**.

Karena `P=D=Q=0`, model **SARIMA(1,0,1)(0,0,0,12)** tidak memiliki komponen AR, differencing, maupun MA musiman yang aktif. Secara struktur model tersebut setara dengan **ARIMA(1,0,1)**, dengan periode musiman `s=12` tetap digunakan dalam kerangka SARIMA.

---

## 🔬 Alur Pengolahan

```text
Dataset Waterpark Sumenep
        ↓
Pemeriksaan Struktur Data
        ↓
Pemeriksaan Nilai 0
        ↓
Interpolasi Linear
        ↓
Uji Stasioneritas ADF
        ↓
Identifikasi ACF & PACF
        ↓
Penentuan Parameter SARIMA
        ↓
Pembagian Training / Testing
        ↓
Pemodelan SARIMA
        ↓
Evaluasi MAE, RMSE, MAPE, R²
        ↓
Rolling Forecast Window 6 → 1 & 12 → 1
        ↓
Ekspor Hasil ke CSV
        ↓
Dashboard Streamlit
```

---

## 📈 Penentuan Parameter SARIMA

Hasil uji Augmented Dickey-Fuller (ADF) pada data asli menunjukkan bahwa data telah stasioner dengan:

* ADF Statistic = **-4,8466**
* p-value = **0,00004425**

Karena p-value < 0,05, data asli dinyatakan stasioner sehingga parameter **d = 0**.

Hasil identifikasi ACF dan PACF pada lag non-musiman digunakan untuk menentukan kandidat `q` dan `p`, sedangkan pemeriksaan lag musiman 12, 24, dan 36 digunakan untuk mengevaluasi komponen musiman. Hasil identifikasi menghasilkan konfigurasi:

```text
SARIMA(1,0,1)(0,0,0,12)
```

Parameter model:

| Parameter | Nilai |
| --------- | ----: |
| p         |     1 |
| d         |     0 |
| q         |     1 |
| P         |     0 |
| D         |     0 |
| Q         |     0 |
| s         |    12 |

Pada pengujian tambahan *seasonal differencing* lag 12, diperoleh ADF Statistic **-2,4764** dan p-value **0,1213**. Karena data asli telah stasioner dan *seasonal differencing* tidak diterapkan pada model, parameter **D tetap ditetapkan 0**.

## 📊 Hasil Evaluasi

| Skenario      |       MAE |       RMSE |      MAPE |      R² |
| ------------- | --------: | ---------: | --------: | ------: |
| Training      | 9558.5038 | 15345.9657 | 148.6090% | -0.2835 |
| Testing       | 4743.1135 |  6402.1999 |  53.3402% | -0.5930 |
| Window 6 → 1  | 8624.3091 | 14806.8334 | 161.4247% | -1.7296 |
| Window 12 → 1 | 6381.7087 |  8786.6993 |  72.0655% | -0.0466 |

Hasil tersebut merupakan performa **baseline SARIMA** pada dataset Waterpark Sumenep. Nilai pada tabel diambil dari hasil evaluasi training, testing, rolling forecast window 6, dan rolling forecast window 12 pada notebook.
Pada skenario rolling forecast, **window 12 → 1** menghasilkan MAE, RMSE, dan MAPE yang lebih rendah dibandingkan window 6 → 1, serta nilai R² yang lebih tinggi. Hasil window 12 tercatat sebagai skenario dengan RMSE terendah di antara dua skenario rolling forecast.

---

## 🖥️ Fitur Dashboard

### 🏠 Beranda

Menampilkan ringkasan dataset, jumlah observasi training/testing, periode data, serta visualisasi historis jumlah kunjungan.

### 📈 Data Historis

Menampilkan tabel dan visualisasi data kunjungan setelah proses preprocessing dan interpolasi linear.

### 🔮 Hasil SARIMA

Menampilkan perbandingan nilai **Aktual** dan **Prediksi** pada data training dan testing menggunakan model SARIMA.

### 🪟 Windowing

Menampilkan hasil *rolling forecast* dengan dua skenario:

* **Window 6 bulan → 1 langkah prediksi**
* **Window 12 bulan → 1 langkah prediksi**

Window 6 menghasilkan 102 observasi prediksi, sedangkan window 12 menghasilkan 96 observasi prediksi.

### 📅 Forecast Masa Depan

Menghasilkan prediksi beberapa periode setelah observasi historis terakhir menggunakan model SARIMA, termasuk **interval prediksi 95%**.

### 📊 Evaluasi Model

Menampilkan nilai:

* MAE
* RMSE
* MAPE
* R²

dalam bentuk tabel dan visualisasi.

### ⚙️ Konfigurasi Model

Menampilkan struktur model:

```text
SARIMA(1,0,1)(0,0,0,12)
```

---

## 📁 Struktur Repository

```text
SARIMA-Forecast-Dashboard-Waterpark-Sumenep-Asta-Tinggi/

├── app.py
├── requirements.txt
├── README.md
├── .gitignore
└── Streamlit Data/
    ├── data_historis.csv
    ├── hasil_training.csv
    ├── hasil_testing.csv
    ├── window_6.csv
    ├── window_12.csv
    ├── metrics.csv
    └── konfigurasi_model.csv
```

> Nama folder data harus sama dengan path yang digunakan pada `app.py`, yaitu `Streamlit Data`.

---

## 📦 File Data

* `data_historis.csv` — Data historis setelah preprocessing dan interpolasi linear.
* `hasil_training.csv` — Aktual dan prediksi pada 75 observasi training.
* `hasil_testing.csv` — Aktual dan prediksi pada 33 observasi testing.
* `window_6.csv` — Hasil rolling forecast dengan histori 6 observasi untuk 1 langkah prediksi.
* `window_12.csv` — Hasil rolling forecast dengan histori 12 observasi untuk 1 langkah prediksi.
* `metrics.csv` — Nilai MAE, RMSE, MAPE, dan R² dari seluruh skenario evaluasi.
* `konfigurasi_model.csv` — Konfigurasi model SARIMA yang digunakan.

---

## 🛠️ Teknologi

* Python
* Pandas
* NumPy
* Statsmodels
* Plotly
* Streamlit

Model:

```text
SARIMA(1,0,1)(0,0,0,12)
```

---

## ▶️ Menjalankan Secara Lokal

Install dependency:

```bash
pip install -r requirements.txt
```

Jalankan dashboard:

```bash
streamlit run app.py
```

Dashboard biasanya tersedia di:

```text
http://localhost:8501
```

---

## ☁️ Deployment

Project disiapkan untuk deployment menggunakan **Streamlit Community Cloud** melalui repository GitHub.

File entrypoint:

```text
app.py
```

Dependency:

```text
requirements.txt
```

Folder data:

```text
Streamlit Data/
```

Pastikan seluruh **7 CSV final** tersedia di repository sebelum deployment.

---

## 🧪 Catatan Metodologis

Dashboard ini merupakan implementasi **baseline SARIMA** untuk peramalan kunjungan wisatawan pada dataset Waterpark Sumenep.

Hasil **Window 6 → 1** dan **Window 12 → 1** merupakan skenario evaluasi *rolling forecast* dan berbeda dari fitur **Forecast Masa Depan**, yang digunakan untuk menghasilkan prediksi setelah periode historis terakhir.

Nilai nol pada dataset ditangani menggunakan **Interpolasi Linear**. Pada dataset yang digunakan, terdapat **3 nilai nol** yang diganti dengan hasil interpolasi.

Tahun **2020 tidak tersedia pada dataset mentah**. Tahun tersebut tidak dibuat, tidak diestimasi, dan tidak diisi menggunakan data sintetis. Dengan demikian, gap kalender pada visualisasi tetap dipertahankan sesuai kondisi dataset asli.

Parameter model yang digunakan adalah:

```text
(p,d,q)(P,D,Q,12)
= (1,0,1)(0,0,0,12)
```

Model SARIMA tersebut digunakan sebagai **baseline forecasting** untuk memberikan acuan performa sebelum diterapkan pendekatan yang lebih kompleks pada penelitian berikutnya.

---

## 👤 Project

**SARIMA Forecast Dashboard — Waterpark Sumenep Asta Tinggi**

Bagian dari kegiatan:

**MBKM Skema Penelitian/Riset — Pemodelan SARIMA untuk Peramalan Kunjungan Wisatawan Berbasis Data Runtun Waktu**

Topik penelitian:

**Pengembangan Model Hibrida CEEMDAN–GRU dengan Optimasi Bayesian untuk Peramalan Kunjungan Wisatawan dalam Mendukung Pengelolaan Pariwisata Berkelanjutan**

Pada tahap ini, SARIMA digunakan sebagai **model pembanding (baseline)** untuk evaluasi terhadap pendekatan hibrida pada tahap penelitian selanjutnya.

---

## 📄 Lisensi

Project ini dibuat untuk kebutuhan akademik dan dokumentasi penelitian.