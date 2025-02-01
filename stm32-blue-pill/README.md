# Dokumentasi Desain PCB STM32F103C8T6

## 1. Kapasitor Decoupling

### **Mengapa Diperlukan?**
- Menstabilkan tegangan suplai (**VDD**) agar tidak berfluktuasi akibat switching internal IC.
- Mengurangi noise frekuensi tinggi yang dapat mengganggu operasi mikrokontroler.
- Menyediakan arus instan saat mikrokontroler mengalami perubahan keadaan logika secara cepat.

### **Penempatan Kapasitor Decoupling**
- Harus **sedekat mungkin** dengan pin daya IC untuk mengurangi efek induktansi jalur PCB.
- Nilai umum yang digunakan:
  - **100nF (0.1µF)** untuk setiap pin **VDD**.
  - **4.7µF – 10µF** elektrolit/tantalum untuk stabilisasi tambahan.
  - Untuk filter noise frekuensi tinggi, bisa ditambahkan **1nF**.

---

## 2. Perbedaan Rangkaian Filter VDD dan VDDA

| Tegangan | Fungsi | Rangkaian Filter |
|----------|--------|-----------------|
| **VDD** | Daya utama inti mikrokontroler dan periferal digital. | Kapasitor **100nF + 4.7µF** untuk stabilisasi tegangan. |
| **VDDA** | Daya untuk bagian analog (ADC, DAC). | Filter **RC atau LC** untuk meredam noise dari VDD. |

- **VDDA lebih sensitif terhadap noise**, sehingga perlu pemfilteran tambahan menggunakan ferrite bead atau resistor.

---

## 3. Filter Ferrite Bead untuk VDDA

### **Mengapa Perlu Filter?**
- Noise dari **switching digital** di VDD bisa mempengaruhi ADC jika langsung masuk ke VDDA.
- Ferrite bead berfungsi sebagai **low-pass filter**, menghilangkan noise frekuensi tinggi.
- Rekomendasi umum:
  - **Ferrite bead 600Ω @ 100MHz** (contoh: Murata BLM18AG601SN1D).
  - **Kapasitor 100nF + 1µF** setelah ferrite bead.

### **Alternatif: Filter RC**
- Jika ferrite bead tidak tersedia, bisa menggunakan **resistor 10Ω + kapasitor 1µF** sebagai pengganti.
- Namun, penggunaan resistor bisa menyebabkan sedikit **drop tegangan pada VDDA**.

---

## 4. Sumber Noise Frekuensi Tinggi

| Sumber Noise | Penyebab | Efek pada PCB |
|-------------|---------|--------------|
| **Switching internal mikrokontroler** | Perubahan logika cepat dalam CPU & GPIO | Lonjakan arus sesaat, fluktuasi tegangan pada VDD. |
| **Regulator Switching (DC-DC Converter)** | Frekuensi switching tinggi (150kHz–1MHz) | Ripple tegangan yang bisa masuk ke VDD/VDDA. |
| **Clock dan Kristal Osilator** | Sinyal clock eksternal (8MHz, 16MHz, dll.) | Noise EMI pada jalur PCB. |
| **Desain PCB yang buruk** | Jalur suplai daya terlalu panjang, ground plane tidak optimal | Ground bounce, crosstalk antar sinyal. |
| **Periferal eksternal (motor, relay, LCD)** | Arus transien tinggi | Gangguan pada ADC, kemungkinan reset mendadak. |

### **Solusi untuk Mengurangi Noise:**
- **Gunakan kapasitor decoupling dengan benar**.
- **Gunakan ferrite bead pada VDDA untuk menghilangkan noise dari VDD**.
- **Pastikan tata letak PCB optimal dengan ground plane yang baik**.
- **Pisahkan jalur daya analog dan digital** jika memungkinkan.

---

## 5. VDDA dan VDD: Mengapa Filter Tetap Menghubungkan Ke 3.3V?

