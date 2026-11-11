+++
title = "Modul Praktikum AWS "
date = 2026-11-10
weight = 10
description = "Panduan praktikum AWS."
draft = false
+++

## Persiapan Cloud Computing 
**Metode:** Teori (30%) & Hands-On Lab (70%)  
**Target Output:** Membangun Arsitektur Web Multi-AZ Terdistribusi (Standar Kasus TechNova)

---

## DAFTAR ALOKASI WAKTU

| Pertemuan / Sesi | Jam Ke- | Modul Utama | Fokus Pembelajaran |
| :--- | :--- | :--- | :--- |
| **Sesi 1** | 1 - 2 | Cloud Foundation & Identity (IAM) | Enforce MFA, IAM Group, User & Policy Least Privilege |
| **Sesi 2** | 3 - 5 | Networking (VPC) & Compute (EC2) | Custom VPC Multi-AZ, Subnetting, IGW, Route Table, Web Cluster |
| **Sesi 3** | 6 - 8 | High Availability & Storage (ALB, RDS, S3) | Application Load Balancer, Private RDS Subnet, S3 Bucket Policy |
| **Sesi 4** | 9 - 10 | Cloud Automation & CLI | AWS CLI Configuration, S3 Data Transfer, User Data Automation |
| **Sesi 5** | 11 - 12 | Simulasi Uji Standar LKS (TechNova) | Final Challenge (Speed & Accuracy), Verifikasi Pengujian & Rubrik |

---

## SESI 1: CLOUD FOUNDATION & IDENTITY MANAGEMENT (IAM)
**Durasi:** 2 Jam  
**Rasio:** 25 Menit Teori | 65 Menit Praktik

### 1. Landasan Teori (30%)
* **AWS Shared Responsibility Model:** AWS bertanggung jawab atas keamanan infrastruktur fisik (*Security OF the Cloud*), sedangkan pelanggan bertanggung jawab atas keamanan OS, firewall, kredensial, dan data (*Security IN the Cloud*).
* **Prinsip Least Privilege:** Pengguna hanya diberikan hak akses minimum yang diperlukan untuk menyelesaikan tugasnya.
* **IAM Components:**
  * **Root Account:** Kredensial super-admin pertama kali; **tidak boleh** digunakan untuk operasional harian.
  * **IAM Users & Groups:** Entitas identitas untuk personil dan pengelompokan hak akses kolektif.
  * **IAM Policies:** Dokumen JSON deklaratif yang mendefinisikan izin (*Effect: Allow/Deny, Action, Resource*).

---

### 2. Hands-on Lab 1: IAM Hardening & Delegation (70%)

#### Skenario Kasus
Perusahaan *TechNova* mewajibkan akun root diamankan dengan MFA. Anda harus menyiapkan akun delegasi operasional bernama `web-admin` yang tergabung dalam grup `WebAdminGroup` dengan izin administrasi penuh.

#### Langkah Kerja

##### A. Amankan Root Account
1. Masuk ke AWS Management Console menggunakan akun email Root.
2. Pada search bar atas, ketik **IAM**, lalu pilih menu **Dashboard**.
3. Di bagian **Security recommendations**, klik **Add MFA**.
4. Beri nama perangkat MFA: `Root-Authenticator`.
5. Buka aplikasi Google Authenticator/Authy di smartphone, pindai QR Code di layar.
6. Masukkan 2 kode TOTP berturut-turut, lalu klik **Add MFA**.

##### B. Membuat User Group & User Baru
1. Di panel kiri IAM, klik **User groups** $\rightarrow$ klik **Create group**.
2. Masukkan **User group name**: `WebAdminGroup`.
3. Pada tabel **Attach permissions policies**, cari dan centang: `AdministratorAccess`.
4. Klik **Create user group**.
5. Di panel kiri, klik **Users** $\rightarrow$ klik **Create user**.
6. Atur identitas pengguna:
   * **User name**: `web-admin`
   * Centang: **Provide user access to the AWS Management Console**.
   * Pilih: **I want to create an IAM user**.
   * Console password: Pilih **Custom password** (buat password kuat).
   * Matikan centang: *Users must change their password at next sign-in* (untuk efisiensi latihan lab).
