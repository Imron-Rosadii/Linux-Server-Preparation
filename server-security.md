**Security Baseline untuk Server Production**.

---

# Server Security Baseline

## Tujuan

Setelah server berhasil dikonfigurasi dan diamankan dengan:

- SSH Hardening
- Firewall
- SELinux

langkah berikutnya adalah menambahkan **lapisan keamanan tambahan** untuk melindungi server dari serangan umum seperti:

- brute force attack
- unauthorized access
- aktivitas mencurigakan pada sistem

Security baseline ini terdiri dari:

- **Fail2Ban** → proteksi brute force login
- **Auditd** → audit aktivitas sistem
- **Log Monitoring** → monitoring log server

Langkah ini biasanya dilakukan oleh **SysAdmin atau DevOps Engineer sebelum server masuk production**.

---

# 1. Install Security Tools

Install paket keamanan yang diperlukan.

```bash
dnf install -y fail2ban fail2ban-firewalld audit
```

Penjelasan:

| Package            | Fungsi                                        |
| ------------------ | --------------------------------------------- |
| fail2ban           | memblokir IP yang mencoba login berulang kali |
| fail2ban-firewalld | integrasi Fail2Ban dengan firewall            |
| audit              | audit aktivitas sistem                        |

---

# 2. Konfigurasi Fail2Ban

Fail2Ban bekerja dengan cara:

1. membaca log server
2. mendeteksi percobaan login gagal
3. memblokir IP penyerang melalui firewall

---

## 2.1 Buat file konfigurasi Fail2Ban

Jangan mengedit file utama langsung.

Buat file konfigurasi baru:

```bash
vi /etc/fail2ban/jail.local
```

Tambahkan konfigurasi berikut.

```ini
[DEFAULT]

bantime  = 3600
findtime = 600
maxretry = 5
backend  = systemd

[sshd]

enabled  = true
port     = 2222
logpath  = %(sshd_log)s
backend  = systemd
```

Penjelasan:

| Parameter | Fungsi                                  |
| --------- | --------------------------------------- |
| bantime   | lama pemblokiran IP (detik)             |
| findtime  | waktu penghitungan percobaan login      |
| maxretry  | jumlah percobaan login sebelum diblokir |

Dalam contoh ini:

- jika IP gagal login **5 kali dalam 10 menit**
- maka IP akan diblokir **1 jam**

---

# 3. Mengaktifkan Fail2Ban

Aktifkan service Fail2Ban.

```bash
systemctl enable fail2ban
systemctl start fail2ban
```

Cek status service.

```bash
systemctl status fail2ban
```

---

# 4. Verifikasi Fail2Ban

Cek jail yang aktif.

```bash
fail2ban-client status
```

Contoh output:

```
Status
|- Number of jail:  1
`- Jail list:   sshd
```

Cek detail jail sshd:

```bash
fail2ban-client status sshd
```

Contoh output:

```
Currently banned: 1
Banned IP list: 192.168.1.10
```

---

# 5. Install Audit System

Linux memiliki sistem audit bawaan bernama **auditd**.

Auditd mencatat:

- login user
- perubahan file penting
- penggunaan command tertentu
- aktivitas sistem sensitif

---

## 5.1 Install auditd

Biasanya sudah tersedia di sistem.

Jika belum:

```bash
dnf install -y audit
```

---

## 5.2 Aktifkan auditd

```bash
systemctl enable auditd
systemctl start auditd
```

Cek status:

```bash
systemctl status auditd
```

---

# 6. Membuat Rule Audit Dasar

Edit file rule audit.

```bash
vi /etc/audit/rules.d/audit.rules
```

Tambahkan rule berikut.

```bash
-w /etc/passwd -p wa -k passwd_changes
-w /etc/shadow -p wa -k shadow_changes
-w /etc/sudoers -p wa -k sudo_changes
-w /var/log/secure -p wa -k auth_log
```

Penjelasan:

| File            | Fungsi           |
| --------------- | ---------------- |
| /etc/passwd     | database user    |
| /etc/shadow     | password user    |
| /etc/sudoers    | konfigurasi sudo |
| /var/log/secure | log autentikasi  |

---

# 7. Reload Audit Rules

Setelah menambahkan rule audit:

```bash
augenrules --load
```

Verifikasi rule aktif:

```bash
auditctl -l
```

---

# 8. Monitoring Log Server

Log adalah sumber utama untuk mengetahui aktivitas server.

Beberapa log penting pada Linux:

| File Log                 | Fungsi                |
| ------------------------ | --------------------- |
| /var/log/secure          | login dan autentikasi |
| /var/log/messages        | log sistem umum       |
| /var/log/audit/audit.log | log auditd            |
| /var/log/cron            | aktivitas cron        |

Contoh melihat log login:

```bash
tail -f /var/log/secure
```

---

# 9. Melihat Aktivitas Login

Untuk melihat login user:

```bash
last
```

Melihat user yang sedang login:

```bash
who
```

Melihat riwayat login gagal:

```bash
lastb
```

---

# 10. Verifikasi Security Baseline

Pastikan semua service berjalan.

```bash
systemctl status fail2ban
systemctl status auditd
```

Cek firewall rule:

```bash
firewall-cmd --list-all
```

Cek jail fail2ban:

```bash
fail2ban-client status
```

---

# Hasil Konfigurasi

Setelah tahap ini selesai, server memiliki sistem keamanan berikut:

- SSH Hardening
- Firewall Protection
- SELinux Enforcement
- Brute Force Protection (Fail2Ban)
- System Audit Logging (Auditd)
- Log Monitoring

Ini merupakan **baseline security standar server Linux production**.

---

# Arsitektur Security Layer

```
Internet
   │
   ▼
Firewall (firewalld)
   │
   ▼
Fail2Ban
   │
   ▼
SSH Hardening
   │
   ▼
SELinux
   │
   ▼
Linux Server
```

Setiap layer menambah perlindungan terhadap server.

---
