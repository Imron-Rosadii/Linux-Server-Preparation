**STEP 8 – SELinux Configuration** .

---

# SELinux Configuration

## Tujuan

**SELinux (Security-Enhanced Linux)** adalah mekanisme keamanan Linux berbasis **Mandatory Access Control (MAC)**.

Berbeda dengan permission Linux biasa (`chmod`, `chown`) yang bersifat **Discretionary Access Control (DAC)**, SELinux mengontrol **apa yang boleh dilakukan proses terhadap file atau resource sistem**.

Contoh perlindungan SELinux:

- web server tidak bisa membaca file sembarangan
- service tidak bisa mengakses port yang tidak diizinkan
- mencegah eksploitasi privilege escalation

Karena itu pada server production **SELinux sebaiknya selalu dalam mode `Enforcing`**.

---

# 1. Mengecek Status SELinux

Gunakan perintah berikut untuk melihat status SELinux.

```bash
getenforce
```

Kemungkinan output:

| Status     | Penjelasan                          |
| ---------- | ----------------------------------- |
| Enforcing  | SELinux aktif dan aturan diterapkan |
| Permissive | aturan hanya dicatat di log         |
| Disabled   | SELinux tidak aktif                 |

Untuk server production, status yang direkomendasikan adalah:

```
Enforcing
```

---

# 2. Melihat Informasi SELinux Lengkap

Untuk melihat informasi SELinux lebih detail:

```bash
sestatus
```

Contoh output:

```
SELinux status:                 enabled
Current mode:                   enforcing
Policy from config file:        enforcing
```

Informasi yang ditampilkan:

- status SELinux
- mode aktif
- policy yang digunakan

---

# 3. Mengaktifkan SELinux Jika Disabled

Jika SELinux dalam keadaan `disabled`, aktifkan melalui file konfigurasi.

Edit file berikut:

```bash
vi /etc/selinux/config
```

Cari baris berikut:

```
SELINUX=disabled
```

Ubah menjadi:

```
SELINUX=enforcing
```

Pilihan mode yang tersedia:

| Mode       | Fungsi                    |
| ---------- | ------------------------- |
| enforcing  | aturan SELinux diterapkan |
| permissive | hanya mencatat log        |
| disabled   | SELinux dimatikan         |

---

# 4. Mengaktifkan Enforcing Tanpa Reboot

Jika SELinux sudah aktif tetapi dalam mode `permissive`, mode dapat diubah tanpa reboot.

```bash
setenforce 1
```

Verifikasi kembali:

```bash
getenforce
```

Output yang diharapkan:

```
Enforcing
```

---

# 5. Mengatur SELinux Context untuk Direktori Web

Ketika membuat direktori baru untuk web server, SELinux harus mengetahui tipe file yang digunakan.

Contoh direktori web:

```
/var/www
```

Tambahkan context SELinux:

```bash
semanage fcontext -a -t httpd_sys_content_t "/var/www(/.*)?"
```

Kemudian terapkan context:

```bash
restorecon -Rv /var/www
```

Penjelasan:

| Perintah          | Fungsi                     |
| ----------------- | -------------------------- |
| semanage fcontext | menambahkan aturan SELinux |
| restorecon        | menerapkan context ke file |

---

# 6. Mengatur Context untuk Direktori Aplikasi

Jika aplikasi disimpan pada direktori lain seperti:

```
/app
```

Tambahkan context SELinux:

```bash
semanage fcontext -a -t httpd_sys_content_t "/app(/.*)?"
```

Terapkan perubahan:

```bash
restorecon -Rv /app
```

Dengan konfigurasi ini, web server dapat membaca file pada direktori tersebut.

---

# 7. Melihat Context SELinux File

Untuk melihat context SELinux pada file atau direktori gunakan:

```bash
ls -Z
```

Contoh output:

```
drwxr-xr-x root root system_u:object_r:httpd_sys_content_t:s0 /var/www
```

Penjelasan bagian context:

```
user:role:type:level
```

Yang paling penting biasanya adalah **type**.

Contoh:

| Type                   | Fungsi                             |
| ---------------------- | ---------------------------------- |
| httpd_sys_content_t    | file static web                    |
| httpd_sys_rw_content_t | file yang boleh ditulis web server |

---

# 8. Mengizinkan Web Server Menulis File

Jika aplikasi perlu menulis file (misalnya upload file), gunakan tipe berikut.

```bash
semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/uploads(/.*)?"
restorecon -Rv /var/www/uploads
```

Dengan tipe ini web server bisa:

- membaca
- menulis
- mengubah file

---

# 9. Troubleshooting SELinux

Jika aplikasi tidak bisa mengakses file, periksa log SELinux.

```bash
journalctl -t setroubleshoot
```

Atau:

```bash
ausearch -m AVC,USER_AVC -ts recent
```

Log ini akan menunjukkan **akses yang ditolak oleh SELinux**.

---

# 10. Best Practice SELinux Production

Untuk server production:

- gunakan mode **Enforcing**
- jangan menonaktifkan SELinux kecuali benar-benar diperlukan
- gunakan **context yang tepat**
- periksa log jika ada masalah

SELinux memberikan **lapisan keamanan tambahan** yang sangat penting pada server Linux.

---

# Verifikasi Konfigurasi

Pastikan konfigurasi SELinux sudah benar.

```bash
getenforce
```

Output harus:

```
Enforcing
```

Cek context direktori:

```bash
ls -Zd /var/www
```

---

# Hasil Konfigurasi

Setelah langkah ini selesai, server memiliki:

- SELinux aktif dalam mode **Enforcing**
- context SELinux yang benar untuk direktori web
- perlindungan terhadap akses file yang tidak sah

Ini merupakan bagian dari **baseline security server Linux production**.

---
