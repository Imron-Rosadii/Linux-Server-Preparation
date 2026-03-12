---

```md
# 📦 STEP 4 — Setup LVM (Logical Volume Manager)

Durasi: ±30 menit

LVM adalah sistem manajemen storage pada Linux yang memungkinkan kita
menggabungkan disk fisik dan membaginya menjadi partisi virtual yang fleksibel.

Keuntungan LVM:

- Storage dapat diperbesar tanpa reinstall OS
- Disk dapat digabung menjadi satu pool
- Logical volume bisa diresize
- Mendukung snapshot backup

---

# 1. Konsep Dasar LVM

LVM memiliki tiga layer utama:

### 1️⃣ Physical Volume (PV)

Physical Volume adalah disk fisik atau partisi yang akan digunakan oleh LVM.

Contoh:

```

/dev/sdb
/dev/sdc1

```

---

### 2️⃣ Volume Group (VG)

Volume Group adalah kumpulan beberapa Physical Volume yang digabung menjadi satu storage pool.

Contoh:

```

VG vg_data
├─ /dev/sdb
└─ /dev/sdc

```

---

### 3️⃣ Logical Volume (LV)

Logical Volume adalah "partisi virtual" yang dibuat dari Volume Group.

Contoh:

```

VG vg_data
├─ lv_app
├─ lv_log
└─ lv_tmp

```

LV inilah yang nantinya diformat filesystem dan di-mount.

---

# 2. Analisis Disk Server

Cek semua disk yang tersedia.

```bash
lsblk
```

Contoh output:

```
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk
├─sda1   8:1    0    1M  0 part
├─sda2   8:2    0    1G  0 part /boot
└─sda3   8:3    0   19G  0 part
  ├─almalinux-root  30G
  └─almalinux-swap  3.5G

sdc      8:32   0   15G  0 disk
└─sdc1   8:33   0   15G  0 part

sdb      8:16   0   10G  0 disk
```

Analisis:

| Disk | Fungsi                     |
| ---- | -------------------------- |
| sda  | Disk sistem                |
| sdc  | Extend root VG             |
| sdb  | Disk kosong untuk aplikasi |

Disk yang akan digunakan:

```
/dev/sdb
```

---

# 3. Membuat Partisi LVM

Masuk ke fdisk.

```bash
fdisk /dev/sdb
```

Langkah di dalam fdisk:

```
n  → buat partisi baru
p  → primary
1  → nomor partisi
Enter → default first sector
Enter → gunakan seluruh disk
t  → ubah tipe
8e → Linux LVM
w  → simpan perubahan
```

Verifikasi:

```bash
lsblk
```

Output:

```
sdb
 └─sdb1
