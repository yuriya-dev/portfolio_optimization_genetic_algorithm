# 📂 Dokumentasi Data Input: Optimasi Portofolio

Dokumen ini menjelaskan tiga komponen data utama yang digunakan sebagai **"bahan bakar" Algoritma Genetika** untuk menentukan komposisi portofolio saham terbaik, serta penjelasan mengenai konsep **Algoritma Genetika (Genetic Algorithm / GA)** itu sendiri.

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

  * **ADRO & PTBA** → Sektor Energi (Batu Bara), cenderung siklikal dan fluktuatif.
  * **BBCA & BBNI** → Sektor Perbankan (Blue Chip), relatif stabil dan defensif.
  * **ANTM** → Sektor Mineral (Emas/Nikel), dipengaruhi harga komoditas global.

---

## 2. Data Return Harian

**File:** `return_harian_saham.csv`

Data ini merepresentasikan **persentase keuntungan atau kerugian harian** dari masing-masing saham.

### Rumus

[
Return = \frac{Harga_{hari_ini} - Harga_{kemarin}}{Harga_{kemarin}}
]

### Contoh Data

| Date       | ADRO.JK | ANTM.JK | BBCA.JK | BBNI.JK | PTBA.JK |
| ---------- | ------- | ------- | ------- | ------- | ------- |
| 2022-01-04 | -0.0295 | -0.0171 | 0.0102  | 0.0446  | 0.0112  |
| ...        | ...     | ...     | ...     | ...     | ...     |

### Analisis & Arti Angka

* **-0.0295 (ADRO):** Harga ADRO turun 2.95%.
* **0.0446 (BBNI):** Harga BBNI naik 4.46%.

### Peran dalam Algoritma Genetika

* Rata-rata return harian (yang disetahunkan) menjadi **Expected Return**.
* GA akan mencari kombinasi bobot saham yang menghasilkan **return total tertinggi**.

---

## 3. Matriks Kovarians

**File:** `matriks_kovarians.csv`

Matriks ini adalah komponen paling krusial untuk **manajemen risiko**. Nilainya menunjukkan hubungan pergerakan antar saham yang telah disetahunkan (× 252 hari bursa).

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

Nilai diagonal adalah **varians**, menunjukkan tingkat risiko masing-masing saham.

* **ADRO (0.192) & ANTM (0.187):** Risiko tinggi (fluktuatif).
* **BBCA (0.053):** Risiko rendah (stabil).

#### B. Non-Diagonal (Hubungan Antar Saham)

Nilai di luar diagonal adalah **kovarians**.

* **ADRO vs PTBA (0.0689):** Korelasi positif tinggi → diversifikasi kurang efektif.
* **BBCA vs ANTM (0.0099):** Hampir tidak berkorelasi → diversifikasi sangat baik.

---

## 4. Sintesis: Cara GA Menggunakan Data

Saat menghitung **Fitness**, GA melakukan:

1. **Ambil Gen (Bobot):** Contoh `{ADRO: 50%, BBCA: 50%}`
2. **Hitung Return:** Dari rata-rata return harian.
3. **Hitung Risiko:** Dari matriks kovarians.

### Keputusan

* Korelasi ADRO–BBCA rendah → risiko gabungan menurun.
* Portofolio dinilai **efisien** (return tinggi, risiko terkendali).
* Kombinasi ini memperoleh **skor fitness tinggi**.

---

## 5. Analogi Biologi dalam Algoritma Genetika

GA mengadopsi konsep evolusi biologi ke dalam pemrograman.

### Tabel Perbandingan

| Konsep Biologi | Dalam Kode Python | Representasi Data |
| -------------- | ----------------- | ----------------- |
| Gen            | Bobot saham       | Float (mis. 0.2)  |
| Kromosom       | Satu portofolio   | Array NumPy       |
| Individu       | Solusi kandidat   | Kromosom          |
| Populasi       | Kumpulan solusi   | List of arrays    |
| Fitness        | Skor kualitas     | Return − Risiko   |

---

### A. Gen

Gen adalah bobot alokasi untuk satu saham.

* 5 saham → 5 gen per individu.
* Contoh: ADRO 20% → gen = `0.20`.

### B. Kromosom & Individu

Kromosom adalah kumpulan gen yang membentuk satu strategi portofolio.

```python
def create_individual():
    w = np.random.rand(n_assets)
    return w / np.sum(w)
```

Contoh output:

```
[0.1, 0.4, 0.2, 0.1, 0.2]
```

### C. Populasi

Populasi adalah sekumpulan individu dalam satu generasi.

* Contoh: `POP_SIZE = 100`.

### D. Fitness (Fungsi Kelayakan)

Menilai kualitas portofolio.

[
Fitness = Return - (\lambda \times Risk)
]

```python
def fitness(weights, risk_aversion=RISK_AVERSION):
    ret = portfolio_return(weights)
    risk = portfolio_risk(weights)
    return ret - risk_aversion * risk
```

### E. Generasi & Alur Evolusi

1. **Inisialisasi:** Populasi awal acak
2. **Evaluasi:** Hitung fitness
3. **Seleksi:** Pilih induk terbaik
4. **Crossover:** Gabungkan gen
5. **Mutasi:** Acak kecil untuk variasi
6. **Iterasi:** Hingga ditemukan **portofolio optimal**
