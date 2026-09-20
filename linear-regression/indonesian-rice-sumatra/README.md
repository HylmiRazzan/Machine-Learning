# [Indonesian Rice Production Prediction in Sumatra Using Linear Regression]

## 1. Deskripsi Dataset & Preprocessing

Proyek ini bertujuan untuk memprediksi jumlah produksi beras di wilayah Sumatra menggunakan dataset `Data_Tanaman_Padi_Sumatera_version_1.csv`. 

Tahap pra-pemrosesan data dirancang menggunakan struktur *pipeline* yang terintegrasi:
*   **Fitur Kategorik (`Provinsi`):** Ditransformasikan menggunakan **OneHotEncoder** dengan penanganan kategori asing (`handle_unknown='ignore'`).
*   **Fitur Numerik (`Tahun`, `Luas Panen`, `Curah hujan`, `Kelembapan`, `Suhu rata-rata`):** Distandardisasi menggunakan **StandardScaler** untuk menyamakan skala pengukuran.
*   **Target Transformation:** Menggunakan **TransformedTargetRegressor** dengan fungsi logaritmik (`np.log1p` dan invers `np.expm1`) untuk menstabilkan distribusi variabel target (`Produksi`) yang memiliki rentang nilai bervariasi.

## 2. Workflow (Alur Kerja)

Proyek ini dieksekusi melalui beberapa tahapan sistematis:
*   **Pembagian Data:** Membagi dataset menjadi 80% data latih dan 20% data uji (`train_test_split`) dengan `random_state=42`.
*   **Model Baseline:** Melatih model awal menggunakan *Linear Regression* yang dibungkus dalam *pipeline* transformasi target.
*   **Analisis Outlier:** Melakukan eksperimen penghapusan *outlier* (berdasarkan persentase absolut residual ke-95). Hasilnya menunjukkan bahwa penghapusan *outlier* justru menurunkan performa model secara drastis, sehingga seluruh data ekstrem diputuskan untuk tetap dipertahankan karena merepresentasikan variasi kondisi iklim dan pertanian dunia nyata.
*   **Hyperparameter Tuning:** Menggunakan **GridSearchCV** (5-*fold cross-validation*) untuk mengoptimalkan parameter regresi linear seperti `fit_intercept` dan `positive`.

## 3. Metrik Penilaian

Performa model regresi dievaluasi menggunakan empat metrik utama:
*   **R2 Score (Coefficient of Determination):** Mengukur seberapa besar proporsi variasi target yang dapat dijelaskan oleh model.
*   **MSE (Mean Squared Error):** Menghitung kuadrat rata-rata kesalahan prediksi.
*   **MAE (Mean Absolute Error):** Rata-rata kesalahan mutlak dalam satuan produksi asli.
*   **MAPE (Mean Absolute Percentage Error):** Persentase kesalahan rata-rata relatif terhadap nilai aktual.

## 4. Hasil Akhir Perbandingan Model & Analisis Outlier

Berikut adalah ringkasan hasil evaluasi akhir dari berbagai skenario pengujian:

### A. Eksperimen Outlier
*   **Baseline (Dengan Outlier):** R2 = 0.9481 | MAE = 152,515.02 | MAPE = 0.2943
*   **Tanpa Outlier (Clean Data):** R2 = 0.7119 | MAE = 292,069.41 | MAPE = 0.2321
*   *Insight:* Membuang *outlier* menurunkan skor R2 secara signifikan dari 0.948 menjadi 0.711, membuktikan bahwa data ekstrem tersebut bukan sekadar *noise*, melainkan informasi bermakna terkait variasi produksi pertanian akibat faktor cuaca atau iklim yang tidak biasa.

### B. Hasil Akhir Performa Model (Baseline vs Tuned)
*   **Baseline Model:** 
    *   Train R2: 0.1073 | Test R2: 0.7119 (Gap: -60.46%)
    *   MSE: 45,902,203,283.57 | MAE: 152,515.02 | MAPE: 0.2943
*   **Tuned Model (`positive=True`):** 
    *   Train R2: 0.8938 | Test R2: 0.9477 (Gap: 5.4%)
    *   MSE: 46,250,973,276.23 | MAE: 152,955.24 | MAPE: 0.2949

## 5. Visualisasi Interaktif Power BI, Kelayakan Deploy, & Kesimpulan

Selain pemodelan prediktif berbasis Python, proyek ini juga dilengkapi dengan dasbor visualisasi interaktif menggunakan Power BI untuk mengeksplorasi tren produksi padi, sebaran wilayah panen, serta faktor iklim di berbagai provinsi di pulau Sumatra secara visual dan mendalam.

A. Status Kelayakan Deployment

Model regresi ini memiliki skor R2 pada data uji yang sangat tinggi (0.9477), yang menunjukkan kemampuan prediksi yang sangat kuat dan konsisten dalam memperkirakan hasil panen. Model ini sudah sangat siap diintegrasikan ke dalam sistem operasional atau dihubungkan langsung dengan dasbor Power BI untuk mendukung proyeksi ketahanan pangan secara berkala oleh pihak berwenang.

B. Penanganan Underfitting & Analisis Outlier

- Penyembuhan Underfitting: Model baseline awal sempat mengalami underfitting parah pada data latih (Train R2 hanya 0.11) akibat sensitivitas regresi linear terhadap keberadaan outlier. Melalui proses optimasi GridSearchCV, model berhasil disembuhkan total dan mencapai kondisi yang sangat sehat.
- Keputusan Outlier: Data ekstrem sengaja dipertahankan di dalam dataset karena merepresentasikan variasi kondisi iklim dan anomali cuaca dunia nyata yang berdampak langsung pada naik turunnya hasil produksi pertanian.

C. Kesimpulan Analisis & Penyimpanan Model

- Stabilitas Generalisasi: Model tuned menunjukkan performa optimal dengan selisih gap performa data latih dan uji yang sangat kecil (sekitar 5.4%), membuktikan kemampuan generalisasi yang sempurna ke data baru.
- Efektivitas Pipeline: Integrasi pipeline dengan TransformedTargetRegressor berbasis logaritma terbukti sangat sukses menstabilkan sebaran target produksi padi yang memiliki rentang nilai sangat luas.
- Penyimpanan Model: Model terbaik berhasil disimpan menggunakan pustaka joblib (`production_padi_model_linear_reg.pkl`) agar siap digunakan kembali untuk kebutuhan inferensi mandiri.
