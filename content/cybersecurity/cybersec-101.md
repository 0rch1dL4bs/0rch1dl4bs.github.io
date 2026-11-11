+++
title = "Cyber Security Fundamental"
description = "Modul pembelajaran Cyber Security 101."
weight = 10
+++

**Pendekatan: 30% Teori | 70% Hands-on Lab & Challenge**

Dokumen ini adalah modul praktikum intensif Cyber Security. Setiap modul memuat porsi teori esensial (30%) dan didominasi oleh instruksi teknis, setup lab, serta tantangan/challenge praktis (70%).

---

## MODUL A: INFRASTRUCTURE HARDENING

### 1. Teori Hardening (30%)
Hardening adalah proses meminimalkan *attack surface* (celah serangan) dengan menonaktifkan layanan yang tidak perlu, menerapkan kebijakan *least privilege*, dan mengamankan konfigurasi default. 

**Layanan (Services) Target yang Wajib Di-Hardening:**
1. **OpenSSH (Linux) / RDP (Windows):** Protokol remote access. Rentan terhadap *brute-force* dan akses root langsung. Wajib menonaktifkan root login dan membatasi akses.
2. **Web Server (Apache/Nginx):** Rentan terhadap *directory listing*, *information disclosure* (versi banner), dan *insecure HTTP headers*.
3. **File Sharing (Samba/SMB, FTP/ProFTPD):** Rentan terhadap *anonymous login*, *null sessions*, dan penggunaan protokol usang (SMBv1, plaintext FTP).
4. **Database (MySQL/PostgreSQL):** Rentan terhadap remote login (bind ke `0.0.0.0`) dan kredensial default bawaan instalasi.
5. **Active Directory (Windows Server):** Layanan Kerberos (rentan terhadap enkripsi lemah RC4), LDAP (rentan terhadap *anonymous bind*), dan DNS.

### 2. Hands-on Lab & Praktik (70%)

#### Lab Praktik: Mengamankan SSH & Apache (Linux)
**Langkah 1: Hardening OpenSSH (`/etc/ssh/sshd_config`)**
```bash
# Edit konfigurasi SSH
sudo nano /etc/ssh/sshd_config

# Pastikan parameter berikut diset:
PermitRootLogin no          # Melarang root login via SSH
PasswordAuthentication no   # Hanya izinkan Public Key
MaxAuthTries 3              # Batasi percobaan login gagal
Port 2222                   # (Opsional) Ganti port default

# Restart service
sudo systemctl restart sshd
```

**Langkah 2: Hardening Apache2 (`/etc/apache2/conf-available/security.conf`)**
```bash
sudo nano /etc/apache2/conf-available/security.conf

# Sembunyikan versi OS dan Apache
ServerTokens Prod
ServerSignature Off

# Matikan Directory Listing pada /etc/apache2/apache2.conf
# Ubah "Options Indexes FollowSymLinks" menjadi "Options -Indexes +FollowSymLinks"

sudo systemctl restart apache2
```

#### 🏆 CHALLENGE: "The Vulnerable Sysadmin"
* **Skenario:** Anda diberikan sebuah VM Ubuntu (Gunakan mesin virtual Linux lokal Anda). Administrator sebelumnya meninggalkan sistem dalam keadaan rentan.
* **Objektif:**
  1. Buat user baru bernama `lks_admin` dan masukkan ke grup `sudo`.
  2. Kunci password user `root` agar tidak bisa login sama sekali (`sudo passwd -l root`).
  3. Instal `vsftpd` (FTP), lalu konfigurasikan file `/etc/vsftpd.conf` agar **Anonymous Login ditolak** (`anonymous_enable=NO`).
  4. Pastikan `ssh` berjalan, tetapi user `lks_admin` akan di-kick jika salah password sebanyak 2 kali.
* **Validasi (Cara Cek):** Coba login ke FTP menggunakan user `anonymous` dan password bebas. Jika koneksi ditolak (Login Failed), Anda berhasil!

