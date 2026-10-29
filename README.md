# Mid-Project Komdat Jarkom 2026/2027 — Implementasi Jaringan Smart City Kota Surabaya

**Kelompok 10** — Khalifa S. (5027251104), Nadia Iqlima (5027251108), Muh. Ridwan (5027251113)
**Tools:** Cisco Packet Tracer 9.0.1 · **Skema pengalamatan:** VLSM · **Deadline:** Minggu, 1 November 2026, 23.59 WIB

> Dokumen ini diperbarui bertahap. Bagian bertanda **[TODO]** belum dikerjakan, **[BUKTI]** menandai tempat screenshot yang harus ditempel.

## Status pengerjaan

| No | Soal | Bobot | Status |
|---|---|---|---|
| 1 | Perancangan topologi | 10 | Topologi & backbone selesai; label LAN, hostname CLI, penjelasan desain **[TODO]** |
| 2 | Tabel subnetting | 5 | Tabel VLSM selesai; tabel CIDR, supernet per router, tabel revisi, format `.xlsx` **[TODO]** |
| 3 | Static routing | 15 | Selesai (semua LAN saling terhubung); bukti ping lengkap **[TODO]** |
| 4 | DHCP | 10 | **Selesai.** 11 LAN memakai pool di router; CCTV Kota memakai server DHCP di SERVER-DC dengan relay (`ip helper-address`) di R-DC |
| 5 | NAT | 5 | **Selesai.** PAT di R-PUSAT; route `10.80.0.0/16` di R-ISP sudah dihapus |
| 6 | Tunnel GRE | 10 | **Selesai.** Tunnel aktif; lalu lintas Balai Kota ↔ Pelabuhan sudah lewat tunnel (terverifikasi tracert); bonus IPsec: lisensi `securityk9` sudah aktif di R-PUSAT dan R-PELABUHAN, konfigurasi dan verifikasi selesai (ISAKMP SA `QM_IDLE`/`ACTIVE`, paket GRE terenkripsi dan terdekripsi) |
| 7 | OSPF (file terpisah) | 10 | **[TODO]** |
| 8 | Layanan aplikasi | 10 | **[TODO]** |
| 9 | Demo | 25 | Persiapan |

---

## 1. Perancangan Topologi

### 1.1 Pemilihan skema: VLSM

Kelompok memilih **VLSM**. Alasan (draft, konfirmasi alasan kelompok):

- VLSM membagi blok sesuai kebutuhan tiap LAN sehingga lebih hemat: seluruh 14 LAN muat di `10.80.0.0/16` dengan **61.728** alamat terpakai (**94,19 %**, 3.808 alamat terbuang).
- Jika memakai CIDR (satu supernet per router), kebutuhan menjadi **118.784** alamat dalam `10.80.0.0/15` (**90,63 %**, 12.288 alamat terbuang).
- Kekurangan VLSM: dua LAN satu router tidak bersebelahan (mis. R-DC memegang `10.80.96.0/19` dan `10.80.239.0/24`), sehingga R-PUSAT perlu 12 route LAN, bukan 6 route supernet.

### 1.2 Alamat dasar

Blok dasar `10.80.0.0/16` (oktet ke-2 = 8 × nomor kelompok 10). Rentang `10.0.0.0/8` adalah blok private (RFC 1918), sedangkan `172.80.x.x` dan `192.80.x.x` bukan rentang private.

### 1.3 Tabel LAN (VLSM)

| LAN | Router | Host | Network / Prefix | Rentang host | Broadcast | Gateway |
|---|---|---|---|---|---|---|
| WiFi Publik Utara | R-UTARA | 11.000 | 10.80.0.0/18 | 10.80.0.1 – 10.80.63.254 | 10.80.63.255 | 10.80.0.1 |
| WiFi Publik Selatan | R-SELATAN | 6.500 | 10.80.64.0/19 | 10.80.64.1 – 10.80.95.254 | 10.80.95.255 | 10.80.64.1 |
| CCTV Kota | R-DC | 6.000 | 10.80.96.0/19 | 10.80.96.1 – 10.80.127.254 | 10.80.127.255 | 10.80.96.1 |
| WiFi Publik Timur | R-TIMUR | 5.500 | 10.80.128.0/19 | 10.80.128.1 – 10.80.159.254 | 10.80.159.255 | 10.80.128.1 |
| Sensor & SCADA Pelabuhan | R-PELABUHAN | 5.000 | 10.80.160.0/19 | 10.80.160.1 – 10.80.191.254 | 10.80.191.255 | 10.80.160.1 |
| WiFi Publik Barat | R-BARAT | 4.500 | 10.80.192.0/19 | 10.80.192.1 – 10.80.223.254 | 10.80.223.255 | 10.80.192.1 |
| Pegawai Balai Kota | R-PUSAT | 1.800 | 10.80.224.0/21 | 10.80.224.1 – 10.80.231.254 | 10.80.231.255 | 10.80.224.1 |
| Sensor Banjir Timur | R-TIMUR | 700 | 10.80.232.0/22 | 10.80.232.1 – 10.80.235.254 | 10.80.235.255 | 10.80.232.1 |
| Sensor Lalu Lintas Utara | R-UTARA | 450 | 10.80.236.0/23 | 10.80.236.1 – 10.80.237.254 | 10.80.237.255 | 10.80.236.1 |
| Puskesmas & Sekolah Barat | R-BARAT | 220 | 10.80.238.0/24 | 10.80.238.1 – 10.80.238.254 | 10.80.238.255 | 10.80.238.1 |
| Server & Storage DC | R-DC | 180 | 10.80.239.0/24 | 10.80.239.1 – 10.80.239.254 | 10.80.239.255 | 10.80.239.1 |
| Kantor Pelabuhan | R-PELABUHAN | 120 | 10.80.240.0/25 | 10.80.240.1 – 10.80.240.126 | 10.80.240.127 | 10.80.240.1 |
| PJU Pintar Selatan | R-SELATAN | 90 | 10.80.240.128/25 | 10.80.240.129 – 10.80.240.254 | 10.80.240.255 | 10.80.240.129 |
| Server Balai Kota | R-PUSAT | 20 | 10.80.241.0/27 | 10.80.241.1 – 10.80.241.30 | 10.80.241.31 | 10.80.241.1 |