- **VDDA** dan **VDD** menggunakan sumber **3.3V yang sama**, namun **VDDA lebih sensitif terhadap noise** dari bagian digital (VDD).
- **Low-pass filter** menggunakan **ferrite bead** dan **kapasitor** bertujuan untuk **mengurangi noise frekuensi tinggi** pada **VDDA** tanpa memisahkan sumber daya sepenuhnya.
- Memisahkan sumber daya untuk VDDA dan VDD bisa dilakukan dengan menggunakan **regulator terpisah**, namun umumnya cukup dengan menggunakan filter untuk penyaringan noise.

---

## 6. Perhitungan Kapasitor untuk Crystal Oscillator (16 MHz)

### **Langkah-langkah Menghitung Kapasitor**

1. **Tentukan Nilai Kapasitansi Seri dan Parasitik dari Kristal**
   - Nilai **C_L** untuk kristal dapat ditemukan di datasheet kristal tersebut (biasanya dalam kisaran 12 pF hingga 30 pF untuk kristal 16 MHz).
   - **Kapasitansi parasitik** PCB biasanya sekitar 2 pF hingga 5 pF.

2. **Rumus Perhitungan Kapasitor Eksternal**
   - Untuk menghitung nilai kapasitor eksternal (C1 dan C2) yang digunakan di sisi kristal, gunakan rumus berikut:
   
   \[
   C_1 = C_2 = 2 \times C_L - C_{parasitik}
   \]

   Di mana:
   - **C_L** adalah kapasitan yang dibutuhkan oleh kristal (misalnya 18 pF),
   - **C_{parasitik}** adalah kapasitansi parasitik (misalnya 5 pF),
   - **C_1** dan **C_2** adalah kapasitor yang dibutuhkan.

### **Contoh Perhitungan**
Misalnya, untuk kristal 16 MHz dengan **C_L = 18 pF** dan **kapasitansi parasitik PCB = 5 pF**:

\[
C_1 = C_2 = 2 \times 18\,\text{pF} - 5\,\text{pF} = 36\,\text{pF} - 5\,\text{pF} = 31\,\text{pF}
\]

Jadi, nilai kapasitor yang digunakan adalah **31 pF** untuk C1 dan C2.

---

## 7. USB (Micro USB) Pinout

Pada desain STM32F103C8T6 yang hanya berfungsi sebagai **peripheral device** (bukan host USB), pin **Shield** dan **Pin ID** dapat **dibiarkan mengambang** atau **floating**.

### **Pin Shield**
- **Fungsi**: Pin ini digunakan untuk **grounding** dan **pelindung** terhadap interferensi elektromagnetik (EMI).
- **Koneksi pada perangkat STM32**: Pin **Shield** dapat **dibiarkan mengambang** atau tidak terhubung langsung ke ground jika perangkat hanya berfungsi sebagai **peripheral device**. Tidak perlu menghubungkannya ke ground.

### **Pin ID**
- **Fungsi**: Pin ini digunakan untuk menentukan apakah perangkat bertindak sebagai **host** atau **peripheral** dalam mode **USB OTG**.
- **Koneksi pada perangkat STM32**: Jika STM32 hanya bertindak sebagai **peripheral device**, pin **ID** dapat **dibiarkan mengambang** atau **floating**. Tidak perlu dihubungkan ke ground karena perangkat tidak akan berfungsi sebagai host USB.

Jadi, pada desain STM32 yang berfungsi sebagai **device-only**, baik pin **Shield** maupun **Pin ID** dapat dibiarkan **floating** tanpa mengganggu fungsionalitas perangkat.

Pada desain USB untuk STM32F103C8T6 yang berfungsi sebagai **peripheral device**, resistor **pull-up 1.5kΩ** perlu ditambahkan pada **pin D+** untuk memungkinkan perangkat dikenali oleh host USB.

### **Fungsi Resistor Pull-up**
- Resistor **1.5kΩ** dihubungkan antara **pin D+** dan **VBUS** (5V) untuk memberi sinyal ke host bahwa perangkat ini adalah **peripheral**.
- Pin **D-** tidak memerlukan resistor pull-up, hanya **D+** yang membutuhkan.
- Dengan resistor ini, perangkat USB dapat berkomunikasi dengan host seperti komputer atau perangkat lainnya.

