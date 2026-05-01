# UTS_Pengolahan_Citra

NAMA : MUHAMMAD ARKHAMULLAH RIFAI ASSHIDIQ
NIM : 312410545

# 🚗 Praktikum 8 UTS — Pengolahan Citra Digital
### Segmentasi Citra dengan Python & OpenCV

| | |
|---|---|
| **Mata Kuliah** | Pengolahan Citra |
| **Dosen** | Muhamad Fatchan, S.Kom., M.Kom. |
| **Nama** | Afdhal Agislam |
| **NIM** | 312410445 |
| **Kelas** | I241E |
| **Program Studi** | Teknik Informatika — Universitas Pelita Bangsa |

---

## 📌 Deskripsi

Praktikum ini mengimplementasikan berbagai metode **segmentasi citra digital** menggunakan Python, OpenCV, dan scikit-image. Input yang digunakan adalah **gambar mobil berwarna** yang dikonversi ke grayscale sebelum diproses.

Segmentasi citra adalah proses memisahkan suatu citra menjadi beberapa bagian (region/segmen) berdasarkan karakteristik tertentu seperti intensitas, warna, atau tekstur.

---

## 🛠️ Dependensi

```bash
pip install opencv-python numpy matplotlib scikit-image scipy
```

| Library | Versi | Kegunaan |
|---|---|---|
| `opencv-python` | ≥ 4.5 | Operasi citra utama |
| `numpy` | ≥ 1.21 | Manipulasi array |
| `matplotlib` | ≥ 3.4 | Visualisasi hasil |
| `scikit-image` | ≥ 0.18 | Watershed, peak detection |
| `scipy` | ≥ 1.7 | Distance transform |

---

## ▶️ Cara Menjalankan

```bash
# Clone repository
git clone https://github.com/afdaal10/UTS_PENGOLAHAN-CITRA.git
cd UTS_PENGOLAHAN-CITRA

# Install dependensi
pip install opencv-python numpy matplotlib scikit-image scipy

# Jalankan program
python main_mobil.py
```

Program akan otomatis menghasilkan **9 file gambar output** di folder yang sama.

---

## 🖼️ Input

Gambar yang digunakan adalah foto **5 mobil berwarna** (hijau, merah, biru, kuning, pink) yang ditumpuk secara vertikal dengan latar belakang putih. Gambar dikonversi ke **grayscale 512×512 piksel** sebelum diproses.

```
Intensitas: min=0, max=255, mean=197.1
```

---

## 📊 Hasil & Penjelasan

### 1. Thresholding

![Thresholding](out_1_thresholding.png)

![Histogram](out_1_histogram.png)

Thresholding adalah metode segmentasi paling sederhana — setiap piksel diklasifikasikan menjadi **objek (putih)** atau **background (hitam)** berdasarkan nilai intensitasnya.

Tiga metode yang diuji:

| Metode | Nilai T | Cara Kerja | Hasil |
|---|---|---|---|
| **Global** | T = 128 | Nilai tetap, semua piksel > 128 → putih | Kurang optimal, background putih ikut terpotong |
| **Otsu** | T = 153 (auto) | Memaksimalkan variansi antar kelas secara otomatis | Lebih baik, T disesuaikan dengan distribusi citra |
| **Adaptif** | Berbeda tiap area | T dihitung per-neighborhood (blockSize=31) | Menangkap detail halus, cocok untuk pencahayaan tidak merata |

**Kesimpulan:** Metode Otsu menghasilkan T=153 (lebih tinggi dari manual T=128) karena gambar ini didominasi background putih (mean=197.1). Metode adaptif menghasilkan detail paling kaya namun lebih sensitif terhadap noise.

---

### 2. Region Growing

![Region Growing](out_2_region_growing.png)

![Multi Region](out_2b_multiregion.png)

Region Growing adalah metode segmentasi berbasis piksel yang tumbuh dari titik awal (seed). Algoritma menggunakan **BFS (Breadth-First Search)** dengan 8-konektivitas.

**Cara kerja:**
1. Tentukan titik seed
2. Periksa semua tetangga 8-arah
3. Tambahkan tetangga ke region jika `|I(tetangga) - I(seed)| ≤ threshold`
4. Ulangi sampai tidak ada tetangga yang memenuhi syarat

**Seed yang diuji:**