### 1.4 Alokasi link antar-router

| Link | Router A | IP A | Router B | IP B | Network |
|---|---|---|---|---|---|
| ISP | R-PUSAT | 203.0.113.41 | R-ISP | 203.0.113.42 | 203.0.113.40/30 |
| L1 | R-PUSAT | 10.80.241.33 | R-DC | 10.80.241.34 | 10.80.241.32/30 |
| L2 | R-PUSAT | 10.80.241.37 | R-UTARA | 10.80.241.38 | 10.80.241.36/30 |
| L3 | R-PUSAT | 10.80.241.41 | R-SELATAN | 10.80.241.42 | 10.80.241.40/30 |
| L4 | R-PUSAT | 10.80.241.45 | R-TIMUR | 10.80.241.46 | 10.80.241.44/30 |
| L5 | R-PUSAT | 10.80.241.49 | R-BARAT | 10.80.241.50 | 10.80.241.48/30 |
| L6 | R-UTARA | 10.80.241.53 | R-PELABUHAN | 10.80.241.54 | 10.80.241.52/30 |
| Tunnel0 (GRE) | R-PUSAT | 192.168.10.1 | R-PELABUHAN | 192.168.10.2 | 192.168.10.0/30 |

Server internet: `8.8.8.8/24`, gateway `8.8.8.1` di R-ISP (`203.0.113.40/30` = 203.0.113.(4 × 10)/30).

### 1.5 Penamaan perangkat

Nama di diagram sudah sesuai tempatnya (R-PUSAT, SW-SERVER-BK, PC-WIFI-UTARA, dst.).
**[TODO]** Hostname di CLI semua router masih `Router`; setel dengan `hostname R-XXXX`.
**[TODO]** Label tiap LAN (nama subnet, Network ID/prefix, jumlah host) belum ditambahkan di topologi.

### 1.6 Desain tiap lokasi

**[TODO]** Jelaskan pilihan switch, jenis perangkat (PC / mobile / server), media link (kabel / wireless), dan penambahan port router, beserta alasannya. Saat ini tiap LAN memakai 1 switch (2960-24TT) + 1 perangkat perwakilan; router 2911 diberi modul serial untuk link backbone.

**[BUKTI]** Screenshot topologi.

---

## 2. Tabel Subnetting

Spreadsheet: `Tabel_VLSM_SmartCity_Surabaya_Kelompok10` (tabel VLSM LAN + alokasi link antar-router).
Hasil verifikasi: 14 subnet VLSM benar (network ID, prefix, kapasitas) dan tidak saling tumpang tindih.

**[TODO]** Tabel CIDR (supernet per router: R-UTARA `10.80.0.0/17`, R-TIMUR `10.80.128.0/18`, R-DC `10.80.192.0/18`, R-BARAT `10.81.0.0/18`, R-SELATAN `10.81.64.0/18`, R-PELABUHAN `10.81.128.0/18`, R-PUSAT `10.81.192.0/20`; total 118.784 alamat dalam `10.80.0.0/15`).
**[TODO]** Tabel revisi (subnet, jawaban UTS, perbaikan, alasan). Calon isi (verifikasi ke lembar UTS):

| Subnet | Jawaban UTS | Perbaikan | Alasan |
|---|---|---|---|
| Kantor Pelabuhan | rentang/broadcast tertulis `10.80.239.x` | `10.80.240.1 – .126`, broadcast `.127` | blok `/25` mulai di `10.80.240.0` |
| PJU Pintar Selatan | network `10.80.240.0`, rentang `.1 – .254` | `10.80.240.128/25`, `.129 – .254` | bertabrakan dengan Kantor Pelabuhan |
| Ukuran blok /18 | 16389 | 16384 | 2^14 |
| WiFi Publik Selatan | network `10.80.69.0` | `10.80.64.0` | kelipatan 8192 |
| Nama segmen | SCADA Pelabuhan | Sensor & SCADA Pelabuhan | samakan dengan soal |
| Akar pohon CIDR | /14 | /15 | total 118.784 alamat |

**[TODO]** Konversi ke `.xlsx` (file sekarang `.xlsm`).

---

## 3. Static Routing

Seluruh LAN saling terhubung hanya dengan static route; default route hanya ke arah atas/keluar. Hasil simulasi 182 pasangan LAN (semua arah): **0 gagal**.

| Router | Default route | Static route lain |
|---|---|---|
| R-PUSAT | `0.0.0.0/0` → `203.0.113.42` (R-ISP) | 12 LAN milik R-DC, R-UTARA, R-PELABUHAN, R-SELATAN, R-TIMUR, R-BARAT; `10.80.241.52/30` (link L6) via R-UTARA |
| R-UTARA | → `10.80.241.37` (R-PUSAT) | `10.80.160.0/19` dan `10.80.240.0/25` via `10.80.241.54` (R-PELABUHAN) |
| R-DC | → `10.80.241.33` | — |
| R-BARAT | → `10.80.241.49` | — |
| R-TIMUR | → `10.80.241.45` | — |
| R-SELATAN | → `10.80.241.41` | — |
| R-PELABUHAN | → `10.80.241.53` (R-UTARA) | `10.80.241.0/27` via tunnel (`192.168.10.1`) |

Perintah dasar: `ip route <network> <mask> <next-hop>`; default route: `ip route 0.0.0.0 0.0.0.0 <next-hop>`.

**[BUKTI]** `show ip route` di R-PUSAT, R-UTARA, dan satu router daun; ping antar-wilayah (dari tiap wilayah ke wilayah lain, sertakan PC sumber).

---

