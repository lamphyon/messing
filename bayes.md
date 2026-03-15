# 📧 Deteksi Spam Email dengan Algoritma Naive Bayes

Selamat datang! Dokumentasi ini menjelaskan konsep dasar di balik pendeteksian email spam menggunakan algoritma **Naive Bayes**, mulai dari teori matematika dasar hingga cara membaca hasil evaluasi modelnya.

---

## 1. Memahami Teori Bayes (Bayes' Theorem)

Sebelum masuk ke algoritmanya, kita harus kenalan dulu dengan **Teori Bayes**. Ini adalah konsep fundamental dalam probabilitas dan statistika untuk menghitung **probabilitas bersyarat** (*conditional probability*). 

Sederhananya: *Bagaimana kita memperbarui keyakinan kita terhadap sesuatu ketika ada informasi atau bukti baru?*

Secara matematis, rumusnya ditulis seperti ini:

$$P(A|B) = \frac{P(B|A) \cdot P(A)}{P(B)}$$

**Keterangan:**
* **$P(A|B)$ (Posterior):** Peluang kejadian A terjadi *jika* kita tahu B sudah terjadi. (Ini hasil akhir yang ingin kita cari).
* **$P(B|A)$ (Likelihood):** Peluang kejadian B terjadi *jika* asumsi A itu benar.
* **$P(A)$ (Prior):** Peluang awal kejadian A *sebelum* ada bukti B.
* **$P(B)$ (Evidence):** Total peluang kejadian B secara keseluruhan.

---

## 2. Bagaimana Naive Bayes Bekerja?

Algoritma **Naive Bayes** meminjam rumus di atas untuk melakukan klasifikasi (misalnya: membedakan pesan Spam dan Aman). 

Kenapa disebut **"Naive"** (Naif)? 
Karena algoritma ini punya asumsi yang sangat kaku: Ia menganggap setiap kata dalam sebuah kalimat itu berdiri sendiri dan tidak punya hubungan satu sama lain. (Contoh: kata "hadiah" dan "gratis" dianggap tidak saling berkaitan, padahal di dunia nyata sering muncul bersamaan).

### Contoh Kasus: Mendeteksi Kata Spam
Bayangkan ada email masuk bertuliskan: "uang gratis". Algoritma akan menghitung seberapa besar peluang email ini adalah SPAM menggunakan variasi rumus Bayes:

$$P(Spam | uang, gratis) \propto P(Spam) \cdot P(uang | Spam) \cdot P(gratis | Spam)$$

**Proses di balik layar:**
1. **Prior $P(Spam)$:** Komputer melihat berapa peluang umum sebuah email itu spam berdasarkan data masa lalu.
2. **Likelihood:** Komputer mengecek seberapa sering kata "uang" dan "gratis" muncul di kumpulan email spam.
3. **Perhitungan:** Komputer mengalikan peluang-peluang tadi. Proses ini dihitung dua kali (satu skor untuk SPAM, satu skor untuk AMAN).
4. **Keputusan:** Kategori dengan skor tertinggi yang akan dipilih menjadi hasil akhir!

---

## 3. Sekilas Tentang Dataset

Model klasifikasi ini membutuhkan data latih. Salah satu dataset publik yang paling populer digunakan untuk eksperimen ini adalah **SMS Spam Collection Dataset** (bisa ditemukan di platform seperti *Kaggle*).

**Karakteristik Dataset:**
* **Format:** Hardcoded message langsung di code nya
* **Struktur:** Umumnya terdiri dari dua kolom utama:
  * `v1` (Label): Kategori pesan, isinya `spam` atau `ham` (aman/bukan spam).
  * `v2` (Teks): Isi pesan aslinya.

**Contoh isi data:**
| Label (`v1`) | Isi Pesan (`v2`) |
| :--- | :--- |
| `ham` | *Ok lar... Joking wif u oni...* |
| `spam` | *Free entry in 2 a wkly comp to win FA Cup final tkts...* |
| `ham` | *U dun say so early hor... U c already then say...* |

---

## 4. Cara Membaca Evaluasi Model

Setelah model selesai dilatih dan diuji, kita akan mendapatkan metrik evaluasi. Berikut adalah panduan membaca hasil performa model (contoh kasus menggunakan akurasi **98.39%**):

* **Accuracy (Akurasi):** Dari keseluruhan data yang diuji, berapa persentase tebakan yang benar? (Contoh: dari 100 email, 98 ditebak dengan tepat).
* **Precision:** Ketika model berteriak *"Ini SPAM!"*, seberapa akurat teriakan itu? (Skor 0.98 berarti 98% tebakan spam-nya memang benar-benar spam).
* **Recall:** Dari semua email SPAM yang *sebenarnya* ada di dataset, berapa banyak yang berhasil ditangkap oleh model? (Skor 0.89 berarti 89% spam berhasil ditangkap, sisanya 11% lolos ke *inbox* pengguna).

### Membaca Confusion Matrix
*Confusion Matrix* membantu kita melihat di mana model melakukan kesalahan:

| | Ditebak AMAN | Ditebak SPAM |
|---|:---:|:---:|
| **Aslinya AMAN** | 964 (Benar) | 2 (Salah Tebak) |
| **Aslinya SPAM** | 16 (Lolos) | 133 (Benar) |

* **True Negative (964):** Email aman yang *sukses* ditebak aman.
* **True Positive (133):** Email spam yang *sukses* ditebak spam.
* **False Positive (2):** Email aman tapi *salah tebak* masuk folder spam (menyebalkan bagi user).
* **False Negative (16):** Email spam yang *lolos* masuk ke kotak masuk utama (berbahaya).

### Berikut adalah hasil Test Run

tarok screenshot
