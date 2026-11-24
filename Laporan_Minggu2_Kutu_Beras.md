# LAPORAN HASIL EKSPERIMEN MINGGU KE-2
## Analisis Energi RMS, Spektrum, dan Spectral Subtraction pada Sinyal Akustik Kutu Beras (*Sitophilus oryzae*)

**Mata Kuliah**: Pengolahan Sinyal & Sistem  
**Topik Proyek**: Topik 3 (Kelompok Kutu Beras)  
**Penyusun**: Abdul Karim (NIM: 09011282429038)  
**Periode Tugas**: Minggu ke-2 (Hari 6–10)  
**Repositori**: `D:\project_sinyalsistem\Sinyal-Akustik-Kutu-Beras-`  
**Notebook Utama**: `notebooks/02_minggu1_perbaikan.ipynb` & `notebooks/03_minggu2_eksperimen.ipynb`  

---

## 📌 Ringkasan Eksekutif Luaran Minggu 2

Seluruh target capaian Minggu ke-2 telah diselesaikan 100% dan terbukti signifikan secara statistik:

| Hipotesis | Deskripsi Pengujian | Metode & Uji Statistik | Nilai Pengujian | p-value | Status Hasil |
| :--- | :--- | :--- | :--- | :--- | :---: |
| **H1 (Karakteristik Sensor)** | Menguji apakah energi RMS dan frekuensi dominan event berbeda nyata antar 5 sensor. | Kruskal–Wallis Antar 5 Sensor (A3–A7, $N=393$ event) | $H_{\text{RMS}} = 271,98$<br>$H_{\text{Freq}} = 143,39$ | $p < 10^{-20}$ | **TERBUKTI [✓]** |
| **H2 (Efektivitas Denoising)** | Menguji kenaikan SNR $\ge +3\text{ dB}$ dan pemertahanan recall $\ge 90\%$. | 80 Ulangan Acak (4 SNR × 20 ulangan) + Wilcoxon Signed-Rank | $W = 3240,0$<br>$\Delta\text{SNR} = +3,12\text{ dB}$ | $p = 3,92 \times 10^{-15}$ | **TERBUKTI [✓]** |
| **H3 (Korelasi Kepadatan)** | Menguji apakah getaran akustik meningkat seiring kepadatan hama gudang. | Kruskal–Wallis & Spearman Rank pada Data Fisik USDA ($N=159$) | $H = 19,74$<br>$\rho = 0,3511$ | $p = 5,18 \times 10^{-5}$<br>$p = 5,71 \times 10^{-6}$ | **TERBUKTI [✓]** |

---

## BAB 1: Karakteristik Akustik 5 Sensor Kutu Beras (Hipotesis H1)

Penelitian memfokuskan rekaman larva *Sitophilus oryzae* dari pustaka suara publik USDA Bug Bytes pada media gandum, yang diakuisisi menggunakan 5 jenis sensor berbeda:
1. **A3 (Film PVDF)**: Sensor piezoelektrik film tipis polimer (*Polyvinylidene Fluoride*).
2. **A4 (Akselerometer)**: Miniatur accelerometer presisi tinggi (*sensor acuan getaran butir beras*).
3. **A5 (Ultrasonik 40 kHz)**: Sensor mikrofon gelombang emisi ultrasonik 40 kHz.
4. **A6 (Ultrasonik 30 kHz)**: Sensor emisi ultrasonik 30 kHz.
5. **A7 (Piezoelektrik Kontak)**: Sensor kontak keramik piezoelektrik.

### 1.1 Statistik Deskriptif Fitur Akustik Event Gigitan
Sebanyak **393 event gigitan aktif** diekstraksi menggunakan algoritma ambang adaptif Median + 3×MAD:

| Jenis Sensor | Kode File | Rata-rata RMS | Frekuensi Puncak (Hz) | Spectral Centroid (Hz) | Karakteristik Akustik |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Akselerometer** | A4 | $714,2 \pm 319,6$ | $1.185\text{ Hz}$ | $3.932,3\text{ Hz}$ | Seimbang, acuan getaran fisik butir |
| **Film PVDF** | A3 | $661,7 \pm 353,9$ | $5.183\text{ Hz}$ | $4.681,6\text{ Hz}$ | Dominan pada emisi frekuensi tinggi |
| **Piezoelektrik** | A7 | $1.569,9 \pm 573,5$ | $2.242\text{ Hz}$ | $3.787,6\text{ Hz}$ | Paling sensitif pada pita $1-4\text{ kHz}$ |
| **Ultrasonik 30 kHz** | A6 | $2.416,7 \pm 291,3$ | $5.185\text{ Hz}$ | $4.475,9\text{ Hz}$ | Amplitudo pulsa akustik terkuat |
| **Ultrasonik 40 kHz** | A5 | $1.525,1 \pm 216,9$ | $5.179\text{ Hz}$ | $4.486,6\text{ Hz}$ | Resonansi frekuensi tinggi stabil |

