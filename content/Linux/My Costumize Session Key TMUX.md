Oh, itu artinya emang belum ada tmux session yang lagi jalan sama sekali — jadi wajar aja `tmux source-file` gagal karena gak ada server buat di-reload.

Tinggal start tmux dari awal aja, config barunya bakal otomatis kepake:

```bash
tmux
```

Abis itu langsung coba test:

- `Ctrl+b` lalu `"` → split horizontal (udah lo confirm work)
- `Ctrl+b` lalu `%` → split vertical
- `Alt+Left/Right/Up/Down` → pindah pane (dari config yang tadi kita tambahin)

Kalau lo mau kasih nama session biar gampang dibedain nanti (apalagi kalau kerjaan lo suka banyak sesi buat MikroTik/server beda-beda), langsung aja:

```bash
tmux new -s main
```

Wkwkwk iya, ini emang paling bikin bingung pemula karena mouse scroll defaultnya suka gak jalan di tmux.

**Cara manual (keyboard):**

1. `Ctrl+b` lalu `[` → masuk ke "copy mode" (mode buat scroll & select text)
2. Abis itu bisa scroll pake:
    - Arrow key atas/bawah (atau `k`/`j` kalau lo pake vim-style)
    - `Ctrl+u` = scroll setengah halaman ke atas
    - `Ctrl+d` = scroll setengah halaman ke bawah
    - `PageUp`/`PageDown` juga bisa
3. Buat keluar dari copy mode: tekan `q`

**Cara lebih enak — pake mouse scroll langsung** (kayak Terminator biasanya), tinggal enable mouse mode:

```bash
cat >> ~/.tmux.conf << 'EOF'
set -g mouse on
EOF
```

Reload:

```bash
tmux kill-server
tmux
```

Abis itu scroll mouse/touchpad langsung jalan buat lihat history, plus bonus lo juga bisa klik buat pindah pane dan drag border buat resize pane — jauh lebih mirip experience Terminator. Coba enable ini, dijamin bakal kerasa beda banget.