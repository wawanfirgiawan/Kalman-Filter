# `Beberapa Pertanyaan tentang Implemntasi Kalman Filter pada Yolo pada tracking atau counting kendaraan`
### **1. Apa Fungsi dari Kalman Filter?**

#### ✅ **Jawaban Singkat**:

Kalman Filter adalah algoritma matematis yang digunakan untuk **memperkirakan keadaan suatu sistem secara akurat dari data yang memiliki noise (gangguan)**.

---

### 🧠 **Dalam Konteks Deteksi & Pelacakan Kendaraan**:

Kalman Filter berfungsi untuk:

* 🔄 **Memprediksi** posisi objek (kendaraan) pada frame berikutnya berdasarkan posisi sebelumnya.
* 🧹 **Menyaring (filtering)** data deteksi dari YOLO yang kadang tidak stabil akibat:

  * Deteksi hilang (karena kendaraan tertutup),
  * Perubahan pencahayaan,
  * Noise visual.

---

### 🔍 **Contoh Kasus Nyata**:

Misal YOLO mendeteksi mobil di frame ke-5, tapi **tidak mendeteksi** di frame ke-6 karena mobil tertutup tiang.
👉 Kalman Filter akan tetap **memperkirakan posisi mobil** di frame ke-6 agar tracking tetap berjalan lancar.
Setelah YOLO mendeteksi lagi di frame ke-7, Kalman akan **menyesuaikan (update)** prediksi.

---

### 📦 Ringkasan Fungsi Kalman Filter:

| Fungsi Kalman Filter | Penjelasan Singkat                                                           |
| -------------------- | ---------------------------------------------------------------------------- |
| Prediksi             | Menebak posisi kendaraan selanjutnya dari posisi sebelumnya                  |
| Koreksi (Update)     | Mengoreksi tebakan tadi jika ada data baru dari YOLO                         |
| Pelacakan Stabil     | Menjaga agar objek tetap "terlacak" walaupun YOLO sempat tidak mendeteksinya |
| Penghalus Gerakan    | Membuat lintasan objek tampak lebih halus dan realistis di video             |

---

Bagus! Sekarang kita bahas:

---

### **2. Bagaimana Cara Kerja Kalman Filter?**

#### 📌 **Inti Utama**:

Kalman Filter bekerja dalam **dua tahap utama** pada setiap frame video:

> **Prediksi ➤ Update (Koreksi)**

---

### 🔁 Tahap 1: **Prediksi**

* Berdasarkan informasi sebelumnya (posisi, kecepatan),
* Kalman Filter **memperkirakan** posisi kendaraan di frame berikutnya.

📌 Misal:

* Di frame ke-5, mobil ada di titik (100, 200), kecepatan ke kanan.
* Maka di frame ke-6, Kalman akan prediksi posisinya mungkin (110, 200).

🧠 Ini disebut **motion model** → diasumsikan gerak linier.

---

### 🎯 Tahap 2: **Update (Koreksi)**

* Saat YOLO mendeteksi posisi baru (observasi),
* Kalman **menggabungkan prediksi** dengan **observasi dari YOLO**.
* Hasil akhirnya adalah posisi yang lebih akurat.

📌 Jika YOLO tidak mendeteksi (misalnya objek terhalang),

* Kalman tetap mempertahankan posisi prediksi.

---

### 🧮 Komponen Utama (secara matematis):

| Simbol | Nama                    | Fungsi                                                                   |
| ------ | ----------------------- | ------------------------------------------------------------------------ |
| `x`    | State vector            | Posisi dan kecepatan kendaraan (misalnya `[pos_x, vel_x, pos_y, vel_y]`) |
| `F`    | State transition matrix | Menentukan bagaimana `x` berubah dari frame ke frame                     |
| `P`    | Error covariance matrix | Menyatakan seberapa pasti kita terhadap estimasi posisi                  |
| `Q`    | Process noise           | Error karena gerakan kendaraan tidak selalu linier/perfek                |
| `R`    | Measurement noise       | Error dari hasil deteksi YOLO                                            |
| `H`    | Observation matrix      | Menyambungkan hasil deteksi ke `x`                                       |

---

### 🔁 Ilustrasi Sederhana:

```plaintext
Frame ke-1
↓
Prediksi posisi kendaraan selanjutnya
↓
YOLO mendeteksi kendaraan (jika ada)
↓
Kalman membandingkan hasil YOLO dengan prediksi
↓
Kalman mengoreksi posisi
↓
Gunakan posisi baru → ulangi untuk frame selanjutnya
```

---

### 📌 Kesimpulan:

Kalman Filter **tidak menggantikan YOLO**, tapi:

> **Melengkapi YOLO dengan pelacakan posisi yang cerdas**, bahkan ketika YOLO tidak bisa mendeteksi objek dalam frame.

---

### **3. Kapan Suatu Kendaraan Dikatakan *Terhitung* (Counted)?**

#### ✅ **Definisi Sederhana**:

Kendaraan dikatakan **terhitung (counted)** **satu kali** jika:

