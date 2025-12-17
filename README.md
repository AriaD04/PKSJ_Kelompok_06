# Implementasi IDS (Intrusion Detection System) dengan Snort

[![License](https://img.shields.io/badge/license-Academic-blue.svg)](LICENSE)
[![VirtualBox](https://img.shields.io/badge/platform-VirtualBox-orange.svg)](https://www.virtualbox.org/)
[![Ubuntu](https://img.shields.io/badge/OS-Ubuntu%2024.04-E95420.svg)](https://ubuntu.com/)
[![Snort](https://img.shields.io/badge/IDS-Snort-red.svg)](https://www.snort.org/)

Proyek Akhir Mata Kuliah **Perancangan Keamanan Sistem dan Jaringan**  
Program Studi Teknik Informatika  
Universitas Maritim Raja Ali Haji (UMRAH) - 2025

---

## 📋 Daftar Isi

- [Tentang Proyek](#-tentang-proyek)
- [Fitur Utama](#-fitur-utama)
- [Arsitektur Sistem](#-arsitektur-sistem)
- [Persyaratan Sistem](#-persyaratan-sistem)
- [Instalasi dan Konfigurasi](#-instalasi-dan-konfigurasi)
  - [1. Setup VirtualBox Network](#1-setup-virtualbox-network)
  - [2. Konfigurasi IP Statis](#2-konfigurasi-ip-statis)
  - [3. Instalasi dan Konfigurasi Snort](#3-instalasi-dan-konfigurasi-snort)
  - [4. Custom Rules](#4-custom-rules)
- [Skenario Pengujian](#-skenario-pengujian)
- [Hasil dan Analisis](#-hasil-dan-analisis)
- [Troubleshooting](#-troubleshooting)
- [Referensi](#-referensi)
- [Tim Pengembang](#-tim-pengembang)
- [Lisensi](#-lisensi)

---

## 🎯 Tentang Proyek

Proyek ini mengimplementasikan **Intrusion Detection System (IDS)** menggunakan **Snort** dalam lingkungan virtual untuk mendeteksi berbagai jenis serangan siber secara real-time, termasuk:

- **Port Scanning** (menggunakan Nmap)
- **Brute Force Login** (SSH)
- **Denial of Service (DoS)** - ICMP Flood

Sistem ini dirancang untuk memberikan peringatan dini terhadap ancaman keamanan jaringan dengan menganalisis lalu lintas data dan mencocokkannya dengan aturan deteksi yang telah dikonfigurasi.

---

## ✨ Fitur Utama

- ✅ **Deteksi Real-time** - Monitoring lalu lintas jaringan secara langsung
- ✅ **Custom Rules** - Aturan deteksi yang dapat disesuaikan
- ✅ **Threshold-based Detection** - Mengurangi false positive
- ✅ **Multi-attack Detection** - Mendeteksi berbagai jenis serangan
- ✅ **Logging & Alerting** - Pencatatan dan peringatan otomatis

---

## 🏗️ Arsitektur Sistem

### Topologi Jaringan

```
┌─────────────────────┐         ┌─────────────────────┐
│   Kali Linux        │         │   Metasploitable2   │
│   (Attacker)        │────────▶│   (Target/Victim)   │
│   192.168.56.10     │         │   192.168.56.20     │
└─────────────────────┘         └─────────────────────┘
          │                              ▲
          │                              │
          │         Monitor Traffic      │
          └──────────────────┬───────────┘
                             │
                    ┌────────▼────────┐
                    │  Ubuntu Server  │
                    │   (IDS/Snort)   │
                    │  192.168.56.30  │
                    └─────────────────┘
```

### Komponen Sistem

| Komponen       | OS/Software          | IP Address    | Peran                         |
| -------------- | -------------------- | ------------- | ----------------------------- |
| **Attacker**   | Kali Linux           | 192.168.56.10 | Mesin penyerang (Nmap, Hydra) |
| **Target**     | Metasploitable2      | 192.168.56.20 | Sistem target yang rentan     |
| **IDS Server** | Ubuntu 24.04 + Snort | 192.168.56.30 | Sistem deteksi intrusi        |

---

## 💻 Persyaratan Sistem

### Hardware Requirements

- **CPU**: Minimal Dual-core (Quad-core recommended)
- **RAM**: Minimal 8 GB (16 GB recommended)
- **Storage**: Minimal 50 GB free space
- **Network**: Host-only adapter support

### Software Requirements

- **Hypervisor**: Oracle VirtualBox 7.0+
- **OS Images**:
  - Kali Linux (Latest)
  - Metasploitable2
  - Ubuntu Server 24.04 LTS

---

## 🚀 Instalasi dan Konfigurasi

### 1. Setup VirtualBox Network

#### 1.1 Konfigurasi VM Kali Linux & Metasploitable2

1. Matikan kedua VM
2. Buka **Settings > Network**
3. **Adapter 1**:
   - Attached to: `Host-only Adapter`
   - Name: `VirtualBox Host-Only Ethernet Adapter`
   - Promiscuous Mode: `Allow All`

#### 1.2 Konfigurasi VM Ubuntu IDS Server

1. Matikan VM
2. Buka **Settings > Network**
3. **Adapter 1** (Manajemen):

   - Attached to: `Host-only Adapter`
   - Name: `VirtualBox Host-Only Ethernet Adapter`
   - Promiscuous Mode: `Deny`

4. **Adapter 2** (Monitoring):
   - Enable Network Adapter: ✅
   - Attached to: `Host-only Adapter`
   - Name: `VirtualBox Host-Only Ethernet Adapter`
   - Promiscuous Mode: `Allow All`

---

### 2. Konfigurasi IP Statis

#### 2.1 Kali Linux (192.168.56.10)

```bash
# Edit file konfigurasi network
sudo nano /etc/network/interfaces

# Tambahkan di bagian bawah file:
# Konfigurasi IP Statis untuk Proyek (Host-only)
auto eth0
iface eth0 inet static
    address 192.168.56.10
    netmask 255.255.255.0
```

**Simpan dan reboot:**

```bash
sudo reboot
```

**Verifikasi:**

```bash
ip addr show eth0
# Output harus menampilkan: inet 192.168.56.10/24
```

---

#### 2.2 Metasploitable2 (192.168.56.20)

```bash
# Login dengan kredensial default: msfadmin/msfadmin
sudo nano /etc/network/interfaces

# Ubah bagian eth0 menjadi:
auto eth0
iface eth0 inet static
    address 192.168.56.20
    netmask 255.255.255.0
    gateway 192.168.56.1
```

**Simpan dan reboot:**

```bash
sudo reboot
```

---

#### 2.3 Ubuntu Server IDS (192.168.56.30)

Ubuntu Server menggunakan **Netplan** untuk konfigurasi jaringan.

```bash
sudo nano /etc/netplan/00-installer-config.yaml
```

**Isi file dengan konfigurasi berikut:**

```yaml
network:
  ethernets:
    enp0s3:
      dhcp4: false
      addresses:
        - 192.168.56.30/24
    enp0s8:
      dhcp4: false
  version: 2
```

**Terapkan konfigurasi:**

```bash
sudo netplan apply
```

**Verifikasi:**

```bash
ip addr
# Cek enp0s3 memiliki IP 192.168.56.30
# Cek enp0s8 tidak memiliki IP (untuk monitoring)
```

---

### 3. Instalasi dan Konfigurasi Snort

#### 3.1 Install Snort

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y build-essential libpcap-dev libpcre3-dev \
    libdumbnet-dev bison flex zlib1g-dev liblzma-dev openssl libssl-dev

# Install Snort
sudo apt install -y snort
```

#### 3.2 Konfigurasi Dasar Snort

**Edit file konfigurasi utama:**

```bash
sudo nano /etc/snort/snort.conf
```

**3.2.1 Set HOME_NET Variable**

Cari baris:

```
ipvar HOME_NET any
```

Ubah menjadi:

```
ipvar HOME_NET 192.168.56.0/24
```

**3.2.2 Aktifkan Preprocessor Port Scan**

Cari bagian `preprocessor sfportscan` dan pastikan tidak ada tanda `#` di depannya:

```
preprocessor sfportscan: \
    proto { all } \
    memcap { 10000000 } \
    sense_level { low }
```

#### 3.3 Validasi Konfigurasi

```bash
sudo snort -T -c /etc/snort/snort.conf
```

**Output yang diharapkan:**

```
Snort successfully validated the configuration!
```

---

### 4. Custom Rules

#### 4.1 Deteksi Brute Force SSH

**Edit file local rules:**

```bash
sudo nano /etc/snort/rules/local.rules
```

**Tambahkan rule:**

```
alert tcp any any -> 192.168.56.20 22 (msg:"BAHAYA: Serangan Brute Force SSH Terdeteksi"; flags:S; threshold: type both, track by_src, count 5, seconds 60; sid:1000001; rev:1;)
```

**Penjelasan Rule:**

- `alert tcp any any -> 192.168.56.20 22` - Monitor TCP ke port SSH target
- `flags:S` - Hanya hitung paket SYN (koneksi baru)
- `threshold: count 5, seconds 60` - Trigger jika ≥5 percobaan dalam 60 detik
- `sid:1000001` - Custom rule ID

---

#### 4.2 Deteksi ICMP Flood (DoS)

**Tambahkan rule di file yang sama:**

```
alert icmp any any -> $HOME_NET any (msg:"BAHAYA: Deteksi ICMP Flooding (DoS)"; threshold: type both, track by_src, count 100, seconds 10; sid:1000002; rev:1;)
```

**Penjelasan Rule:**

- `alert icmp any any -> $HOME_NET any` - Monitor semua paket ICMP
- `threshold: count 100, seconds 10` - Trigger jika ≥100 paket dalam 10 detik
- `track by_src` - Lacak berdasarkan IP sumber

**Validasi rules:**

```bash
sudo snort -T -c /etc/snort/snort.conf
```

---

## 🧪 Skenario Pengujian

### Menjalankan Snort dalam Mode Console

**Pada Ubuntu IDS Server:**

```bash
sudo snort -A console -q -c /etc/snort/snort.conf -i enp0s8
```

**Penjelasan parameter:**

- `-A console` - Tampilkan alert di console
- `-q` - Quiet mode (minimal output)
- `-c /etc/snort/snort.conf` - File konfigurasi
- `-i enp0s8` - Interface monitoring (adapter 2)

---

### Pengujian 1: Port Scanning

**Dari Kali Linux (Attacker):**

```bash
sudo nmap -Pn -sS 192.168.56.20
```

**Expected Output di Snort:**

```
[**] [1:1421:11] SNMP AgentX/tcp request [**]
[**] [1:1418:11] SNMP request tcp [**]
Classification: Attempted Information Leak
```

---

### Pengujian 2: Brute Force SSH

#### Persiapan File Password List

```bash
# Buat file password di Kali Linux
nano pass.txt

# Isi dengan beberapa password:
password
123456
admin
msfadmin
user
```

#### Metode 1: Menggunakan Nmap NSE (Recommended)

```bash
sudo nmap -Pn -p 22 --script ssh-brute \
    --script-args userdb=users.txt,passdb=pass.txt \
    192.168.56.20
```

#### Metode 2: Menggunakan Hydra (Alternatif)

```bash
hydra -l msfadmin -P pass.txt ssh://192.168.56.20
```

**Expected Output di Snort:**

```
[**] [1:1000001:1] BAHAYA: Serangan Brute Force SSH Terdeteksi [**]
```

---

### Pengujian 3: ICMP Flood (DoS)

#### Variasi 1: Menggunakan hping3

```bash
sudo hping3 --flood --icmp 192.168.56.20
```

**Expected Output di Snort:**

```
[**] [1:469:3] ICMP PING NMAP [**]
Classification: Attempted Information Leak
```

#### Variasi 2: Menggunakan Ping Flood

```bash
sudo ping -f 192.168.56.20
```

**Expected Output di Snort:**

```
[**] [1:1000002:1] BAHAYA: Deteksi ICMP Flooding (DoS) [**]
```

---

## 📊 Hasil dan Analisis

### Ringkasan Keberhasilan Deteksi

| Skenario         | Tool     | Status        | Custom Rule         |
| ---------------- | -------- | ------------- | ------------------- |
| Port Scanning    | Nmap     | ✅ Terdeteksi | Preprocessor bawaan |
| Brute Force SSH  | Nmap NSE | ✅ Terdeteksi | SID: 1000001        |
| DoS - hping3     | hping3   | ✅ Terdeteksi | Signature bawaan    |
| DoS - Ping Flood | ping -f  | ✅ Terdeteksi | SID: 1000002        |

### Key Findings

1. **Threshold Mechanism**: Sangat efektif mengurangi false positive dengan hanya trigger pada aktivitas mencurigakan intensitas tinggi

2. **Signature vs Behavior Detection**:

   - Snort prioritaskan signature-based detection untuk tool spesifik (hping3 → Nmap signature)
   - Fallback ke behavior-based detection (threshold) untuk serangan generic (ping flood)

3. **Custom Rules Effectiveness**: Rules dengan threshold parameter bekerja optimal untuk mendeteksi serangan volumetrik

---

## 🔧 Troubleshooting

### Problem 1: Hydra "kex error"

**Issue**:

```
[ERROR] could not connect to ssh://192.168.56.20:22 - kex error
```

**Cause**: Incompatibilitas algoritma enkripsi antara Kali Linux modern dan OpenSSH lama di Metasploitable2

**Solution**: Gunakan Nmap NSE sebagai alternatif (lihat Pengujian 2)

---

### Problem 2: Snort tidak mendeteksi traffic

**Checklist**:

1. Pastikan Promiscuous Mode `Allow All` pada adapter monitoring
2. Verifikasi Snort berjalan di interface yang benar (`enp0s8`)
3. Cek HOME_NET sudah sesuai (`192.168.56.0/24`)
4. Pastikan tidak ada firewall yang memblok

**Debug command**:

```bash
# Cek interface
ip addr

# Test packet capture manual
sudo tcpdump -i enp0s8 -n

# Validasi rules
sudo snort -T -c /etc/snort/snort.conf
```

---

### Problem 3: "Snort successfully validated" tapi tidak ada output

**Solution**:

```bash
# Pastikan parameter -q tidak menghalangi output
# Gunakan -A full untuk logging lengkap
sudo snort -A full -c /etc/snort/snort.conf -i enp0s8
```

---

## 📚 Referensi

1. Stallings, W. (2021). _Network Security Essentials: Applications and Standards_ (7th ed.). Pearson Education
2. Anderson, R. J. (2021). _Security Engineering: A Guide to Building Dependable Distributed Systems_ (3rd ed.). Wiley
3. Cisco Systems, Inc. (2025). "Snort: The Open Source IDS/IPS"
4. Nmap Project. (2025). "Nmap Network Scanning: The Official Project Guide"
5. Oracle Corporation. (2025). "Oracle VM VirtualBox User Manual: Version 7.1"

---

## 👥 Tim Pengembang

**Kelompok 6 - Teknik Informatika UMRAH**

| NIM        | Nama                        |
| ---------- | --------------------------- |
| 2201020107 | Aria Dimas Mastur           |
| 2201020121 | Ahmad Zeldiyan              |
| 2201020058 | Abdul Arafah                |
| 2201020120 | Juandienova Abdullah Sofyan |
| 2201020097 | Fauzan Hanif                |

**Dosen Pengampu**: Dr. Hendra Kurniawan, S.Kom., M.Sc.Eng

---

## 📄 Lisensi

Proyek ini dibuat untuk keperluan akademik di **Universitas Maritim Raja Ali Haji (UMRAH)** dalam mata kuliah **Perancangan Keamanan Sistem dan Jaringan**.

---

## 🙏 Acknowledgments

Terima kasih kepada:

- Dr. Hendra Kurniawan atas bimbingan selama pengerjaan proyek
- Snort Community atas dokumentasi dan rules yang comprehensive
- Offensive Security untuk Kali Linux
- Rapid7 untuk Metasploitable2

---

## 📞 Kontak

Untuk pertanyaan atau diskusi lebih lanjut mengenai proyek ini, silakan hubungi tim melalui repository issues.

---

**⭐ Jika repository ini bermanfaat, jangan lupa berikan star!**

**Universitas Maritim Raja Ali Haji (UMRAH)**  
Fakultas Teknik dan Teknologi Kemaritiman  
Program Studi Teknik Informatika  
2025