### **Koneksi**
- **Pin D+** dihubungkan ke **VBUS** melalui resistor **1.5kΩ**.

### **Catatan**
- Penambahan resistor pull-up 1.5kΩ pada pin D+ ini **tidak berlaku untuk semua mikrokontroler STM32**. Namun, pada **STM32F103C8T6**, resistor ini sangat diperlukan agar perangkat dapat dikenali dan berfungsi sebagai peripheral USB.


## 8. Pull-up Resistor pada I2C

Pada komunikasi **I2C**, resistor **pull-up** diperlukan pada jalur **SDA (Serial Data)** dan **SCL (Serial Clock)** karena protokol I2C menggunakan **open-drain**. Tanpa resistor ini, jalur komunikasi bisa **floating**, yang menyebabkan kesalahan dalam transmisi data.

### **Fungsi Pull-up Resistor**
1. **Menjaga Level Logika "High"**  
   - Perangkat I2C hanya dapat menarik jalur ke LOW (GND) tetapi tidak bisa secara aktif mendorongnya ke HIGH.  
   - Resistor pull-up memastikan bahwa saat tidak ada perangkat yang menarik ke LOW, **SDA dan SCL tetap berada di level HIGH**.

2. **Menentukan Kecepatan Transisi Sinyal**  
   - Resistor pull-up mempengaruhi seberapa cepat sinyal berubah dari LOW ke HIGH.
   - Untuk **I2C Fast Mode (400kHz)**, biasanya digunakan **resistor 2.2kΩ** untuk memastikan transisi yang cepat.

3. **Mencegah Floating Signal & Error**  
   - Tanpa pull-up resistor, jalur **SDA** dan **SCL** bisa berada dalam kondisi tidak stabil (floating), menyebabkan komunikasi I2C gagal atau error.

### **Nilai Pull-up Resistor Berdasarkan Kecepatan I2C**
| Kecepatan I2C | Nilai Resistor Pull-up |
|--------------|----------------------|
| **100 kHz (Standard Mode)** | **4.7kΩ – 10kΩ** |
| **400 kHz (Fast Mode)** | **1.8kΩ – 4.7kΩ** (umumnya **2.2kΩ**) |
| **1 MHz (Fast Mode Plus)** | **1kΩ – 2.2kΩ** |
| **3.4 MHz (High-Speed Mode)** | **200Ω – 1kΩ** |

### **Cara Menghubungkan Pull-up Resistor**
- **SDA → Resistor 2.2kΩ → VCC (3.3V atau 5V, sesuai tegangan I2C)**  
- **SCL → Resistor 2.2kΩ → VCC (3.3V atau 5V, sesuai tegangan I2C)**  

*(Pastikan nilai VCC sesuai dengan level tegangan yang digunakan oleh perangkat I2C! Jika STM32 menggunakan 3.3V, maka pull-up harus ke 3.3V.)*

### **Kesimpulan**
- **Pull-up resistor sangat penting dalam komunikasi I2C** untuk menjaga kestabilan sinyal.  
- Nilai **2.2kΩ** digunakan untuk **I2C Fast Mode (400kHz)** agar transisi sinyal cepat dan stabil.  
- Tanpa resistor pull-up, **SDA dan SCL bisa floating**, menyebabkan error atau komunikasi yang tidak berjalan.  

## 9. Pemilihan Footprint Komponen

Memilih footprint yang tepat sangat penting agar komponen dapat dipasang dengan baik dan proses soldering berjalan lancar. Berikut adalah panduan dalam memilih footprint yang sesuai.

### **1. Jenis Footprint Berdasarkan Komponen**
Komponen PCB memiliki dua jenis utama berdasarkan cara pemasangannya:
- **Through-Hole (THT)** → Memiliki lubang pada PCB untuk kaki komponen. Cocok untuk solder manual.
- **Surface Mount Device (SMD)** → Tidak memiliki lubang, hanya pad tembaga. Cocok untuk reflow soldering.