## 4. DHCP

**Soal:**
- Interface router dan seluruh server menggunakan **IP statis**.
- Perangkat pengguna mendapat **IP dinamis** secara otomatis, lengkap dengan gateway dan DNS server (soal 8), tanpa bentrok dengan alamat IP statis.
- Khusus subnet CCTV Kota (subnet R-DC) mendapatkan alamat dari server DHCP di LAN Server dan Storage DC, bukan dari router.

![Teks soal 4](screenshots/item4-teks-soal.png)

### 4.1 Rancangan

- Interface router dan server (SERVER-DC, SERVER-BK) tetap statis; LAN Server & Storage DC dan LAN Server Balai Kota tidak memakai DHCP.
- 11 LAN pengguna memakai pool DHCP di router yang melayani LAN tersebut: R-PUSAT (Pegawai), R-UTARA (WiFi Utara, Sensor Lalu Lintas Utara), R-SELATAN (WiFi Selatan, PJU Pintar Selatan), R-TIMUR (WiFi Timur, Sensor Banjir Timur), R-BARAT (WiFi Barat, Puskesmas & Sekolah Barat), R-PELABUHAN (Sensor & SCADA Pelabuhan, Kantor Pelabuhan).
- CCTV Kota (`10.80.96.0/19`, gateway `10.80.96.1` di R-DC): pool dibuat di server DHCP SERVER-DC (`10.80.239.2`), dan R-DC meneruskan permintaan lewat `ip helper-address 10.80.239.2` pada interface CCTV. R-DC tidak memiliki pool untuk CCTV.
- DNS yang dibagikan: `10.80.239.2` (SERVER-DC). **Asumsi:** sesuai soal 8 DNS berada di SERVER-DC; jika ternyata server lain, cukup ubah baris `dns-server` di setiap pool.

### 4.2 Cara menentukan alamat gateway (port router) setiap LAN

Gateway pada pool (`default-router`) harus sama dengan alamat port router di LAN tersebut, dan alamat itu juga dikecualikan dari pool. Cara memastikannya:

1. **Tanya router langsung:** `show ip interface brief` di router yang bersangkutan menampilkan setiap port beserta alamat IP-nya (alamat inilah milik router).
2. **Lihat PC:** *Default Gateway* pada konfigurasi IP PC adalah port router di LAN tersebut.
3. **Aturan tabel VLSM:** router selalu mendapat alamat host pertama setiap subnet (`.1` untuk LAN `10.80.64.0/19` → `10.80.64.1`; `10.80.240.128/25` → `10.80.240.129`).
4. **Link antar-router (/30):** hanya berisi dua alamat host, satu untuk tiap router (mis. L3: R-PUSAT `10.80.241.41`, R-SELATAN `10.80.241.42`).

### 4.3 Konfigurasi pool di router

Contoh (R-PUSAT, LAN Pegawai Balai Kota):

```
enable
configure terminal
ip dhcp excluded-address 10.80.224.1
ip dhcp pool PEGAWAI
 network 10.80.224.0 255.255.248.0
 default-router 10.80.224.1
 dns-server 10.80.239.2
 exit
end
write memory
```

| Perintah | Fungsi |
|---|---|
| `ip dhcp excluded-address <ip>` | Alamat yang tidak boleh dibagikan (gateway/port router, alamat statis). Bisa berupa satu alamat atau rentang `<awal> <akhir>`. Berlaku untuk seluruh router, bukan satu pool. |
| `ip dhcp pool <nama>` | Membuat pool; prompt berubah menjadi `(dhcp-config)#`. Nama hanya label. |
| `network <alamat> <mask>` | Rentang alamat yang dibagikan (seluruh subnet LAN). |
| `default-router <ip>` | Gateway yang diberikan ke PC (port router di LAN itu). |
| `dns-server <ip>` | DNS yang diberikan ke PC. |

Daftar pool:

| Router | Pool | Network | Gateway (dikecualikan) |
|---|---|---|---|
| R-PUSAT | PEGAWAI | `10.80.224.0/21` | `10.80.224.1` |
| R-UTARA | WIFI-UTARA | `10.80.0.0/18` | `10.80.0.1` |
| R-UTARA | LALIN | `10.80.236.0/23` | `10.80.236.1` |
| R-SELATAN | WIFI-SELATAN | `10.80.64.0/19` | `10.80.64.1` |
| R-SELATAN | PJU | `10.80.240.128/25` | `10.80.240.129` |
| R-TIMUR | WIFI-TIMUR | `10.80.128.0/19` | `10.80.128.1` |
| R-TIMUR | BANJIR | `10.80.232.0/22` | `10.80.232.1` |
| R-BARAT | WIFI-BARAT | `10.80.192.0/19` | `10.80.192.1` |
| R-BARAT | PUSKESMAS | `10.80.238.0/24` | `10.80.238.1` |
| R-PELABUHAN | SCADA | `10.80.160.0/19` | `10.80.160.1` |
| R-PELABUHAN | KANTOR | `10.80.240.0/25` | `10.80.240.1` |

(CCTV Kota memakai pool di SERVER-DC, bukan di router; lihat 4.5.)

### 4.4 Hasil pengujian

PC diubah dari *Static* ke *DHCP*; semua menampilkan **DHCP request successful**:

