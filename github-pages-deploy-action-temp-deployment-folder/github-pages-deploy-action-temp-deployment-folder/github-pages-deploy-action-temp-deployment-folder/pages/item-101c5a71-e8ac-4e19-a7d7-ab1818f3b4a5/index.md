Panduan Konfigurasi SSH Tanpa Password via Cloudflare Tunnel
------------------------------------------------------------

Halaman ini merangkum langkah-langkah, penyebab kegagalan umum, serta rekomendasi keamanan untuk mengkonfigurasi akses SSH tanpa password (_public key authentication_) ke server yang menggunakan **Cloudflare Tunnel** (`home.pesanweb.my.id`) pada port non-standar (Port 2222).

* * *

### 1\. Poin Penting Konfigurasi SSH

*   **Menyalin Public Key ke Server:** Kunci publik client (isi file `~/.ssh/id_ed25519.pub`) harus didaftarkan ke file `~/.ssh/authorized_keys` di server tujuan.
*   **Izin File (Permissions):** File dan direktori SSH di server harus memiliki permission yang tepat agar tidak ditolak oleh daemon SSH:
    *   Direktori `~/.ssh`: `chmod 700 ~/.ssh`
    *   File `authorized_keys`: `chmod 600 ~/.ssh/authorized_keys`
*   **ProxyCommand Cloudflare Tunnel:** Karena domain di-routing melalui Cloudflare Tunnel, IP publik domain (misal `104.21.15.39`) adalah Anycast CDN Cloudflare yang menolak koneksi direct TCP SSH. Sambungan harus menggunakan `ProxyCommand` yang memanggil binary `cloudflared`.

* * *

### 2\. Kesalahan & Penyebab SSH Ditolak

Berikut adalah beberapa kendala yang terjadi saat percobaan awal beserta analisis alasannya:

1.  **Koneksi Langsung Tanpa Proxy Timeout:**
    
    Perintah seperti `ssh -p 2222 root@home.pesanweb.my.id` gagal / timeout karena SSH mencoba menghubungi IP Cloudflare CDN langsung di port 2222. IP Cloudflare tidak membuka port TCP SSH secara terbuka.
    
2.  **Permission Denied (publickey, password):**
    
    Terjadi karena public key milik client belum dimasukkan ke `authorized_keys` milik user `root` di server. Karena kunci ditolak, SSH mencoba fallback ke autentikasi password, namun di lingkungan non-interaktif/headless (tanpa `/dev/tty`), permintaan password gagal total.
    
3.  **Mismatched Public Key:**
    
    Key yang dipasang di server berbeda dengan key milik client yang aktif di `~/.ssh/id_ed25519`. Selalu pastikan string public key yang dicopy cocok sepenuhnya.
    

* * *

### 3\. Langkah-Langkah Konfigurasi yang Benar

#### Langkah A: Konfigurasi di Sisi Server (Target)

    # 1. Buat direktori .ssh jika belum ada
    mkdir -p ~/.ssh
    
    # 2. Tambahkan isi public key client ke authorized_keys
    echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIRENLguqILk+3TYt2mX9lXNbZ3E/1o+jOiWI0EMf7uS hermes-agent-access" >> ~/.ssh/authorized_keys
    
    # 3. Set permission file
    chmod 700 ~/.ssh
    chmod 600 ~/.ssh/authorized_keys
    

#### Langkah B: Konfigurasi di Sisi Client (\`~/.ssh/config\`)

Tambahkan entri berikut pada file `~/.ssh/config` lokal:

    Host home.pesanweb.my.id
        User root
        Port 2222
        ProxyCommand cloudflared access ssh --hostname %h
        IdentityFile ~/.ssh/id_ed25519
        StrictHostKeyChecking no
        UserKnownHostsFile /dev/null
        IdentitiesOnly yes
    

#### Langkah C: Uji Coba Login

Jalankan perintah berikut untuk login tanpa perlu password:

    ssh home.pesanweb.my.id
    

* * *

### 4\. Rekomendasi Keamanan

*   **Non-aktifkan Password Authentication:** Ubah konfigurasi `/etc/ssh/sshd_config` di server menjadi `PasswordAuthentication no` dan `PubkeyAuthentication yes`, lalu restart SSHD (`systemctl restart sshd`). Ini mencegah serangan brute-force password.
*   **Gunakan Cloudflare Tunnel Persistent Service:** Pastikan `cloudflared` terpasang sebagai systemd service (`cloudflared service install <token>`) agar tunnel tetap aktif otomatis setelah server direboot.
*   **Tidak Perlu Service Client Background:** Client tidak perlu menjalankan daemon background seperti `cloudflared access tcp ... &`, cukup gunakan `ProxyCommand` di SSH config agar koneksi dipanggil secara _on-demand_ saat melakukan SSH.