| Seed | Posisi | Mobil |
|---|---|---|
| (256, 420) | Bawah | Mobil pink |
| (280, 310) | Tengah | Mobil kuning |
| (256, 210) | Atas-tengah | Mobil biru |

**Pengaruh threshold:**
- **T=15** → Region sempit, hanya piksel sangat mirip dengan seed yang masuk
- **T=30** → Region lebih luas, mulai mencakup seluruh badan mobil
- **T=50** → Region sangat luas, bisa merembes ke mobil lain yang intensitasnya berdekatan

**Kesimpulan:** Pemilihan seed yang tepat (di tengah objek) dan nilai threshold yang sesuai sangat menentukan kualitas segmentasi. Gambar nyata lebih sulit karena intensitas antar bagian mobil bervariasi.

---

### 3. Deteksi Tepi

![Deteksi Tepi](out_3_deteksi_tepi.png)

Deteksi tepi bertujuan menemukan batas/kontur objek dalam citra dengan mendeteksi perubahan intensitas yang tajam.

**Operator yang dibandingkan:**

| Operator | Cara Kerja | Karakteristik |
|---|---|---|
| **Sobel Gx** | Kernel 3×3 arah horizontal | Mendeteksi tepi vertikal (sisi kiri-kanan mobil) |
| **Sobel Gy** | Kernel 3×3 arah vertikal | Mendeteksi tepi horizontal (atap & bawah mobil) |
| **Sobel Magnitude** | √(Gx²+Gy²) | Tepi lengkap, hasil agak tebal |
| **LoG** | Gaussian blur → Laplacian | Sensitif terhadap detail, menghasilkan double edge |
| **Canny (30/80)** | Non-max suppression + hysteresis | Tepi bersih tapi threshold rendah masih ada noise |
| **Canny (50/150)** | Non-max suppression + hysteresis | **Terbaik** — tepi sangat bersih dan lengkap |
| **Canny (100/200)** | Non-max suppression + hysteresis | Terlalu ketat, beberapa tepi halus hilang |

**Kesimpulan:** Pada gambar mobil yang kompleks, metode **Canny dengan T=50/150** memberikan hasil terbaik — kontur mobil terdeteksi jelas tanpa noise berlebihan.

---

### 4. K-Means Clustering

![K-Means](out_4_kmeans.png)

![K-Means Color](out_4b_kmeans_color.png)

K-Means mengelompokkan piksel berdasarkan kemiripan intensitas. Setiap piksel dianggap sebagai titik data 1D dan dikelompokkan ke dalam K cluster.

**Centroid yang ditemukan:**

| K | Centroid Intensitas | Interpretasi |
|---|---|---|
| K=2 | [58, 248] | Background gelap vs area terang |
| K=3 | [42, 149, 255] | Gelap / abu-abu / putih |
| K=4 | [21, 92, 174, 255] | 4 tingkat kecerahan berbeda |
| K=5 | [16, 72, 115, 177, 255] | Paling detail, tiap mobil mulai terpisah |

**Analisis K=5 (sesuai jumlah mobil):**
- Cluster 1 (I≈16): Area sangat gelap (ban, kaca gelap)
- Cluster 2 (I≈72): Bagian gelap mobil
- Cluster 3 (I≈115): Abu-abu sedang
- Cluster 4 (I≈177): Abu-abu terang
- Cluster 5 (I≈255): Background putih & area sangat terang

**Kesimpulan:** K=5 dipilih karena sesuai dengan jumlah mobil dalam gambar, menghasilkan segmentasi paling informatif.

---

### 5. Watershed Segmentation

![Watershed](out_5_watershed.png)

Watershed adalah metode segmentasi berbasis topografi — citra dianggap seperti peta ketinggian dan "air" mengalir dari titik terendah.

**6 Tahap proses:**

1. **Citra Asli** → Input grayscale mobil
2. **Otsu Threshold** → Binarisasi dengan T=153, memisahkan objek dari background
3. **Morphological Opening** → Menghilangkan noise kecil, memperhalus tepi objek
4. **Distance Transform** → Setiap piksel putih diberi nilai jarak ke background terdekat. Area **merah/kuning = pusat region** (paling jauh dari tepi)
5. **Peak Markers** → Titik puncak distance transform dijadikan seed watershed (terdeteksi **19 marker** karena kompleksitas gambar nyata)
6. **Hasil Watershed** → 19 region dipisahkan dengan batas putih, masing-masing diberi warna berbeda