| PC | Alamat | Mask | Gateway | DNS |
|---|---|---|---|---|
| PC-PEGAWAI | `10.80.224.2` | `255.255.248.0` | `10.80.224.1` | `10.80.239.2` |
| PC-WIFI-UTARA | `10.80.0.2` | `255.255.192.0` | `10.80.0.1` | `10.80.239.2` |
| PC-LALIN | `10.80.236.2` | `255.255.254.0` | `10.80.236.1` | `10.80.239.2` |
| PC-WIFI-SELATAN | `10.80.64.2` | `255.255.224.0` | `10.80.64.1` | `10.80.239.2` |
| PC-PJU | `10.80.240.130` | `255.255.255.128` | `10.80.240.129` | `10.80.239.2` |
| PC-WIFI-TIMUR | `10.80.128.2` | `255.255.224.0` | `10.80.128.1` | `10.80.239.2` |
| PC-BANJIR | `10.80.232.2` | `255.255.252.0` | `10.80.232.1` | `10.80.239.2` |
| PC-WIFI-BARAT | `10.80.192.2` | `255.255.224.0` | `10.80.192.1` | `10.80.239.2` |
| PC-PUSKESMAS | `10.80.238.2` | `255.255.255.0` | `10.80.238.1` | `10.80.239.2` |
| PC-SCADA | `10.80.160.2` | `255.255.224.0` | `10.80.160.1` | `10.80.239.2` |
| PC-KANTOR | `10.80.240.2` | `255.255.255.128` | `10.80.240.1` | `10.80.239.2` |

Konfigurasi router:

![Pool R-PUSAT](screenshots/item4-konfigurasi-pool-pegawai-rpusat.png)
![Pool R-UTARA](screenshots/item4-konfigurasi-pool-rutara.png)
![Pool R-SELATAN](screenshots/item4-konfigurasi-pool-rselatan.png)
![Pool R-TIMUR](screenshots/item4-konfigurasi-pool-rtimur.png)
![Pool R-BARAT](screenshots/item4-konfigurasi-pool-rbarat.png)
![Pool R-PELABUHAN](screenshots/item4-konfigurasi-pool-rpelabuhan.png)

Hasil pada PC:

![PC-PEGAWAI](screenshots/item4-dhcp-pc-pegawai.png)
![PC-WIFI-UTARA](screenshots/item4-dhcp-pc-wifi-utara.png)
![PC-LALIN](screenshots/item4-dhcp-pc-lalin.png)
![PC-WIFI-SELATAN](screenshots/item4-dhcp-pc-wifi-selatan.png)
![PC-PJU](screenshots/item4-dhcp-pc-pju.png)
![PC-WIFI-TIMUR](screenshots/item4-dhcp-pc-wifi-timur.png)
![PC-BANJIR](screenshots/item4-dhcp-pc-banjir.png)
![PC-WIFI-BARAT](screenshots/item4-dhcp-pc-wifi-barat.png)
![PC-PUSKESMAS](screenshots/item4-dhcp-pc-puskesmas.png)
![PC-SCADA](screenshots/item4-dhcp-pc-scada.png)
![PC-KANTOR](screenshots/item4-dhcp-pc-kantor.png)

### 4.5 CCTV Kota: server DHCP di SERVER-DC dan relay di R-DC

Permintaan DHCP berupa *broadcast* dan tidak melewati router. SERVER-DC (`10.80.239.2`) berada di LAN Server & Storage DC, sedangkan perangkat CCTV di LAN CCTV Kota (`10.80.96.0/19`), keduanya di belakang R-DC. R-DC meneruskan broadcast itu ke server dengan `ip helper-address`. R-DC tidak memiliki `ip dhcp pool` untuk CCTV, sesuai soal.

**Mencari port CCTV di R-DC** (`show ip interface brief`): `GigabitEthernet0/0` = `10.80.96.1` (LAN CCTV Kota), `GigabitEthernet0/1` = `10.80.239.1` (LAN Server & Storage DC), `Serial0/3/0` = `10.80.241.34` (link L1 ke R-PUSAT).

![show ip interface brief di R-DC](screenshots/item4-rdc-show-ip-interface-brief.png)

**Server DHCP di SERVER-DC** (Services → DHCP, Service On): pool `CCTV`, gateway `10.80.96.1`, DNS `10.80.239.2`, start IP `10.80.96.2`, mask `255.255.224.0`, maksimum 8189 pengguna (jumlah host /19 dikurangi gateway). Pool bawaan `serverPool` tidak dipakai dan tidak diubah (Packet Tracer tidak mengizinkan pengeditan `serverPool`, sehingga pool baru ditambahkan).

![Pool CCTV di SERVER-DC](screenshots/item4-server-dc-pool-cctv.png)

**Relay di R-DC:**

```
configure terminal
interface GigabitEthernet0/0
 ip helper-address 10.80.239.2
 end
write memory
```

![Konfigurasi relay di R-DC](screenshots/item4-konfigurasi-relay-rdc.png)

**Hasil:** PC-CCTV diubah ke DHCP dan menerima `10.80.96.2`, mask `255.255.224.0`, gateway `10.80.96.1`, DNS `10.80.239.2` (**DHCP request successful**), berarti permintaan berhasil diteruskan ke SERVER-DC.

![PC-CCTV memperoleh alamat dari SERVER-DC](screenshots/item4-dhcp-pc-cctv.png)

**Catatan:** alamat dari DHCP tidak dijamin selalu sama. Server memberi alamat bebas pertama (di sini `.2` karena hanya ada satu PC) dan biasanya mengingat alamat yang pernah diberikan selama *lease* berlaku. Untuk alamat tetap pada satu perangkat diperlukan reservasi DHCP, yang tidak diminta soal. Saat demo, alamat PC sebaiknya dibaca dari konfigurasi IP/`ipconfig`, bukan diasumsikan.


## 5. NAT

**Soal:** "Seluruh host dapat mengakses server 8.8.8.8 menggunakan satu IP publik milik R-PUSAT. R-ISP tidak boleh memiliki rute ke alamat privat internal."

![Teks soal 5](screenshots/item5-teks-soal.png)

Satu IP publik dipakai bersama oleh seluruh host, sehingga teknik yang dipakai adalah **PAT** (NAT *overload*): R-PUSAT mengganti alamat sumber private menjadi `203.0.113.41` (port ISP R-PUSAT, `Serial0/1/0`) dan membedakan setiap percakapan lewat nomor port (untuk ping: ICMP identifier). Tabel terjemahan dicatat di R-PUSAT sehingga balasan dari 8.8.8.8 dikembalikan ke PC yang meminta.

