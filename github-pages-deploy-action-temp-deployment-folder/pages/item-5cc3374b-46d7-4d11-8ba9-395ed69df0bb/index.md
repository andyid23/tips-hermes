Ringkasan Sesi
--------------

Sesi ini membahas model agen, konfigurasi, environment, instalasi, cara pakai, langkah troubleshooting, dan rekomendasi.

Model Utama & Fallback
----------------------

*   **Model utama (efektif saat ini):** `poolside/laguna-s-2.1-free` via provider `commandcode` (sumppod). Diatur di `/opt/data/home/.hermes/config.yaml` (user-level override).
*   **Repo config:** `hermes-repo/config.yaml` menyebut `glm-5.3-flash-free` via `opencode-free`, tapi di-timpa oleh user override.
*   **Fallback (user override):** `ling-3.1-flash` (commandcode) → `laguna-s-2.1-free` (commandcode) → `mimo-v2.5-free` (opencode-free).
*   **Perubahan hari ini:** Di `hermes-repo/config.yaml`, fallback commandcode urutan ke-4 diganti dari `deepseek/deepseek-v4.1-flash-fast` menjadi `deepseek/deepseek-v4.1-flash-cmc`.

MOA (Mixture of Agents)
-----------------------

*   **Reference models:** `mimo-v2.5-free` & `ling-3.0-flash-fin-free` (opencode-free) dijalankan paralel.
*   **Aggregator:** `nemotron-3-ultra-free` (opencode-free) menggabungkan jawaban.
*   **Catatan:** MOA adalah preset terpisah, bukan pengganti `model.default`. Tetap aktif di config (`moa.enabled: true`).

Environment / Env Vars
----------------------

*   **User config:** `/opt/data/home/.hermes/config.yaml` (API key via `SUMPOOD_API_KEY` untuk commandcode).
*   **fcc-server:** `/opt/data/home/.fcc/.env` (mengelola key `OPENROUTER_API_KEY`, `NVIDIA_NIM_API_KEY`, `OLLAMA_API_KEY` + `MODEL_FALLBACKS`).
*   **Secrets:** `.env` sudah di-`.gitignore` di `hermes-repo` (dihentikan tracking) tapi file lokal tetap ada.

Instalasi & Status Layanan
--------------------------

*   **fcc-claude/fcc-server:** Berada di `/opt/data/home/.fcc/`. Saat ini tidak terlihat proses aktif di `ps` - kemungkinan berjalan on-demand, via tmux/screen, pm2, atau systemd user.
*   **opencode:** Binary di `~/.opencode/bin` (tidak selalu di PATH). Biasanya dipanggil on-demand, bukan daemon permanen.
*   **kilocode:** Tidak terdeteksi proses/service aktif saat pengecekan.
*   **Refresh model:** Ada job "nightly free-model refresh" di history commit (pola malam), tapi jam pastinya tidak terlihat di crontab/systemd timer saat ini.

Cara Pakai
----------

*   **Ganti model per sesi:** Pakai `/model <provider:model>` di TUI/CLI (langsung berlaku, tidak mengubah config global).
*   **Ganti model default:** Edit `/opt/data/home/.hermes/config.yaml` (ubah `model.default`, `provider`, `api_key_env`, dan urutan `fallback_providers`).
*   **Ganti cadangan:** Tambah/urutkan model di `fallback_providers`. Jika kuota habis, fallback akan mencoba urutan berikutnya.
*   **Lewat fcc/opencode:** Tergantung jalur akses - `MODEL_FALLBACKS` di `.fcc/.env` bisa berpengaruh di sisi proxy.

Langkah Rekomendasi
-------------------

*   **Prinsip konfigurasi:** Pahami precedence (user override > repo config > built-in). Yang aktif saat ini ditentukan oleh `~/.hermes/config.yaml`.
*   **Kelola memori:** Gabungkan fakta serupa jadi 1 baris. Pindahkan prosedur ke skill via `skill_manage`. Hanya simpan fakta universal di `memory`.
*   **Baca file efisien:** Gunakan `offset/limit` kalau tidak butuh seluruh file.
*   **Tugas berat/iteratif:** Delegasikan ke subagent agar konteks utama tetap bersih.
*   **Konsolidasi:** Jika budget memory/profile >80%, prioritaskan konsolidasi ketimbang menambah entri baru.
*   **Biarkan kompresi:** Fitur `compression` aktif - biarkan bekerja, tapi waspada saat membaca file besar berturut-turut.
*   **Skill baru/tugas berulang:** Simpan ke skill di sesi terpisah atau lanjutkan di sesi yang sama, lalu perbarui skill jika ada temuan/pitfall baru.
*   **Keamanan:** Jangan commit secrets. Repo private aman untuk melacak file, tapi tetap lebih baik menghindari jika bisa dipublikasi.