> Titik tengah (centroid) dari deteksi kendaraan **melewati garis hitung (counting line)** dan **belum pernah dihitung sebelumnya**.

---

### 🧩 Dalam Praktik (Kode Python-nya):

```python
if line_y - 5 <= center_y <= line_y + 5:  # kendaraan menyentuh garis deteksi
    if track_id not in counted_objects:  # belum pernah dihitung sebelumnya
        counted_objects[track_id] = True
        vehicle_counts[class_name] += 1
```

---

### 🧠 Komponen Penting:

| Komponen          | Fungsi                                                                |
| ----------------- | --------------------------------------------------------------------- |
| `line_y`          | Koordinat Y dari garis horizontal (misal: 400)                        |
| `center_y`        | Titik tengah dari bounding box kendaraan                              |
| `track_id`        | ID unik dari objek (hasil tracking Kalman atau BoT-SORT)              |
| `counted_objects` | Set/daftar kendaraan yang **sudah dihitung**, agar tidak double count |

---

### 🧠 Kenapa Butuh `track_id`?

Tanpa tracking ID, setiap kendaraan bisa dihitung berkali-kali saat tetap berada di garis.
Dengan **tracking ID**, kita tahu:

> “Ini kendaraan yang sama yang sudah dihitung sebelumnya.”

---

### 🧪 Contoh Visual:

```
Frame 1: 🚗 (ID: 12) belum sampai garis → tidak dihitung
Frame 2: 🚗 (ID: 12) tepat di garis → dihitung
Frame 3: 🚗 (ID: 12) sudah lewat garis → tidak dihitung lagi (karena ID 12 sudah tercatat)
```

---

### 🧾 Hasil Akhir:

```plaintext
Total Cars : 5
Total Buses: 2
Total Trucks: 3
```

---

### **4. Bagaimana Matriks Kalman Filter Dihitung? Kapan Nilainya True atau False?**

Kalman Filter menggunakan **matriks-matriks** untuk menghitung prediksi dan update. Mari kita jabarkan secara ringkas dan intuitif:

---

### 🔢 Matriks yang Digunakan:

| Notasi | Nama                         | Fungsi                                                                                         |
| ------ | ---------------------------- | ---------------------------------------------------------------------------------------------- |
| `x`    | State vector                 | Menyimpan posisi dan kecepatan kendaraan → contoh: `[pos_x, vel_x, pos_y, vel_y]`              |
| `F`    | State transition matrix      | Menggambarkan perubahan state antar waktu (frame) → misalnya, posisi baru = posisi + kecepatan |
| `P`    | Covariance matrix            | Menyimpan ketidakpastian terhadap prediksi state                                               |
| `Q`    | Process noise covariance     | Menyatakan error dari gerakan → kendaraan bisa tidak selalu bergerak lurus                     |
| `H`    | Observation matrix           | Menghubungkan hasil deteksi YOLO ke vektor state (`x`)                                         |
| `R`    | Observation noise covariance | Error dalam pengukuran (misalnya, YOLO mendeteksi kurang tepat)                                |
| `z`    | Observation vector           | Data hasil deteksi YOLO: `[pos_x, pos_y]`                                                      |

---

### 🔁 Langkah Komputasi:

#### 1. **Prediksi**:

```python
x = F @ x      # prediksi posisi baru
P = F @ P @ F.T + Q  # prediksi ketidakpastian
```

#### 2. **Update (Jika ada observasi baru)**:

```python
y = z - H @ x             # error antara deteksi dan prediksi
S = H @ P @ H.T + R       # ketidakpastian gabungan
K = P @ H.T @ np.linalg.inv(S)  # Kalman Gain
x = x + K @ y             # update posisi
P = (I - K @ H) @ P       # update ketidakpastian
```

---

### ⚖️ Kapan `True` dan `False`?

* Jika deteksi YOLO **ada** dan akurat → digunakan untuk **update state**.
* Jika YOLO **tidak mendeteksi** (false detection atau hilang karena occlusion), maka:

  * **Update dilewati**
  * **Prediksi tetap dijalankan**
  * Maka hasil tracking tetap lanjut walau YOLO “gagal sementara”.

✅ Ini yang menjadikan Kalman Filter **stabil dan kuat terhadap gangguan visual**.

---

### **5. Car, Truck, dan Bus Itu Apa?**

#### 📘 Berdasarkan **COCO Dataset** (yang digunakan oleh YOLOv8):

| ID Kelas | Nama Kelas | Penjelasan Singkat                                          |
| -------- | ---------- | ----------------------------------------------------------- |
| `2`      | `Car`      | Kendaraan kecil pribadi seperti sedan, hatchback            |
| `5`      | `Bus`      | Kendaraan besar untuk angkut orang (misal bus kota)         |
| `7`      | `Truck`    | Kendaraan besar untuk angkut barang (pickup, truk logistik) |

---

### 🔎 Kenapa ini penting?

Supaya saat program memfilter deteksi, kita bisa memilih hanya kelas-kelas tertentu:

```python
classes = [2, 5, 7]  # Car, Bus, Truck
```