7. Klik **Next**. Pada halaman **Set permissions**, pilih opsi **Add user to group**.
8. Centang grup `WebAdminGroup`, klik **Next**, lalu klik **Create user**.
9. Salin URL Login Console khusus akun Anda (format: `https://<account-id>.signin.aws.amazon.com/console`).

##### C. Verifikasi Akses
1. Buka jendela *Incognito / Private Browsing*.
2. Buka URL login konsol yang telah disalin.
3. Login sebagai user `web-admin`. Pastikan dashboard AWS terbuka normal tanpa menggunakan kredensial root.

---

## SESI 2: NETWORKING ARCHITECTURE (VPC) & COMPUTE (EC2)
**Durasi:** 3 Jam   
**Rasio:** 35 Menit Teori | 100 Menit Praktik

### 1. Landasan Teori (30%)
* **Amazon Virtual Private Cloud (VPC):** Jaringan virtual privat yang terisolasi secara logis pada AWS.
* **CIDR Notation:** Pengalamatan IP berbasis blok. Prefix `/16` setara dengan $65.536$ alamat IP privat, sedangkan prefix `/24` setara dengan $256$ alamat IP (AWS mencadangkan 5 alamat IP pertama & terakhir per subnet).
* **Public vs Private Subnet:**
  * **Public Subnet:** Tabel rute mengarahkan rute default `0.0.0.0/0` ke **Internet Gateway (IGW)**.
  * **Private Subnet:** Tidak memiliki rute langsung ke IGW; akses internet keluar hanya melalui NAT Gateway.
* **Security Group vs Network ACL:** Security Group bersifat *stateful* (aturan inbound otomatis mengizinkan respon outbound terkait) dan beroperasi pada level instance/ENI.

---

### 2. Hands-on Lab 2: Topologi VPC Multi-AZ & Web Server Cluster (70%)

```
Topologi yang Dibangun:
VPC: 10.30.0.0/16 (Region: us-east-1)
 ├── Internet Gateway (IGW)
 ├── Route Table Publik (0.0.0.0/0 -> IGW)
 ├── AZ us-east-1a:
 │    └── Public Subnet 1: 10.30.10.0/24 --> [EC2: Cloud Web 1]
 ├── AZ us-east-1b:
 │    └── Public Subnet 2: 10.30.20.0/24 --> [EC2: Cloud Web 2]
 └── AZ us-east-1a:
      └── Private Subnet 1: 10.30.50.0/24 (Untuk RDS Database)
```

#### Langkah Kerja

##### A. Membuat Virtual Private Cloud (VPC)
1. Pilih Region: **US East (N. Virginia) `us-east-1`**.
2. Masuk ke dashboard **VPC** $\rightarrow$ klik **Create VPC**.
3. Pilih opsi **VPC only**.
4. Isi parameter:
   * **Name tag**: `VPC-Cloud`
   * **IPv4 CIDR block**: `10.30.0.0/16`
5. Klik **Create VPC**.
6. Klik VPC yang baru dibuat $\rightarrow$ menu **Actions** $\rightarrow$ **Edit VPC settings** $\rightarrow$ centang **Enable DNS hostnames** $\rightarrow$ klik **Save changes**.

##### B. Membuat Subnet
Di menu **Subnets** $\rightarrow$ klik **Create subnet** (pilih VPC: `VPC-Cloud`):
1. **Public Subnet 1**:
   * Name: `Cloud-Public-1`
   * Availability Zone: `us-east-1a`
   * IPv4 CIDR block: `10.30.10.0/24`
2. Klik **Add new subnet** $\rightarrow$ **Public Subnet 2**:
   * Name: `Cloud-Public-2`
   * Availability Zone: `us-east-1b`
   * IPv4 CIDR block: `10.30.20.0/24`
3. Klik **Add new subnet** $\rightarrow$ **Private Subnet 1**:
   * Name: `Cloud-Private-1`
   * Availability Zone: `us-east-1a`
   * IPv4 CIDR block: `10.30.50.0/24`
