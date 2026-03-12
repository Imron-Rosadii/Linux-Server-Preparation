````md
# 📚 Panduan Setup Server CentOS

## Konfigurasi Awal Server (Network & Disk Check)

---

# 📝 STEP 1: CEK KONDISI AWAL (±5 menit)

Login sebagai **root** lalu cek kondisi awal server.

```bash
# Cek user yang sedang login
whoami
```
````

Output yang benar:

```
root
```

Jika output `root`, berarti Anda sudah login sebagai **administrator server**.

---

## 1.1 Cek Versi CentOS

```bash
cat /etc/centos-release
```

Contoh output:

```
CentOS Linux release 8.5.2111
```

Ini penting karena **metode konfigurasi network berbeda antara CentOS 7 dan CentOS 8/9**.

---

## 1.2 Cek IP Address Saat Ini (DHCP)

```bash
ip addr show
```

atau

```bash
ifconfig
```

Biasanya server baru masih menggunakan **DHCP**.

---

## 1.3 Cek Disk yang Tersedia

```bash
lsblk
```

Contoh output:

```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   40G  0 disk
├─sda1   8:1    0    1G  0 part /boot
└─sda2   8:2    0   39G  0 part /
sdb      8:16   0  100G  0 disk
sdc      8:32   0  100G  0 disk
```

Keterangan:

| Disk | Fungsi                 |
| ---- | ---------------------- |
| sda  | Disk utama OS          |
| sdb  | Disk tambahan (data)   |
| sdc  | Disk tambahan (backup) |

---

## 1.4 Catat Informasi Penting

Sebelum lanjut konfigurasi, catat informasi berikut:

```
Interface Network : __________ (contoh: ens192 / eth0 / ens33)
IP Address Saat Ini : __________
Gateway : __________
DNS Server : __________
Disk Tambahan : __________ (/dev/sdb /dev/sdc)
```

Untuk cek gateway:

```bash
ip route | grep default
```

Untuk cek DNS:

```bash
cat /etc/resolv.conf
```

---

# 📝 STEP 2: SETUP NETWORK STATIC (±10 menit)

Server production **tidak boleh menggunakan DHCP**, karena IP bisa berubah.

Gunakan **IP Static**.

---

# 2.1 Tentukan Konfigurasi Network

Contoh konfigurasi:

```
IP Address : 192.168.1.100
Netmask    : 255.255.255.0 (/24)
Gateway    : 192.168.1.1
DNS Server : 8.8.8.8, 8.8.4.4
Hostname   : prod-web-01.example.local
```

---

# 2.2 Set Hostname

Set hostname server.

```bash
hostnamectl set-hostname prod-web-01.example.local
```

Verifikasi:

```bash
hostname
hostnamectl
```

Tambahkan ke `/etc/hosts` agar resolusi lokal berfungsi.

```bash
echo "127.0.0.1 prod-web-01.example.local prod-web-01" >> /etc/hosts
```

---

# 2.3 Konfigurasi IP Static

Metode berbeda tergantung versi CentOS.

---

# 🔹 Cara untuk CentOS 7

### Cek file interface

```bash
ls /etc/sysconfig/network-scripts/ifcfg-*
```

Contoh:

```
/etc/sysconfig/network-scripts/ifcfg-ens192
```

---

### Backup file konfigurasi

```bash
cp /etc/sysconfig/network-scripts/ifcfg-ens192 \
/etc/sysconfig/network-scripts/ifcfg-ens192.backup
```

---

### Edit konfigurasi

```bash
vi /etc/sysconfig/network-scripts/ifcfg-ens192
```

Isi konfigurasi:

```
TYPE=Ethernet
BOOTPROTO=static
DEFROUTE=yes
NAME=ens192
DEVICE=ens192
ONBOOT=yes

IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
```

Simpan file:

```
ESC
:wq
```

---

# 🔹 Cara untuk CentOS 8 / CentOS 9

Gunakan **NetworkManager (nmcli)**.

### Cek koneksi network

```bash
nmcli connection show
```

Contoh output:

```
NAME      UUID                                  TYPE      DEVICE
ens192    xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  ethernet  ens192
```

---

### Set IP Static

```bash
nmcli connection modify ens192 ipv4.addresses 192.168.1.100/24

nmcli connection modify ens192 ipv4.gateway 192.168.1.1

nmcli connection modify ens192 ipv4.dns "8.8.8.8 8.8.4.4"

nmcli connection modify ens192 ipv4.method manual
```

---

### Terapkan konfigurasi

```bash
nmcli connection up ens192
```

---

# 2.4 Restart Network & Verifikasi

### CentOS 7

```bash
systemctl restart network
```

### CentOS 8 / 9

```bash
systemctl restart NetworkManager
```

---

# Verifikasi IP Baru

```bash
ip addr show ens192
```

---

# Test Koneksi Network

Test koneksi ke gateway:

```bash
ping -c 3 192.168.1.1
```

Test koneksi ke internet:

```bash
ping -c 3 8.8.8.8
```

Test DNS resolution:

```bash
ping -c 3 google.com
```

---

# Jika Ping Gagal

Periksa routing:

```bash
ip route show
```

Periksa konfigurasi interface:

```bash
cat /etc/sysconfig/network-scripts/ifcfg-ens192
```

---

# ✅ Checklist Setup Network

Pastikan semua berhasil:

- [ ] Hostname sudah di-set
- [ ] IP sudah static
- [ ] Gateway benar
- [ ] DNS berfungsi
- [ ] Bisa ping gateway
- [ ] Bisa ping internet
- [ ] Bisa resolve DNS

---