Maka yang dihitung adalah kendaraan **bermotor besar**, bukan sepeda atau orang.

---

## **6. Bagaimana Pengujian Sistem: Apa Maksud dari Actual, Predict, dan Unvalid?**

Pengujian dilakukan untuk mengukur **seberapa akurat sistem mendeteksi dan menghitung kendaraan**. Dalam konteks ini, kita membandingkan:

| Istilah   | Makna dalam Pengujian Sistem                                                  |
| --------- | ----------------------------------------------------------------------------- |
| `Actual`  | Jumlah kendaraan **yang sebenarnya muncul** dalam video (ground truth)        |
| `Predict` | Jumlah kendaraan yang **terdeteksi dan dihitung oleh sistem** (YOLO+tracking) |
| `Unvalid` | Jumlah kendaraan yang **tidak terdeteksi atau salah deteksi** oleh sistem     |

---

### 📋 Contoh Tabel Hasil:

| Jenis Kendaraan | Actual (Ground Truth) | Predict (Deteksi Sistem) | Unvalid (Error) |
| --------------- | --------------------- | ------------------------ | --------------- |
| Car             | 100                   | 97                       | 3               |
| Bus             | 25                    | 24                       | 1               |
| Truck           | 30                    | 28                       | 2               |

---

### 🧠 Penjelasan:

* **Actual**:

  * Didapat dari **penghitungan manual** oleh pengamat atau anotasi frame-by-frame pada video.
  * Ini adalah “fakta lapangan”.

* **Predict**:

  * Output dari sistem YOLO + Kalman / ByteTrack.
  * Dihitung berdasarkan logika crossing line dan ID tracking.

* **Unvalid**:

  * Error = `Actual - Predict` (dalam konteks ini hanya **false negative** / kendaraan tidak terdeteksi).
  * Bisa disebabkan oleh:

    * Occlusion (terhalang kendaraan lain)
    * Kendaraan terlalu kecil atau terlalu cepat
    * Deteksi yang tidak stabil

---

### 📊 Tujuan Pengujian:

1. **Evaluasi akurasi sistem**

   * Seberapa dekat hasil deteksi dengan kenyataan
   * Mengukur *recall* sistem

2. **Menemukan kelemahan**

   * Frame mana yang sering gagal
   * Kelas kendaraan apa yang sering miss

3. **Bandingkan metode**

   * Misalnya: YOLO vs YOLO + Kalman → mana yang lebih baik prediksinya

---

## **7. Visualisasi Hasil Pengujian Aktual (Hasil vs Prediksi)**

Visualisasi hasil pengujian dilakukan untuk **membandingkan antara jumlah kendaraan sebenarnya (actual)** dengan **jumlah kendaraan yang terdeteksi oleh sistem (predict)**. Tujuan utama visualisasi ini adalah untuk:

* Menilai performa sistem secara visual
* Mengetahui kelas kendaraan mana yang paling sering salah deteksi
* Menyampaikan hasil ke pihak non-teknis secara lebih mudah

---

### 🔢 Contoh Data Hasil Pengujian:

| Kendaraan | Actual | Predicted | Unvalid |
| --------- | ------ | --------- | ------- |
| Car       | 100    | 97        | 3       |
| Bus       | 25     | 24        | 1       |
| Truck     | 30     | 28        | 2       |

---

### 📊 Visualisasi dengan Diagram Batang

```python
import matplotlib.pyplot as plt

# Data dummy
labels = ['Car', 'Bus', 'Truck']
actual = [100, 25, 30]
predicted = [97, 24, 28]
unvalid = [a - p for a, p in zip(actual, predicted)]

x = range(len(labels))
width = 0.3

# Buat diagram batang
plt.figure(figsize=(8, 5))
plt.bar(x, actual, width=width, label='Actual', color='skyblue')
plt.bar([i + width for i in x], predicted, width=width, label='Predicted', color='green')
plt.bar([i + 2*width for i in x], unvalid, width=width, label='Unvalid', color='red')

# Label dan format
plt.xticks([i + width for i in x], labels)
plt.ylabel('Jumlah Kendaraan')
plt.title('Perbandingan Actual, Predict, dan Unvalid')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```

---

### ✅ Kesimpulan Visualisasi:

* Jika selisih antara `actual` dan `predict` besar, maka performa sistem perlu diperbaiki.
* Visualisasi ini membantu evaluasi sistem sebelum deployment atau integrasi lapangan.

---

## **8. Apakah Codingan Video untuk Visualisasi Aktual Menggunakan Kalman Filter?**

Jawaban: **YA.**

* Jika kamu **secara manual mengimpor dan menggunakan `KalmanFilter` dari `filterpy`**, contohnya:

  ```python
  from filterpy.kalman import KalmanFilter
  ```

  Maka ya, **sistem menggunakan Kalman Filter**.

---

Cek di kode:

* Apakah ada `from filterpy.kalman import KalmanFilter`?

  * Jika **ya** → Kalman digunakan
  * Cek pada folder **/content/drive/MyDrive/Program/VachileDetection/ultralytics/ultralytics/trackers/utils**

---
