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
| `interface` (*name*; Default: ) | Nama interfaces fisik teratas dari VLAN yang work |
| `l2mtu` (*integer*; Default: )  | Layer2 MTU. Untuk VLAN, nilai ini tidak dapat dikonfigurasi |
| `mtu` (`integer`;Default: 1500) | MTU Layer3 |
| `use-service-tag` (`yes | no`; Default: ) | 802.1ad compatible Service Tag |
| `vlan-id` (`integer: 4095`; Default: 1) | Identifier atau tag Virtual LAN yang digunakan untuk membedakan VLAN. Harus sama untuk semua komputer bahwa termasuk VLAN yang sama |

> **Catatan:**<br> 
> MTU harus diatur ke 1500 byte sama seperti pada interfaces Ethernet. Tetapi ini mungkin tidak berfungsi dengan beberapa Ethernet card yang tidak mendukung receiving/transmitting paket Ethernet ukuran full dengan header VLAN yang ditambahkan (1500 byte data + 4 byte header VLAN + 14 byte header Ethernet). Dalam situasi ini MTU 1496 dapat digunakan, tetapi perhatikan bahwa ini akan menyebabkan paket fragmentasi jika paket yang terlalu besar harus dikirim melalui interface. Pada saat yang sama ingat bahwa MTU 1496 dapat menyebabkan masalah jika jalur discovery MTU tidak berfungsi dengan benar antara source dan destination.

### Contoh Leyer2 VLAN
***

> **Peringatan:**<br>
> Konfigurasi ini diketahui menyebabkan masalah dengan perangkat vendor lain, terutama di jaringan yang mengaktifkan STP, Anda harus menggunakan penyaringan bridge VLAN sebagai gantinya jika Anda menggunakan RouterOS v6.41 atau yang lebih baru. Anda dapat membaca lebih lanjut tentang ini di [Sini](https://wiki.mikrotik.com/wiki/Manual:Layer2_misconfiguration#VLAN_in_bridge_with_a_physical_interface).

#### Port based VLAN tagging #1 (Trunk and Access ports)

![Secenario Vlan]({{"/images/scenario-vlan.png" | absolute_url}})

#### RouterOS
- Tambahkan interfaces VLAN yang diperlukan pada interface ethernet untuk menjadikannya sebagai port trunk VLAN
```bash
/interface vlan
add interface=ether1 vlan-id=100 name=eth1-vlan100
add interface=ether1 vlan-id=200 name=eth1-vlan200
add interface=ether1 vlan-id=300 name=eth1-vlan300
```

- Tambahkan IP Address untuk setiap VLAN dengan subnet yang berbeda
```bash
/ip address
add address=192.168.1.1/24 interface=eth1-vlan100
add address=192.168.2.1/24 interface=eth1-vlan200
add address=192.168.3.1/24 interface=eth1-vlan300
```

#### RouterOS-to-Switch
- Tambahkan interfaces VLAN yang diperlukan pada interface ethernet untuk menjadikannya sebagai port trunk VLAN
```bash
/interface vlan
add interface=ether1 vlan-id=100 name=eth1-vlan100
add interface=ether1 vlan-id=200 name=eth1-vlan200
add interface=ether1 vlan-id=300 name=eth1-vlan300
```

- Tambahkan bridge untuk setiap VLAN
```bash
/interface bridge
add name=bridge-vlan100
add name=bridge-vlan200
add name=bridge-vlan300
```

- Tambahkan interfaces VLAN ke bridge yang sesuai dan interfaces ethernet di mana untuk keperluan traffic untagged 
```bash
/interface bridge port
add bridge=bridge-vlan100 interface=eth1-vlan100
add bridge=bridge-vlan100 interface=ether2
add bridge=bridge-vlan200 interface=eth1-vlan200
add bridge=bridge-vlan200 interface=ether3
add bridge=bridge-vlan300 interface=eth1-vlan300
add bridge=bridge-vlan300 interface=ether4
```

- Tambahkan IP Address pada PC1, PC2, dan PC3 dan Uji koneksi ke semua VLAN
##### PC1
```bash
PC1> ip 192.168.1.2/24 192.168.1.1
Checking for duplicate address...
PC1 : 192.168.1.2 255.255.255.0 gateway 192.168.1.1
```
```bash
PC1> ping 192.168.1.1
84 bytes from 192.168.1.1 icmp_seq=1 ttl=64 time=27.085 ms
84 bytes from 192.168.1.1 icmp_seq=2 ttl=64 time=21.402 ms
84 bytes from 192.168.1.1 icmp_seq=3 ttl=64 time=17.312 ms
84 bytes from 192.168.1.1 icmp_seq=4 ttl=64 time=18.555 ms
84 bytes from 192.168.1.1 icmp_seq=5 ttl=64 time=17.339 ms
```
##### PC2
```bash
PC2> ip 192.168.2.2/24 192.168.2.1
Checking for duplicate address...
PC1 : 192.168.2.2 255.255.255.0 gateway 192.168.2.1
```
```bash
PC2> ping 192.168.2.1
84 bytes from 192.168.2.1 icmp_seq=1 ttl=64 time=14.927 ms
84 bytes from 192.168.2.1 icmp_seq=2 ttl=64 time=24.165 ms
84 bytes from 192.168.2.1 icmp_seq=3 ttl=64 time=29.632 ms
84 bytes from 192.168.2.1 icmp_seq=4 ttl=64 time=23.061 ms
84 bytes from 192.168.2.1 icmp_seq=5 ttl=64 time=18.035 ms
```
##### PC3
```bash
PC3> ip 192.168.3.2/24 192.168.3.1
Checking for duplicate address...
PC1 : 192.168.3.2 255.255.255.0 gateway 192.168.3.1
```
```bash
PC3> ping 192.168.3.1
84 bytes from 192.168.3.1 icmp_seq=1 ttl=64 time=8.499 ms
84 bytes from 192.168.3.1 icmp_seq=2 ttl=64 time=17.415 ms
84 bytes from 192.168.3.1 icmp_seq=3 ttl=64 time=17.776 ms
84 bytes from 192.168.3.1 icmp_seq=4 ttl=64 time=14.304 ms
84 bytes from 192.168.3.1 icmp_seq=5 ttl=64 time=21.216 ms
```

![Iklan]({{"/images/ads.jpg" | absolute_url}})

**... To be continue**



