---
layout: post
title: VLAN Mikrotik
excerpt: "Virtual Local Area Network (VLAN) adalah metode Layer 2 yang mengijinkan beberapa Virtual LAN pada satu interface fisik (ethernet, nirkabel, dll.), Memberikan ability untuk memisahkan LAN secara efisien."
date: 2018/04/27
---

#### Pendahuluan
***
Virtual Local Area Network (VLAN) adalah metode Layer 2 yang memungkinkan beberapa Virtual LAN pada satu interface fisik (ethernet, nirkabel, dll.), Memberikan ability untuk memisahkan LAN secara efisien.

Anda bisa menggunakan MikroTik RouterOS (serta Cisco IOS, Linux dan sistem router lainnya) untuk mark paket-paket ini serta untuk accept dan route yang dimark.

Karena VLAN berkerja pada OSI Layer 2, VLAN dapat digunakan sama seperti interface jaringan lainnya tanpa batasan apa pun. VLAN juga berjalan mulus melewati Ethernet Bridge biasa.

#### 802.1Q
***
Protokol yang paling umum digunakan untuk Virtual LAN (VLAN) adalah IEEE 802.1Q. Ini adalah protokol enkapsulasi standar yang mendefinisikan bagaimana memasukkan identifier four-byte VLAN  ke header Ethernet. (lihat Gambar berikut:)
![Gambar Header VLAN](/blog/images/Image12001.gif)<br>
Source: [https://wiki.mikrotik.com/images/f/fb/Image12001.gif](https://wiki.mikrotik.com/images/f/fb/Image12001.gif)

Setiap VLAN diperlakukan sebagai subnet terpisah. Ini berarti bahwa secara default, host dalam VLAN tertentu tidak dapat berkomunikasi dengan host yang merupakan anggota VLAN lain, meskipun mereka terhubung dalam switch yang sama. Jadi jika Anda ingin komunikasikan inter-VLAN, maka Anda memerlukan router. RouterOS mendukung hingga 4095 interfaces VLAN, masing-masing dengan ID VLAN yang unik, per interfaces. Prioritas VLAN juga dapat digunakan dan dimanipulasi.

Ketika VLAN menjangkau lebih dari satu switch, link inter-switch harus menjadi 'trunk', di mana paket-paket ditandai untuk menunjukkan VLAN darimana mereka berasal. Sebuah trunk membawa traffic multiple-VLAN; ini seperti link point-to-point yang membawa paket-paket yang ditandai antar switch atau antara switch dan router.<br>
![Vlan]({{"/images/vlan.png" | absolute_url}})

#### Q-in-Q
***
Aslinya 802.1Q hanya mengizinkan satu header vlan, sedangkan Q-in-Q di sisi lain memungkinkan dua atau lebih header vlan. Di RouterOS Q-in-Q dapat dikonfigurasi dengan menambahkan satu interface vlan di atas yang lain. Contoh:
```bash
/interface vlan
add name=vlan1 vlan-id=11 interface=ether1
add name=vlan2 vlan-id=12 interface=vlan1
```
Jika ada paket yang dikirim melalui interfaces 'vlan2', dua tag vlan akan ditambahkan ke header ethernet - '11' dan '12'.

![Iklan]({{"/images/ads.jpg" | absolute_url}})

#### Properties
***

| Property  | Penjelasan |
|:----------|:-----------|
| `arp` (`disabled | enabled | proxy-arp | reply-only`; Default: `enabled`) | Mode Address Resolution Protocol |
| `interface` (`name`; Default: ) | Nama interfaces fisik teratas dari VLAN yang work |
| `l2mtu` (`integer`; Default: )  | Layer2 MTU. Untuk VLAN, nilai ini tidak dapat dikonfigurasi |

**... To be continue**



