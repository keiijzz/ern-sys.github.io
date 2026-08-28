
Ringkasan hasilnya:

|ISP|Throughput|Packet Loss|Delay|Jitter|Indeks|
|---|---|---|---|---|---|
|IndiHome|2,09 Mbps|0,86%|6,83 ms|10,17 ms|3,67 (Bagus)|
|**VNET Dedicated**|**10,29 Mbps** 🏆|1,62%|**0,88 ms** 🏆|**1,48 ms** 🏆|3,67 (Bagus)|
|VNET Broadband|2,20 Mbps|**0,23%** 🏆|6,41 ms|7,94 ms|3,67 (Bagus)|
|iForte Dedicated|3,06 Mbps|2,46% ⚠️|2,66 ms|4,28 ms|3,33 (Bagus)|

**Temuan menarik buat laporan PKL-nya:**

- **VNET Dedicated jelas paling unggul** secara angka — throughput 5x lipat lebih tinggi, delay & jitter jauh lebih rendah. Cocok banget sama karakteristik "dedicated line" yang low-contention.
- **iForte Dedicated** punya packet loss tertinggi (2,46%) — dikit di atas ambang "Sangat Bagus". Ini poin yang bisa digali lebih dalam sama anak PKL lo, kenapa dedicated line malah paling banyak loss (mungkin volume trafik TCP yang jauh lebih besar di sampel itu).
- IndiHome & VNET Broadband karakteristiknya mirip — masuk akal karena keduanya sama-sama jalur non-dedicated.

Yang masih nge-gantung sama kayak sebelumnya: **kategori Throughput** butuh data bandwidth kontrak resmi tiap ISP buat difinalisasi — itu placeholder di dokumen (Bab IV & tabel indeks).

Kalau anak PKL-nya udah dapet data kontrak bandwidth-nya, kirim aja, gue update sekalian.