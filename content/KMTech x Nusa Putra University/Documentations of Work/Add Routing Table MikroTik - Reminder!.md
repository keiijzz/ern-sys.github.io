Ketemu, dan ralat: tebakan gue soal mangle tadi salah layer. Mangle lo nggak masalah, yang bolong itu `/ip route rule`.

Rule 0-10 itu semacam daftar "kalau tujuannya subnet ini, cari rute di main". Itu yang bikin balasan ke VLAN lama nggak nyasar ke tabel ISP. Sekarang liat daftar subnet yang kecover (172.16.0.0/19, 192.168.0.0/22, 192.168.10.0/24, 192.168.20.0/24, dll) dan bandingin sama address-list lo. Yang **nggak ada** rule dst-nya:

- 192.168.xxx.0/24 (Smartclass)
- 192.168.xxx.0/24 (Ruang Meeting)
- 192.168.xxx.0/24 (Auditorium)
- 192.168.40.0/25, 50.0/25, 60.0/25, 70.0/24, 90.0/27
- 10.90.1.0/24

Alurnya kira-kira begini. Request dari Smartclass ke 103.xxx.xxx.xxx masuk dengan mulus karena rule 4 (dst=.199 → main). Tapi balasan router keluar dengan source 103.xxx.xxx.xxx, kena rule 12 (`src-address=.199 → to-VNET-SOHO`). Tujuannya 192.168.xxx.x, nggak ada rule dst yang nyelamatin, jadi dicari di tabel to-VNET-SOHO yang isinya cuma default route. Balasannya dibuang ke internet lewat WAN dan nggak pernah nyampe ke client. "Nusa Putra University" jalan karena subnet-nya (kemungkinan 172.16.0.0/19 atau 192.168.0.0/22) udah punya rule dst.

Pola yang sama kemungkinan besar juga bikin **inter-VLAN ke/dari VLAN baru itu mati**. Mangle ngasih routing-mark berdasarkan src doang, jadi kalau dst-nya nggak di-exclude, paketnya ikut ke tabel ISP. Coba ping dari VLAN Mahasiswa ke host di Smartclass. Kalau gagal, itu konfirmasi.

Fix cepat buat Smartclass:

```
/ip route rule add dst-address=192.168.xxx.0/24 action=lookup table=main place-before=0 comment="Smartclass -> main"
```

Tapi menurut gue ini bakal keulang tiap lo bikin VLAN baru, dan lo udah kelupaan 9 kali. Mending satu rule payung:

```
/ip route rule add dst-address=192.168.0.0/16 action=lookup table=main place-before=0
/ip route rule add dst-address=10.0.0.0/8 action=lookup table=main place-before=0
/ip route rule add dst-address=172.16.0.0/12 action=lookup table=main place-before=0
```

Ini aman karena `action=lookup` (bukan `lookup-only-in-table`): kalau main nggak punya rute ke tujuan itu, rule dilewatin dan lanjut ke rule bawahnya. Yang perlu lo cek cuma kalau ada rute private yang sengaja cuma ada di tabel ISP, misalnya akses ke modem IndiHome di 192.168.1.1 lewat tabel to-IndiHome. Rute itu bakal ketimpa kalau main juga punya rute yang overlap ke subnet tersebut.

Buktiin sebelum fix kalau mau: torch di interface WAN SOHO dengan filter dst-address=192.168.111.0/24, terus akses dari WiFi Smartclass. Kalau muncul trafik di situ, balasan lo bocor ke WAN, dan teori ini kebukti.

Rule `dst-address-type=local` yang gue kasih tadi nggak bakal nolong di kasus ini. Kalau udah lo pasang, hapus aja biar mangle tetap bersih.