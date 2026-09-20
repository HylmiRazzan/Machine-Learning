# [EV Battery Failure Classification Using Tree-Based Models]

## 1. Deskripsi Dataset & Feature Engineering

Proyek ini bertujuan untuk melakukan klasifikasi biner guna mendeteksi potensi kegagalan pada baterai kendaraan listrik (*Electric Vehicle* / EV) melalui variabel target `battery_failure` (1 = *failure*, 0 = *no failure*). 

Tahap awal proyek melibatkan pembersihan data (*data cleaning*) dengan memeriksa persentase nilai kosong serta menghapus kolom ID dan atribut yang tidak relevan (seperti `vehicle_id`, `battery_serial`, `manufacturing_year`, dan beberapa atribut suhu yang berlebihan). Selain itu, dilakukan rekayasa fitur (*Feature Engineering*) untuk memperkaya informasi model:
*   **`degradation_rate`**: Rasio penurunan kapasitas baterai terhadap jumlah siklus penggunaan (`capacity_loss_percent` dibagi `cycle_count`).
*   **`thermal_charging_stress`**: Metrik tekanan termal saat pengisian daya, yang diperoleh dari perkalian rasio pengisian cepat (`fast_charge_ratio`) dengan suhu maksimum sel (`cell_temperature_max`).

## 2. Workflow (Alur Kerja)

Proyek ini dibangun secara sistematis menggunakan pendekatan berbasis *pipeline*:
*   **Pembagian Data (Train-Test Split):** Membagi data dengan proporsi 80% pelatihan dan 20% pengujian, serta menerapkan parameter `stratify=y` untuk menjaga proporsi kelas target yang tidak seimbang di kedua bagian data.
*   **Preprocessing Pipeline:** Menangani nilai kosong (*missing values*) secara terpisah menggunakan imputasi median untuk kolom numerik dengan *outlier*, imputasi rata-rata untuk kolom tanpa *outlier*, serta penerapan **OneHotEncoder** untuk seluruh fitur kategorik melalui `ColumnTransformer`.
*   **Komparasi Model & Hyperparameter Tuning:** Melatih empat algoritma *tree-based* secara independen, yaitu **Decision Tree**, **Random Forest**, **XGBoost**, dan **LightGBM**. Proses pencarian parameter terbaik dilakukan menggunakan **RandomizedSearchCV** dengan 3-*fold cross-validation* berdasarkan skor **ROC-AUC**.
*   **Evaluasi Model:** Masing-masing model dievaluasi menggunakan *Classification Report* dan *Learning Curve* untuk memantau stabilitas performa.
*   **Penyimpanan Model Final:** Model terbaik yang terpilih (**XGBoost**) diekspor menggunakan pustaka `joblib` ke dalam format file `.pkl` agar siap diintegrasikan.

## 3. Penanganan Imbalanced Data & Metrik Penilaian

Dataset pada proyek ini memiliki karakteristik kelas target yang sangat tidak seimbang (*imbalanced dataset*):
*   **Strategi Penanganan:** Untuk mencegah model menjadi bias terhadap kelas mayoritas, diterapkan teknik *stratification* saat pembagian data, serta pemanfaatan parameter pembobotan kelas (`class_weight='balanced'` pada Decision Tree, Random Forest, dan LightGBM; serta `scale_pos_weight=ratio` pada XGBoost).
*   **Fokus Metrik:** Evaluasi performa sangat menitikberatkan pada metrik **Recall** untuk kelas minoritas (Kelas 1 / *Failure*). Dalam kasus bisnis ini, melewatkan deteksi kerusakan baterai (*false negative*) jauh lebih berbahaya daripada memunculkan peringatan palsu (*false positive*), sehingga keselamatan operasional menjadi prioritas utama.

## 4. Hasil Komparasi Model & Analisis Recall

Berikut adalah ringkasan performa metrik *Recall* (terutama untuk kelas kegagalan baterai / Kelas 1) dari masing-masing model sebelum model terbaik dipilih:

*   **Decision Tree:** Berhasil menangkap 94% dari total kasus kegagalan aktual (*Recall* Kelas 1 = 0.94), namun memiliki tingkat *precision* yang masih rendah untuk alarm palsu.
*   **Random Forest:** Menunjukkan peningkatan performa dengan nilai *Recall* Kelas 1 mencapai 95%, memberikan stabilitas yang lebih baik dibanding Decision Tree.
*   **LightGBM:** Mencatatkan performa *Recall* Kelas 1 yang sangat tinggi yaitu 96%, serta akurasi keseluruhan yang menyentuh angka 94%.

### XGBoost (Model Terbaik)
Setelah melalui proses optimasi parameter, **XGBoost** terpilih sebagai model paling optimal karena tidak hanya mempertahankan *Recall* tinggi pada kelas minoritas, tetapi juga memiliki tingkat *precision* dan keseimbangan evaluasi yang paling unggul.

## 5. Analisis Kesimpulan

Berdasarkan hasil eksperimen dan evaluasi yang divisualisasikan melalui kurva pembelajaran (*learning curve*), beberapa poin kesimpulan utama dapat ditarik:

*   **Solusi Data Tidak Seimbang:** Variabel target `battery_failure` berhasil ditangani dengan sangat efektif melalui penerapan teknik *stratify* saat pembagian data, serta penggunaan parameter pembobotan kelas (`class_weight` dan `scale_pos_weight`) guna memberikan penalti yang adil selama proses pelatihan model.
*   **Prioritas Metrik Recall:** Fokus utama diarahkan pada peningkatan nilai *Recall* untuk kelas minoritas (Kelas 1). Dalam skenario deteksi kerusakan komponen kendaraan listrik, melewatkan tanda-tanda kegagalan jauh lebih berisiko dibanding mendeteksi alarm palsu, sehingga mitigasi risiko dapat berjalan optimal.
*   **Kestabilan dan Kesehatan Model:** Berdasarkan analisis kurva pembelajaran, meskipun pola skor tampak rapat, selisih performa antara data pelatihan dan data validasi tergolong sangat kecil. Hal ini mengonfirmasi bahwa model berada dalam kondisi yang sangat sehat dan terhindar dari gejala *overfitting*.
*   **Pemilihan Model Final:** Di antara seluruh algoritma yang diuji, keluarga model *boosting* seperti **XGBoost** dan **LightGBM** mencatatkan hasil paling unggul. Karena XGBoost menunjukkan performa keseluruhan yang sedikit lebih superior dengan tingkat kesalahan alarm palsu yang lebih minimal, model tersebut resmi dipilih dan disimpan sebagai solusi prediktif final.

Berikut adalah rincian *Classification Report* dari XGBoost pada data uji (*test set*)
*   **Parameter Terbaik:** `{'classifier__subsample': 0.8, 'classifier__n_estimators': 100, 'classifier__max_depth': 4, 'classifier__learning_rate': 0.1}`

```text
Classification Report untuk XGBoost:
              precision    recall  f1-score   support

           0       1.00      0.94      0.97     36016
           1       0.63      0.96      0.76      3984

    accuracy                           0.94     40000
   macro avg       0.81      0.95      0.86     40000
weighted avg       0.96      0.94      0.94     40000
