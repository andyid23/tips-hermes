Ringkasan
---------

**Command Code** (`cmd` / `cmdc` / `command-code`) adalah coding agent CLI. **jev-nudge** adalah _mod stop-hook_ yang mencegah agent berhenti sebelum tugas tuntas: setiap kali agent mau berhenti, mod bertanya ke model `typesafe/jev` — "apakah perlu disenggol?" Jika ya, agent dipaksa lanjut.

Diuji headless di mesin ini (Command Code 1.88.1): satu prompt menghasilkan 4 senggolan, lalu run berhenti sendiri saat Jev menilai pekerjaan selesai.

    [cmd-mod-jev-nudge] jev nudge 1: unfinished work found (68%), continuing
    [cmd-mod-jev-nudge] jev nudge 2: unfinished work found (68%), continuing
    [cmd-mod-jev-nudge] jev nudge 3: unfinished work found (74%), continuing
    [cmd-mod-jev-nudge] jev nudge 4: unfinished work found (70%), continuing
    stopReason: end_turn   finalText: こんにちは — 5 done. All done.

Instalasi & autentikasi
-----------------------

    npm i -g command-code@latest          # prefix npm harus milik user (~/.npm-global)
    npm install -g cmd-mod-jev-nudge      # nama paket TIDAK ber-scope
    cmd mods add -g cmd-mod-jev-nudge     # daftarkan lewat mod manager (wajib)
    cmd mods list                         # harus muncul: cmd-mod-jev-nudge · user

**Penting:** `npm i -g` saja tidak mendaftarkan mod (`cmd mods list` tetap kosong). Paket `@commandcodeai/cmd-mod-jev-nudge` **404** — nama aslinya `cmd-mod-jev-nudge`.

Autentikasi headless tidak memakai `cmd login` (butuh TTY/Ink raw mode). CLI membaca **`COMMAND_CODE_API_KEY`** (bukan `CMD_API_KEY`, bukan `COMMANDCODE_API_KEY`):

    export COMMAND_CODE_API_KEY=$(awk -F= '/^COMMAND_CODE_API_KEY=/{print $2}' /opt/data/.env)
    cmd -p "say OK" --max-turns 2        # terautentikasi kalau mencetak OK

Langkah pakai
-------------

1.  **Auth** — set `COMMAND_CODE_API_KEY` (atau `cmd login` di terminal interaktif).
2.  **Pastikan mod aktif** — `cmd mods list`. Mod auto-load; tidak perlu `--mod`.
3.  **Jalankan tugas** — `cmd -n "nama-sesi" "prompt"` atau headless `cmd -p "prompt"`.
4.  **Tuning** — `--mod-option jev-nudge-threshold=0.6` dan kawan-kawan (tabel di bawah).
5.  **Kelola sesi** — `-c`, `-r`, `--fork-session`, `/sessions`, `/tree`.
6.  **Verifikasi** — `--output-format json` lalu grep event `jev nudge`.

Prompt + sesi: beda yang sering tertukar
----------------------------------------

*   **Session** = percakapan persisten (disimpan, punya id/nama). **Task** = pekerjaan yang ingin tuntas. **Prompt** = pesan Anda; tiap prompt = satu _user turn_.
*   `cmd "prompt"` → TUI interaktif. `cmd -p "prompt"` → sekali jalan, cetak hasil, keluar.
*   `-c` = lanjutkan sesi terakhir; `-r "nama"` = sesi tertentu; `--session <id>` = by id; tambah `--fork-session` = cabang baru tanpa merusak aslinya.
*   `--max-turns N` _memotong_ kerja; jev-nudge _memperpanjang_ kerja sampai tuntas. Pakai keduanya.
*   `--safe-mode` mematikan semua mod sekali run; `--mod-option jev-nudge-max=0` mematikan hanya jev.

Flag jev-nudge
--------------

Flag

Default

Arti

`jev-nudge-threshold`

0.5

Probabilitas minimum untuk menyenggol

`jev-nudge-max`

tanpa batas

Batas nudge per run; `0` = mati

`jev-nudge-verbose`

false

Tampilkan juga keputusan yang tidak jadi nudge

`jev-nudge-base-url`

https://api.commandcode.ai

Root API

`jev-nudge-model`

typesafe/jev

Model penilai

Fakta operasional: maksimum **8 nudge per user turn** (batas Command Code). Jev memberi skor 0.5–0.8 saat masih ada kerja, <0.1 saat selesai. Ia **tidak** menyenggol bila agent sedang menunggu Anda (izin/info/pilihan) atau bila nudge sebelumnya tanpa progres. Tanpa API key, mod dorman (diam), bukan error. `DEBUG=true` mencatat setiap keputusan ke stderr.

Resep siap pakai
----------------

### 1\. Tugas nyata (headless, aman)

    cmd -n "fix-tests" \
        --permission-mode accept-edits \
        --mod-option jev-nudge-threshold=0.55 \
        --mod-option jev-nudge-max=5 \
        -p "perbaiki semua test yang gagal sampai suite hijau" \
        --max-turns 40 --output-format json | grep -E 'jev nudge|"finalText"'

### 2\. Verifikasi mod benar-benar menyenggol

    cmd -p "say hi in five languages. ONE hi per reply, then say all done." \
        --max-turns 10 --output-format json | grep -o 'jev nudge [0-9]*: [^"]*'

### 3\. Sesi panjang / eksperimen

    cmd -r "refactor-auth" --fork-session \
        --mod-option jev-nudge-verbose=true "lanjutkan refactor"

Rekomendasi
-----------

*   Pakai jev untuk tugas multi-langkah yang **jelas selesainya** (refactor, migrasi, rantai perbaikan test).
*   Selalu pasang `--max-turns` sebagai rem (30–50). Tanpa itu, salah tafsir "belum selesai" bisa berputar sampai 8 nudge dan biaya naik.
*   Mulai dari threshold default **0.5**; naikkan ke 0.6–0.7 kalau terlalu agresif, turunkan ke 0.4 kalau agent berhenti terlalu cepat.
*   Headless: pakai `--permission-mode accept-edits` supaya tidak berhenti di prompt izin (agent "waiting" → jev sengaja diam).
*   `jev-nudge-verbose=true` hanya untuk mencoba/mendiagnosis.
*   Fork sesi sebelum eksperimen agar riwayat asli tetap utuh.
*   Jangan kirim API key ke chat; simpan di `.env` dan ekspor sebagai `COMMAND_CODE_API_KEY`.