4. Klik **Create subnet**.
5. Pilih `Cloud-Public-1` $\rightarrow$ **Actions** $\rightarrow$ **Edit subnet settings** $\rightarrow$ centang **Enable auto-assign public IPv4 address** $\rightarrow$ Simpan. Lakukan hal yang sama untuk `Cloud-Public-2`.

##### C. Konfigurasi Internet Gateway & Routing
1. Menu **Internet gateways** $\rightarrow$ klik **Create internet gateway**.
   * Name tag: `VPC-Cloud-IGW` $\rightarrow$ klik **Create**.
2. Pilih IGW tersebut $\rightarrow$ klik **Actions** $\rightarrow$ **Attach to VPC** $\rightarrow$ pilih `VPC-Cloud` $\rightarrow$ klik **Attach**.
3. Menu **Route tables** $\rightarrow$ klik **Create route table**:
   * Name: `RT-Public` | VPC: `VPC-Cloud` $\rightarrow$ **Create**.
4. Pilih `RT-Public` $\rightarrow$ tab **Routes** $\rightarrow$ klik **Edit routes**:
   * Klik **Add route**: Destination `0.0.0.0/0` $\rightarrow$ Target pilih **Internet Gateway** (`VPC-Cloud-IGW`).
   * Klik **Save changes**.
5. Tab **Subnet associations** $\rightarrow$ klik **Edit subnet associations**:
   * Centang `Cloud-Public-1` dan `Cloud-Public-2` $\rightarrow$ klik **Save associations**.

##### D. Konfigurasi Security Group
1. Menu **Security groups** $\rightarrow$ klik **Create security group**.
2. Isi formulir:
   * **Security group name**: `Web-SG`
   * **Description**: `Allow HTTP and SSH access`
   * **VPC**: Pilih `VPC-Cloud`
3. Atur **Inbound rules**:
   * **Rule 1**: Type `SSH` (Port 22), Source `Anywhere-IPv4` (`0.0.0.0/0`).
   * **Rule 2**: Type `HTTP` (Port 80), Source `Anywhere-IPv4` (`0.0.0.0/0`).
4. Klik **Create security group**.

##### E. Provisioning EC2 Instance Cluster
1. Buka layanan **EC2** $\rightarrow$ klik **Launch instances**.
2. Konfigurasi **Cloud Web 1**:
   * **Name**: `Cloud Web 1`
   * **AMI**: Amazon Linux 2 AMI (HVM) Kernel 5.10 / Amazon Linux 2023
   * **Instance type**: `t2.micro` (atau `t3.micro`)
   * **Key pair**: Buat key pair baru `lks-key` (.pem) dan unduh ke komputer.
   * **Network settings** (klik *Edit*):
     * VPC: `VPC-Cloud`
     * Subnet: `Cloud-Public-1`
     * Auto-assign public IP: `Enable`
     * Firewall: *Select existing security group* $\rightarrow$ pilih `Web-SG`.
   * **Advanced details** $\rightarrow$ kolom **User data**:
     ```bash
     #!/bin/bash
     yum update -y
     yum install -y httpd php
     systemctl start httpd
     systemctl enable httpd
     echo "<h1>TechNova Web Cluster - Server: Cloud Web 1</h1>" > /var/www/html/index.php
     ```
3. Klik **Launch instance**.
4. Ulangi proses di atas untuk meluncurkan **Cloud Web 2**:
   * **Name**: `Cloud Web 2`
   * Subnet: `Cloud-Public-2`
   * Security Group: `Web-SG`
   * **User data**:
     ```bash
     #!/bin/bash
     yum update -y
     yum install -y httpd php
     systemctl start httpd
     systemctl enable httpd
     echo "<h1>TechNova Web Cluster - Server: Cloud Web 2</h1>" > /var/www/html/index.php
     ```
5. Klik **Launch instance**.

##### F. Verifikasi Akses Web
1. Salin **Public IPv4 address** dari `Cloud Web 1`, buka di tab peramban baru. Pastikan muncul tulisan: `TechNova Web Cluster - Server: Cloud Web 1`.
2. Buka **Public IPv4 address** dari `Cloud Web 2`. Pastikan muncul tulisan server 2.

