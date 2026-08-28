# Navigasi & Anatomi Struktur File System di Linux
Tujuan: **Paham Peta Rumah Linux** 

## Skenario Nyata
Jam 2 Malam, Alarm DutyPager bunyi, *"Disk usage on Server DB-01 is 98%."* atau *"Nginx Configuration System Syntax error".*

Secara nyata, lu sebagai Engineer harus tau masalah apa itu:
- Cari tau config nginx dimana: `/etc/nginx`
- Cari log error dan disk penuh: `/var/log`
- Cari tau ukuran file atau folder tanpa bikin server hang: `ls -lhS` atau `du`

![[Pasted image 20260828104326.png]]

## Daftar Command:
![[Pasted image 20260828104421.png]]

--- 
# My Lab Hands-On
- `cd -` balik ke directory sebelumnya tanpa harus mengetikan directory panjang. misalkan kayak di DE, lu seperti pencet `alt+tab` dengan dua tab doang. nah seperti itu di terminalnya.
- `cd ` cd polos hanya akan membawa kamu ke directory home. misalkan, `/home/edrill`
- cari file terbaru dengan menggunakan `head`. `ls -lht /etc | head -n 10`. berarti mencari file terbaru dengan fungsi head dengan nilai 10 baris saja.
- berarti bisa menggunakan `tail` > `ls -lht /etc/ | tail -n 10`. berarti menunjukan file 10 baris dari bawah.
- `CTRL+Z` freeze session. untuk lanjut session bisa di pake menggunkan `fg`.
- `tree -L 2 /etc` jadi ini tuh ngasih `directory list dengan tree maksimal 2 folder aja yang keliatan, ke dalemnya kaga` begitu lah hasil penelitian gue nya. wkwkkw

---
# Hirarcy of Access Control
![[Pasted image 20260828110012.png]]

[[index]]