### 5.1 Kondisi awal

R-PUSAT sudah punya default route `Gateway of last resort is 203.0.113.42` (R-ISP). Interface *outside* = `Serial0/1/0`; interface *inside* = `GigabitEthernet0/0`, `GigabitEthernet0/1`, `Serial0/1/1`, `Serial0/2/0`, `Serial0/2/1`, `Serial0/3/0`, `Serial0/3/1`.

![R-PUSAT sebelum NAT](screenshots/item5-sebelum-nat-rpusat.png)

R-ISP masih memiliki route `S 10.80.0.0/16 [1/0] via 203.0.113.41` yang membuat ping bisa berhasil tanpa NAT, sehingga harus dihapus:

![R-ISP sebelum route dihapus](screenshots/item5-risp-sebelum-hapus-rute.png)

### 5.2 Konfigurasi di R-PUSAT

```
configure terminal
access-list 1 permit 10.80.0.0 0.0.255.255
interface Serial0/1/0
 ip nat outside
 exit
interface range GigabitEthernet0/0 - 1
 ip nat inside
 exit
interface Serial0/1/1
 ip nat inside
 exit
(sama untuk Serial0/2/0, Serial0/2/1, Serial0/3/0, Serial0/3/1)
ip nat inside source list 1 interface Serial0/1/0 overload
end
write memory
```

| Perintah | Fungsi |
|---|---|
| `access-list 1 permit 10.80.0.0 0.0.255.255` | Daftar alamat sumber yang boleh diterjemahkan (seluruh blok 10.80.0.0/16). *Wildcard mask* `0.0.255.255` = kebalikan subnet mask. |
| `ip nat outside` / `ip nat inside` | Menandai sisi luar (ke ISP) dan sisi dalam (jaringan kota). NAT hanya bekerja untuk paket yang masuk dari interface inside dan keluar lewat interface outside, sehingga lalu lintas internal tidak berubah. |
| `ip nat inside source list 1 interface Serial0/1/0 overload` | Ganti alamat sumber yang cocok dengan ACL 1 menjadi alamat `Serial0/1/0` (`203.0.113.41`), dibagi bersama dengan port sebagai pembeda. |

Catatan: `ip nat inside` pada `Tunnel0` ditolak Packet Tracer (`% Invalid input detected`). Ini tidak berpengaruh pada hasil pengujian karena seluruh PC yang diuji berhasil. `interface range` hanya menerima satu jenis interface, sehingga port Serial diatur satu per satu.

![Konfigurasi NAT di R-PUSAT](screenshots/item5-konfigurasi-nat-rpusat.png)

Hasil `show ip nat statistics` (sebelum ada lalu lintas):

![show ip nat statistics](screenshots/item5-show-ip-nat-statistics-awal.png)

### 5.3 Hapus route di R-ISP

```
configure terminal
no ip route 10.80.0.0 255.255.0.0 203.0.113.41
end
write memory
```

Setelah dihapus, tabel routing R-ISP hanya berisi jaringan yang terhubung langsung (`8.8.8.0/24` dan `203.0.113.40/30`), tanpa route ke `10.80.x.x`:

![R-ISP setelah route dihapus](screenshots/item5-risp-setelah-hapus-rute.png)

### 5.4 Pengujian

Semua ping ke `8.8.8.8` berhasil setelah route di R-ISP dihapus (paket pertama pada PC-PEGAWAI timeout karena ARP, kemudian normal):

| PC | LAN | Hasil |
|---|---|---|
| PC-PEGAWAI (`10.80.224.2`) | Pegawai Balai Kota (R-PUSAT) | 3 dari 4 balasan, TTL 126 |
| PC-SCADA (`10.80.160.2`) | Sensor & SCADA Pelabuhan | 4 dari 4 balasan, TTL 125 |
| PC-LALIN (`10.80.236.2`) | Sensor Lalu Lintas Utara (R-UTARA) | 4 dari 4 balasan, TTL 125 |

![ping PC-PEGAWAI](screenshots/item5-ping-pegawai-ke-8888.png)
![ping PC-SCADA](screenshots/item5-ping-scada-ke-8888.png)
![ping PC-LALIN](screenshots/item5-ping-lalin-ke-8888.png)

Di screenshot PC-PEGAWAI juga terlihat ping internal ke `10.80.240.2` (PC-KANTOR) masih berhasil, artinya lalu lintas internal tidak ikut diterjemahkan.

Tabel terjemahan di R-PUSAT (`show ip nat translations`): alamat *inside local* `10.80.236.2` (PC-LALIN) diterjemahkan menjadi *inside global* `203.0.113.41` dengan ICMP identifier 1–4 sebagai pembeda. Entri milik PC lain sudah kedaluwarsa saat screenshot diambil.

![show ip nat translations](screenshots/item5-show-ip-nat-translations.png)

---

## 6. Tunnel GRE

### 6.1 Konfigurasi

| | R-PUSAT | R-PELABUHAN |
|---|---|---|
| Tunnel0 | `192.168.10.1/30` | `192.168.10.2/30` |
| Tunnel source | `10.80.241.37` (Serial0/2/1, link L2) | `10.80.241.54` (Serial0/3/0, link L6) |
| Tunnel destination | `10.80.241.54` | `10.80.241.37` |

Status: `Tunnel0 is up, line protocol is up`; ping PC-SCADA → SERVER-BK 0 % loss; tracert menampilkan hop `192.168.10.1`.

### 6.2 Lalu lintas Balai Kota ↔ Pelabuhan harus lewat tunnel

Subnet Balai Kota: `10.80.224.0/21` (Pegawai), `10.80.241.0/27` (Server). Subnet Pelabuhan: `10.80.160.0/19` (SCADA), `10.80.240.0/25` (Kantor).

Pemeriksaan awal menemukan dua route yang melewati R-UTARA, bukan tunnel:

