Ini keknya proyek DevOps-mu banget — bikin Gedung A jadi routed segment sendiri pake OSPF, bukan cuma L2 extension via media converter+switch kayak sekarang. Gue coba breakdown dan gambarin revisinya.

**Konsep perubahan:** Sekarang jalur ke Gedung A itu: fiber → media converter → switch L2 D-Link → jadi switch itu cuma nerusin VLAN 13 (192.168.0.0/22) apa adanya dari core, artinya broadcast domain-nya nyambung sampe ke sana lewat fiber. Kalau fiber putus/link flapping, atau ada broadcast storm di Gedung A, efeknya bisa kerasa ke core juga.

Kalau mau masuk konsep OSPF, artinya lo taro **router baru di Gedung A** (bisa MikroTik hEX/RB750Gr3 kalau throughput ga gede, atau CCR kalau butuh routing lebih berat), terus link fiber core↔Gedung A jadi **link routed point-to-point** (misal /30), bukan trunk VLAN. Switch L2 D-Link yang sekarang jadi turun jabatan — cuma access switch buat distribusi ke AP, bukan gateway lagi.**Yang berubah dari topo lama:**

- Gateway VLAN 13 Gedung A (192.168.0.0/22) pindah dari core ke router baru di Gedung A — jadi core cuma nge-route ke sana, ga nge-bridge lagi.
- Link fiber core↔Gedung A jadi P2P routed link (misal 10.99.0.0/30), OSPF neighbor adjacency di situ.
- Switch D-Link & AP Ruijie tetep di posisinya, cuma sekarang di belakang router, bukan langsung nempel ke core lewat trunk.

**Contoh config kerangka (RouterOS v7 style):**

Di core CCR1036:

```
/routing ospf instance
add name=inst-nusaputra router-id=10.255.255.1
/routing ospf area
add name=area0 instance=inst-nusaputra area-id=0.0.0.0
/routing ospf interface-template
add area=area0 interfaces=ether-fiber-gedungA network-type=point-to-point
```

Di router Gedung A (baru):

```
/routing ospf instance
add name=inst-nusaputra router-id=10.255.255.2
/routing ospf area
add name=area0 instance=inst-nusaputra area-id=0.0.0.0
/routing ospf interface-template
add area=area0 interfaces=ether-to-core network-type=point-to-point
add area=area0 interfaces=bridge-vlan13 passive
```

`network-type=point-to-point` penting biar ga ada DR/BDR election yang ga perlu di link 2-router doang.

Kalau ke depannya lo rencana nambah gedung lain juga (B, C, dst), gue saranin sekalian pikirin **multi-area OSPF** — core jadi ABR area 0, tiap gedung punya area sendiri (area 1, 2, dst). Sekarang karena baru 1 gedung, area 0 doang udah cukup, tapi kalau desain awal udah nyiapin itu, migrasinya nanti lebih mulus.

Router buat Gedung A-nya sendiri, lo mau pake hardware yang mana? Ada spare MikroTik di lokasi atau perlu direncanain beli baru?

---
# Implements OSPF

Oke, gue susun full config dari CCR1036 (core) sampe router baru di Gedung A. Gue pake skema IP baru buat link fiber-nya: **10.99.99.0/30** (core `.1`, router Gedung A `.2`), dan VLAN 13 Gedung A (192.168.0.0/22) yang lama di-**disable** di core, bukan dihapus — biar ada jalur rollback kalau ada yang salah.

Asumsi interface (sesuaiin sama port fisik lo yang sebenernya):

- Core: port fiber ke Gedung A = `sfp2-gedungA`, VLAN lama = `vlan13-gedungA`
- Router Gedung A: `ether1` = uplink ke core (fiber/media converter), `ether2` = trunk turun ke switch D-Link

## 1. Core CCR1036 — nonaktifin VLAN lama & pasang link OSPF baru

