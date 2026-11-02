Latar Belakang
--------------

Di beberapa provider seperti SumoPod, **nggak disediain raw TCP port forwarding** buat SSH langsung (IP:Port). Satu-satunya cara biar bisa SSH stabil adalah dengan menjalankan `cloudflared` di dalam container dan membuat SSH tunnel lewat Cloudflare.

Langkah-langkah
---------------

### 1) Tambahin public key client ke target (wajib di /root/.ssh/)

Dari laptop/client ambil public key:

    cat ~/.ssh/id_ed25519.pub

Buka web console target dan login sebagai **root**, lalu tambahin ke `authorized_keys` milik root:

    mkdir -p /root/.ssh
    chmod 700 /root/.ssh
    echo "<PASTE_PUBLIC_KEY_KAMU>" >> /root/.ssh/authorized_keys
    chmod 600 /root/.ssh/authorized_keys
    cat /root/.ssh/authorized_keys

**Penting:** `sshd` membaca key dari `/root/.ssh/authorized_keys`, bukan dari home user `hermes`.

### 2) Setup Cloudflare Tunnel di container

Cek binary cloudflared (biasanya ada di `/opt/data/home/.local/bin/cloudflared`).

    export HOME=/opt/data/home
    export CLOUDFLARED_HOME=/opt/data/home/.cloudflared
    mkdir -p $CLOUDFLARED_HOME
    
    /opt/data/home/.local/bin/cloudflared tunnel login
    /opt/data/home/.local/bin/cloudflared tunnel create ssh-hermes-<nama>

Catat **Tunnel UUID** dan file `<UUID>.json` yang muncul.

### 3) Salin cert & credentials ke lokasi writable

File `cert.pem` dan credentials JSON kerap dimiliki `root` dan nggak bisa dibaca user `hermes`. Salin dari web console **root**:

    mkdir -p /opt/data/cf-ssh
    cp /opt/data/home/.cloudflared/cert.pem /opt/data/cf-ssh/
    cp /opt/data/home/.cloudflared/<UUID>.json /opt/data/cf-ssh/
    chown -R hermes:hermes /opt/data/cf-ssh
    chmod 600 /opt/data/cf-ssh/*

### 4) Buat config, route DNS & jalankan tunnel

Buat config di sesi `hermes`:

    cat > /opt/data/cloudflared-ssh-config.yml << 'EOF'
    tunnel: <UUID>
    credentials-file: /opt/data/cf-ssh/<UUID>.json
    ingress:
      - hostname: ssh-<nama>.<domain>.com
        service: ssh://localhost:22
      - service: http_status:404
    EOF

Route DNS dan jalankan di background:

    export HOME=/opt/data/home
    export CLOUDFLARED_HOME=/opt/data/cf-ssh
    
    /opt/data/home/.local/bin/cloudflared tunnel route dns ssh-hermes-<nama> ssh-<nama>.<domain>.com
    
    nohup /opt/data/home/.local/bin/cloudflared tunnel --config /opt/data/cloudflared-ssh-config.yml run <UUID> > /opt/data/cloudflared-ssh.log 2>&1 &

### 5) Setup di laptop/client

Tambahin ke `~/.ssh/config`:

    Host hermes-tunnel
        HostName ssh-<nama>.<domain>.com
        User root
        Port 22
        ProxyCommand cloudflared access ssh --hostname %h
        StrictHostKeyChecking accept-new
        ServerAliveInterval 60
        ServerAliveCountMax 120
        IdentitiesOnly yes

Lalu coba konek:

    ssh hermes-tunnel

Catatan Penting & Troubleshooting
---------------------------------

*   **Key location:** Wajib ditambahin ke `/root/.ssh/authorized_keys`. Kalo cuma ditambahin ke home user lain, SSH tetap ditolak.
*   **Permission issues:** Kalo `cert.pem` atau credentials JSON dimiliki `root` dan nggak bisa dibaca `hermes`, **copy** ke folder writable (mis. `/opt/data/cf-ssh/`), ubah owner ke `hermes`, dan set `CLOUDFLARED_HOME=/opt/data/cf-ssh`.
*   **ProxyCommand wajib:** Karena nggak ada raw TCP, client harus pakai `ProxyCommand cloudflared access ssh --hostname %h`.
*   **Tunnel name vs UUID:** `tunnel route dns` pakai **nama tunnel**, `tunnel run` bisa pakai **UUID**.
*   **Background tetap jalan:** Pakai `nohup` biar tunnel nggak mati pas web console disconnect.

Skill Terkait
-------------

Langkah ini udah disimpan di skill `ssh-access-hermes` (kategori `software-development`) biar bisa dipake buat setup serupa ke instance Hermes lain.