---

## SESI 3: HIGH AVAILABILITY (ALB), STORAGE (S3), & DATABASE (RDS)
**Durasi:** 3 Jam   
**Rasio:** 35 Menit Teori | 100 Menit Praktik

### 1. Landasan Teori (30%)
* **Application Load Balancer (ALB):** Beroperasi pada Layer 7 (HTTP/HTTPS) model OSI. Menerima traffic masuk dan mendistribusikannya secara cerdas ke instance backend berdasarkan target group dan *health check*.
* **Amazon Simple Storage Service (S3):** *Object storage* terdistribusi yang sangat tahan (*11 nines durability*), berbasis *key-value*, ideal untuk file statis web.
* **Amazon Relational Database Service (RDS):** Layanan database relasional terkelola (patching otomatis, backup, multi-AZ failover). Ditempatkan di subnet privat agar tidak bisa dieksploitasi langsung dari internet.

---

### 2. Hands-on Lab 3: Load Balancer, S3 Bucket, & DB Subnet Group (70%)

#### Langkah Kerja

##### A. Membuat Target Group & Application Load Balancer
1. Di panel EC2, pilih **Target groups** $\rightarrow$ klik **Create target group**.
   * Target type: **Instances**
   * Target group name: `TG-Web-Cluster`
   * Protocol / Port: `HTTP` / `80`
   * VPC: `VPC-Cloud`
   * Health check path: `/index.php`
2. Klik **Next**.
3. Di bagian **Available instances**, centang `Cloud Web 1` dan `Cloud Web 2`.
4. Klik **Include as pending below** $\rightarrow$ klik **Create target group**.
5. Di panel kiri, klik **Load balancers** $\rightarrow$ klik **Create load balancer**.
6. Pilih **Application Load Balancer** $\rightarrow$ klik **Create**:
   * **Load balancer name**: `ALB-TechNova`
   * **Scheme**: `Internet-facing`
   * **IP address type**: `IPv4`
   * **Network mapping**:
     * VPC: `VPC-Cloud`
     * Mappings: Centang AZ `us-east-1a` (pilih `Cloud-Public-1`) dan AZ `us-east-1b` (pilih `Cloud-Public-2`).
   * **Security groups**: Pilih `Web-SG`.
   * **Listeners and routing**: Protocol HTTP (Port 80) $\rightarrow$ Default action: forward to `TG-Web-Cluster`.
7. Klik **Create load balancer**. Tunggu hingga statusnya berubah dari *Provisioning* menjadi **Active** (2–3 menit).
8. Salin **DNS name** dari load balancer (contoh: `ALB-TechNova-12345.us-east-1.elb.amazonaws.com`).
9. Akses URL DNS tersebut pada browser dan lakukan refresh berulang kali. Halaman web akan bergantian menampilkan respon dari *Cloud Web 1* dan *Cloud Web 2*.

##### B. Membuat Penyimpanan S3 (Static Assets)
1. Buka layanan **Amazon S3** $\rightarrow$ klik **Create bucket**.
2. Masukkan parameter:
   * **Bucket name**: `lks-technova-<nomorpeserta>-storage` (nama harus unik global, huruf kecil dan strip saja).
   * **AWS Region**: `US East (N. Virginia) us-east-1`.
   * Hapus centang **Block all public access** (konfirmasi centang kotak peringatan di bawahnya).
