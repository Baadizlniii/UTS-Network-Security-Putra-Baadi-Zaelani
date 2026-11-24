# LAPORAN UTS NETWORK SECURITY
## Kasus 5 — Dokumentasi Tugas 1–7

**Nama:** Putra Baa'di Zaelani
**NIM:** 24552011131
**Kelas:** TIF RP 24 E CNS
**Mata Kuliah:** Keamanan Jaringan

---

## Penentuan Kasus

Jumlah digit NIM:

`2 + 4 + 5 + 5 + 2 + 0 + 1 + 1 + 1 + 3 + 1 = 25`

`25 mod 10 = 5`

Dengan demikian, kasus yang dikerjakan adalah **Kasus 5**.

## Parameter Praktikum

| Parameter | Nilai |
|---|---|
| Container target | `target-k5` |
| Volume | `vol-k5` |
| File laporan | `laporan-k5.txt` |
| Domain DNS | `k5.exfil.lab` |

Seluruh pengujian dilakukan pada lingkungan laboratorium praktikum.

---

# TUGAS 1 — DOCKER DAN VOLUME
## Pertemuan 1

### 1. Tujuan
Memahami penggunaan Docker Volume untuk menyimpan data secara persisten dan menguji komunikasi antarcontainer menggunakan Docker Compose.

### 2. Langkah pengerjaan

1. Membuat volume menggunakan `docker volume create vol-k5`.
2. Menjalankan container Alpine dan menyimpan NIM ke `/data/laporan-k5.txt`.
3. Menghapus container pertama menggunakan `docker rm`.
4. Menjalankan container baru dengan volume yang sama, kemudian membaca file menggunakan `cat`.
5. Menjalankan layanan lab menggunakan Docker Compose.
6. Menguji komunikasi attacker ke target dengan mengakses `http://target`.
7. Membersihkan container dan network setelah praktikum selesai.

### 3. Hasil pengujian
Data NIM berhasil disimpan pada Docker Volume dan tetap dapat dibaca setelah container pertama dihapus. Hal ini menunjukkan bahwa volume menyimpan data secara terpisah dari siklus hidup container.

Pengujian komunikasi antarcontainer juga dilakukan dengan mengakses layanan Nginx dari container attacker.

### 4. Kesimpulan
Docker Volume berguna untuk mempertahankan data meskipun container dihapus. Docker Compose mempermudah pengelolaan beberapa container dan komunikasi dalam jaringan laboratorium.

---

# TUGAS 2 — KRIPTOGRAFI
## Pertemuan 2

### 1. Tujuan
Memahami enkripsi dan dekripsi menggunakan AES-256-CBC, pemeriksaan integritas dengan SHA-256, serta pembuatan sertifikat digital.

### 2. Langkah pengerjaan

1. Membuat file laporan sebagai data awal.
2. Mengenkripsi file menggunakan OpenSSL dengan algoritma AES-256-CBC.
3. Menggunakan PBKDF2 dan salt dalam proses enkripsi.
4. Melakukan dekripsi terhadap file terenkripsi.
5. Menghitung hash SHA-256 file asli dan hasil dekripsi.
6. Membuat sertifikat X.509 menggunakan pasangan kunci RSA 2048-bit dengan CN sesuai domain kasus:
   ```bash
   docker run --rm -v "$PWD":/work uts-openssl sh -c \
     'openssl req -x509 -newkey rsa:2048 -nodes \
      -keyout key-k5.pem -out cert-k5.pem -days 30 \
      -subj /CN=k5.exfil.lab && \
      openssl x509 -in cert-k5.pem -noout -subject -issuer -dates'
   ```
7. Memeriksa subject, issuer, dan masa berlaku sertifikat.

### 3. Hasil pengujian
Berdasarkan output praktikum, hash SHA-256 file asli dan hasil dekripsi terlihat sama. Hal tersebut menunjukkan bahwa hasil dekripsi sesuai dengan file awal.