Contoh package dan footprint yang sering digunakan:
| **Komponen** | **Jenis Package** | **Footprint PCB** |
|-------------|-----------------|------------------|
| **STM32F103C8T6** | LQFP-48 | LQFP-48 (7x7mm, 0.5mm pitch) |
| **Resistor SMD** | 0805 | 0805 (2.0mm x 1.25mm) |
| **Kapasitor SMD** | 0603 | 0603 (1.6mm x 0.8mm) |
| **LED SMD** | 1206 | 1206 (3.2mm x 1.6mm) |
| **Konektor USB Micro** | SMD/THT | USB-Micro-B-5P |
| **Crystal 16MHz** | HC-49S | HC-49S (Through-hole) atau SMD-3225 |
| **AMS1117-3.3V** | SOT-223 | SOT-223 (SMD) |

### **2. Faktor yang Harus Diperhatikan dalam Memilih Footprint**
1. **Sesuai dengan Datasheet Komponen**  
   - Periksa dimensi kaki, pitch (jarak antar pin), dan ukuran pad solder.  
2. **Ukuran Pad dan Lubang Solder**  
   - **THT** → Lubang harus sedikit lebih besar dari kaki komponen agar mudah dipasang.  
   - **SMD** → Pad harus cukup besar untuk soldering manual atau reflow.  
3. **Clearance Antar Komponen**  
   - Pastikan ada ruang cukup agar soldering tidak sulit.  
   - Untuk sinyal high-speed, gunakan **clearance minimal 0.2mm – 0.5mm**.  
4. **Metode Soldering**  
   - **Manual** → Gunakan footprint yang lebih besar agar lebih mudah disolder.  
   - **Reflow** → Gunakan footprint standar sesuai datasheet.  
5. **Orientasi & Polaritas Komponen**  
   - Pin 1 pada IC harus sesuai dengan marking di footprint PCB.  
   - LED, kapasitor elektrolit, dan dioda harus memiliki tanda polaritas.  

### **3. Cara Memeriksa Footprint Sebelum Produksi**
- Gunakan **3D Viewer di software PCB** (seperti KiCad atau Altium) untuk mengecek dimensi.  
- Cetak layout PCB dalam skala **1:1**, lalu tempelkan komponen fisik untuk memastikan kecocokan.  
- Periksa **clearance antar komponen** terutama pada area padat seperti konektor dan IC.  
- Pastikan **solder pad cukup besar** agar mudah disolder.  
### **4. Jenis-Jenis Package Komponen dan Penggunaannya**

| **Jenis Package** | **Deskripsi** | **Digunakan untuk** | **Gambar** |
|------------------|--------------|----------------------|------------|
| **SOIC (Small Outline IC)** | IC dengan kaki menonjol keluar di kedua sisi, lebih besar dari TSSOP. | IC dengan jumlah pin sedang, seperti op-amp atau mikrokontroler kecil. | *(Tambahkan gambar di sini)* |
| **TSSOP (Thin Shrink Small Outline Package)** | Versi lebih tipis dan rapat dari SOIC, menghemat ruang PCB. | IC kecil dengan banyak pin seperti regulator atau sensor. | *(Tambahkan gambar di sini)* |
| **QFP (Quad Flat Package)** | IC dengan kaki di keempat sisi, tersedia dalam berbagai ukuran. | STM32F103C8T6 (LQFP-48), mikrokontroler, FPGA. | *(Tambahkan gambar di sini)* |
| **QFN (Quad Flat No-Lead)** | IC tanpa kaki menonjol, hanya pad solder di bawahnya. | IC kecil dengan performa tinggi, konverter daya, sensor digital. | *(Tambahkan gambar di sini)* |
| **BGA (Ball Grid Array)** | IC dengan bola solder di bawahnya, membutuhkan reflow soldering. | Prosesor, memori flash, IC berkecepatan tinggi. | *(Tambahkan gambar di sini)* |
| **0805 (SMD Resistor/Kapasitor)** | 2.0mm x 1.25mm, mudah disolder secara manual. | Resistor dan kapasitor SMD umum. | *(Tambahkan gambar di sini)* |
| **0603 (SMD Resistor/Kapasitor)** | 1.6mm x 0.8mm, lebih kecil dari 0805. | Resistor dan kapasitor untuk desain yang lebih ringkas. | *(Tambahkan gambar di sini)* |
| **0402 (SMD Resistor/Kapasitor)** | 1.0mm x 0.5mm, sulit disolder manual. | Desain berukuran kecil dengan kepadatan tinggi. | *(Tambahkan gambar di sini)* |