**Kesimpulan:** Jumlah region yang banyak (19) menunjukkan kompleksitas gambar mobil nyata dibanding citra sintetis (5 region). Hal ini karena mobil memiliki banyak komponen berbeda (badan, ban, kaca, lampu).

---

### 6. Evaluasi Segmentasi

![Evaluasi](out_6_evaluasi.png)

Evaluasi mengukur seberapa akurat hasil segmentasi dibandingkan dengan **ground truth** (area yang benar). Ground truth yang digunakan adalah area badan mobil pink (mobil paling bawah).

**Metrik evaluasi:**

| Metrik | Formula | Arti |
|---|---|---|
| **Pixel Accuracy (PA)** | `Σ(pred==gt) / total` | Persentase piksel yang diklasifikasi benar |
| **IoU (Jaccard)** | `\|A∩B\| / \|A∪B\|` | Overlap antara prediksi dan ground truth |
| **Dice Coefficient** | `2\|A∩B\| / (\|A\|+\|B\|)` | Mirip IoU, lebih sensitif terhadap false negative |
| **Precision** | `TP / (TP+FP)` | Seberapa tepat area yang diprediksi |
| **Recall** | `TP / (TP+FN)` | Seberapa lengkap area yang terdeteksi |

**Hasil evaluasi:**

| Prediksi | PA | IoU | Dice | Precision | Recall |
|---|---|---|---|---|---|
| A — Hampir sempurna | 0.992 | **0.963** | **0.981** | 1.000 | 0.963 |
| B — Under-segmentation | 0.903 | 0.531 | 0.694 | 1.000 | 0.531 |
| C — Over-segmentation | 0.885 | 0.644 | 0.784 | 0.644 | 1.000 |
| D — Posisi salah | 0.584 | 0.000 | 0.000 | 0.000 | 0.000 |

**Kode warna visualisasi:**
- 🟢 **Hijau = True Positive (TP)** → Area yang benar diprediksi sebagai objek
- 🔴 **Merah = False Positive (FP)** → Area salah diprediksi sebagai objek (over)
- 🔵 **Biru = False Negative (FN)** → Area objek yang tidak terdeteksi (under)

**Kesimpulan:** Prediksi A mendapat IoU=0.963 (sangat tinggi). Prediksi D (posisi salah) mendapat IoU=0.000 karena tidak ada overlap sama sekali dengan ground truth.

---

## 📁 Struktur File

```
UTS_PENGOLAHAN-CITRA/
│
├── main_mobil.py              # Kode utama program
├── mobil.jpg                  # Gambar input
├── README.md                  # Dokumentasi ini
│
├── out_1_histogram.png        # Output: Histogram intensitas
├── out_1_thresholding.png     # Output: Perbandingan thresholding
├── out_2_region_growing.png   # Output: Region growing 8 kombinasi
├── out_2b_multiregion.png     # Output: Multi-region 3 seed
├── out_3_deteksi_tepi.png     # Output: Perbandingan deteksi tepi
├── out_4_kmeans.png           # Output: K-Means K=2,3,4,5
├── out_4b_kmeans_color.png    # Output: K-Means K=5 berwarna
├── out_5_watershed.png        # Output: Watershed 6 tahap
└── out_6_evaluasi.png         # Output: Evaluasi IoU & Dice
```

---

## 📝 Kesimpulan Umum

| Metode | Kelebihan | Kekurangan |
|---|---|---|
| **Thresholding Global** | Cepat, sederhana | Tidak adaptif terhadap variasi pencahayaan |
| **Thresholding Otsu** | T optimal otomatis | Masih satu nilai global |
| **Thresholding Adaptif** | Fleksibel per area | Sensitif noise |
| **Region Growing** | Kontrol penuh via seed | Sensitif terhadap seed dan threshold |
| **Deteksi Tepi (Canny)** | Tepi bersih dan tipis | Hanya menghasilkan kontur, bukan region |
| **K-Means** | Mudah diimplementasi | Perlu menentukan K manual |
| **Watershed** | Memisahkan objek berdekatan | Over-segmentation pada gambar kompleks |

Untuk gambar nyata seperti foto mobil, **kombinasi metode** (misalnya Canny untuk deteksi tepi + Watershed untuk segmentasi region) menghasilkan hasil yang lebih baik dibanding satu metode saja.

---