3. Klik **Create bucket**.
4. Masuk ke detail bucket $\rightarrow$ tab **Permissions** $\rightarrow$ pilih **Bucket policy** $\rightarrow$ klik **Edit**. Masukkan policy izin baca publik berikut:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "PublicReadGetObject",
               "Effect": "Allow",
               "Principal": "*",
               "Action": "s3:GetObject",
               "Resource": "arn:aws:s3:::lks-technova-<nomorpeserta>-storage/*"
           }
       ]
   }
   ```
   *(Ganti `<nomorpeserta>` sesuai nama bucket Anda)* $\rightarrow$ klik **Save changes**.

##### C. Konfigurasi Amazon RDS (Private Database)
1. Buat Subnet Tambahan untuk syarat minimal Multi-AZ DB Subnet Group:
   * Masuk ke VPC Subnets $\rightarrow$ **Create Subnet**:
     * Name: `Cloud-Private-2`
     * VPC: `VPC-Cloud`
     * AZ: `us-east-1b`
     * CIDR: `10.30.60.0/24` $\rightarrow$ klik **Create**.
2. Buat Security Group khusus Database:
   * Buat SG dengan nama `DB-SG` pada `VPC-Cloud`.
   * **Inbound rules**: Port `3306` (MySQL/Aurora), Source arahkan ke Security Group `Web-SG` (hanya web server yang dapat berkomunikasi ke database).
3. Buka layanan **RDS** $\rightarrow$ menu **Subnet groups** $\rightarrow$ klik **Create DB subnet group**:
   * Name: `rds-private-group`
   * VPC: `VPC-Cloud`
   * Add subnets: Pilih AZ `us-east-1a` dan `us-east-1b`, lalu pilih CIDR `10.30.50.0/24` dan `10.30.60.0/24`.
   * Klik **Create**.
4. Masuk ke menu **Databases** $\rightarrow$ klik **Create database**:
   * Engine: **MySQL**
   * Templates: **Free tier**
   * DB instance identifier: `technova-db`
   * Master username: `admin` | Password: `Password123!`
   * Connectivity:
     * VPC: `VPC-Cloud`
     * DB Subnet Group: `rds-private-group`
     * Public access: **No**
     * VPC Security Group: Pilih `DB-SG`
5. Klik **Create database** (status database akan menjadi *Creating*).

---

## SESI 4: CLOUD AUTOMATION & AWS CLI
**Durasi:** 2 Jam   
**Rasio:** 20 Menit Teori | 70 Menit Praktik

### 1. Landasan Teori (30%)
* **Command Line Interface (CLI):** Antarmuka terpadu untuk mengelola seluruh *resource* AWS secara programatik melalui terminal, esensial untuk otomatisasi dan integrasi pipeline DevOps (CI/CD).
* **Infrastructure as Code (IaC):** Pendekatan mengelola infrastruktur melalui file definisi/kode yang terversi (seperti AWS CloudFormation atau HashiCorp Terraform), bukan klik manual pada konsol web.

---

### 2. Hands-on Lab 4: Otomasi File Transfer & Audit via AWS CLI (70%)

#### Langkah Kerja

##### A. Konfigurasi CLI dari Instance EC2
1. Hubungkan terminal komputer Anda ke instance `Cloud Web 1` via SSH:
   ```bash
   chmod 400 lks-key.pem
   ssh -i lks-key.pem ec2-user@<IP_PUBLIK_CLOUD_WEB_1>
   ```
2. Buat IAM Access Key untuk user `web-admin`:
   * Buka Console AWS $\rightarrow$ IAM $\rightarrow$ Users $\rightarrow$ `web-admin` $\rightarrow$ Security credentials.
   * Klik **Create access key** $\rightarrow$ pilih **Command Line Interface (CLI)** $\rightarrow$ simpan file `.csv` yang berisi *Access Key ID* dan *Secret Access Key*.
3. Pada terminal SSH instance EC2, inisialisasi kredensial:
   ```bash
   aws configure
   ```
   * *AWS Access Key ID*: Masukkan Access Key Anda
   * *AWS Secret Access Key*: Masukkan Secret Key Anda
   * *Default region name*: `us-east-1`
   * *Default output format*: `json`

##### B. Manipulasi Penyimpanan Objek via CLI
1. Buat file dummy aset web lokal di server:
   ```bash
   echo "Selamat Datang di Portal LKS Cloud Computing" > /home/ec2-user/logo-test.txt
   ```
2. Salin file tersebut ke dalam S3 Bucket menggunakan perintah AWS CLI:
   ```bash
   aws s3 cp /home/ec2-user/logo-test.txt s3://lks-technova-<nomorpeserta>-storage/
   ```
3. Verifikasi daftar isi bucket melalui CLI:
   ```bash
   aws s3 ls s3://lks-technova-<nomorpeserta>-storage/
   ```
4. Uji akses via browser:
   Buka `https://lks-technova-<nomorpeserta>-storage.s3.amazonaws.com/logo-test.txt`. File teks harus dapat dibaca langsung.

