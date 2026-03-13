**Server Monitoring Setup**.

---

# Server Monitoring Setup

## Tujuan

Monitoring server diperlukan untuk:

- mengetahui penggunaan **CPU**
- memantau **memory usage**
- melihat **disk I/O**
- memantau **network traffic**
- mendeteksi **bottleneck sistem**

Tanpa monitoring, administrator tidak bisa mengetahui kapan server:

- overload
- kehabisan disk
- mengalami masalah performa

Monitoring dasar ini biasanya digunakan sebelum implementasi sistem monitoring besar seperti:

- Prometheus
- Grafana
- Zabbix
- ELK Stack

---

# 1. Install Monitoring Tools

Install beberapa tools monitoring yang sering digunakan oleh SysAdmin.

```bash
dnf install -y htop sysstat iotop dstat nmon
```

Penjelasan package:

| Package | Fungsi                                      |
| ------- | ------------------------------------------- |
| htop    | monitoring CPU dan memory secara interaktif |
| sysstat | menyediakan tool iostat, mpstat, sar        |
| iotop   | monitoring disk I/O                         |
| dstat   | monitoring resource secara real time        |
| nmon    | monitoring performa server                  |

---

# 2. Monitoring CPU dan Memory

Gunakan `htop` untuk melihat penggunaan CPU dan memory.

```bash
htop
```

Informasi yang ditampilkan:

- penggunaan CPU per core
- penggunaan RAM
- proses yang berjalan
- load average

Keuntungan `htop` dibanding `top`:

- tampilan lebih interaktif
- bisa kill process langsung
- lebih mudah membaca resource usage

---

# 3. Monitoring Disk Usage

Cek kapasitas disk dengan:

```bash
df -h
```

Contoh output:

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   12G   35G  25% /
```

Penjelasan:

| Kolom | Fungsi                    |
| ----- | ------------------------- |
| Size  | total kapasitas disk      |
| Used  | disk yang sudah digunakan |
| Avail | ruang disk tersisa        |
| Use%  | persentase penggunaan     |

Jika penggunaan disk mendekati **80%**, biasanya perlu dilakukan pembersihan log atau penambahan storage.

---

# 4. Monitoring Disk I/O

Disk I/O menunjukkan aktivitas baca tulis disk.

Gunakan perintah:

```bash
iostat -x 1
```

Penjelasan parameter:

| Parameter | Fungsi                       |
| --------- | ---------------------------- |
| -x        | menampilkan statistik detail |
| 1         | refresh setiap 1 detik       |

Informasi penting yang perlu diperhatikan:

- `%util` → tingkat penggunaan disk
- `await` → waktu tunggu disk

Jika `%util` mendekati **100%**, disk kemungkinan menjadi bottleneck.

---

# 5. Monitoring Network

Untuk melihat statistik network gunakan:

```bash
dstat -n
```

Output akan menunjukkan:

- trafik incoming
- trafik outgoing

Contoh monitoring lengkap:

```bash
dstat -cdnm
```

Penjelasan parameter:

| Parameter | Fungsi  |
| --------- | ------- |
| c         | CPU     |
| d         | disk    |
| n         | network |
| m         | memory  |

---

# 6. Monitoring Disk Activity per Process

Gunakan `iotop` untuk melihat proses yang menggunakan disk.

```bash
iotop
```

Informasi yang ditampilkan:

- proses yang membaca disk
- proses yang menulis disk
- penggunaan I/O setiap proses

Tool ini sangat berguna ketika server mengalami **disk slowdown**.

---

# 7. Monitoring Load Average

Load average menunjukkan jumlah proses yang menunggu CPU.

Cek dengan:

```bash
uptime
```

Contoh output:

```
load average: 0.25, 0.30, 0.28
```

Interpretasi:

| Nilai        | Makna           |
| ------------ | --------------- |
| < jumlah CPU | normal          |
| = jumlah CPU | CPU mulai penuh |
| > jumlah CPU | server overload |

Contoh:

Jika server memiliki **4 CPU core**, load ideal sebaiknya **di bawah 4**.

---

# 8. Monitoring Memory

Cek penggunaan memory dengan:

```bash
free -m
```

Contoh output:

```
              total   used   free
Mem:           4096   1024   3072
Swap:          2048      0   2048
```

Penjelasan:

| Kolom | Fungsi             |
| ----- | ------------------ |
| total | total RAM          |
| used  | RAM yang digunakan |
| free  | RAM yang tersedia  |

---

# 9. Monitoring Process

Melihat proses yang berjalan:

```bash
ps aux
```

Melihat proses berdasarkan penggunaan CPU tertinggi:

```bash
ps aux --sort=-%cpu | head
```

Melihat proses berdasarkan penggunaan memory tertinggi:

```bash
ps aux --sort=-%mem | head
```

---

# 10. Monitoring System Resource Secara Historis

Tool `sar` dari package `sysstat` dapat menyimpan statistik sistem.

Aktifkan service sysstat:

```bash
systemctl enable sysstat
systemctl start sysstat
```

Cek penggunaan CPU historis:

```bash
sar -u 1 5
```

Cek penggunaan memory:

```bash
sar -r
```

Cek network:

```bash
sar -n DEV
```

---

# Verifikasi Monitoring Tools

Pastikan tools monitoring sudah tersedia.

```bash
which htop
which iostat
which dstat
which iotop
```

Jika path muncul seperti:

```
/usr/bin/htop
```

berarti tool sudah terinstall.

---

# Hasil Setup Monitoring

Setelah tahap ini selesai, server memiliki kemampuan monitoring berikut:

- CPU monitoring
- memory monitoring
- disk usage monitoring
- disk I/O monitoring
- network monitoring
- process monitoring

Monitoring ini membantu administrator mendeteksi masalah performa sejak awal.

---

# Arsitektur Monitoring Dasar

```
Linux Server
      │
      ▼
Monitoring Tools
(htop, iostat, sar)
      │
      ▼
SysAdmin / DevOps
```

Administrator dapat memonitor server secara real-time melalui tools tersebut.

---

💡
