---
featured_image: "/uploads/justakazh/packet-sniffing.jpg"
author_description: Aku nulis karena aku pelupa
author_avatar: "/uploads/justakazh/1656477958868.jpeg"
author_facebook: https://facebook.com
author_twitter: https://twitter.com/akazh18
author_instagram: https://instagram.com
author_donation: https://paypal.com/jawagalawbgt
author_web: https://blog.hackstreet.id
title: HTTP(S) Traffic Sniffing (Bahasa Indonesia)
date: 2022-12-24T17:00:00+07:00
author: Justakazh
tags:
- http
- wireshark
- hacking
- sniffing
categories: penetration testing

---
# **INTRODUCTION**

Pada tulisan kali ini kita akan membahas tentang HTTP(S) Traffic Sniffing, disini kita akan melakukan Sniffing Traffic komunikasi dan mengetahui perbedaan HTTP dan HTTPS dalam sisi Security selain itu juga kita akan melakukan sniffing untuk mendapatkan username dan password. tulisan ini diadaptasi dari ine.com dimana ine.com menyediakan lab untuk training yang lengkap.

Untuk persiapannya sendiri disini saya menggunakan sebuah tools diantaranya adalah:

* Web Browser
* Wireshark

sebelum kita melangkah lebih lanjut terdahadap point utama tulisan ini kita harus tahu terlebih dahulu apa itu HTTP serta perbedaan antara HTTP dan HTTPS.

**HTTP** (Hyper Text Transfer Protocol) merupakan sebuah protokol jaringan application layer yang dikembangkan untuk membantu proses transfer antar komputer. Sedangkan **HTTPS** (Hyper Text Transfer Protocol Secure) hampir sama seperti HTTP bedanya adalah HTTPS mengandung huruf S yang artinya Secure, HTTP mengirimkan data dalam bentuk Clear-Text sedangkan HTTPS mengirimkan data dalam bentuk enkripsi, tentunya HTTP dapat dibaca secara jelas oleh seseorang yang melakukan Sniffing, sedangkan HTTPS tidak bisa dibaca oleh seseorang yang melakukan Sniffing.

# Traffic Sniffing

![https://miro.medium.com/max/4800/1](https://miro.medium.com/max/4800/1*g_iYqcmaeUkD1x_gHWPUuw.webp)

gambar diatas merupakan contoh bagaimana sniffing bekerja, penyerang hanya mendengarkan komunikasi atau traffic yang dikirimkan dari komputer satu ke komputer yang lainnya atau dari client ke server. disini penyerang dapat mengetahui aktifitas apa saja yang dilakukan oleh korban, bahkan penyerang dapat mengetahui informasi sensitif seperti username,password,kartu kredit dan lain sebagainya.

# Practical

![](https://miro.medium.com/max/4800/1*cUE5s8ZYj414yOukc4VQvA.webp)

pada kasus kali ini kita akan melakukan triffic sniffing dan melihat perbedaan secara langsung antara protokol HTTP dan HTTPS. pada lab yang saya gunakan disini sudah disiapkan sebuah web server dengan IP address yang berbeda diantaranya adalah

* [http://10.54.15.68](http://10.54.15.68)
* [https://10.54.15.15](https://10.54.15.15)

disini kita akan melakukan sniffing dengan menggunakan wireshark, kita bisa mendownload wireshark melalui [link ini](https://www.wireshark.org/download.html)

ketika sudah melakukan installasi terhadap wireshark silahkan jalankan wireshark

![](https://miro.medium.com/max/4800/1*zOFua4ht5GDcY6ySnslN9A.webp)

pada halaman utama wireshark kita akan ditunjukan interface yang terdapat pada jaringan kita. dikarenakan saya menggunakan vpn untuk terhubung pada lab saya saya akan memilih interface **tap0**

![](https://miro.medium.com/max/4800/1*KHzpy52N__PVU2U684fAzw.webp)

secara default wireshark akan melakukan listening secara otomatis selanjutnya mari kita akses lab yang menggunakan HTTP [http://10.54.15.68](http://10.54.15.68)

![](https://miro.medium.com/max/4800/1*EnOPyQctGuCN7jqVQKsyxw.webp)

disini kita ditunjukan sebuah halaman login page, disini saya mencoba untuk login menggunakan username dan password yang telah disediakan dari lab tersebut

![](https://miro.medium.com/max/4800/1*kszOahfzTWNE0xArg8CYsw.webp)

pada wireshark akan menampilkan sebuah informasi traffic, disini kita akan berfokus pada traffic yang menggunakan protocol HTTP seperti yang dilihat pada gambar diatas tersebut traffic tersebut melakukan pengiriman post data pada /login.php mari kita coba untuk membaca lebih detail mengenai post data yang dikirimkan tersebut dengan melakukan **follow** > **TCP Stream**

![](https://miro.medium.com/max/4800/1*ehbeFK1Q3uFDCc-y4GOhAw.webp)

pada gambar diatas tersebut kita telah mendapatkan username dan password yang dikirimkan tadi.

selanjutnya mari kita coba untuk mengakses server yang menggunakan protokol HTTPS [https://10.54.15.15](https://10.54.15.15)

![](https://miro.medium.com/max/4800/1*zfZhaLsnqt5OnrdWiE40pA.webp)

disini kita juga ditunjukan halaman login mari kita lakukan login dengan menggunakan username dan password yang telah disediakan oleh lab

![](https://miro.medium.com/max/4800/1*zdVUToy959kdwhD8A7L50g.webp)

pada wireshark dapat dilihat bahwa semua traffic yang dikirimkan tersebut telah terenkripsi.

# Kesimpulan

dari sini dapat disimpulkan bahwa protokol HTTP dapat dilakukan Sniffing dan Traffic tersebut dapat dibaca dengan jelas tidak seperti HTTPS semua traffic akan di enkripsi sehingga seseorang yang melakukan Sniffing tentu saja tidak bisa membaca data yang dikirimkan