💡 **Catatan**:  
- **QFP lebih mudah disolder manual dibanding QFN dan BGA** karena memiliki kaki yang terlihat.  
- **QFN dan BGA membutuhkan soldering reflow dan sulit diperbaiki secara manual**.  
- **Komponen ukuran 0805 lebih mudah disolder dibanding 0603 atau 0402**, sehingga lebih cocok untuk pemula.  

🔹 Untuk menambahkan gambar, bisa menggunakan format markdown berikut setelah mengunggah gambarnya:  
### **Kesimpulan**
- Gunakan **footprint yang sesuai datasheet** untuk menghindari kesalahan produksi.  
- Perhatikan **ukuran pad, lubang solder, dan clearance** agar proses soldering lebih mudah.  
- Lakukan pengecekan dengan **3D viewer atau cetak layout skala 1:1** sebelum memesan PCB.  
- Pastikan **IC, konektor, dan komponen dengan polaritas** memiliki marking yang jelas agar tidak salah pasang.  

# Faktor yang Mempengaruhi Kualitas Jalur pada PCB  

Desain jalur pada PCB sangat berpengaruh terhadap kualitas sinyal, terutama pada sinyal frekuensi tinggi. Berikut adalah faktor-faktor utama yang mempengaruhi kualitas jalur:  

## 1. Impedansi Jalur  

### **Apa itu Impedansi?**  
Impedansi (\(Z\)) adalah hambatan total dalam jalur PCB yang terdiri dari resistansi, induktansi, dan kapasitansi. Nilai impedansi sangat penting dalam desain **high-speed signal** dan **differential pairs**.  

### **Faktor yang Mempengaruhi Impedansi:**  
| Faktor | Pengaruh terhadap Impedansi |
|--------|-----------------------------|
| **Lebar jalur** | Jalur lebih lebar → Impedansi lebih rendah |
| **Jarak ke ground plane** | Jarak lebih dekat → Impedansi lebih rendah |
| **Konstanta dielektrik PCB (\( \varepsilon_r \))** | \( \varepsilon_r \) lebih tinggi → Impedansi lebih rendah |
| **Ketebalan tembaga** | Tembaga lebih tebal → Impedansi lebih rendah |
| **Jarak antar pasangan diferensial** | Jarak lebih dekat → Impedansi lebih rendah |

### **Dampak jika impedansi tidak sesuai:**  
- **Refleksi sinyal**, yang menyebabkan distorsi data.  
- **Penurunan kecepatan komunikasi** akibat sinyal yang tidak terdeteksi dengan baik.  
- **Ketidaksesuaian dengan standar (USB, HDMI, PCIe, dll.)**.  

---

## 2. Induktansi Jalur  

### **Apa itu Induktansi?**  
Induktansi adalah kemampuan jalur PCB untuk menyimpan energi dalam bentuk medan magnet ketika arus melewatinya.  

### **Faktor yang Meningkatkan Induktansi:**  
| Faktor | Pengaruh terhadap Induktansi |
|--------|-----------------------------|
| **Jalur lebih panjang** | Induktansi lebih tinggi |
| **Jalur lebih sempit** | Induktansi lebih tinggi |
| **Jarak lebih jauh dari ground plane** | Induktansi lebih tinggi |
| **Tidak adanya ground return path yang dekat** | Induktansi lebih tinggi |