Sertifikat digital berhasil dibuat dengan Common Name sesuai domain kasus, menggunakan RSA 2048-bit dan masa berlaku 30 hari.

| Field | Nilai |
|---|---|
| Subject | `CN=k5.exfil.lab` |
| Issuer | `CN=k5.exfil.lab` |

**Perbedaan sertifikat self-signed dan sertifikat yang ditandatangani CA:** Sertifikat self-signed memiliki Subject dan Issuer yang sama karena ditandatangani oleh kunci privat miliknya sendiri, sehingga tidak dipercaya secara otomatis oleh pihak lain. Sertifikat yang ditandatangani CA memiliki Issuer berupa pihak ketiga tepercaya, sehingga dapat diverifikasi dan dipercaya oleh sistem lain tanpa konfigurasi tambahan.

### 4. Kesimpulan
AES-256-CBC digunakan untuk menjaga kerahasiaan data. SHA-256 membantu memeriksa integritas file, sedangkan sertifikat X.509 dapat digunakan untuk mengidentifikasi suatu entitas secara digital.

---

# TUGAS 3 — ARP SPOOFING
## Pertemuan 3

### 1. Tujuan
Memahami protokol ARP, melakukan simulasi ARP spoofing pada jaringan laboratorium, dan mempelajari mitigasi menggunakan static ARP.

### 2. Langkah pengerjaan

1. Membuat jaringan khusus praktikum dengan nama `lab-arp-k5`.
2. Menjalankan container korban, gateway, dan penyerang.
3. Memeriksa alamat IP setiap container menggunakan `docker inspect`.
4. Melihat tabel ARP korban menggunakan `arp -a` sebelum spoofing.
5. Menjalankan ARP spoofing dari penyerang ke korban:
   ```bash
   arpspoof -i eth0 -t 172.18.0.2 172.18.0.3
   ```
6. Memeriksa kembali tabel ARP korban setelah beberapa detik.
7. Menerapkan static ARP untuk menetapkan pemetaan IP gateway ke MAC address yang benar.
8. Memeriksa kembali tabel ARP untuk memastikan konfigurasi mitigasi.

### 3. Hasil pengujian
Alamat IP yang diamati pada praktikum:
- Korban: `172.18.0.2`
- Gateway: `172.18.0.3`
- Penyerang: `172.18.0.4`

| Kondisi | MAC untuk `172.18.0.3` |
|---|---|
| Sebelum spoofing | |
| Setelah spoofing | |
| Setelah mitigasi (static ARP) | `62:43:02:f2:1c:ce` (`PERM`) |

**Mitigasi lain selain static ARP:** Dynamic ARP Inspection (DAI) pada switch, yang memvalidasi pasangan IP–MAC pada setiap paket ARP terhadap basis data tepercaya dan menolak paket ARP yang tidak sesuai.

### 4. Kesimpulan
ARP spoofing merupakan teknik manipulasi pemetaan alamat IP dan MAC pada jaringan lokal. Static ARP dapat menjadi salah satu langkah mitigasi dengan menetapkan pemetaan alamat secara manual.

---

# TUGAS 4 — ANALISIS WPA2 HANDSHAKE
## Pertemuan 4

### 1. Tujuan
Memahami proses autentikasi WPA2 dan menganalisis file handshake menggunakan Aircrack-ng serta wordlist praktikum.

### 2. Langkah pengerjaan

1. Menyiapkan file capture `handshake-k5.cap`.
2. Menyiapkan wordlist `wordlist-k5.txt`.
3. Menjalankan Aircrack-ng terhadap file capture.
4. Mengamati proses pengujian kandidat kata sandi.
5. Memeriksa status hasil analisis.

