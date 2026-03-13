---

# User Management

## Tujuan

Pada tahap ini kita akan:

- membuat user administrator untuk mengelola server
- membuat user khusus untuk menjalankan aplikasi
- mengkonfigurasi **SSH key authentication**
- memberikan akses **sudo** yang aman

Mengapa penting?

Dalam server production **tidak disarankan menggunakan root secara langsung**, karena:

- berisiko terhadap kesalahan sistem
- sulit melakukan audit aktivitas user
- meningkatkan risiko keamanan

Karena itu dibuat **user admin khusus** yang memiliki akses sudo.

---

# 1. Membuat User Administrator

Buat user bernama `sysadmin`.

```bash
useradd -m -G wheel -s /bin/bash sysadmin
```

Penjelasan parameter:

| Parameter      | Fungsi                                  |
| -------------- | --------------------------------------- |
| `-m`           | membuat home directory `/home/sysadmin` |
| `-G wheel`     | menambahkan user ke group `wheel`       |
| `-s /bin/bash` | menetapkan shell bash                   |

Pada distribusi Linux berbasis **RHEL (CentOS, Rocky, AlmaLinux)**,
group `wheel` biasanya digunakan untuk **akses sudo**.

---

# 2. Mengatur Password User

Set password untuk user `sysadmin`.

```bash
passwd sysadmin
```

Masukkan password yang kuat.

Contoh password policy yang baik:

- minimal **12 karakter**
- kombinasi **huruf besar**
- huruf kecil
- angka
- simbol

Contoh:

```
J4k4rt4!23$%^
```

Password yang kuat membantu mencegah **brute force attack**.

---

# 3. Membuat User untuk Aplikasi

Selain user administrator, biasanya dibuat user khusus untuk menjalankan aplikasi.

Tujuan:

- meningkatkan keamanan
- memisahkan hak akses sistem dan aplikasi

Buat user bernama `appuser`.

```bash
useradd -m -s /sbin/nologin appuser
```

Penjelasan:

| Parameter          | Fungsi                       |
| ------------------ | ---------------------------- |
| `-m`               | membuat home directory       |
| `-s /sbin/nologin` | mencegah user login ke shell |

User ini hanya digunakan oleh aplikasi dan **tidak dapat login ke server**.

---

# 4. Mengunci Password User Aplikasi

Untuk keamanan tambahan, password user aplikasi dapat dikunci.

```bash
passwd -l appuser
```

Perintah ini akan **menonaktifkan login berbasis password**.

---

# 5. Verifikasi User

Pastikan user berhasil dibuat.

```bash
grep appuser /etc/passwd
```

Contoh output:

```
appuser:x:1001:1001::/home/appuser:/sbin/nologin
```

Penjelasan field:

| Field          | Keterangan           |
| -------------- | -------------------- |
| username       | nama user            |
| UID            | user ID              |
| GID            | group ID             |
| home directory | direktori home user  |
| shell          | shell yang digunakan |

---

# 6. Setup SSH Key untuk Administrator

Login menggunakan **SSH key** jauh lebih aman dibanding password.

Buat direktori `.ssh`.

```bash
mkdir -p /home/sysadmin/.ssh
chmod 700 /home/sysadmin/.ssh
```

Penjelasan permission:

| Permission | Fungsi                                       |
| ---------- | -------------------------------------------- |
| 700        | hanya pemilik yang dapat mengakses directory |

---

# 7. Menambahkan Public Key

Buat file `authorized_keys`.

```bash
vi /home/sysadmin/.ssh/authorized_keys
```

Paste **public key** dari komputer Anda.

Public key biasanya berada di:

```
~/.ssh/id_rsa.pub
```

Contoh isi public key:

```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQD... user@laptop
```

---

# 8. Mengatur Permission SSH

Set permission yang benar agar SSH dapat membaca key.

```bash
chmod 600 /home/sysadmin/.ssh/authorized_keys
chown -R sysadmin:sysadmin /home/sysadmin/.ssh
```

Penjelasan:

| Permission | Fungsi                                 |
| ---------- | -------------------------------------- |
| 600        | hanya user yang dapat membaca file     |
| chown      | memastikan file dimiliki user sysadmin |

Jika permission salah, SSH biasanya akan menolak login key.

---

# 9. Konfigurasi Akses Sudo

Untuk memberikan akses administratif kepada user `sysadmin`, konfigurasi sudo diperlukan.

Gunakan perintah:

```bash
visudo -f /etc/sudoers.d/sysadmin
```

Tambahkan konfigurasi berikut.

```
sysadmin ALL=(ALL) ALL
```

Penjelasan:

| Field    | Makna                                      |
| -------- | ------------------------------------------ |
| sysadmin | nama user                                  |
| ALL      | semua host                                 |
| (ALL)    | bisa menjalankan command sebagai user lain |
| ALL      | semua command diperbolehkan                |

Dengan konfigurasi ini, user `sysadmin` dapat menjalankan command administrator menggunakan `sudo`.

Contoh penggunaan:

```bash
sudo systemctl restart nginx
```

---

# 10. Verifikasi Akses Sudo

Login sebagai user `sysadmin`.

```bash
su - sysadmin
```

Tes akses sudo:

```bash
sudo whoami
```

Jika berhasil, output akan:

```
root
```

Artinya user `sysadmin` memiliki akses administrator.

---

# Kesimpulan

Pada tahap ini server sudah memiliki:

- user administrator (`sysadmin`)
- user aplikasi (`appuser`)
- SSH key authentication
- akses sudo yang aman

---