| Router | Route lama | Perbaikan |
|---|---|---|
| R-PUSAT | `10.80.240.0/25` via `10.80.241.38` (R-UTARA) | `no ip route 10.80.240.0 255.255.255.128 10.80.241.38` lalu `ip route 10.80.240.0 255.255.255.128 192.168.10.2` |
| R-PELABUHAN | `10.80.224.0/21` lewat default route (R-UTARA) | `ip route 10.80.224.0 255.255.248.0 192.168.10.1` |

Status perbaikan: **selesai dan terverifikasi**.

- R-PUSAT: `show ip route 10.80.240.0` menampilkan `Known via "static"` dengan next hop `192.168.10.2`, dan route lama via `10.80.241.38` sudah tidak ada.
- R-PELABUHAN: sebelumnya `show ip route 10.80.224.0` menjawab `% Subnet not in table`; setelah `ip route 10.80.224.0 255.255.248.0 192.168.10.1`, entri `10.80.224.0/21` muncul dengan next hop `192.168.10.1`, lalu disimpan dengan `write memory`.
**Bukti pengujian (tracert):**

1. Dari PC-PEGAWAI ke PC-KANTOR (`tracert 10.80.240.2`): hop 1 `10.80.224.1` (R-PUSAT), hop 2 **`192.168.10.2`** (ujung tunnel di R-PELABUHAN), hop 3 `10.80.240.2` (PC-KANTOR). Alamat tunnel muncul di tengah jalur, jadi paket melewati tunnel dan tidak melewati R-UTARA.

   ![tracert PC-PEGAWAI ke PC-KANTOR](screenshots/item6-tracert-pegawai-ke-kantor.png)

2. Dari PC-KANTOR ke SERVER-BK (`tracert 10.80.241.2`): hop 1 `10.80.240.1` (R-PELABUHAN), hop 2 **`192.168.10.1`** (ujung tunnel di R-PUSAT), hop 3 `10.80.241.2` (SERVER-BK). Arah sebaliknya juga lewat tunnel.

   Catatan membaca hasil `tracert`: angka `ms` adalah waktu pulang-pergi (round trip) tiap paket uji, dan tiap hop dikirim 3 paket uji. Tanda `*` pada paket pertama hop terakhir berarti satu paket uji tidak dibalas sebelum timeout. Hal ini wajar: paket pertama ke sebuah tujuan hilang selagi ARP (pencarian alamat MAC dari alamat IP) belum selesai, sedangkan dua paket berikutnya dibalas, sehingga jalur sudah benar.

   ![tracert PC-KANTOR ke SERVER-BK](screenshots/item6-tracert-kantor-ke-serverbk.png)

3. Dari PC-SCADA ke PC-PEGAWAI (`tracert 10.80.224.2`): hop 1 `10.80.160.1` (R-PELABUHAN), hop 2 **`192.168.10.1`** (ujung tunnel di R-PUSAT), hop 3 `10.80.224.2` (PC-PEGAWAI). Lalu lintas SCADA ↔ Balai Kota juga melewati tunnel.

   ![tracert PC-SCADA ke PC-PEGAWAI](screenshots/item6-tracert-scada-ke-pegawai.png)

Simulasi tabel routing seluruh router menunjukkan kedelapan arus Balai Kota ↔ Pelabuhan (Pegawai dan Server Balai Kota ↔ SCADA dan Kantor Pelabuhan, masing-masing dua arah) melewati tunnel, dan seluruh 182 pasangan LAN tetap saling terhubung.

### 6.3 Penjelasan

**Mengapa router memilih jalur tunnel dibanding jalur ke pelabuhan lewat R-UTARA?** Router memilih route berdasarkan tabel routing. Untuk prefix Balai Kota ↔ Pelabuhan, route statis menunjuk ke alamat ujung tunnel (`192.168.10.x`) sebagai next hop, sehingga paket diteruskan ke interface Tunnel0, bukan ke R-UTARA. Route ke R-UTARA tidak dipasang untuk prefix tersebut, jadi tunnel satu-satunya jalur yang dipakai.

**Mengapa GRE saja tidak mengenkripsi data?** GRE hanya melakukan *encapsulation*: paket asli dibungkus paket IP baru (header IP luar `10.80.241.37 → 10.80.241.54` + header GRE). Isi paket asli tetap terbaca oleh siapa pun yang menangkapnya. Router di tengah (R-UTARA) hanya membaca header luar. Overhead sekitar 24 byte, sebab itu MTU tunnel 1476. Enkripsi memerlukan IPsec (bonus +5).

**Apa fungsi tunnel pada topologi ini?** Pada topologi ini jalur fisik tetap melewati R-UTARA, sehingga tunnel tidak mempercepat atau memperpendek jalur. Fungsinya: (1) membentuk link logis langsung antara R-PUSAT dan R-PELABUHAN sehingga lalu lintas Balai Kota ↔ Pelabuhan menjadi satu aliran dengan dua ujung tetap; (2) menyembunyikan alamat jaringan yang dibawa dari router di tengah (R-UTARA hanya perlu rute ke alamat ujung tunnel `10.80.241.37` dan `10.80.241.54`); (3) menjadi dasar untuk enkripsi IPsec. Pada jaringan nyata, tunnel dipakai untuk menghubungkan dua lokasi melalui jaringan milik pihak lain (misalnya ISP) yang tidak mengenal alamat private kita: paket asli dibungkus paket baru beralamat publik yang bisa diteruskan ISP. Biayanya sekitar 24 byte per paket (header IP luar 20 byte + header GRE 4 byte), sehingga MTU tunnel 1476 byte.

**Perbedaan GRE dan NAT:** GRE membungkus paket tanpa mengubah isi paket asli (alamat private tetap), sedangkan NAT (soal 5) mengganti alamat sumber private dengan alamat publik.

### 6.4 Bonus: IPsec di atas tunnel GRE (+5)

