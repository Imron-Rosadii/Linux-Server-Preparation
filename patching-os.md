# Update System dan Install Package Dasar

## Tujuan

Pada tahap ini kita akan melakukan:

- update sistem operasi
- install package dasar server
- konfigurasi timezone
- mengaktifkan service penting

Langkah ini penting karena:

- memastikan server memiliki **patch keamanan terbaru**
- menyiapkan **tool administrasi server**
- memastikan sistem siap untuk konfigurasi selanjutnya

Distribusi Linux yang dibahas pada panduan ini:

- **CentOS 7**
- **CentOS 8**
- **Rocky Linux**
- **AlmaLinux**

Rocky Linux dan AlmaLinux merupakan **pengganti CentOS yang kompatibel dengan RHEL (Red Hat Enterprise Linux)**.

---

# 1. Membersihkan Cache Repository

Sebelum melakukan update sistem, disarankan untuk membersihkan cache repository agar sistem menggunakan metadata terbaru.

### CentOS 7

```bash id="rm8yc5"
yum clean all
yum makecache
```

### CentOS 8 / Rocky Linux / AlmaLinux

```bash id="gy9cx7"
dnf clean all
dnf makecache
```

Penjelasan:

| Perintah  | Fungsi                              |
| --------- | ----------------------------------- |
| clean all | menghapus cache repository lama     |
| makecache | membuat metadata repository terbaru |

Langkah ini membantu mencegah error dependency saat update package.

---

# 2. Update Sistem Operasi

Setelah repository siap, lakukan update seluruh package sistem.

### CentOS 7

```bash id="1ovcph"
yum update -y
```

### CentOS 8 / Rocky Linux / AlmaLinux

```bash id="1nfy7b"
dnf update -y
```

Penjelasan:

- `update` memperbarui seluruh package
- `-y` menjawab otomatis konfirmasi instalasi

Update sistem akan:

- memperbaiki bug
- menginstall patch keamanan
- memperbarui kernel dan library

Proses ini biasanya memakan waktu **5–10 menit** tergantung koneksi internet.

---

# 3. Install Repository Tambahan (EPEL)

Beberapa package penting seperti **fail2ban** tersedia pada repository tambahan bernama **EPEL (Extra Packages for Enterprise Linux)**.

Install repository EPEL terlebih dahulu.

### CentOS 7

```bash id="npeklf"
yum install -y epel-release
```

### CentOS 8 / Rocky Linux / AlmaLinux

```bash id="imw74g"
dnf install -y epel-release
```

Setelah EPEL aktif, server dapat menginstall berbagai package tambahan.

---

# 4. Install Package Dasar Server

Install package yang biasanya dibutuhkan untuk administrasi server.

### CentOS 7

```bash id="ntksy4"
yum install -y vim wget curl git \
net-tools bind-utils telnet \
htop tcpdump lsof \
lvm2 mdadm smartmontools \
firewalld fail2ban \
chrony \
bash-completion \
policycoreutils-python-utils
```

### CentOS 8 / Rocky Linux / AlmaLinux

```bash id="x4pab6"
dnf install -y vim wget curl git \
net-tools bind-utils telnet \
htop tcpdump lsof \
lvm2 mdadm smartmontools \
firewalld fail2ban \
chrony \
bash-completion \
policycoreutils-python-utils
```

---

# 5. Penjelasan Package yang Diinstall

## Tool Administrasi

| Package | Fungsi                        |
| ------- | ----------------------------- |
| vim     | editor teks untuk konfigurasi |
| wget    | download file dari internet   |
| curl    | tool HTTP request             |
| git     | version control               |

---

## Network Tools

| Package    | Fungsi                                   |
| ---------- | ---------------------------------------- |
| net-tools  | command seperti `ifconfig` dan `netstat` |
| bind-utils | tool DNS seperti `dig` dan `nslookup`    |
| telnet     | testing koneksi port                     |

---

## Monitoring dan Debugging

| Package | Fungsi                                         |
| ------- | ---------------------------------------------- |
| htop    | monitoring CPU dan memory                      |
| tcpdump | analisa network traffic                        |
| lsof    | melihat proses yang menggunakan port atau file |

---

## Storage Management

| Package       | Fungsi                    |
| ------------- | ------------------------- |
| lvm2          | Logical Volume Manager    |
| mdadm         | software RAID management  |
| smartmontools | monitoring kesehatan disk |

---

## Security Tools

| Package         | Fungsi                      |
| --------------- | --------------------------- |
| firewalld       | firewall Linux              |
| fail2ban        | proteksi brute force attack |
| policycoreutils | tool konfigurasi SELinux    |

---

## System Utility

| Package         | Fungsi                    |
| --------------- | ------------------------- |
| chrony          | sinkronisasi waktu server |
| bash-completion | auto complete command     |

---

# 6. Mengatur Timezone Server

Agar waktu server sesuai dengan lokasi Indonesia, set timezone ke WIB.

```bash id="q1quy9"
timedatectl set-timezone Asia/Jakarta
```

Cek timezone:

```bash id="ay29jv"
timedatectl
```

Contoh output:

```
Time zone: Asia/Jakarta (WIB)
```

Pengaturan timezone penting untuk:

- log server
- cron job
- monitoring sistem

---

# 7. Mengaktifkan Service Penting

Beberapa service perlu diaktifkan agar berjalan otomatis saat server boot.

---

## Firewall

Aktifkan firewall.

```bash id="t3whsd"
systemctl enable firewalld
systemctl start firewalld
```

Cek status firewall.

```bash id="3vme0q"
systemctl status firewalld
```

Firewall digunakan untuk mengontrol akses jaringan ke server.

---

## Network Time Protocol (Chrony)

Chrony digunakan untuk sinkronisasi waktu server dengan NTP server.

Aktifkan chrony.

```bash id="a4bqlk"
systemctl enable chronyd
systemctl start chronyd
```

Cek status:

```bash id="sr7rsh"
systemctl status chronyd
```

Verifikasi sinkronisasi waktu.

```bash id="sz2ilp"
chronyc tracking
```

---

# 8. Verifikasi Waktu Server

Pastikan waktu server sudah benar.

```bash id="qj52y1"
date
```

Cek konfigurasi waktu.

```bash id="l52e9n"
timedatectl status
```

Pastikan:

- timezone sudah benar
- NTP aktif
- waktu sistem sinkron

---

# Kesimpulan

Setelah langkah ini selesai, server sudah memiliki:

- sistem yang sudah diperbarui
- repository tambahan
- tool administrasi server
- firewall aktif
- proteksi brute force
- sinkronisasi waktu server
