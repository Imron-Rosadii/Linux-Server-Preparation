---

# SSH Hardening

## Tujuan

Pada tahap ini kita akan meningkatkan keamanan akses SSH ke server.

Hardening ini bertujuan untuk:

* mencegah **brute force attack**
* menonaktifkan **login root**
* membatasi user yang boleh login
* menggunakan **SSH key authentication**
* mengganti **port default SSH**

Konfigurasi ini sering digunakan pada **server production**.

Target hasil hardening:

* port SSH tidak menggunakan default `22`
* login root dinonaktifkan
* login hanya menggunakan **SSH key**
* hanya user tertentu yang dapat login
* firewall dan SELinux tetap aman
* tidak kehilangan akses server

Estimasi waktu konfigurasi sekitar **15–20 menit**.

---

# 1. Backup Konfigurasi SSH

Sebelum melakukan perubahan konfigurasi, selalu buat backup file konfigurasi.

```bash
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
```

Jika terjadi kesalahan konfigurasi, file dapat dikembalikan dengan:

```bash
cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
```

Backup sangat penting untuk mencegah **server tidak dapat diakses melalui SSH**.

---

# 2. Generate SSH Key di Laptop

SSH key digunakan untuk autentikasi tanpa password dan jauh lebih aman.

Di laptop Anda jalankan:

```bash
ssh-keygen
```

Tekan **Enter** untuk menggunakan konfigurasi default.

File yang akan dibuat:

```
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

Penjelasan:

| File       | Fungsi                         |
| ---------- | ------------------------------ |
| id_rsa     | private key (jangan dibagikan) |
| id_rsa.pub | public key (dikirim ke server) |

---

# 3. Copy SSH Key ke Server

Sebelum melakukan hardening, pastikan login menggunakan SSH key sudah berfungsi.

Gunakan perintah berikut dari laptop:

```bash
ssh-copy-id sysadmin@192.168.249.132
```

Setelah itu lakukan test login:

```bash
ssh sysadmin@192.168.249.132
```

Jika berhasil login **tanpa memasukkan password**, maka SSH key sudah aktif.

---

# 4. Edit Konfigurasi SSH

Buka file konfigurasi SSH server.

```bash
vi /etc/ssh/sshd_config
```

Tambahkan atau ubah konfigurasi berikut.

---

## Mengganti Port SSH

Port default SSH adalah **22**, yang sering menjadi target brute force.

Ganti menjadi port lain, misalnya:

```
Port 2222
```

---

## Menonaktifkan Login Root

Untuk meningkatkan keamanan, login root harus dimatikan.

```
PermitRootLogin no
```

Administrator sebaiknya login menggunakan user biasa lalu menggunakan `sudo`.

---

## Mengaktifkan SSH Key Authentication

Pastikan autentikasi menggunakan key diaktifkan.

```
PubkeyAuthentication yes
```

---

## Menonaktifkan Login Password

⚠️ Lakukan langkah ini **hanya setelah SSH key berhasil digunakan**.

```
PasswordAuthentication no
```

Hal ini mencegah **brute force password attack**.

---

## Membatasi User yang Boleh Login

Batasi hanya user tertentu yang boleh mengakses server.

```
AllowUsers sysadmin
```

Dengan konfigurasi ini hanya user `sysadmin` yang dapat login melalui SSH.

---

## Membatasi Percobaan Login

Tambahkan pembatasan jumlah percobaan login.

```
MaxAuthTries 3
MaxSessions 2
LoginGraceTime 30
```

Penjelasan:

| Parameter      | Fungsi                   |
| -------------- | ------------------------ |
| MaxAuthTries   | maksimal percobaan login |
| MaxSessions    | maksimal session SSH     |
| LoginGraceTime | waktu maksimal login     |

---

## Timeout Session

Agar session tidak dibiarkan terbuka terlalu lama.

```
ClientAliveInterval 300
ClientAliveCountMax 2
```

Artinya:

- server akan mengecek koneksi setiap **5 menit**
- jika tidak ada respon, koneksi akan ditutup

---

## Nonaktifkan Fitur yang Tidak Dibutuhkan

```
X11Forwarding no
PrintMotd no
```

Hal ini mengurangi potensi celah keamanan.

---

# 5. Update SELinux untuk Port Baru

Jika port SSH diganti, SELinux harus diberi izin untuk port tersebut.

Install tool yang dibutuhkan:

```bash
dnf install policycoreutils-python-utils -y
```

Tambahkan port SSH baru:

```bash
semanage port -a -t ssh_port_t -p tcp 2222
```

Cek konfigurasi port:

```bash
semanage port -l | grep ssh
```

Contoh output:

```
ssh_port_t tcp 22, 2222
```

---

# 6. Membuka Port pada Firewall

Tambahkan port baru pada firewall.

```bash
firewall-cmd --permanent --add-port=2222/tcp
```

Reload firewall:

```bash
firewall-cmd --reload
```

Verifikasi:

```bash
firewall-cmd --list-ports
```

Output yang diharapkan:

```
2222/tcp
```

---

# 7. Validasi Konfigurasi SSH

Sebelum restart service, lakukan pengecekan konfigurasi.

```bash
sshd -t
```

Jika tidak ada output, berarti konfigurasi **valid**.

Jika ada error, perbaiki terlebih dahulu sebelum melanjutkan.

---

# 8. Restart Service SSH

Restart service SSH untuk menerapkan konfigurasi.

```bash
systemctl restart sshd
```

Cek apakah port baru sudah aktif:

```bash
ss -tlnp | grep 2222
```

Contoh output:

```
LISTEN 0 128 0.0.0.0:2222
```

---

# 9. Testing Koneksi SSH

⚠️ Jangan menutup session SSH lama sebelum testing selesai.

Buka terminal baru dari laptop dan coba login:

```bash
ssh -p 2222 sysadmin@192.168.249.132
```

Jika berhasil login, konfigurasi sudah berhasil.

Setelah itu barulah Anda bisa logout dari session lama.

---

# 10. Verifikasi Keamanan

Cek port SSH aktif:

```bash
ss -tlnp | grep ssh
```

Cek firewall:

```bash
firewall-cmd --list-ports
```

Cek konfigurasi SELinux:

```bash
semanage port -l | grep ssh
```

---

# Hasil Hardening

Setelah konfigurasi selesai, server memiliki fitur keamanan berikut:

- port SSH tidak menggunakan default
- login root dinonaktifkan
- autentikasi menggunakan SSH key
- login dibatasi hanya untuk user tertentu
- firewall dan SELinux dikonfigurasi dengan benar
- pembatasan percobaan login
- session timeout otomatis

Konfigurasi ini merupakan **standar keamanan dasar server Linux production**.

---

# Diagram Akses Server

Arsitektur koneksi SSH setelah hardening:

```
Laptop
   │
   │ SSH Key
   │ Port 2222
   ▼
Linux Server
   │
   └── sysadmin (sudo access)
```

Root login langsung tidak diperbolehkan.

---
