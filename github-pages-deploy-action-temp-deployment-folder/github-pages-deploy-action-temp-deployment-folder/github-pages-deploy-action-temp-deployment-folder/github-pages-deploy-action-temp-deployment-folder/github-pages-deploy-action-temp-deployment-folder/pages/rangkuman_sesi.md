# Rangkuman Sesi - Free Claude Code & HAXCMS

## Agent yang Digunakan
- **Claude Code** (2.1.267) - Anthropic's official CLI
- **OpenCode** (1.18.30) - Provider-agnostic, open-source (gratis, built-in free models)
- **Codex** (0.154.0) - OpenAI's coding agent
- **Pi** (0.85.1) - Earendil coding agent
- **Cline** (3.0.61) - VS Code extension (CLI wrapper)
- **Command Code CLI** (cmd/cmdc) - Dengan mod jev-nudge untuk memastikan agent tidak berhenti sebelum tugas tuntas

## Free Claude Code (fcc-server) - Proxy Unifying

### Instalasi
```bash
curl -fsSL https://raw.githubusercontent.com/Alishahryar1/free-claude-code/main/scripts/install.sh | bash
```

### Konfigurasi PATH
```bash
export PATH="$HOME/.local/bin:$HOME/.opencode/bin:$HOME/.npm-global/bin:$HOME/.local/share/uv/tools/free-claude-code/bin:$PATH"
```

### Menjalankan Proxy
```bash
fcc-server &  # background
# atau
nohup fcc-server > ~/.fcc/server.log 2>&1 &
```

- **Port:** http://127.0.0.1:8082
- **Admin UI:** http://127.0.0.1:8082/admin
- **Endpoints:** /claude, /codex, /opencode, /pi, /cline (OpenAI-compatible)

### API Keys
- Bisa via Admin UI di http://127.0.0.1:8082/admin atau via `~/.fcc/.env`
- **OpenCode** tidak butuh API key (gratis)
- Keys tersimpan: OPENROUTER_API_KEY, NVIDIA_NIM_API_KEY, OLLAMA_API_KEY, dll

## HAXCMS - Membuat Situs

### Instalasi HAX CLI
```bash
npm i -g @haxtheweb/create
npm link  # atau langsung via global
```

### Membuat Situs Baru
```bash
cd /path/where/you/want/sites
hax site nama-situs --y --no-i
```

Contoh: `hax site ustadzhikmah --y --no-i`

### Konfigurasi Git (wajib)
```bash
git config --global user.email "nama@email.com"
git config --global user.name "Nama Anda"
```

### Menjalankan & Mengelola Situs
```bash
cd nama-situs
hax start  # Interactive menu
# atau
hax site site:status  # cek status
```

## Command Code + jev-nudge (Mod Stop-Hook)

### Instalasi
```bash
npm i -g command-code@latest
npm install -g cmd-mod-jev-nudge
cmd mods add -g cmd-mod-jev-nudge
```

### Autentikasi (Headless)
```bash
export COMMAND_CODE_API_KEY=$(grep COMMAND_CODE_API_KEY /opt/data/.env | cut -d= -f2)
```

### Penggunaan
```bash
cmd -n "nama-sesi" --permission-mode accept-edits "prompt tugas"
cmd -p "prompt" --max-turns 40 --output-format json  # headless
```

**Kunci:** jev-nudge mencegah agent berhenti sebelum pekerjaan benar-benar selesai. Set `--max-turns` sebagai rem untuk mengontrol biaya.

## Langkah Praktis (Rekomendasi)

1. **Setup fcc-server** - Jalankan proxy untuk mengakses semua agent via satu endpoint
2. **Gunakan OpenCode** - Untuk tugas gratis tanpa API key
3. **Buat situs HAXCMS** - `hax site <nama> --y --no-i` dengan git config sudah diset
4. **Untuk tugas kompleks** - Gunakan Command Code + jev-nudge dengan `--max-turns` terbatas
5. **Kelola via Admin UI** - http://127.0.0.1:8082/admin untuk monitoring & config keys

## Tips Penting
- **PATH order matters** untuk fcc-server
- **Admin UI local-only** (127.0.0.1) - pakai SSH tunnel untuk remote
- **jev-nudge** maksimal 8 nudge per user turn
- **HAXCMS** butuh git user identity untuk auto-commit
- **Node/npm** harus terinstall untuk HAX CLI