### 3. Hasil pengujian
Berdasarkan hasil Aircrack-ng:
- ESSID: `Cyber`
- BSSID: `CC:B1:71:57:B2:18`
- Paket terbaca: `37.771`
- Handshake terdeteksi: `1`
- Kandidat yang diuji: `5/6 (83,33%)`
- Kecepatan pengujian: `389,41 k/s`
- Status: **KEY FOUND!**

**Empat langkah WPA2 4-way handshake:**
1. **Message 1 (AP → Client):** AP mengirim ANonce ke client.
2. **Message 2 (Client → AP):** Client menghasilkan SNonce, menghitung PTK, lalu mengirim SNonce beserta MIC ke AP.
3. **Message 3 (AP → Client):** AP mengirim GTK terenkripsi beserta MIC ke client.
4. **Message 4 (Client → AP):** Client mengonfirmasi penerimaan, komunikasi terenkripsi dapat dimulai.

### 4. Kesimpulan
Analisis handshake WPA2 membantu memahami proses autentikasi jaringan Wi-Fi. Aircrack-ng dapat digunakan untuk menguji kecocokan kandidat kata sandi terhadap data handshake yang tersedia.

---

# TUGAS 5 — ICMP DENGAN SCAPY
## Pertemuan 5

### 1. Tujuan
Memahami protokol ICMP dan menggunakan Scapy untuk mengirim paket ICMP serta mengamati respons dari host tujuan.

### 2. Langkah pengerjaan

1. Menyiapkan script Python menggunakan Scapy.
2. Menjalankan script pada lingkungan container praktikum.
3. Mengirim ICMP Echo Request ke alamat tujuan `10.89.2.2`.
4. Mengamati informasi TTL dan tipe pesan ICMP pada respons.
5. Mempelajari konsep Smurf Attack dan langkah mitigasinya.

### 3. Hasil pengujian
- Alamat tujuan: `10.89.2.2`
- Alamat pengirim: `10.89.2.5`
- TTL balasan: `64`
- Tipe ICMP: `0` (Echo Reply)

### 4. Pembahasan Smurf Attack
Smurf Attack merupakan serangan Denial of Service yang memanfaatkan ICMP Echo Request dan alamat broadcast. Serangan dapat terjadi ketika alamat IP korban dipalsukan sebagai sumber paket sehingga banyak host mengirimkan balasan kepada korban. Mitigasi dapat dilakukan dengan membatasi respons ICMP broadcast dan menerapkan penyaringan paket pada jaringan.

### 5. Kesimpulan
Scapy dapat digunakan untuk membuat dan mengirim paket jaringan secara terprogram. Praktikum ini membantu memahami ICMP Echo Request, Echo Reply, serta konsep penyalahgunaan ICMP pada Smurf Attack.

---

# TUGAS 6 — SYN FLOOD DAN TCP THREE-WAY HANDSHAKE
## Pertemuan 6

### 1. Tujuan
Memahami proses TCP Three-Way Handshake dan mempelajari simulasi SYN Flood secara terbatas pada lingkungan laboratorium.

### 2. Langkah pengerjaan

1. Menjalankan container target dan attacker menggunakan Docker Compose.
2. Memeriksa jumlah koneksi `SYN_RECV` sebelum serangan.
3. Menjalankan SYN flood di background, lalu memeriksa `SYN_RECV` selagi serangan masih berlangsung.
4. Mengamati lalu lintas TCP menggunakan `tcpdump` saat satu koneksi HTTP normal dibuat.
5. Membersihkan lingkungan praktikum setelah pengujian.

### 3. Pembahasan TCP Three-Way Handshake
1. **SYN:** Client meminta untuk memulai koneksi.
2. **SYN-ACK:** Server memberikan respons terhadap permintaan.
3. **ACK:** Client mengirimkan konfirmasi terakhir.

### 4. Hasil pengujian

| Kondisi | Jumlah `SYN_RECV` |
|---|---|
| Sebelum serangan | |
| Saat SYN Flood berlangsung | |

