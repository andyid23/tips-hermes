Masalah
-------

Saat diminta “rangkum ke situs tips-hermes”, situs itu **sudah ada** sebagai repo GitHub (`andyid23/tips-hermes`) dengan branch `master` (sumber) dan `gh-pages` (hasil build). Karena tidak dicek dulu, sempat dibuat situs baru di scratch lewat `hax site create`. Push-nya gagal dan hasilnya nyaris menjadi dua riwayat yang berbeda.

Gejala yang muncul
------------------

*   `hax site create` berhasil, tetapi repo tujuan sudah berisi situs lain.
*   GitHub API menjawab `422 Repository creation failed` dengan pesan _“name already exists on this account”_.
*   `git ls-remote` menunjukkan branch `master` dan `gh-pages` yang sudah lama ada.

Urutan yang benar
-----------------

    # 1. Cek dulu apakah situs sudah ada di GitHub
    git ls-remote --heads https://github.com/<user>/<site>.git
    
    # 2. Kalau ada — clone, jangan create
    git clone https://github.com/<user>/<site>.git /opt/data/cache/scratch/<site>
    cd /opt/data/cache/scratch/<site>
    git pull --rebase
    
    # 3. Tambah halaman di clone asli
    hax site node:add --title "Judul" --content "$(cat /opt/data/cache/scratch/halaman.html)" --slug "slug" --y
    
    # 4. Commit + push ke branch sumber
    git add -A && git commit -m "add page: Judul" && git push origin HEAD:master
    
    # 5. Buat situs baru HANYA kalau ls-remote kosong dan tidak ada site.json lokal
    

Kalau terlanjur membuat situs di scratch
----------------------------------------

*   Buang situs percobaan itu; jangan push.
*   Ambil hanya file HTML kontennya, masukkan ke clone repo yang asli.
*   Dengan begitu remote tidak pernah mendapat situs duplikat atau riwayat bercabang.

Catatan tambahan
----------------

*   Repo HAXCMS yang sudah pernah terbit biasanya sudah memakai branch `gh-pages` untuk deploy — periksa workflow sebelum menambal `branch: gh-pages` lagi.
*   Halaman baru digenerate sebagai `pages/<item-id>/index.html`, bukan `pages/<slug>/`; ambil path sebenarnya dari `site.json`.
*   URL slug (mis. `/cmd-jev-nudge/`) memang 404 di `curl` karena GitHub Pages menyajikan `404.html` SPA — verifikasi lewat path `pages/item-…/index.html`, lalu berikan URL slug ke pengguna untuk dibuka di browser.