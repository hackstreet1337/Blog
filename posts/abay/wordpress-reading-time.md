---
featured_image: "/uploads/abay/reading-time.webp"
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
title: WordPress Reading Time
date: 2021-06-20T17:00:00+07:00
author: abay
tags:
- php
- wordpress
categories: programming

---
Pengen nyoba niruin fitur _reading time_ dari Medium di WordPress. Sebenernya ada beberapa plugins yang bisa langsung pake, tapi kali ini bikin yang tanpa plugin.

Buka dashboard, ke halaman **Tampilan** > **Edit Tema**, cari file `single.php`. Tambahin kode:

    function reading_time($content) {
    	// ambil konten
    	$the_content = $content;
    	// hitung jumlah kata
    	$words = str_word_count( strip_tags( $the_content ) );
    	// pembulatan dan pembagian 200 kata per menit
    	$minute = floor( $words / 200 );
    	// hitung total waktu baca
    	$estimate = $minute . ' minute' . ( $minute == 1 ? '' : 's' );
    	// tampilkan output
    	$output = $estimate . ' read.';
    	// return hasil estimasi waktu baca
    	return $output;
    }

Panggil functions `reading_time` di sekitar tanggal post dan/atau author post, contoh:

    <div class="entry-meta">
    	<?php
    	incise_posted_on();
    	incise_author_outside_loop();
    	echo " • " . reading_time(get_the_content());
    	?>
    </div>

Function [`get_the_content()`](https://developer.wordpress.org/reference/functions/get_the_content/) buat ambil isi konten. Cara ini juga bisa dipake di homepage WordPress—saat looping posts. Tinggal `get_the_content` sama function ambil konten by `post_id` nya aja.