- Durasi simulasi: `15 detik`
- Paket yang tertangkap tcpdump:

**Mitigasi SYN flood:** menggunakan SYN cookies, yaitu server menyimpan informasi koneksi dalam bentuk cookie terenkripsi di dalam nomor urut SYN-ACK, dan baru mengalokasikan resource penuh setelah ACK balasan diverifikasi valid.

### 5. Kesimpulan
SYN Flood memanfaatkan permintaan koneksi TCP dalam jumlah besar sehingga berpotensi membebani sumber daya server. Pemantauan status koneksi dan lalu lintas jaringan dapat membantu administrator mengidentifikasi aktivitas yang tidak normal.

---

# TUGAS 7 — DNS EXFILTRATION
## Pertemuan 7

### 1. Tujuan
Memahami penyalahgunaan DNS sebagai media pengiriman data dan mempelajari cara mendeteksi pola query DNS yang mencurigakan.

### 2. Langkah pengerjaan

1. Menjalankan layanan DNS dan attacker menggunakan Docker Compose.
2. Mengubah NIM menjadi representasi hexadecimal.
3. Mengirim query DNS menggunakan `dig` ke domain praktikum.
4. Menyimpan log DNS ke file.
5. Menganalisis log menggunakan script Python.
6. Mengembalikan hexadecimal menjadi teks menggunakan `xxd -r -p`.
7. Membersihkan container dan network setelah pengujian.

### 3. Hasil pengujian

NIM yang digunakan: `24552011131`

Hasil konversi hexadecimal: `3234353532303131313331`

Query DNS yang dikirim: `3234353532303131313331.k5.exfil.lab`

Jawaban DNS: `10.0.0.99`

Hasil analisis log:
- `exfil.lab: 2`
- `google.com: 1`

Hasil decoding hexadecimal kembali menghasilkan `24552011131`, sesuai dengan NIM yang digunakan pada praktikum.

**Mengapa teknik ini sulit diblokir firewall biasa:** DNS exfiltration memanfaatkan protokol DNS yang hampir selalu diizinkan keluar oleh firewall (port 53), sehingga lalu lintasnya tampak seperti resolusi nama domain yang normal.

**Indikator deteksi:** panjang label subdomain yang tidak wajar atau frekuensi query ke satu domain yang tinggi dalam waktu singkat.

### 4. Kesimpulan
DNS Exfiltration merupakan teknik penyalahgunaan protokol DNS untuk mengirimkan data melalui query DNS. Analisis log DNS dapat membantu mendeteksi pola query yang tidak biasa. Konversi hexadecimal juga dapat digunakan untuk membaca kembali data yang dikodekan.

---

# KESIMPULAN AKHIR

Berdasarkan praktikum Kasus 5, terdapat tujuh materi keamanan jaringan yang telah dipelajari, yaitu Docker dan Volume, Kriptografi, ARP Spoofing, Analisis WPA2 Handshake, ICMP dengan Scapy, SYN Flood, dan DNS Exfiltration.

Praktikum memberikan pemahaman mengenai cara kerja teknologi jaringan, potensi kerentanan, proses pengujian, dan langkah mitigasi dalam lingkungan laboratorium.

---

# LAMPIRAN SCREENSHOT PRAKTIKUM

## Screenshot Tugas 1
![Tugas 1 - 1.png](../04-bukti-pengerjaan/screenshot/tugas-1/1.png)
![Tugas 1 - 2.png](../04-bukti-pengerjaan/screenshot/tugas-1/2.png)
![Tugas 1 - 3.png](../04-bukti-pengerjaan/screenshot/tugas-1/3.png)
![Tugas 1 - 4.png](../04-bukti-pengerjaan/screenshot/tugas-1/4.png)
![Tugas 1 - 5.png](../04-bukti-pengerjaan/screenshot/tugas-1/5.png)
![Tugas 1 - 6.png](../04-bukti-pengerjaan/screenshot/tugas-1/6.png)
![Tugas 1 - 7.png](../04-bukti-pengerjaan/screenshot/tugas-1/7.png)

