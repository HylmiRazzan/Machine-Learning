# [Housing Price Prediction Using Tree-Based Regressors]

## 1. Deskripsi Dataset & Feature Engineering

Proyek ini menggunakan dataset harga rumah (`kc_house_data.csv`). Untuk memaksimalkan performa model regresi, dilakukan pembersihan data awal dengan membuang kolom yang tidak relevan (`id` dan `date`), serta melakukan rekayasa fitur (*Feature Engineering*) secara ekstensif guna mengekstrak informasi yang lebih mendalam:

*   **`house_age`**: Menghitung umur rumah berdasarkan tahun pembuatan (`yr_built`) dikurangkan dari tahun referensi (2015).
*   **`is_renovated`**: Variabel biner (1/0) untuk menandai apakah rumah pernah direnovasi atau tidak berdasarkan `yr_renovated`.
*   **`effective_age`**: Umur efektif rumah dengan mengambil nilai maksimum antara tahun pembuatan dan tahun renovasi terakhir.
*   **`has_basement`**: Menandai keberadaan ruang bawah tanah (`sqft_basement > 0`).
*   **`living_lot_ratio`**: Rasio perbandingan antara luas bangunan (`sqft_living`) dan luas tanah (`sqft_lot`).
*   **`sqft_per_floor`**: Luas area per lantai berdasarkan jumlah lantai (`floors`).
*   **`bath_bed_ratio`**: Rasio jumlah kamar mandi terhadap jumlah kamar tidur.
*   **`dist_to_center`**: Menghitung jarak spasial Euclidean rumah terhadap titik tengah koordinat wilayah (*median latitude* dan *longitude*).

Setelah proses rekayasa fitur selesai, kolom asli yang sudah direpresentasikan ulang (`yr_built`, `yr_renovated`) dihapus untuk menghindari multikolinearitas.

## 2. Workflow (Alur Kerja)

Proyek ini dibangun secara terstruktur menggunakan alur kerja berbasis *pipeline* untuk memastikan tidak terjadi *data leakage*:

*   **Pembagian Data (Train-Test Split):** Membagi dataset dengan proporsi 80% data latih dan 20% data uji menggunakan `random_state=42`.
*   **Preprocessing Pipeline:** Seluruh kolom numerik distandardisasi menggunakan **RobustScaler** di dalam `ColumnTransformer` agar tahan terhadap keberadaan *outlier*.
*   **Pemodelan & Komparasi Algoritma:** Mengeksplorasi empat keluarga algoritma *tree-based* regressor, yaitu **Decision Tree (DT)**, **Random Forest (RF)**, **Gradient Boosting Machine (GBM)**, dan **XGBoost (XGB)**.
*   **Hyperparameter Tuning:** Masing-masing model dievaluasi dalam dua kondisi, yakni model *Baseline* (parameter default) dan model *Tuned* yang dioptimasi menggunakan **GridSearchCV** (dengan 3-fold cross-validation dan *scoring* berbasis $R^2$).
*   **Evaluasi Model:** Setiap model diuji menggunakan fungsi evaluasi khusus untuk memantau skor $R^2$, MAE, dan MAPE pada data uji.
*   **Model Interpretability:** Menganalisis fitur paling berpengaruh pada model terbaik menggunakan **Feature Importance** (grafik batang) dan **SHAP Summary Plot** untuk melihat arah dampak setiap fitur terhadap prediksi harga rumah.

## 3. Metrik Penilaian

Evaluasi performa model regresi ini menggunakan tiga metrik utama:
*   **$R^2$ Score (Coefficient of Determination):** Mengukur seberapa baik variasi dari variabel target (harga rumah) dapat dijelaskan oleh model.
*   **MAE (Mean Absolute Error):** Rata-rata kesalahan mutlak dalam satuan mata uang, memberikan gambaran besar kesalahan prediksi secara langsung.
*   **MAPE (Mean Absolute Percentage Error):** Mengukur persentase kesalahan rata-rata relatif terhadap nilai aktual.

## 4. Hasil Perbandingan Model (Baseline vs Tuned)

Berikut adalah ringkasan hasil evaluasi dari keempat algoritma yang telah diuji:

| Model | Kondisi | $R^2$ Score | MAE | MAPE | Catatan |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Decision Tree** | Baseline | 0.6902 | 106,527.09 | 0.1872 | Overfit |
| | Tuned | 0.6381 | 124,010.98 | 0.2213 | - |
| **Random Forest** | Baseline | 0.8614 | 72,596.50 | 0.1310 | Overfit |
| | Tuned | 0.7845 | 90,655.90 | 0.1602 | - |
| **Gradient Boosting** | Baseline | 0.8638 | 80,341.79 | 0.1501 | - |
| | Tuned | **0.8786** | **75,477.97** | **0.1397** | - |
| **XGBoost** | Baseline | 0.8766 | 71,258.55 | 0.1304 | Overfit |
| | Tuned | **0.8885** | **73,428.44** | **0.1374** | **Performa Terbaik** |

### 5. Analisis Faktor Harga dan Kelayakan Model

Berdasarkan hasil pengujian mendalam, berikut adalah rincian kesimpulan dari performa model **XGBoost Tuned**:

#### A. Faktor Utama Penentu Harga Rumah
*   **Andalan Fitur:** Model secara konsisten mengandalkan tingkat kualitas bangunan (*grade*) dan luas bangunan (*sqft_living*) sebagai pertimbangan utama dalam membuat estimasi nilai.
*   **Pengaruh Lokasi:** Faktor letak geografis (koordinat lintang atau *latitude*) terbukti memberikan pengaruh paling signifikan dalam mengubah nominal harga akhir sebuah rumah.
*   **Kesesuaian Logika Bisnis:** Model bekerja secara rasional dengan mematok estimasi harga yang lebih tinggi pada rumah yang memiliki kualitas bangunan prima, ukuran luas, berada dekat dengan pusat kota, atau memiliki akses ke tepian air (*waterfront*).

#### B. Evaluasi Kelayakan Model untuk Produksi
*   **Bebas Overfitting:** Selisih skor $R^2$ antara data latih dan data uji tercatat hanya sebesar 2.37% (berada jauh di bawah batas aman 5%), yang menandakan model sangat stabil dan aman digunakan.
*   **Tingkat Akurasi Tinggi:** Perolehan skor $R^2$ pada data uji mencapai 88.85%, menunjukkan kemampuan prediksi yang sangat andal dalam mengenali pola harga pasar.
*   **Toleransi Kesalahan Wajar:** Nilai *Mean Absolute Percentage Error* (MAPE) berada di angka 13.74%. Angka ini sangat normal dan masuk dalam kategori sangat diterima untuk standar prediksi industri properti (*real estate*), di mana kisaran 10% sampai 15% dianggap sebagai batas error yang ideal.
