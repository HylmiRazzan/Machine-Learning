# [Jakarta Air Quality Analysis & Predictive Modeling]

## 1. Deskripsi Dataset & Feature Engineering

Proyek ini menganalisis Indeks Standar Pencemar Udara (ISPU) di DKI Jakarta menggunakan dataset `ispu_dki_all.csv`. Tahap pra-pemrosesan awal mencakup pembersihan data dengan menghapus kolom yang tidak relevan (`tanggal`, `max`, `critical`) serta menyaring kategori yang tidak valid atau memiliki jumlah sampel terlalu sedikit (seperti "TIDAK ADA DATA" dan "BERBAHAYA").

Untuk memperkaya informasi model, dilakukan rekayasa fitur (*Feature Engineering*):
*   **`gas_total`**: Akumulasi total konsentrasi gas polutan (`so2`, `co`, `o3`, `no2`).
*   **`pm10_co_ratio`**: Rasio perbandingan antara partikulat `pm10` terhadap karbon monoksida (`co`).
*   **`avg_pollutant`**: Rata-rata keseluruhan nilai polutan utama.

## 2. Workflow (Alur Kerja)

Proyek ini dibangun menggunakan *pipeline* *Machine Learning* yang terstruktur:
*   **Pembagian Data:** Membagi dataset menjadi 80% data latih dan 20% data uji dengan teknik *stratify* berdasarkan variabel target kategori ISPU (diubah ke bentuk ordinal integer 0 hingga 3).
*   **Preprocessing Pipeline:** Mengatasi *missing values* menggunakan imputasi median dan modus, menerapkan transformasi logaritmik (`np.log1p`) untuk menangani distribusi data yang miring (*skewed*), serta menormalisasi fitur numerik menggunakan **RobustScaler**.
*   **Reduksi Dimensi (PCA):** Menerapkan **Principal Component Analysis (PCA)** dengan ambang batas varians `0.95` untuk mereduksi dimensi fitur secara efisien.
*   **Pemodelan & Hyperparameter Tuning:** Menggunakan **Logistic Regression** (multiclass) dengan penanganan data tidak seimbang (`class_weight='balanced'`), serta dioptimasi menggunakan **GridSearchCV** dengan metrik evaluasi `f1_macro`.

## 3. Metrik Penilaian

Evaluasi model multikelas ini menggunakan beberapa metrik utama:
*   **Accuracy Score:** Mengukur persentase prediksi yang benar secara keseluruhan.
*   **Classification Report (Precision, Recall, F1-Score):** Menilai performa model secara terperinci pada tiap kategori tingkat polusi[cite: 1].
*   **ROC-AUC Score (OvR - Macro):** Mengukur kemampuan model dalam membedakan antar kelas secara keseluruhan[cite: 1].

## 4. Hasil Evaluasi Model (Tuned)

Berdasarkan hasil pencarian parameter terbaik menggunakan **GridSearchCV**, diperoleh konfigurasi parameter optimal:
*   **Parameter Terbaik:** `{'model__C': 1, 'model__class_weight': 'balanced', 'model__max_iter': 100, 'model__solver': 'lbfgs'}`[cite: 1]

Performa model pada data uji mencatatkan hasil sebagai berikut:
*   **Akurasi Keseluruhan:** 64.68% (0.65)
*   **ROC-AUC Score (Macro OvR):** 0.8855

## 5. Visualisasi Interaktif Power BI, Kelayakan Deploy, & Kesimpulan

Selain pemodelan prediktif berbasis Python, proyek ini juga dilengkapi dengan dasbor visualisasi interaktif menggunakan **Power BI** untuk mengeksplorasi tren kualitas udara harian, sebaran polutan berdasarkan stasiun pemantauan, serta perbandingan tingkat polusi secara visual di wilayah DKI Jakarta.

### A. Status Kelayakan Deployment
Model ini memiliki skor ROC-AUC yang sangat baik (0.8855), yang menunjukkan bahwa model memiliki kemampuan peringkat (*ranking ability*) yang kuat dalam membedakan tingkat polusi udara. Namun, dengan akurasi di angka 64.68%, model ini **belum sepenuhnya siap untuk sistem otomatisasi penuh berisiko tinggi**, melainkan lebih ideal diposisikan sebagai **alat bantu analitis pendukung keputusan** atau diintegrasikan langsung dengan dasbor Power BI untuk pemantauan tren harian oleh tim terkait.

### B. Perbedaan AUC dan Accuracy
*   **Accuracy (Akurasi):** Menghitung total tebakan yang benar dibagi dengan seluruh data uji. Metrik ini sangat intuitif, tetapi bisa sangat menyesatkan jika distribusi data tidak seimbang (*imbalanced data*), karena model yang hanya menebak kelas mayoritas pun bisa mendapat akurasi tinggi.
*   **AUC (Area Under the Curve / ROC-AUC):** Mengukur probabilitas kemampuan model dalam membedakan antar kelas secara akurat di berbagai tingkat ambang batas (*threshold*). Nilai AUC tidak terpengaruh oleh bias kelas mayoritas, sehingga sangat handal untuk mengukur kualitas pemisahan model pada kasus multikelas.

### C. Kesimpulan Analisis
*   **Efektivitas Pipeline & PCA:** Kombinasi transformasi logaritmik, penskalaan *RobustScaler*, dan reduksi dimensi PCA berhasil menyederhanakan kompleksitas data tinggi secara efektif.
*   **Penanganan Kelas Tidak Seimbang:** Penggunaan `class_weight='balanced'` membantu meningkatkan nilai *Recall* pada kelas-kelas minoritas sehingga model mampu mendeteksi kondisi ekstrem dengan lebih baik.
*   **Potensi Integrasi Solusi:** Integrasi antara model prediktif berbasis *Machine Learning* dan dasbor interaktif Power BI memberikan solusi komprehensif bagi pemangku kepentingan untuk memantau serta menganalisis tingkat polusi udara di Jakarta secara menyeluruh.

### Classification Report
*   **Kelas 0:** BAIK
*   **Kelas 1:** SEDANG
*   **Kelas 2:** TIDAK SEHAT
*   **Kelas 3:** SANGAT TIDAK SEHAT
```text
              precision    recall  f1-score   support

           0       0.36      0.92      0.52        63
           1       0.79      0.59      0.68       638
           2       0.60      0.66      0.63       365
           3       0.57      0.95      0.71        41

    accuracy                           0.65      1107
   macro avg       0.58      0.78      0.63      1107
weighted avg       0.70      0.65      0.65      1107