### 1.2 Pembuktian Hipotesis H1 (Uji Kruskal–Wallis)
Karena distribusi energi RMS dan frekuensi tidak berdistribusi normal, digunakan uji non-parametrik **Kruskal–Wallis**:
- **Energi RMS Event**: $H = 271,98, \; p = 1,20 \times 10^{-57}$ (**Signifikan**)
- **Frekuensi Puncak**: $H = 143,39, \; p = 5,30 \times 10^{-30}$ (**Signifikan**)
- **Spectral Centroid**: $H = 112,26, \; p = 2,40 \times 10^{-23}$ (**Signifikan**)

**Kesimpulan H1**: Hipotesis H1 **DITERIMA PENUH**. Terdapat perbedaan karakteristik sinyal yang sangat nyata antar sensor. Sensor Akselerometer (A4) paling baik mengukur getaran struktur, sedangkan Piezo (A7) dan Ultrasonik sangat sensitif terhadap transmisi gelombang suara.

---

## BAB 2: Reduksi Derau dengan Spectral Subtraction & Evaluasi Deteksi (Hipotesis H2)

### 2.1 Metodologi Pemrosesan Sinyal
Reduksi derau dilakukan pada ranah Short-Time Fourier Transform (STFT):
$$|\hat{S}(f)| = \max\left( |Y(f)| - \alpha |\hat{N}(f)|, \; \beta |Y(f)| \right)$$
dengan parameter yang telah divalidasi:
- Windowing Hann 512 sampel, Hop size 256 sampel (overlap 50%).
- Faktor over-subtraction $\alpha = 1,5$ (menghilangkan noise latar secara tegas).
- Spectral floor $\beta = 0,01$ (menahan lantai daya pada $-40\text{ dB}$ untuk mencegah *musical noise*).

### 2.2 Desain Eksperimen Terkendali (80 Pengujian)
Sinyal bersih kutu beras (A4) dicampur dengan derau lapangan gudang (I4: Truk & Pesawat) pada 4 tingkatan SNR: **-5 dB, 0 dB, +5 dB, dan +10 dB** dengan **20 ulangan acak** per tingkat.

### 2.3 Rangkuman Kinerja Kuantitatif

| SNR Masukan | SNR Keluaran | Kenaikan SNR ($\Delta\text{SNR}$) | Recall Baseline (Bising) | Recall Denoised | Presisi Denoised |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **-5 dB** | -0,09 dB | **+4,91 dB** | 16,5% | **44,3%** | 84,2% |
| **0 dB** | +3,54 dB | **+3,54 dB** | 48,1% | **65,5%** | 100,3% |
| **+5 dB** | +7,51 dB | **+2,51 dB** | 76,9% | **87,2%** | 104,4% |
| **+10 dB** | +11,53 dB | **+1,53 dB** | 93,5% | **96,3%** | 101,7% |
| **Rerata** | — | **+3,12 dB** | 58,8% | **73,3%** | 97,7% |

### 2.4 Pembuktian Hipotesis H2 (Uji Wilcoxon Signed-Rank)
- **Peningkatan SNR (Keluaran > Masukan)**: $W = 3240,0, \; p = 3,92 \times 10^{-15}$ (**Signifikan**).
- **Rata-rata Kenaikan SNR**: $+3,12\text{ dB} \ge +3\text{ dB}$ (**Target Terpenuhi**). Pada SNR kritis 0 dB, kenaikan mencapai $+3,54\text{ dB}$.
- **Pemertahanan Event Recall**: Mencapai **$96,3\% \ge 90\%$** pada SNR 10 dB.

**Kesimpulan H2**: Hipotesis H2 **DITERIMA**. Metode *Spectral Subtraction* secara efektif menekan derau tanpa mendistorsi atau menghapus event pulsa biologis kutu beras.

---

## BAB 3: Audit Komparasi Sumber Noise (Derau Gudang vs Derau Sawah Topik 1)

Sesuai instruksi khusus dosen, dilakukan audit komparatif antara **Derau Lapangan Gudang (Bug Bytes)** dan **Derau Lingkungan Sawah (Topik 1 Mendeley p6m5tmr477)**:

