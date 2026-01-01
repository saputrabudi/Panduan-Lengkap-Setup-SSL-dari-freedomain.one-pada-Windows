# Panduan Lengkap Setup SSL dari freedomain.one pada Windows

Panduan ini menjelaskan langkah-langkah lengkap untuk setup SSL certificate dari freedomain.one untuk domain Anda, mulai dari install tools hingga konfigurasi di XAMPP.

## 📋 Daftar Isi

1. [Install IIS Express](#1-install-iis-express)
2. [Install OpenSSL](#2-install-openssl)
3. [Download SSL Certificate dari freedomain.one](#3-download-ssl-certificate-dari-freedomainone)
4. [Extract dan Prepare File SSL](#4-extract-dan-prepare-file-ssl)
5. [Konfigurasi SSL untuk XAMPP](#5-konfigurasi-ssl-untuk-xampp)
6. [Verifikasi dan Testing](#6-verifikasi-dan-testing)
7. [Troubleshooting](#7-troubleshooting)

---

## 1. Install IIS Express

IIS Express diperlukan untuk testing dan development. Meskipun kita akan menggunakan XAMPP, IIS Express berguna untuk verifikasi certificate.

### Langkah Install IIS Express:

1. **Download IIS Express:**
   - Kunjungi: https://www.microsoft.com/en-us/download/details.aspx?id=48264
   - Atau cari "IIS Express download" di Google
   - Download file: `iisexpress_amd64_en-US.msi` (untuk Windows 64-bit)

2. **Install:**
   - Double-click file `.msi` yang sudah didownload
   - Ikuti wizard installation
   - Pilih lokasi install (default: `C:\Program Files\IIS Express`)
   - Klik **Install** dan tunggu hingga selesai

3. **Verifikasi Install:**
   ```powershell
   # Buka PowerShell
   iisexpress.exe /?
   ```
   Jika muncul help menu, berarti install berhasil.

---

## 2. Install OpenSSL

OpenSSL diperlukan untuk convert dan manage SSL certificate.

### Opsi A: Install dengan Chocolatey (Disarankan)

1. **Install Chocolatey (jika belum ada):**
   ```powershell
   # Buka PowerShell sebagai Administrator
   Set-ExecutionPolicy Bypass -Scope Process -Force
   [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
   iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
   ```

2. **Install OpenSSL:**
   ```powershell
   choco install openssl
   ```

### Opsi B: Download Manual

1. **Download OpenSSL:**
   - Kunjungi: https://slproweb.com/products/Win32OpenSSL.html
   - Download: **Win64 OpenSSL v3.x.x** (Light version cukup)
   - Atau: **Win64 OpenSSL v1.1.x** (jika v3 tidak kompatibel)

2. **Install:**
   - Double-click installer
   - Pilih lokasi install: `C:\Program Files\OpenSSL-Win64\`
   - Pilih "Copy OpenSSL DLLs to: The OpenSSL binaries (/bin) directory"
   - Klik **Install**

3. **Verifikasi Install:**
   ```powershell
   openssl version
   ```
   Atau:
   ```powershell
   & "C:\Program Files\OpenSSL-Win64\bin\openssl.exe" version
   ```

---

## 3. Download SSL Certificate dari freedomain.one

freedomain.one menyediakan SSL certificate gratis yang valid selama 90 hari dan bisa diperpanjang.

### Langkah Download:

1. **Login ke freedomain.one:**
   - Buka: https://freedomain.one
   - Login dengan akun Anda

2. **Akses SSL Management:**
   - Klik menu **"DOMAINS"** di sidebar
   - Atau langsung ke: https://freedomain.one/Direct.sv?cmd=userSSLMgm

3. **Pilih Domain:**
   - Pilih domain yang ingin Anda setup SSL
   - Klik **"Download SSL"** button

4. **Download File:**
   - File akan terdownload dalam format ZIP
   - Simpan file ZIP ke folder yang mudah diakses, contoh: `C:\SSL-Certificates\`

5. **Informasi SSL:**
   - **Issued Date**: Tanggal sertifikat dibuat
   - **Expiration Date**: Tanggal sertifikat expire (90 hari dari issued)
   - **Issuer**: ZeroSSL (Certificate Authority)
   - **Valid untuk**: Domain utama dan wildcard (*.domain.com)

---

## 4. Extract dan Prepare File SSL

Setelah download, extract dan siapkan file SSL untuk digunakan.

### Langkah Extract:

1. **Extract File ZIP:**
   - Klik kanan file ZIP yang didownload
   - Pilih **"Extract All..."**
   - Pilih folder tujuan, contoh: `C:\SSL-Certificates\yourdomain.com\`
   - Klik **Extract**

2. **File yang Didapat:**
   Setelah extract, biasanya Anda akan mendapatkan:
   - `yourdomain.com.cer` atau `yourdomain.com.crt` - Certificate file
   - `yourdomain.com.key` - Private key file
   - `ca.cer` atau `ca.crt` - CA certificate (intermediate)
   - `fullchain.cer` atau `fullchain.crt` - Full chain certificate (opsional)

3. **Verifikasi File:**
   ```powershell
   # Buka PowerShell
   cd C:\SSL-Certificates\yourdomain.com
   Get-ChildItem
   ```
   
   Pastikan file berikut ada:
   - ✅ Certificate file (`.cer` atau `.crt`)
   - ✅ Private key file (`.key`)
   - ✅ CA certificate (`.cer` atau `.crt`)

4. **Buat Fullchain (Jika Belum Ada):**
   
   Jika tidak ada file `fullchain`, buat dengan menggabungkan certificate dan CA:
   
   ```powershell
   # Buka PowerShell di folder SSL
   $cert = Get-Content "yourdomain.com.cer" -Raw
   $ca = Get-Content "ca.cer" -Raw
   $fullchain = $cert + "`n" + $ca
   $fullchain | Out-File "fullchain.cer" -Encoding ASCII -NoNewline
   ```

5. **Convert ke Format yang Diperlukan (Jika Perlu):**
   
   **Convert .cer ke .crt (untuk Apache):**
   ```powershell
   # Copy file dan rename
   Copy-Item "yourdomain.com.cer" "yourdomain.com.crt"
   Copy-Item "ca.cer" "ca.crt"
   Copy-Item "fullchain.cer" "fullchain.crt"
   ```
   
   **Convert ke PFX (untuk Windows Certificate Store):**
   ```powershell
   $opensslPath = "C:\Program Files\OpenSSL-Win64\bin\openssl.exe"
   $pfxPassword = "YourSecurePassword123!"
   
   & $opensslPath pkcs12 -export `
       -out "yourdomain.com.pfx" `
       -inkey "yourdomain.com.key" `
       -in "fullchain.cer" `
       -password "pass:$pfxPassword"
   ```

---

## 5. Konfigurasi SSL untuk XAMPP

Setelah file SSL siap, konfigurasi Apache di XAMPP untuk menggunakan SSL.

### Step 1: Copy Certificate ke Folder Apache

```powershell
# Buka PowerShell sebagai Administrator
$domain = "yourdomain.com"  # Ganti dengan domain Anda
$xamppPath = "C:\xampp"  # Sesuaikan dengan lokasi XAMPP Anda
$sslSource = "C:\SSL-Certificates\$domain"  # Folder SSL Anda
$sslTarget = "$xamppPath\apache\conf\ssl"

# Buat folder ssl jika belum ada
if (-not (Test-Path $sslTarget)) {
    New-Item -ItemType Directory -Path $sslTarget -Force | Out-Null
}

# Copy file
Copy-Item "$sslSource\$domain.cer" "$sslTarget\$domain.crt" -Force
Copy-Item "$sslSource\$domain.key" "$sslTarget\$domain.key" -Force
Copy-Item "$sslSource\fullchain.cer" "$sslTarget\$domain-fullchain.crt" -Force
Copy-Item "$sslSource\ca.cer" "$sslTarget\ca.crt" -Force
```

### Step 2: Aktifkan mod_ssl

1. **Buka file:** `C:\xampp\apache\conf\httpd.conf`

2. **Cari baris berikut (biasanya sekitar baris 180-200):**
   ```apache
   #LoadModule ssl_module modules/mod_ssl.so
   ```

3. **Hapus tanda `#` untuk mengaktifkan:**
   ```apache
   LoadModule ssl_module modules/mod_ssl.so
   ```

4. **Aktifkan httpd-ssl.conf:**
   
   Cari baris:
   ```apache
   #Include conf/extra/httpd-ssl.conf
   ```
   
   Hapus tanda `#`:
   ```apache
   Include conf/extra/httpd-ssl.conf
   ```

### Step 3: Konfigurasi Virtual Host dengan SSL

1. **Buka file:** `C:\xampp\apache\conf\extra\httpd-vhosts.conf`

2. **Tambahkan konfigurasi berikut di akhir file:**

   ```apache
   # Virtual Host untuk yourdomain.com dengan SSL
   <VirtualHost *:443>
       ServerName yourdomain.com
       ServerAlias www.yourdomain.com
       DocumentRoot "C:/xampp/htdocs"
       
       SSLEngine on
       SSLCertificateFile "C:/xampp/apache/conf/ssl/yourdomain.com.crt"
       SSLCertificateKeyFile "C:/xampp/apache/conf/ssl/yourdomain.com.key"
       SSLCertificateChainFile "C:/xampp/apache/conf/ssl/yourdomain.com-fullchain.crt"
       
       <Directory "C:/xampp/htdocs">
           Options Indexes FollowSymLinks
           AllowOverride All
           Require all granted
       </Directory>
       
       ErrorLog "logs/yourdomain.com-ssl-error.log"
       CustomLog "logs/yourdomain.com-ssl-access.log" common
   </VirtualHost>

   # Redirect HTTP ke HTTPS
   <VirtualHost *:80>
       ServerName yourdomain.com
       ServerAlias www.yourdomain.com
       Redirect permanent / https://yourdomain.com/
   </VirtualHost>
   ```

3. **Sesuaikan dengan domain dan folder Anda:**
   - Ganti `yourdomain.com` dengan domain Anda
   - Ganti `C:/xampp/htdocs` dengan folder aplikasi Anda (jika berbeda)
   - Sesuaikan path certificate jika berbeda

4. **Aktifkan httpd-vhosts.conf:**
   
   Buka `C:\xampp\apache\conf\httpd.conf` dan pastikan ada:
   ```apache
   Include conf/extra/httpd-vhosts.conf
   ```

### Step 4: Test Konfigurasi Apache

```powershell
# Buka PowerShell
cd C:\xampp\apache\bin
.\httpd.exe -t
```

Jika muncul **"Syntax OK"**, berarti konfigurasi benar.

### Step 5: Restart Apache

1. **Buka XAMPP Control Panel**
2. **Klik "Stop" pada Apache**
3. **Tunggu beberapa detik**
4. **Klik "Start" pada Apache**

Atau via command line:
```powershell
# Stop
C:\xampp\apache_stop.bat

# Start
C:\xampp\apache_start.bat
```

---

## 6. Verifikasi dan Testing

### 6.1. Cek Apache Berjalan

```powershell
# Cek process Apache
Get-Process | Where-Object { $_.ProcessName -like "*httpd*" }

# Cek port 443
netstat -ano | findstr :443
```

### 6.2. Test SSL Connection

```powershell
# Test dengan OpenSSL
openssl s_client -connect yourdomain.com:443 -servername yourdomain.com

# Test dengan PowerShell
Test-NetConnection -ComputerName yourdomain.com -Port 443
```

### 6.3. Test di Browser

1. **Buka browser**
2. **Akses:** `https://yourdomain.com`
3. **Cek icon gembok** di address bar
4. **Klik icon gembok** untuk lihat detail certificate:
   - **Issuer**: ZeroSSL RSA Domain Secure Site CA
   - **Valid until**: Sesuai expiration date
   - **Domain**: yourdomain.com

### 6.4. Cek Log Error (Jika Ada Masalah)

```
C:\xampp\apache\logs\yourdomain.com-ssl-error.log
C:\xampp\apache\logs\error.log
```

---

## 7. Troubleshooting

### Masalah: "Not Secure" masih muncul di browser

**Kemungkinan penyebab:**
1. Apache belum di-restart
2. Certificate tidak terbaca
3. DocumentRoot salah
4. DNS belum mengarah ke IP server

**Solusi:**
1. **Restart Apache** dari XAMPP Control Panel
2. **Cek log error:**
   ```
   C:\xampp\apache\logs\error.log
   ```
3. **Verifikasi file certificate:**
   ```powershell
   Get-ChildItem C:\xampp\apache\conf\ssl\
   ```
4. **Test konfigurasi:**
   ```powershell
   C:\xampp\apache\bin\httpd.exe -t
   ```

### Masalah: Apache tidak bisa start

**Cek error:**
```powershell
C:\xampp\apache\bin\httpd.exe -t
```

**Kemungkinan masalah:**
- Syntax error di httpd.conf atau httpd-vhosts.conf
- Port 443 sudah digunakan aplikasi lain
- mod_ssl tidak aktif

**Solusi:**
1. Perbaiki syntax error sesuai output `httpd.exe -t`
2. Cek port 443:
   ```powershell
   netstat -ano | findstr :443
   ```
3. Pastikan mod_ssl aktif di httpd.conf

### Masalah: Certificate tidak valid

**Cek certificate:**
```powershell
openssl x509 -in C:\xampp\apache\conf\ssl\yourdomain.com.crt -text -noout
```

**Pastikan:**
- Certificate tidak expired
- Domain match dengan ServerName
- Fullchain lengkap

### Masalah: 403 Forbidden

**Solusi:**
1. Cek permission folder DocumentRoot
2. Pastikan Directory directive benar:
   ```apache
   <Directory "C:/xampp/htdocs">
       Options Indexes FollowSymLinks
       AllowOverride All
       Require all granted
   </Directory>
   ```

### Masalah: Port 443 sudah digunakan

**Cek aplikasi yang menggunakan port 443:**
```powershell
netstat -ano | findstr :443
```

**Solusi:**
1. Stop aplikasi yang menggunakan port 443
2. Atau ubah port Apache ke 4443 (dan update konfigurasi)

---

## 📋 Checklist Lengkap

- [ ] IIS Express terinstall
- [ ] OpenSSL terinstall
- [ ] SSL certificate didownload dari freedomain.one
- [ ] File SSL di-extract dan disiapkan
- [ ] Certificate di-copy ke folder Apache
- [ ] mod_ssl diaktifkan
- [ ] Virtual Host SSL dikonfigurasi
- [ ] Apache di-restart
- [ ] DNS mengarah ke IP server
- [ ] Test https://yourdomain.com berhasil
- [ ] Tidak ada warning "Not Secure"

---

## 🔄 Renewal SSL Certificate

SSL certificate dari freedomain.one valid selama **90 hari** dan bisa diperpanjang dalam **30 hari** sebelum expire.

### Cara Renewal:

1. **Login ke freedomain.one**
2. **Akses SSL Management**
3. **Pilih domain**
4. **Klik "Renew SSL"** (jika tersedia)
5. **Download certificate baru**
6. **Replace file certificate lama dengan yang baru**
7. **Restart Apache**

---

## 📞 Informasi Penting

- **SSL Provider**: freedomain.one (ZeroSSL)
- **Validitas**: 90 hari
- **Renewal**: 30 hari sebelum expire
- **Format**: PEM (.cer, .crt, .key)
- **Wildcard**: Mendukung wildcard (*.domain.com)

---

## ✅ Script Otomatis

Untuk memudahkan, Anda bisa menggunakan script PowerShell yang sudah dibuat:

1. **Setup SSL XAMPP:**
   ```powershell
   .\setup-ssl-xampp.ps1 -XamppPath "C:\xampp" -Domain "yourdomain.com"
   ```

2. **Import Certificate:**
   ```powershell
   .\import-cert-simple.ps1 -PfxFile "path\to\certificate.pfx"
   ```

---

## 🎯 Quick Reference

**File Penting:**
- Apache Config: `C:\xampp\apache\conf\httpd.conf`
- Virtual Host: `C:\xampp\apache\conf\extra\httpd-vhosts.conf`
- SSL Config: `C:\xampp\apache\conf\extra\httpd-ssl.conf`
- Certificate: `C:\xampp\apache\conf\ssl\`

**Command Penting:**
```powershell
# Test konfigurasi
C:\xampp\apache\bin\httpd.exe -t

# Restart Apache
C:\xampp\apache_stop.bat
C:\xampp\apache_start.bat

# Test SSL
openssl s_client -connect yourdomain.com:443
```

---

**Selamat! SSL certificate sudah dikonfigurasi untuk XAMPP! 🎉**

By Saputra Budi



