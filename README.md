# Jarkom-Modul-3-A08-2022

| **No** | **Nama**                   | **NRP**    |
| ------ | -------------------------- | ---------- |
| 1      | Dhani Rizki A. C. T. P.    | 5025201226 |
| 2      | Khariza Azmi Alfajira H.   | 5025201044 |
| 3      | Farros Hilmi Syafei        | 5025201012 |

## **Nomor 1**
**Loid bersama Franky berencana membuat peta tersebut dengan kriteria WISE sebagai DNS Server, Westalis sebagai DHCP Server, Berlint sebagai Proxy Server** <br>

## **Nomor 2**
**Ostania sebagai DHCP Relay** <br>

## **Nomor 3**
**Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server. Client yang melalui Switch1 mendapatkan range IP dari [prefix IP].1.50 - [prefix IP].1.88 dan [prefix IP].1.120 - [prefix IP].1.155** <br>

## **Nomor 4**
**Semua client yang ada HARUS menggunakan konfigurasi IP dari DHCP Server. Client yang melalui Switch3 mendapatkan range IP dari [prefix IP].3.10 - [prefix IP].3.30 dan [prefix IP].3.60 - [prefix IP].3.85** <br>

## **Nomor 5**
**Client mendapatkan DNS dari WISE dan client dapat terhubung dengan internet melalui DNS tersebut.** <br>

## **Nomor 6**
**Lama waktu DHCP server meminjamkan alamat IP kepada Client yang melalui Switch1 selama 5 menit sedangkan pada client yang melalui Switch3 selama 10 menit. Dengan waktu maksimal yang dialokasikan untuk peminjaman alamat IP selama 115 menit.** <br>

## **Nomor 7**
**Loid dan Franky berencana menjadikan Eden sebagai server untuk pertukaran informasi dengan alamat IP yang tetap dengan IP [prefix IP].3.13** <br>

## **Nomor 8**
**SSS, Garden, dan Eden digunakan sebagai client Proxy agar pertukaran informasi dapat terjamin keamanannya, juga untuk mencegah kebocoran data.** <br>
**Pada Proxy Server di Berlint, Loid berencana untuk mengatur bagaimana Client dapat mengakses internet. Artinya setiap client harus menggunakan Berlint sebagai HTTP & HTTPS proxy** <br><br>
Rubah proxy http pada SSS, Garden, dan Eden menggunakan command export http_proxy=”http://[ALAMAT IP SERVER PROXY]:[PORT SERVER PROXY]” & export https_proxy=”[sama seperti sebelumnya]”, dalam kasus kami command tersebut menjadi export http_proxy=”http://10.3.2.3:8080” & https_proxy=”http://10.3.2.3:8080”.

## **Nomor 9**
**Client hanya dapat mengakses internet diluar (selain) hari & jam kerja (senin-jumat 08.00 - 17.00) dan hari libur (dapat mengakses 24 jam penuh)** <br><br>
Pertama kita perlu membuat file squid.conf di /etc/squid, lalu kita isi seperti berikut:
![alt text](https://github.com/ObligatedUsername/Jarkom-Modul-3-A08-2022/blob/main/assets/No.9/9a.png) <br>

Lalu untuk membatasi akses internet tergantung variabel” tertentu dapat digunakan direktif acl, untuk soal ini dapat digunakan acl tipe time, dengan direktif lengkap untuk hari dan jam kerja: acl WORKING_TIME time MTWHF 08:00-17:00, dan menggunakan direktif http_access deny WORKING_TIME untuk melarang akses internet selama jam & hari kerja. Setelah itu restart service squid dengan service squid restart.

## **Nomor 10**
**Adapun pada hari dan jam kerja sesuai nomor 8, client hanya dapat mengakses domain loid-work.com dan franky-work.com (IP tujuan domain dibebaskan)** <br><br>
Melanjutkan dari soal sebelumnya, kita hanya perlu menambahkan direktif acl baru dengan tipe dstdomain, atau domain tujuan, seperti berikut: acl ALLOW_LIST dstdomain “/etc/squid/allowed.acl”. Lalu kita buat file baru bername allowed.acl dan dipindah sesuai letak path pada acl. Kita isi allowed.acl dengan loid-work.com & franky-work.com seperti berikut:
![alt text](https://github.com/ObligatedUsername/Jarkom-Modul-3-A08-2022/blob/main/assets/No.10/10a.png) <br>

Setelah itu restart squid.

## **Nomor 11**
**Saat akses internet dibuka, client dilarang untuk mengakses web tanpa HTTPS. (Contoh web HTTP: http://example.com)** <br><br>
Kita hanya perlu melarang akses port 80 menggunakan direktif acl HTTPPORT port 80 lalu menambahkan http_access deny !WORKING_TIME HTTPPORT sehingga saat bukan jam kerja, hanya dapat mengakses web menggunakan HTTPS.

## **Nomor 12**
**Agar menghemat penggunaan, akses internet dibatasi dengan kecepatan maksimum 128 Kbps pada setiap host (Kbps = kilobit per second; lakukan pengecekan pada tiap host, ketika 2 host akses internet pada saat bersamaan, keduanya mendapatkan speed maksimal yaitu 128 Kbps)** <br><br>
Untuk melihat kecepatan maksimum akses internet, kita menggunakan speedtest-cli yang dapat diinstall menggunakan apt-get install speedtest-cli, dan juga melakukan command export PYTHONHTTPSVERIFY=0 agar speedtest dapat dijalankan. Lalu kita gunakan direktif” sebagai berikut:
![alt text](https://github.com/ObligatedUsername/Jarkom-Modul-3-A08-2022/blob/main/assets/No.12/12a.png) <br>

kita ingin membuat delay pool sebanyak 1 dengan tipe kelas 1 dan parameter 16000/64000 ( 16000\*8 = 128 Kbps ), lalu kita atur akses untuk semua. Setelah restart service squid, kita lakukan speedtest –secure (karena akses internet melalui http sudah diblokir) dan didapat:
![alt text](https://github.com/ObligatedUsername/Jarkom-Modul-3-A08-2022/blob/main/assets/No.12/12b.png) <br>

Bisa dilihat download speed sekitar 128 Kbps.

## **Nomor 13**
**Setelah diterapkan, ternyata peraturan nomor 11 mengganggu produktifitas saat hari kerja, dengan demikian pembatasan kecepatan hanya diberlakukan untuk pengaksesan internet pada hari libur** <br>
Kita hanya perlu membuat direktif acl tipe time baru: acl WEEKEND_TIME time SA 00:00-24:00, lalu mengubah delay_access dari all menjadi WEEKEND_TIME. Jika tanggal sistem berada pada hari Sabtu dan Minggu, akan didapat kecepatan seperti sebelumnya, sedangkan saat jam & hari kerja, tidak akan dapat dilakukan speedtest karena akses diblokir. Apabila dilakukan speedtest –secure pada hari kerja tetapi di luar jam kerja, maka akan didapat kecepatan seperti normal seperti berikut:
![alt text](https://github.com/ObligatedUsername/Jarkom-Modul-3-A08-2022/blob/main/assets/No.13/13a.png) <br>