---

## MODUL B: OFFENSIVE / RED-TEAM CTF

### 1. Teori Offensive (30%)
Fase *Red Teaming* berfokus pada menemukan, mengeksploitasi, dan mengambil alih sistem sesuai panduan *OWASP* dan teknik eksploitasi biner.
* **Web Exploitation:** Mengeksploitasi kelemahan input aplikasi (SQLi, XSS, Command Injection, SSRF).
* **Binary Exploitation (Pwn):** Memanipulasi memori (Stack/Heap overflow) untuk mengeksekusi *shellcode* atau *Return-Oriented Programming (ROP)* mem-bypass proteksi seperti NX dan ASLR.
* **Cryptography:** Memecahkan algoritma enkripsi karena kesalahan implementasi (misal: modulus RSA yang bocor, kunci AES yang lemah).
* **Boot2Root:** Siklus penuh mulai dari *Reconnaissance / Scanning* -> *Initial Access* (Eksploitasi service) -> *Privilege Escalation* (Mendapatkan akses root/Administrator).

### 2. Hands-on Lab & Praktik (70%)

#### Lab Praktik: SQL Injection Manual (Web)
Misalkan ada form login dengan query di backend: `SELECT * FROM users WHERE username='$user' AND password='$password'`
```sql
-- Payload input username:
admin' OR '1'='1' -- 

-- Hasil query di backend menjadi:
SELECT * FROM users WHERE username='admin' OR '1'='1' -- AND password=''
-- Kondisi '1'='1' selalu BENAR (TRUE), sehingga login berhasil sebagai user pertama (admin).
```

#### 🏆 CHALLENGE 1: Web Exploitation (Command Injection)
* **Setup:** Instal OWASP Juice Shop lokal menggunakan Docker:
  `docker run --rm -p 3000:3000 bkimminich/juice-shop`
* **Objektif:** Selesaikan tantangan "Admin Section". Akses dashboard admin yang seharusnya disembunyikan.
* **Hint:** Cek file `main.js` di browser DevTools (F12) untuk menemukan *path* yang tersembunyi (Directory Traversal/Information Disclosure).

#### 🏆 CHALLENGE 2: Binary Exploitation (Buffer Overflow / ret2win)
* **Setup:** Buat program C sederhana bernama `vuln.c`:
  ```c
  #include <stdio.h>
  #include <unistd.h>
  void get_flag() {
      printf("LKS{c0ngr4ts_y0u_pwn3d_th1s}\n");
  }
  void vuln() {
      char buffer[32];
      gets(buffer); // VULNERABLE FUNCTION
  }
  int main() { vuln(); return 0; }
  ```
  Kompilasi tanpa proteksi memori: 
  `gcc vuln.c -o vuln -fno-stack-protector -no-pie`
* **Objektif:** Tulis script Python menggunakan `pwntools` untuk mengirim input A melebihi 32 byte, menimpa EIP/RIP, dan mengarahkannya ke alamat memori fungsi `get_flag()`.
* **Hint:** Gunakan `gdb` atau `objdump -d vuln | grep get_flag` untuk mencari alamat memori `get_flag`.

#### 🏆 CHALLENGE 3: Boot2Root (Privilege Escalation)
* **Skenario:** Asumsikan Anda sudah mengeksploitasi web server dan mendapat akses shell terbatas sebagai user biasa (`www-data`).
* **Objektif:** Lakukan Privilege Escalation menggunakan manipulasi file SUID.
* **Setup Lab SUID:** (Jalankan sebagai root di VM lab Anda) `cp /usr/bin/find /tmp/find && chmod +s /tmp/find`.
* **Challenge Task:** Sebagai user biasa, manfaatkan `/tmp/find` yang memiliki hak akses SUID root untuk mendapatkan shell root!
* **Solusi Cepat:** `/tmp/find . -exec /bin/sh -p \; -quit`

---