**Tujuan.** Mengenkripsi lalu lintas di dalam tunnel. Tanpa IPsec, isi paket yang dibungkus GRE tetap terbaca oleh siapa pun yang menangkap paket (termasuk R-UTARA), karena GRE hanya membungkus. Dengan IPsec (mode GRE over IPsec), R-UTARA hanya melihat paket ESP antara `10.80.241.37` dan `10.80.241.54`; isi paket asli terenkripsi.

**1. Persiapan lisensi.** Router 2911 memerlukan paket teknologi `securityk9` (awalnya `disable`). Dilakukan pada R-PUSAT dan R-PELABUHAN:

```
configure terminal
license boot module c2900 technology-package securityk9
end
write memory
reload
```

Verifikasi dengan `show version | begin Technology`: baris `security` menampilkan `securityk9` pada kolom *Current* (Type: Evaluation). Jika kolom *Next reboot* masih `None`, perintah lisensi diulang.

**2. Konfigurasi R-PUSAT.**

```
crypto isakmp policy 10
 encryption aes 256
 authentication pre-share
 group 2
 exit
crypto isakmp key K10SmartCity address 10.80.241.54
crypto ipsec transform-set TS-GRE esp-aes 256 esp-sha-hmacaccess-list 110 permit gre host 10.80.241.37 host 10.80.241.54
crypto map CMAP-GRE 10 ipsec-isakmp
 set peer 10.80.241.54
 set transform-set TS-GRE
 match address 110
 exit
interface Serial0/2/1
 crypto map CMAP-GRE
 end
write memory
```

**3. Konfigurasi R-PELABUHAN** (cerminan; alamat ditukar, antarmuka `Serial0/3/0`):

```
crypto isakmp policy 10
 encryption aes 256
 authentication pre-share
 group 2
 exit
crypto isakmp key K10SmartCity address 10.80.241.37
crypto ipsec transform-set TS-GRE esp-aes 256 esp-sha-hmacaccess-list 110 permit gre host 10.80.241.54 host 10.80.241.37
crypto map CMAP-GRE 10 ipsec-isakmp
 set peer 10.80.241.37
 set transform-set TS-GRE
 match address 110
 exit
interface Serial0/3/0
 crypto map CMAP-GRE
 end
write memory
```

Status: **selesai dan terverifikasi** (lihat bagian 5). Catatan: perintah `mode transport` tidak tersedia di Packet Tracer (sub-mode transform-set tidak muncul), sehingga kedua router memakai mode tunnel bawaan.

**4. Penjelasan konfigurasi**

| Bagian | Fungsi |
|---|---|
| `crypto isakmp policy 10` | Kebijakan fase 1 (IKE): cara kedua router bernegosiasi dengan aman. Angka 10 hanya nomor/prioritas kebijakan (nomor terkecil dicoba lebih dulu). |
| `encryption aes 256` | Percakapan negosiasi dienkripsi dengan AES kunci 256 bit. |
| `authentication pre-share` | Kedua router saling membuktikan identitas dengan kata sandi bersama (pre-shared key). |
| `group 2` | Grup Diffie-Hellman: cara membuat kunci rahasia yang sama di kedua ujung tanpa mengirim kunci itu lewat jaringan. |
| `crypto isakmp key K10SmartCity address ...` | Kata sandi bersama untuk peer di alamat tersebut; harus sama di kedua router. |
| `crypto ipsec transform-set TS-GRE esp-aes 256 esp-sha-hmac` | Fase 2: cara melindungi data. `TS-GRE` hanya nama; ESP mengenkripsi dengan AES-256; SHA-HMAC memeriksa keutuhan paket. |
| mode (bawaan: tunnel) | Transform set memakai mode tunnel (`show crypto ipsec transform-set` menampilkan `will negotiate = { Tunnel, }`): seluruh paket GRE dibungkus ESP dengan header IP baru. Perintah `mode transport` tidak dipakai karena sub-mode transform-set tidak muncul di Packet Tracer; kedua router harus memakai mode yang sama. |
| `access-list 110 permit gre host A host B` | Menentukan lalu lintas yang dienkripsi: hanya paket GRE antara dua ujung tunnel. |
| `crypto map CMAP-GRE 10 ipsec-isakmp` | Aturan gabungan: peer (`set peer`), cara perlindungan (`set transform-set`), lalu lintas yang dicakup (`match address`). |
| `interface Serial0/x/x` → `crypto map CMAP-GRE` | Mengaktifkan aturan pada antarmuka tempat paket tunnel keluar. |

**Istilah**
- **ISAKMP / IKE:** protokol negosiasi sebelum data dilindungi; hasilnya berupa *security association* (kesepakatan algoritma dan kunci).
- **AES:** algoritma enkripsi simetris (kunci sama di kedua ujung); 256 adalah panjang kunci dalam bit.
- **Diffie-Hellman:** metode pertukaran kunci: dua pihak menghasilkan rahasia yang sama lewat jalur terbuka tanpa mengirim rahasia itu.
- **ESP:** protokol IPsec (nomor protokol IP 50) yang mengenkripsi data.
- **SHA-HMAC:** sidik jari (hash) paket yang dicampur kunci rahasia; penerima menghitung ulang, jika berbeda paket dianggap diubah atau dipalsukan dan dibuang.
- Enkripsi AES muncul dua kali dengan fungsi berbeda: pada `isakmp policy` untuk melindungi negosiasi, pada `transform-set` untuk melindungi paket data.

**5. Pengujian dan bukti** **[BUKTI]**
- `show crypto isakmp sa`: status `QM_IDLE` (fase 1 berhasil).
- `show crypto ipsec sa`: nilai `#pkts encaps` dan `#pkts decaps` naik setelah ping (paket dienkripsi dan didekripsi).
- Ping/tracert PC-PEGAWAI ↔ PC-KANTOR tetap berhasil setelah IPsec aktif.
- (Opsional) Simulation mode: paket di R-UTARA tampil sebagai ESP, bukan GRE dengan isi terbaca.

**Hasil pengujian (R-PUSAT):**