## Screenshot Tugas 2
![Tugas 2 - 1.png](../04-bukti-pengerjaan/screenshot/tugas-2/1.png)
![Tugas 2 - 2.png](../04-bukti-pengerjaan/screenshot/tugas-2/2.png)

## Screenshot Tugas 3
![Tugas 3 - 1.png](../04-bukti-pengerjaan/screenshot/tugas-3/1.png)
![Tugas 3 - 2.png](../04-bukti-pengerjaan/screenshot/tugas-3/2.png)
![Tugas 3 - 3.png](../04-bukti-pengerjaan/screenshot/tugas-3/3.png)
![Tugas 3 - 4.png](../04-bukti-pengerjaan/screenshot/tugas-3/4.png)
![Tugas 3 - 5.png](../04-bukti-pengerjaan/screenshot/tugas-3/5.png)
![Tugas 3 - 6.png](../04-bukti-pengerjaan/screenshot/tugas-3/6.png)
![Tugas 3 - 7.png](../04-bukti-pengerjaan/screenshot/tugas-3/7.png)
![Tugas 3 - 8.png](../04-bukti-pengerjaan/screenshot/tugas-3/8.png)
![Tugas 3 - 9.png](../04-bukti-pengerjaan/screenshot/tugas-3/9.png)
![Tugas 3 - 10.png](../04-bukti-pengerjaan/screenshot/tugas-3/10.png)

## Screenshot Tugas 4
![Tugas 4 - 1.png](../04-bukti-pengerjaan/screenshot/tugas-4/1.png)
![Tugas 4 - 2.png](../04-bukti-pengerjaan/screenshot/tugas-4/2.png)
![Tugas 4 - 3.png](../04-bukti-pengerjaan/screenshot/tugas-4/3.png)
![Tugas 4 - 4.png](../04-bukti-pengerjaan/screenshot/tugas-4/4.png)
![Tugas 4 - 5.png](../04-bukti-pengerjaan/screenshot/tugas-4/5.png)
![Tugas 4 - 6.png](../04-bukti-pengerjaan/screenshot/tugas-4/6.png)

## Screenshot Tugas 5
![Tugas 5 - 1.png](../04-bukti-pengerjaan/screenshot/tugas-5/1.png)

## Screenshot Tugas 6
![Tugas 6 - 1.png](../04-bukti-pengerjaan/screenshot/tugas-6/1.png)
![Tugas 6 - 2.png](../04-bukti-pengerjaan/screenshot/tugas-6/2.png)
![Tugas 6 - 3.png](../04-bukti-pengerjaan/screenshot/tugas-6/3.png)
![Tugas 6 - 4.png](../04-bukti-pengerjaan/screenshot/tugas-6/4.png)
![Tugas 6 - 5.png](../04-bukti-pengerjaan/screenshot/tugas-6/5.png)
![Tugas 6 - 6.png](../04-bukti-pengerjaan/screenshot/tugas-6/6.png)

## Screenshot Tugas 7
![Tugas 7 - 1.png](../04-bukti-pengerjaan/screenshot/tugas-7/1.png)
![Tugas 7 - 2.png](../04-bukti-pengerjaan/screenshot/tugas-7/2.png)
![Tugas 7 - 3.png](../04-bukti-pengerjaan/screenshot/tugas-7/3.png)
![Tugas 7 - 4.png](../04-bukti-pengerjaan/screenshot/tugas-7/4.png)
![Tugas 7 - 5.png](../04-bukti-pengerjaan/screenshot/tugas-7/5.png)
![Tugas 7 - 6.png](../04-bukti-pengerjaan/screenshot/tugas-7/6.png)
![Tugas 7 - 7.png](../04-bukti-pengerjaan/screenshot/tugas-7/7.png)