### **Dampak Induktansi Tinggi:**  
- **Menurunkan integritas sinyal** akibat distorsi dan ringing.  
- **Menyebabkan overshoot dan undershoot pada sinyal digital**.  
- **Menimbulkan EMI (Electromagnetic Interference) yang lebih besar**.  

✅ **Solusi:**  
- Gunakan **ground plane** yang dekat dengan jalur.  
- Jaga agar jalur tetap **pendek dan lebar**.  
- Untuk sinyal high-speed, gunakan **jalur diferensial** untuk mengurangi EMI.  

---

## 3. Kapasitansi Jalur  

### **Apa itu Kapasitansi?**  
Kapasitansi adalah kemampuan jalur untuk menyimpan muatan listrik dalam medan listrik antara jalur dan ground plane.  

### **Faktor yang Meningkatkan Kapasitansi:**  
| Faktor | Pengaruh terhadap Kapasitansi |
|--------|-----------------------------|
| **Jarak kecil antara jalur dan ground plane** | Kapasitansi lebih besar |
| **Jalur lebih lebar di dekat ground plane** | Kapasitansi lebih besar |
| **Material PCB dengan konstanta dielektrik tinggi (\( \varepsilon_r \))** | Kapasitansi lebih besar |

### **Dampak Kapasitansi Tinggi:**  
- **Sinyal menjadi lebih lambat** karena waktu propagasi meningkat.  
- **Pelemahan sinyal (signal attenuation)**, terutama pada frekuensi tinggi.  
- **Cross-talk antar jalur meningkat**, menyebabkan interferensi antar sinyal.  

✅ **Solusi:**  
- Jaga jarak antara jalur dan ground plane sesuai perhitungan impedansi.  
- Gunakan **material dengan konstanta dielektrik rendah** untuk mengurangi efek kapasitif.  
- Gunakan jalur **lebih sempit** untuk mengurangi luas permukaan yang bersentuhan dengan ground plane.  

---

## 4. Resistansi Jalur  

### **Apa itu Resistansi?**  
Resistansi adalah hambatan listrik yang terjadi akibat sifat material tembaga dan panjang jalur pada PCB.  

### **Faktor yang Meningkatkan Resistansi:**  
| Faktor | Pengaruh terhadap Resistansi |
|--------|-----------------------------|
| **Jalur lebih panjang** | Resistansi lebih tinggi |
| **Jalur lebih sempit** | Resistansi lebih tinggi |
| **Ketebalan tembaga lebih tipis** | Resistansi lebih tinggi |

### **Dampak Resistansi Tinggi:**  
- **Tegangan drop lebih besar**, terutama pada jalur daya.  
- **Panas berlebih pada PCB**, yang dapat menyebabkan degradasi komponen.  
- **Penurunan performa sinyal**, terutama pada frekuensi tinggi.  

✅ **Solusi:**  
- Gunakan **jalur lebih lebar** untuk daya tinggi.  
- Gunakan **tembaga lebih tebal** (misalnya 2 oz/ft² dibandingkan 1 oz/ft²).  
- Minimalkan panjang jalur untuk mengurangi resistansi total.  

---

## 5. Kesimpulan  

Untuk mendesain PCB dengan kualitas sinyal yang baik, pertimbangkan faktor-faktor berikut:  

| Faktor | Efek Positif | Efek Negatif |
|--------|-------------|-------------|
| **Impedansi yang sesuai** | Mengurangi refleksi, kompatibel dengan standar komunikasi | Jika salah, sinyal bisa terdistorsi |
| **Induktansi rendah** | Mengurangi EMI dan ringing | Jika tinggi, menyebabkan gangguan sinyal |
| **Kapasitansi terkontrol** | Mempertahankan kecepatan sinyal | Jika tinggi, bisa memperlambat sinyal dan menyebabkan cross-talk |
| **Resistansi rendah** | Mengurangi panas dan tegangan drop | Jika tinggi, menyebabkan penurunan performa dan overheat |