- `show crypto isakmp sa`: peer `10.80.241.54` (R-PELABUHAN) dari `10.80.241.37` (R-PUSAT), state **`QM_IDLE`**, status **`ACTIVE`** (negosiasi fase 1 berhasil).
- `show crypto ipsec sa`: antarmuka `Serial0/2/1`, crypto map `CMAP-GRE`; lalu lintas yang dilindungi adalah protokol **47 (GRE)** antara `10.80.241.37` dan `10.80.241.54`; `#pkts encaps: 5`, `#pkts encrypt: 5`, `#pkts decaps: 4`, `#pkts decrypt: 4` (paket benar-benar dienkripsi dan didekripsi). `#send errors: 1` berasal dari paket pertama yang hilang saat negosiasi berlangsung.
- `ping 10.80.240.2` dari PC-PEGAWAI: paket pertama *Request timed out* (negosiasi IKE dan ARP masih berjalan), 3 dari 4 paket berikutnya dibalas (4, 8, 6 ms, TTL 126), sehingga komunikasi Balai Kota ↔ Pelabuhan tetap berjalan lewat tunnel yang kini terenkripsi.

![show crypto isakmp sa dan show crypto ipsec sa di R-PUSAT](screenshots/item6-bonus-ipsec-show-crypto-rpusat.png)

![ping PC-PEGAWAI ke PC-KANTOR setelah IPsec aktif](screenshots/item6-bonus-ipsec-ping-pegawai-kantor.png)

Catatan: bagian atas tangkapan layar ping (tracert) diambil sebelum IPsec diaktifkan.

**Hasil pengujian (R-PELABUHAN):** `show crypto isakmp sa` menampilkan peer `10.80.241.37` (R-PUSAT) dari `10.80.241.54` (R-PELABUHAN) dengan state **`QM_IDLE`** dan status **`ACTIVE`**, sehingga kedua ujung tunnel sama-sama membentuk security association.

![show crypto isakmp sa di R-PELABUHAN](screenshots/item6-bonus-ipsec-isakmp-rpelabuhan.png)

---

## 7. OSPF (file terpisah) — **[TODO]**

Rencana: salin file utama setelah item 1–6 dan 8 selesai; link cadangan L7 R-DC ↔ R-UTARA (`10.80.241.56/30`, port Gi0/2 di kedua router); ganti route statis antar-router dengan OSPF; `passive-interface` di sisi LAN; default route hanya di R-PUSAT (`default-information originate`); uji putus salah satu link backbone (L1/L2/L7).

## 8. Layanan Aplikasi — **[TODO]**

Domain `k10.smartcity.lab`: DNS di SERVER-DC (`www`, `ftp`, `mail`); web "Portal Smart City Surabaya" di SERVER-BK; FTP di SERVER-DC dengan akun login (CCTV Kota dapat mengunggah); email antarpegawai Balai Kota dan Pelabuhan.

## 9. Demo — persiapan

Uji 1 (kelompok, 15 menit): test case soal 1–8 + tanya jawab. Uji 2 (individu): tiap anggota menemukan, memperbaiki, dan menjelaskan satu kesalahan dalam 10 menit. Semua anggota wajib punya file `.pkt` di laptop dan memahami seluruh konfigurasi.

---

## Catatan perubahan

| Tanggal | Perubahan |
|---|---|
| 2026-10-29 | File `.pkt` dibaca: topologi, IP, dan static route sesuai tabel VLSM; 0 dari 182 pasangan LAN gagal. Ditemukan: hostname CLI belum diset, label LAN belum ada, 2 route belum lewat tunnel, R-ISP masih punya route ke `10.80.0.0/16`. |
| 2026-10-29 | Route tunnel diperbaiki: R-PUSAT ke `10.80.240.0/25` via `192.168.10.2`, R-PELABUHAN ke `10.80.224.0/21` via `192.168.10.1`. Diverifikasi dengan 2 tracert (PC-PEGAWAI → PC-KANTOR, PC-KANTOR → SERVER-BK); keduanya melewati hop tunnel. |
| 2026-10-29 | Bonus IPsec: lisensi `securityk9` diaktifkan dan reload pada R-PUSAT dan R-PELABUHAN (kolom *Current* = securityk9). Konfigurasi crypto (ISAKMP, transform-set, crypto map) disiapkan; menunggu eksekusi dan verifikasi. |
| 2026-10-29 | Bonus IPsec dikonfigurasi di R-PUSAT (`Serial0/2/1`) dan R-PELABUHAN (`Serial0/3/0`) dengan mode tunnel bawaan (`mode transport` tidak tersedia). Verifikasi: ISAKMP SA `QM_IDLE`/`ACTIVE`, `#pkts encaps/decaps` > 0, ping PC-PEGAWAI → PC-KANTOR berhasil. |
| 2026-10-29 | Bukti tambahan soal 6: tracert PC-SCADA → PC-PEGAWAI lewat tunnel; `show crypto isakmp sa` di R-PELABUHAN `QM_IDLE`/`ACTIVE`. Soal 6 (termasuk bonus IPsec) dinyatakan selesai. |
| 2026-10-30 | Soal 5: PAT dikonfigurasi di R-PUSAT (outside `Serial0/1/0`, ACL 1 = `10.80.0.0/16`), route `10.80.0.0/16` di R-ISP dihapus. Ping ke 8.8.8.8 berhasil dari PC-PEGAWAI, PC-SCADA, PC-LALIN; `show ip nat translations` menunjukkan terjemahan ke `203.0.113.41`. Soal 5 selesai. |
| 2026-10-30 | Soal 4: pool DHCP di 6 router untuk 11 LAN pengguna (DNS `10.80.239.2`, asumsi SERVER-DC); CCTV Kota lewat server DHCP SERVER-DC + `ip helper-address` di R-DC (Gi0/0). Seluruh PC uji menerima alamat, gateway, dan DNS. Soal 4 selesai. |