```
# Nonaktifin VLAN 13 Gedung A yang lama (disable dulu, jangan dihapus)
/interface vlan
disable [find name="vlan13-gedungA"]

/ip address
disable [find interface="vlan13-gedungA"]

/ip dhcp-server
disable [find interface="vlan13-gedungA"]

# IP baru buat link P2P ke router Gedung A
/ip address
add address=10.99.99.1/30 interface=sfp2-gedungA comment="P2P link ke Router-GedungA - OSPF"

# OSPF instance & area (skip kalau udah ada di core lo)
/routing ospf instance
add name=inst-nusaputra router-id=10.255.255.1

/routing ospf area
add name=area0 instance=inst-nusaputra area-id=0.0.0.0

# Daftarin interface link ke OSPF, point-to-point biar ga ada DR/BDR election
/routing ospf interface-template
add area=area0 interfaces=sfp2-gedungA network-type=point-to-point

# Izinin OSPF lewat firewall input (protocol 89), taro sebelum rule drop
/ip firewall filter
add chain=input protocol=ospf in-interface=sfp2-gedungA action=accept place-before=[find chain=input action=drop]
```

## 2. Router baru Gedung A — full config dari nol

```
# IP link P2P ke core
/ip address
add address=10.99.99.2/30 interface=ether1 comment="P2P link ke Core-CCR1036 - OSPF"

# VLAN 13 Gedung A pindah gateway-nya kesini
/interface vlan
add name=vlan13-gedungA vlan-id=13 interface=ether2

/ip address
add address=192.168.0.1/22 interface=vlan13-gedungA comment="Gateway VLAN 13 GedungA - baru, pindah dari core"

# DHCP server VLAN 13 Gedung A (pindah dari core juga)
/ip pool
add name=pool-gedungA ranges=192.168.0.10-192.168.3.250

/ip dhcp-server
add name=dhcp-gedungA interface=vlan13-gedungA address-pool=pool-gedungA lease-time=1d disabled=no

/ip dhcp-server network
add address=192.168.0.0/22 gateway=192.168.0.1 dns-server=10.10.10.X comment="DHCP GedungA - ganti dns-server sesuai Unbound lo"

# OSPF instance & area
/routing ospf instance
add name=inst-nusaputra router-id=10.255.255.2

/routing ospf area
add name=area0 instance=inst-nusaputra area-id=0.0.0.0

# Daftarin interface: link ke core (point-to-point) + VLAN GedungA (passive, cuma diadvertise, ga nyari neighbor)
/routing ospf interface-template
add area=area0 interfaces=ether1 network-type=point-to-point
add area=area0 interfaces=vlan13-gedungA passive=yes

# Firewall dasar
/ip firewall filter
add chain=input protocol=ospf action=accept
add chain=input connection-state=established,related action=accept
add chain=input in-interface=ether1 action=drop comment="drop yang aneh2 dari arah core"
```

## 3. Verifikasi setelah kedua sisi jalan

```
/routing ospf neighbor print
# harusnya muncul neighbor dengan state Full, address 10.99.99.1 ↔ 10.99.99.2

/routing ospf route print
# harus keliatan 192.168.0.0/22 kepelajarin lewat OSPF dari router Gedung A

/ping 192.168.0.1 src-address=10.10.10.1
# tes dari core, harus nyampe ke gateway baru Gedung A
```

Beberapa hal yang perlu lo cek manual:

- **DNS server** di DHCP network Gedung A gue kasih placeholder `10.10.10.X` — ganti sesuai IP Unbound lo di VLAN 99 Server.
- **Firewall NAT/masquerade** — kalau sebelumnya core yang nge-NAT-in VLAN 13 Gedung A, cek lagi apakah router Gedung A baru ini perlu jalur balik lewat core atau langsung NAT sendiri. Kalau semua traffic tetep keluar lewat core buat internet, ga perlu NAT di router Gedung A, cukup pastiin default route dia ke `10.99.99.1` (via OSPF, area 0 udah otomatis dapet default kalau core originate default route — kalau belum, tambahin `default-originate=always` di OSPF instance core).
- **Port switch D-Link ke arah AP** — karena sekarang cuma satu VLAN lewat situ, port trunk bisa disederhanain jadi access port kalau AP-nya juga cuma butuh satu VLAN, tapi kalau masih ada VLAN lain numpang di situ, tetep trunking aja.Config di atas udah lengkap dua sisi. Tinggal sesuaiin nama interface fisik lo yang bener, terus tes OSPF neighbor-nya dulu sebelum aktifin production traffic. Kalau nanti mau nambah gedung lain juga, kabarin — tinggal duplikasi pola ini per gedung dengan subnet link P2P beda-beda (misal 10.99.99.4/30 buat gedung berikutnya).