```

---

# 4. Membuat Physical Volume

Inisialisasi partisi sebagai Physical Volume.

```bash
pvcreate /dev/sdb1
```

Verifikasi:

```bash
pvs
```

Contoh output:

```
PV         VG        PSize
/dev/sda3  almalinux 19G
/dev/sdc1  almalinux 15G
/dev/sdb1            10G
```

---

# 5. Membuat Volume Group

Buat Volume Group baru untuk aplikasi.

```bash
vgcreate vg_data /dev/sdb1
```

Verifikasi:

```bash
vgs
```

Contoh:

```
VG        VSize
almalinux 34G
vg_data   10G
```

---

# 6. Membuat Logical Volume

Alokasi storage:

| LV     | Ukuran | Fungsi          |
| ------ | ------ | --------------- |
| lv_app | 4G     | aplikasi        |
| lv_log | 3G     | log aplikasi    |
| lv_tmp | 2G     | temporary files |

Buat LV:

```bash
lvcreate -n lv_app -L 4G vg_data
lvcreate -n lv_log -L 3G vg_data
lvcreate -n lv_tmp -L 2G vg_data
```

Verifikasi:

```bash
lvs
```

Output:

```
LV      VG      LSize
lv_app  vg_data 4G
lv_log  vg_data 3G
lv_tmp  vg_data 2G
```

---

# 7. Membuat Filesystem

Gunakan filesystem default Linux enterprise yaitu **XFS**.

```bash
mkfs.xfs /dev/vg_data/lv_app
mkfs.xfs /dev/vg_data/lv_log
mkfs.xfs /dev/vg_data/lv_tmp
```

---

# 8. Membuat Mount Point

Buat direktori tempat filesystem akan di-mount.

```bash
mkdir /app
mkdir -p /var/log/app
mkdir /tmp/app
```

Penjelasan:

| Direktori    | Fungsi         |
| ------------ | -------------- |
| /app         | aplikasi utama |
| /var/log/app | log aplikasi   |
| /tmp/app     | file sementara |

---

# 9. Mount Logical Volume

Mount filesystem ke direktori.

```bash
mount /dev/vg_data/lv_app /app
mount /dev/vg_data/lv_log /var/log/app
mount /dev/vg_data/lv_tmp /tmp/app
```

Verifikasi:

```bash
df -h
```

Contoh output:

```
Filesystem            Size  Used Avail Mounted on
/dev/vg_data/lv_app    4G    20M  3.9G  /app
/dev/vg_data/lv_log    3G    10M  2.9G  /var/log/app
/dev/vg_data/lv_tmp    2G     5M  1.9G  /tmp/app
```

---

# 10. Setup Auto Mount

Backup fstab terlebih dahulu.

```bash
cp /etc/fstab /etc/fstab.backup
```

Ambil UUID.

```bash
blkid
```

Contoh output:

```
/dev/vg_data/lv_app: UUID="xxxx"
/dev/vg_data/lv_log: UUID="yyyy"
/dev/vg_data/lv_tmp: UUID="zzzz"
```

Edit fstab.

```bash
vi /etc/fstab
```

Tambahkan:

```
UUID=xxxx /app xfs defaults,noatime 0 0
UUID=yyyy /var/log/app xfs defaults,noatime 0 0
UUID=zzzz /tmp/app xfs defaults,noatime,nosuid,noexec 0 0
```

---

# 11. Test Konfigurasi fstab

Jalankan:

```bash
mount -a
```

Jika tidak ada error berarti konfigurasi benar.

---

# 12. Set Permission

Atur permission direktori.

```bash
chmod 755 /app
chmod 755 /var/log/app
chmod 1777 /tmp/app
```

Penjelasan:

| Permission | Fungsi                |
| ---------- | --------------------- |
| 755        | direktori normal      |
| 1777       | sticky bit untuk /tmp |

---

# 13. Struktur Storage Akhir

Server akan memiliki layout seperti ini:

```
Disk
├─ sda → OS
├─ sdc → extend root
└─ sdb → storage aplikasi

LVM
├─ VG almalinux
│  ├─ root
│  └─ swap
│
└─ VG vg_data
   ├─ lv_app
   ├─ lv_log
   └─ lv_tmp
```

Mount point:

```
/
/boot
/app
/var/log/app
/tmp/app
```

---

# 14. Best Practice Production

Server production biasanya memisahkan filesystem berikut:

```
/
/var
/var/log
/tmp
/home
/app
/backup
```

Tujuannya:

Jika log penuh maka **root filesystem tidak ikut penuh** sehingga server tetap stabil.

---

# 15. Verifikasi Akhir

Cek storage:

```bash
lsblk
```

Cek mount:

```bash
df -h
```

Cek LVM:

```bash
pvs
vgs
lvs
```

---

# 16. Kesimpulan

Setelah langkah ini selesai:

- Disk berhasil ditambahkan ke LVM
- Logical Volume dibuat untuk aplikasi
- Filesystem diformat
- Mount point aktif
- Auto mount saat reboot

Server sekarang memiliki storage layout yang siap digunakan untuk environment production.

```

---



```

```


```