**Rekomendasi umum untuk jalur PCB:**  
- Gunakan **impedansi terkendali** untuk sinyal high-speed.  
- Pastikan **ground plane selalu dekat dengan jalur** untuk mengurangi EMI dan meningkatkan return path.  
- Gunakan **jalur lebar untuk daya**, dan **jalur sempit dengan perhitungan yang sesuai untuk sinyal high-speed**.  
- Pilih material PCB yang sesuai untuk kebutuhan desain (misalnya FR4 atau Rogers untuk RF).  

Dengan memahami konsep ini, desain PCB akan lebih optimal dalam performa, daya tahan, dan kualitas sinyal. 🚀  

# Differential Pair pada Desain PCB (USB)

Pada desain PCB dengan sinyal **differential pair** seperti USB, penting untuk mengatur beberapa parameter untuk menjaga kualitas sinyal dan memastikan impedansi sesuai standar.

## 1. **Width (Lebar Jalur)**
- **Width** adalah lebar dari jalur **differential pair** (misalnya D+ dan D- untuk USB).
- Lebar jalur berpengaruh pada **impedansi** sinyal.
- Untuk USB, impedansi yang diinginkan adalah sekitar **90Ω**.
- Lebar jalur yang lebih lebar akan menghasilkan **impedansi lebih rendah**, sementara jalur yang lebih sempit menghasilkan **impedansi lebih tinggi**.
- Lebar jalur harus disesuaikan dengan **ketebalan tembaga** pada PCB dan **konstanta dielektrik** bahan PCB.

## 2. **Gap (Jarak Antar Jalur)**
- **Gap** adalah jarak antara dua jalur yang membentuk differential pair (D+ dan D-).
- Jarak yang tepat antara jalur ini penting untuk menjaga **impedansi**.
- Jika gap terlalu kecil, impedansi bisa terlalu rendah, menyebabkan **refleksi** dan **distorsi sinyal**.
- Jika gap terlalu besar, impedansi bisa terlalu tinggi, mempengaruhi kualitas sinyal.
- Pada USB, gap umumnya sekitar **0.2 mm – 0.25 mm**, tergantung pada lebar jalur dan bahan PCB.

## 3. **Via Gap (Jarak Via)**
- **Via gap** adalah jarak antara **via** yang digunakan untuk menghubungkan jalur pada layer yang berbeda.
- **Via gap** yang tepat memastikan integritas sinyal dan menghindari masalah **refleksi**.
- Penggunaan **via** dalam jalur differential pair sebaiknya diminimalisir untuk menjaga kualitas sinyal.
- Jika menggunakan via, pastikan jarak dan ukurannya sesuai dengan standar desain impedansi.

## Mengapa Hal Ini Penting?
- **Integritas sinyal** sangat penting pada desain dengan sinyal kecepatan tinggi seperti USB.
- Mengatur **width**, **gap**, dan **via gap** dengan benar membantu menjaga kualitas sinyal dan memastikan transmisi data yang stabil.
- Tanpa pengaturan yang tepat, dapat muncul masalah seperti **data corruption**, **distorsi sinyal**, atau **komunikasi USB yang tidak stabil**.

---


<!-- ## 7. Kesimpulan
- Kapasitor decoupling wajib digunakan untuk menstabilkan VDD dan mengurangi noise.
- VDDA membutuhkan filter tambahan (**RC atau LC**) untuk memastikan tegangan analog bersih.
- Noise frekuensi tinggi bisa berasal dari switching internal, clock, switching regulator, atau layout PCB yang buruk.
- Ferrite bead lebih efektif dibanding resistor dalam menyaring noise tanpa menyebabkan drop tegangan signifikan.
- **VDDA tetap terhubung dengan 3.3V yang sama dengan VDD**, namun menggunakan filter untuk mengurangi noise dari VDD.
- Untuk kristal 16 MHz, kapasitor C1 dan C2 biasanya dihitung sebagai **31 pF** jika kapasitansi parasitik sekitar 5 pF. -->