## MODUL C: DEFENSIVE / BLUE-TEAM CTF

### 1. Teori Defensive (30%)
Fase ini berfokus pada respons insiden, membedah malware, dan analisis jejak digital.
* **Digital Forensics:** Proses identifikasi, pelestarian, ekstraksi, dan dokumentasi bukti digital. Meliputi *Disk* (memulihkan file terhapus via `binwalk`/`photorec`), *Memory* (RAM dump malware), dan *Network* (analisis PCAP).
* **Reverse Engineering (RE):** Membongkar file eksekusi untuk memahami logikanya, biasanya untuk menganalisis malware atau memecahkan algoritma proteksi (analisis statis dan dinamis).
* **SIEM & Log Analysis:** Mencari anomali pada log agregat (contoh: ribuan request 404 dari 1 IP indikasi *Dirbusting*, atau anomali login pada Windows Event Logs).

### 2. Hands-on Lab & Praktik (70%)

#### Lab Praktik: Memory Forensics dengan Volatility 3
Asumsi: Terjadi insiden komputer lambat dan diduga ada malware tanpa file (fileless malware) yang berjalan di RAM.
```bash
# Menampilkan daftar proses pada file RAM dump Windows
python3 vol.py -f memdump.mem windows.pslist

# Mencari injeksi memori (kode berbahaya yang disuntikkan ke proses sah)
python3 vol.py -f memdump.mem windows.malfind

# Mengekstrak riwayat perintah di CMD yang diketik oleh hacker
python3 vol.py -f memdump.mem windows.cmdline
```

#### 🏆 CHALLENGE 1: Network Forensics (Wireshark)
* **Setup:** Cari dan unduh file pcap latihan HTTP dari repository Github publik (misal: *Malware-Traffic-Analysis.net*).
* **Objektif:** Ditemukan indikasi data rahasia perusahaan dicuri via protokol HTTP yang tidak dienkripsi.
* **Task:** Buka file `.pcap` di Wireshark. Filter menggunakan kueri `http.request.method == "POST"`. Cari paket yang memuat file dokumen atau kredensial login. Gunakan fitur *File -> Export Objects -> HTTP* untuk menyelamatkan file yang dicuri tersebut dari jaringan.

#### 🏆 CHALLENGE 2: Log Analysis (SIEM / Syslog Threat Hunting)
* **Setup:** Buat file log simulasi `auth.log` palsu dengan memasukkan 50 percobaan login gagal untuk user `root` dari IP `192.168.1.5`, lalu 1 login sukses dari IP yang sama.
* **Objektif:** Menggunakan Command Line Tools Linux untuk mendeteksi *Threat Hunting*.
* **Task:** Temukan IP address yang melakukan serangan Brute-Force SSH dan deteksi pada jam berapa akses tersebut akhirnya berhasil (Accepted).
* **Hint Command (Grep is your friend):** 
  ```bash
  cat auth.log | grep "Failed password" | awk '{print $11}' | sort | uniq -c
  cat auth.log | grep "Accepted password"
  ```

#### 🏆 CHALLENGE 3: Reverse Engineering (Basic Static Analysis)
* **Setup:** Unduh tool **Ghidra** atau cukup gunakan tool CLI bawaan Linux (`strings` & `ltrace`). Buat program C sederhana yang berisi hardcoded password: `if(strcmp(input, "LKS_2026_Secret_Flag") == 0)`. Kompilasi program tersebut menjadi `crackme.bin`.
* **Objektif:** Temukan password rahasia (flag) di dalam file biner tersebut tanpa melihat source code-nya!
* **Task:**
  1. Jalankan perintah `strings crackme.bin | grep LKS` untuk mencari teks yang dapat dibaca manusia.
  2. Alternatif dinamis: Jalankan `ltrace ./crackme.bin` dan masukkan password sembarangan (misal: "test"). Amati output `ltrace` yang akan menampilkan fungsi `strcmp` saat membandingkan input Anda dengan string aslinya.