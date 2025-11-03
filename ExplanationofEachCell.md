# 🧬 Optimasi Portofolio dengan Algoritma Genetika

## Deskripsi singkat:
Notebook ini melakukan optimasi proporsi investasi antara berbagai instrumen keuangan — seperti Saham Indonesia (misal: ADRO, BBCA, ANTM, PTBA), Crypto (misal: BTC, ETH, BNB), maupun Reksadana — menggunakan Genetic Algorithm (GA).

Program mencari kombinasi bobot portofolio terbaik untuk memaksimalkan return dengan risiko minimal berdasarkan parameter risk aversion (λ).

---

## ⚙️ 1. Impor Library

Menyediakan dependensi utama untuk manipulasi data (pandas, numpy), pengambilan data (yfinance), dan visualisasi (matplotlib, seaborn).

---

## 📈 2. Konfigurasi Parameter

Cell ini menentukan parameter awal:

| Parameter | Deskripsi |
|-----------|-----------|
| TICKERS | Daftar aset |
| POP_SIZE | Ukuran populasi |
| GENERATIONS | Jumlah iterasi evolusi |
| RISK_AVERSION (λ) | Faktor penalti risiko |
| RANDOM_SEED | Reproducibility |

---

## 📁 3. Persiapan Folder Output

Notebook memastikan folder `data/` dan `outputs/` sudah tersedia.

---

## 💾 4–5. Pengunduhan dan Pembersihan Data

Mengambil harga saham dari Yahoo Finance, mengekstrak kolom Adj Close (harga penutupan disesuaikan), dan menghitung return harian.

**Rumus:**

$$r_{t,i} = \frac{P_{t,i} - P_{t-1,i}}{P_{t-1,i}}$$

Kemudian dihitung rata-rata return dan kovarians tahunan:

$$\bar{r}_i = \text{mean}(r_i) \times 252$$

$$\Sigma = \text{cov}(r_i) \times 252$$

---

## 📤 6. Ekspor Data Awal

Tiga file yang dihasilkan:

- `data/harga_penutupan_saham.csv`
- `data/return_harian_saham.csv`
- `data/matriks_kovarians.csv`

---

## 🧬 8–9. Inisialisasi Algoritma Genetika

Setiap kromosom merepresentasikan portofolio investasi berupa bobot tiap saham:

$$w = [w_1, w_2, ..., w_n]$$

dengan:

$$\sum_{i=1}^{n} w_i = 1, \quad w_i \geq 0$$

Populasi awal dihasilkan acak:

$$\text{Populasi}_0 = \{w_1, w_2, ..., w_{\mathrm{POP\_SIZE}}\}$$

---

## 💡 10. Fungsi Return dan Risiko Portofolio

**Return portofolio:**

$$R_p = \sum_{i=1}^{n} w_i \bar{r}_i = w^T \bar{r}$$

**Risiko (standar deviasi portofolio):**

$$\sigma_p = \sqrt{w^T \Sigma w}$$

---

## 🏆 11. Fungsi Fitness

Fungsi objektif yang dimaksimalkan:

$$f(w) = R_p - \lambda \sigma_p$$

di mana:

- $R_p$ = return ekspektasi portofolio
- $\sigma_p$ = risiko portofolio
- $\lambda$ = faktor aversi risiko

---

## 🔁 12–14. Seleksi, Crossover, dan Mutasi

### Seleksi (Tournament)

Beberapa individu dipilih acak, pemenang dengan fitness tertinggi:

$$w_{\text{best}} = \max_{w_k} f(w_k)$$

### Crossover (BLX-alpha)

Kombinasi genetik dua parent:

$$c = \alpha p_1 + (1 - \alpha) p_2$$

dengan $\alpha \in [0, 1]$

### Mutasi

Menambahkan gangguan acak:

$$w'_i = w_i + \epsilon, \quad \epsilon \sim \mathcal{N}(0, \sigma^2)$$

---

## 🔄 15. Proses Evolusi

Untuk setiap generasi $t$:

1. Hitung fitness semua individu
2. Pertahankan elit terbaik
3. Hasilkan anak baru dengan crossover + mutasi
4. Catat:

$$\text{BestFitness}(t) = \max f_t(w)$$

$$\text{AvgFitness}(t) = \frac{1}{N} \sum f_t(w)$$

---

## 🧩 16. Portofolio Optimal

Output akhir:

$$w^* = \arg\max_w f(w)$$

**Contoh hasil:**

| Ticker | Weight |
|--------|--------|
| ADRO.JK | 0.3731 |
| BBCA.JK | 0.1411 |
| ANTM.JK | 0.0000 |
| BBNI.JK | 0.1980 |
| PTBA.JK | 0.2879 |

---

## 📊 18. Visualisasi Fitness

Menampilkan evolusi fungsi fitness:

$$f_t = R_t - \lambda \sigma_t$$

📁 disimpan di `outputs/fitness_plot.png`

---

## 📈 19. Efficient Frontier

Plot menunjukkan portofolio acak (titik hijau–biru) dan hasil GA (titik merah):

$$\text{Objective} = R_p - \lambda \sigma_p$$

📁 disimpan di `outputs/efficient_frontier.png`

---

## 🥧 21. Komposisi Portofolio

Visualisasi distribusi bobot tiap saham.

📁 disimpan di `outputs/portfolio_pie.png`

---

## 🧱 22. Visualisasi Bobot (Bar Chart)

Bar chart menunjukkan kontribusi numerik masing-masing saham terhadap total modal.

📁 disimpan di `outputs/portfolio_bar.png`

---

## 📘 Dataset dan Hasil Akhir

**Contoh data harga:**

| Tanggal | ADRO.JK | ANTM.JK | BBCA.JK | BBNI.JK | PTBA.JK |
|---------|---------|---------|---------|---------|---------|
| 2022-01-03 | 908.5 | 1944.5 | 6616.5 | 2772.3 | 1227.6 |
| 2022-01-04 | 881.6 | 1911.3 | 6684.2 | 2896.0 | 1241.4 |
| ... | ... | ... | ... | ... | ... |
| 2025-10-31 | 1885.0 | 3100.0 | 8525.0 | 4380.0 | 2400.0 |

---

## 🧾 Rumus Tambahan

**Normalisasi bobot:**

$$w' = \frac{w}{\sum_i w_i}$$

**Return ekspektasi portofolio:**

$$E[R_p] = w^T \bar{r}$$

**Risiko (standard deviation):**

$$\sigma_p = \sqrt{w^T \Sigma w}$$

**Fitness GA:**

$$f(w) = E[R_p] - \lambda \sigma_p$$

---