##### C. Query Status Infrastruktur secara Programatik
Jalankan query JSON untuk memeriksa status instance EC2 yang sedang berjalan:
```bash
aws ec2 describe-instances \
    --filters "Name=instance-state-name,Values=running" \
    --query "Reservations[*].Instances[*].[InstanceId,PrivateIpAddress,PublicIpAddress,State.Name]" \
    --output table
```

---

## SESI 5: SIMULASI UJI STANDAR LKS (KASUS TECHNOVA) & EVALUASI
**Durasi:** 2 Jam   
**Rasio:** 10 Menit Briefing | 75 Menit Uji Praktik Mandiri

### 1. Lembar Kerja Uji Kompetensi Siswa

#### Instruksi Kerja
Kerjakan skenario simulasi di bawah ini secara mandiri tanpa melihat modul panduan. Anda berperan sebagai Cloud Engineer yang bertanggung jawab membangun infrastruktur sistem aplikasi TechNova dari nol dalam waktu 75 menit.

```
Checklist Tugas Siswa:
[ ] 1. IAM: Buat user "web-admin" dalam group "WebAdminGroup" (Policy: AdministratorAccess).
[ ] 2. VPC: Buat VPC "VPC-Cloud" (10.30.0.0/16).
[ ] 3. Subnets:
      - Cloud-Public-1 (10.30.10.0/24) di us-east-1a
      - Cloud-Public-2 (10.30.20.0/24) di us-east-1b
      - Cloud-Private-1 (10.30.50.0/24) di us-east-1a
[ ] 4. Gateway & Routing: Attach IGW ke VPC, asosiasikan default route ke dua subnet publik.
[ ] 5. Compute: Deploy dua EC2 Amazon Linux ("Cloud Web 1" & "Cloud Web 2") di masing-masing AZ.
[ ] 6. Application Server: Install web server Apache/Nginx + PHP via script User Data otomatis.
[ ] 7. Load Balancer: Deploy ALB "ALB-TechNova" yang merutekan traffic port 80 ke kedua instance.
[ ] 8. Verifikasi: URL DNS ALB berhasil me-load balancing konten dari kedua server web.
```

---

### 2. Format Bukti Pengumpulan (Dokumentasi Tangkapan Layar)
Siswa wajib mengumpulkan laporan PDF berisi tangkapan layar verifikasi:
1. **Screen 1:** Halaman IAM User & User Group dengan kebijakan terpasang.
2. **Screen 2:** Peta Resource Map VPC yang menampilkan VPC, 3 Subnet, dan Route Table yang terhubung ke IGW.
3. **Screen 3:** Status instance EC2 `Cloud Web 1` dan `Cloud Web 2` berstatus *Running* dengan *2/2 checks passed*.
4. **Screen 4:** Tampilan browser saat mengakses DNS Application Load Balancer yang menampilkan konten web bergantian.

---

### 3. Rubrik Penilaian Standar Industri (Measurement & Judgement)

| Sub-Kriteria | Parameter Pengukuran (Measurement) | Bobot | Skor Tercapai |
| :--- | :--- | :--- | :--- |
| **Keamanan (IAM)** | User `web-admin` berada di group yang benar; MFA terkonfigurasi. | 15% | |
| **Jaringan (VPC)** | Blok CIDR VPC dan subnetting multi-AZ sesuai spesifikasi topologi. | 25% | |
| **Compute & Web** | Server web terpasang otomatis via script; melayani port 80 secara mandiri. | 25% | |
| **High Availability** | Target group berstatus *healthy*; ALB mendistribusikan traffic ke 2 AZ. | 25% | |
| **Manajemen Bukti** | Tangkapan layar lengkap, rapi, dan sesuai dengan format standar CIS. | 10% | |
| **TOTAL** | | **100%** | |