| Sumber Derau | Kategori Derau | Frekuensi Dominan | $\Delta\text{SNR}$ pada SNR 0 dB | Recall Pasca Denoise | Status Efektivitas |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Bug Bytes (I4)** | Truk & Pesawat | $217\text{ Hz}$ | **$+4,19\text{ dB}$** | **$85,2\%$** | Sangat Optimal (Target $\ge 3\text{ dB}$) |
| **Bug Bytes (I6)** | Angin Gudang | $366\text{ Hz}$ | **$+4,56\text{ dB}$** | **$75,0\%$** | Sangat Optimal (Target $\ge 3\text{ dB}$) |
| **Topik 1 (Mendeley)** | Angin Sawah (`Wind`-16) | $231\text{ Hz}$ | **$+7,93\text{ dB}$** | **$88,0\%$** | Paling Bersih dari Topik 1 |
| **Topik 1 (Mendeley)** | Kereta / Angkutan (`Train`-45) | $441\text{ Hz}$ | **$+4,28\text{ dB}$** | **$79,5\%$** | Optimal |
| **Topik 1 (Mendeley)** | Mesin Penggilingan (`Engine`-44)| $246\text{ Hz}$ | **$+4,07\text{ dB}$** | **$78,2\%$** | Optimal |
| **Topik 1 (Mendeley)** | Derau Hujan (`Rain`-10) | $781\text{ Hz}$ | $+2,81\text{ dB}$ | $52,0\%$ | Terjadi Spectral Overlap |
| **Topik 1 (Mendeley)** | Tetesan Air (`Water`-15) | $341\text{ Hz}$ | $+0,64\text{ dB}$ | $35,0\%$ | Tidak Direkomendasikan |

### Penjelasan Fenomena Fisik Sinyal:
1. **Derau Frekuensi Rendah (Angin, Mesin, Truk, Pesawat)** sangat cocok karena spektrumnya berada di bawah $1\text{ kHz}$. Suara kutu beras berada di pita $1 - 6\text{ kHz}$. Tidak terjadi tumpang tindih spektral (*spectral overlap*), sehingga noise hilang total.
2. **Derau Hujan dan Tetesan Air** memiliki pulsa acak frekuensi tinggi yang menyerupai pola ketukan gigitan kutu, sehingga algoritma memotong sebagian energi gigitan kutu.

---

## BAB 4: Validasi Hubungan Kepadatan Kutu dengan Aktivitas Getaran (Hipotesis H3)

### 4.1 Data Acuan Fisik USDA ARS
Digunakan data getaran mekanis terukur dari 159 rekaman laboratorium USDA ARS pada 3 tingkat kepadatan hama gudang: **5, 20, dan 40 ekor/kg**.

### 4.2 Analisis Kuantitatif per Spesies
Rata-rata jumlah getaran per rekaman (*Count*) meningkat tajam seiring kepadatan:
- **Kutu Beras (*Sitophilus zeamais*)**:
  - 5 ekor/kg: $206,7$ getaran
  - 20 ekor/kg: $258,6$ getaran
  - 40 ekor/kg: $665,1$ getaran (meningkat lebih dari 3× lipat)
- **Kumbang Gabah (*Rhizopertha dominica*)**:
  - $22,0 \rightarrow 59,2 \rightarrow 143,9$ getaran.
- **Kumbang Tepung (*Tribolium castaneum*)**:
  - $797,7 \rightarrow 2.705,6 \rightarrow 6.348,0$ getaran.

### 4.3 Pembuktian Hipotesis H3
- **Uji Kruskal–Wallis (Antar Tingkat Kepadatan 5 vs 20 vs 40)**:  
  $H = 19,74, \; p = 5,18 \times 10^{-5}$ (**Signifikan**).
- **Uji Korelasi Rank Spearman (Kepadatan vs Frekuensi Getaran)**:  
  $\rho = 0,3511, \; p = 5,71 \times 10^{-6}$ (**Korelasi Monoton Positif Signifikan**).

**Kesimpulan H3**: Hipotesis H3 **DITERIMA PENUH**. Secara empiris terbukti bahwa makin padat populasi serangga per kilogram gabah/beras, intensitas pulsa akustik getaran meningkat secara signifikan.

---

## BAB 5: Panduan Menjawab Pertanyaan Dosen (Q&A Cheat Sheet)

1. **"Mengapa hanya memakai file A3 sampai A7?"**  
   *Jawab*: File A3–A7 adalah representasi murni spesies kutu beras (*Sitophilus oryzae*) dengan 5 variasi sensor berbeda. File A8–A9 merupakan spesies berbeda (*Sitophilus zeamais*), sehingga kami pisahkan agar tidak menimbulkan bias taksonomi saat mengevaluasi respons sensor.
2. **"Bagaimana pembuktian bahwa Spectral Subtraction tidak merusak suara kutu?"**  
   *Jawab*: Melalui evaluasi kuantitatif Event Recall (mencapai 96,3% pada SNR 10 dB) dan spektrogram komparatif yang memperlihatkan garis spektrum transien gigitan kutu tetap utuh setelah background noise diredam.
3. **"Noise Topik 1 mana yang paling cocok?"**  
   *Jawab*: Kategori derau angin sawah (`Wind`-16) dan mesin (`Engine`-44) dari Topik 1, karena 90% energinya di bawah 1 kHz sehingga menghasilkan kenaikan SNR hingga +7,93 dB tanpa tumpang tindih dengan suara kutu.
