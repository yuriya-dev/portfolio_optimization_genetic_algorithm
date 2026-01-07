# 📂 Dokumentasi Data Input: Optimasi Portofolio

Dokumen ini menjelaskan tiga komponen data utama yang digunakan sebagai **"bahan bakar" Algoritma Genetika (Genetic Algorithm / GA)** untuk menentukan komposisi portofolio saham terbaik, serta penjelasan konsep GA itu sendiri.

---

## 1. Data Harga Penutupan

**File:** `harga_penutupan_saham.csv`

File ini berisi data mentah pergerakan harga saham historis. Nilai yang digunakan biasanya adalah **Adjusted Close**, yaitu harga penutupan yang telah disesuaikan dengan dividen atau pemecahan saham.

### Contoh Data

| Date       | ADRO.JK | ANTM.JK | BBCA.JK | BBNI.JK | PTBA.JK |
| ---------- | ------- | ------- | ------- | ------- | ------- |
| 2022-01-03 | 840.73  | 1944.58 | 6573.09 | 2772.39 | 1227.65 |
| ...        | ...     | ...     | ...     | ...     | ...     |

### Analisis & Konteks

* **Fungsi:** Sebagai data dasar (*raw material*). Algoritma tidak menggunakan nominal harga secara langsung, melainkan **perubahan harganya**.
* **Sektor Saham:**

  * **ADRO & PTBA** → Energi (Batu Bara), cenderung siklikal dan fluktuatif
  * **BBCA & BBNI** → Perbankan (*Blue Chip*), relatif stabil dan defensif
  * **ANTM** → Mineral (Emas/Nikel), dipengaruhi harga komoditas global

---

## 2. Data Return Harian

**File:** `return_harian_saham.csv`

Data ini merepresentasikan **persentase keuntungan atau kerugian harian** investor.

### Rumus Return Harian

$$
\text{Return} = \frac{Harga_{hari_ini} - Harga_{kemarin}}{Harga_{kemarin}}
$$

### Contoh Data

| Date       | ADRO.JK | ANTM.JK | BBCA.JK | BBNI.JK | PTBA.JK |
| ---------- | ------- | ------- | ------- | ------- | ------- |
| 2022-01-04 | -0.0295 | -0.0171 | 0.0102  | 0.0446  | 0.0112  |
| ...        | ...     | ...     | ...     | ...     | ...     |

### Analisis & Arti Angka

* **-0.0295 (ADRO):** Harga ADRO turun 2.95%
* **0.0446 (BBNI):** Harga BBNI naik 4.46%

### Peran dalam Algoritma Genetika

* Rata-rata return (disetahunkan) menjadi **Expected Return**
* GA mencari kombinasi bobot saham dengan **return portofolio tertinggi**

---

## 3. Matriks Kovarians

**File:** `matriks_kovarians.csv`

Matriks ini merupakan komponen paling krusial dalam **manajemen risiko**. Nilainya menunjukkan hubungan pergerakan antar saham yang telah disetahunkan (× 252 hari bursa).

### Contoh Data

| Ticker  | ADRO.JK | ANTM.JK | BBCA.JK | BBNI.JK | PTBA.JK |
| ------- | ------- | ------- | ------- | ------- | ------- |
| ADRO.JK | 0.1920  | 0.0512  | 0.0187  | 0.0187  | 0.0689  |
| ANTM.JK | 0.0512  | 0.1874  | 0.0099  | 0.0113  | 0.0375  |
| BBCA.JK | 0.0187  | 0.0099  | 0.0538  | 0.0350  | 0.0117  |
| BBNI.JK | 0.0187  | 0.0113  | 0.0350  | 0.0882  | 0.0191  |
| PTBA.JK | 0.0689  | 0.0375  | 0.0117  | 0.0191  | 0.1093  |

### Cara Membaca Matriks

#### A. Diagonal Utama (Risiko Individu)

* Nilai diagonal adalah **varians** (risiko masing-masing saham)
* **ADRO (0.192) & ANTM (0.187):** Risiko tinggi (*high risk*)
* **BBCA (0.053):** Risiko rendah (*low risk*), berperan sebagai penyeimbang portofolio

#### B. Non-Diagonal (Hubungan Antar Saham)

* Nilai di luar diagonal adalah **kovarians**
* **ADRO vs PTBA (0.0689):** Korelasi positif tinggi → diversifikasi lemah
* **BBCA vs ANTM (0.0099):** Hampir tidak berkorelasi → diversifikasi sangat baik

---

## 4. Sintesis: Cara GA Menggunakan Data

Saat menghitung **fitness** sebuah portofolio, GA melakukan langkah berikut:

1. **Ambil Gen (Bobot)**
   Contoh: `{ADRO: 50%, BBCA: 50%}`
2. **Hitung Return** menggunakan rata-rata return harian
3. **Hitung Risiko** menggunakan matriks kovarians

### Keputusan

* Korelasi ADRO–BBCA rendah → risiko gabungan menurun
* Portofolio dinilai **efisien**
* GA memberikan **skor fitness tinggi**

---

## 5. Analogi Biologi dalam Algoritma Genetika

GA mengadaptasi konsep evolusi biologi ke dalam pemrograman optimasi.

### Tabel Perbandingan

| Konsep Biologi | Dalam GA        | Representasi Data |
| -------------- | --------------- | ----------------- |
| Gen            | Bobot saham     | Float (0–1)       |
| Kromosom       | Portofolio      | Array NumPy       |
| Individu       | Solusi kandidat | Kromosom          |
| Populasi       | Kumpulan solusi | List individu     |
| Fitness        | Skor kualitas   | Return − Risiko   |

---

## Penjelasan Detail Konsep

### A. Gen

* Bobot alokasi dana satu saham
* Nilai antara **0.0 – 1.0**
* Contoh: BBCA = 30% → gen = `0.30`

### B. Kromosom & Individu

* Gabungan gen membentuk satu strategi portofolio
* Struktur:

  ```
  [ADRO, ANTM, BBCA, BBNI, PTBA]
  ```
* Contoh:

  ```
  [0.10, 0.20, 0.30, 0.25, 0.15]
  ```

### C. Populasi

* Kumpulan individu dalam satu generasi
* Contoh: `POP_SIZE = 100`

### D. Fitness (Fungsi Kelayakan)

$$
\text{Fitness} = \text{Return} - (0.5 \times \text{Risk})
$$

**Simulasi:**

* Individu Agresif → Skor = 7.5
* Individu Defensif → Skor = 10.0

➡️ **Individu Defensif menang**

### E. Generasi & Alur Evolusi

1. Inisialisasi populasi acak
2. Evaluasi fitness
3. Seleksi induk terbaik
4. Crossover
5. Mutasi kecil
6. Iterasi hingga solusi optimal
