# 🌐 Networking untuk Penetration Testing

> Panduan networking fundamental untuk security professional — mencakup konsep, tools, dan teknik analisis manual.

---

## 📋 Daftar Isi
- [OSI Model](#osi-model)
- [TCP/IP](#tcpip)
- [DNS](#dns)
- [HTTP/HTTPS](#httphttps)
- [Firewall, Proxy & CDN](#firewall-proxy--cdn)
- [Tools: Wireshark](#wireshark)
- [Tools: tcpdump](#tcpdump)
- [Tools Networking Lainnya](#tools-networking-lainnya)
- [Manual Tricks untuk Pentest](#manual-tricks-untuk-pentest)

---

## 🏗️ OSI Model

OSI (Open Systems Interconnection) adalah model referensi 7 layer komunikasi jaringan.

```
Layer 7 — Application   → HTTP, HTTPS, FTP, DNS, SMTP
Layer 6 — Presentation  → SSL/TLS, enkripsi, encoding
Layer 5 — Session       → Manajemen sesi, autentikasi
Layer 4 — Transport     → TCP, UDP (port numbers)
Layer 3 — Network       → IP, ICMP, routing
Layer 2 — Data Link     → MAC address, ARP, Ethernet
Layer 1 — Physical      → Kabel, sinyal, bit
```

### Relevansi untuk Pentest

| Layer | Serangan Umum |
|-------|--------------|
| 7 - Application | SQLi, XSS, CSRF, RCE |
| 6 - Presentation | SSL stripping, cert spoofing |
| 5 - Session | Session hijacking, fixation |
| 4 - Transport | Port scanning, SYN flood |
| 3 - Network | IP spoofing, MITM |
| 2 - Data Link | ARP spoofing, MAC flooding |
| 1 - Physical | Tapping, jamming |

---

## 🔌 TCP/IP

### TCP — Transmission Control Protocol

**Three-Way Handshake:**
```
Client  →  SYN        →  Server
Client  ←  SYN-ACK    ←  Server
Client  →  ACK        →  Server
[Koneksi terbentuk]
```

**TCP Flags:**
```
SYN  — Inisiasi koneksi
ACK  — Acknowledgment
FIN  — Tutup koneksi normal
RST  — Reset koneksi (paksa tutup)
PSH  — Push data segera
URG  — Data urgent
```

**Relevansi Pentest:**
```bash
# Nmap menggunakan TCP flags untuk scan types
-sS  → SYN scan (kirim SYN, tunggu SYN-ACK, tidak complete handshake)
-sT  → Full connect scan (complete 3-way handshake)
-sF  → FIN scan (bypass beberapa firewall)
-sN  → NULL scan (tidak ada flags)
-sX  → Xmas scan (FIN+PSH+URG)
```

### UDP — User Datagram Protocol

- Tidak ada handshake, connectionless
- Lebih cepat tapi tidak reliable
- Digunakan: DNS (53), DHCP (67/68), SNMP (161), NTP (123)

```bash
# UDP scan dengan nmap (lebih lambat)
nmap -sU target.com

# UDP + service detection
nmap -sU -sV target.com
```

### Port Ranges

```
0-1023     → Well-known ports (butuh root untuk bind)
1024-49151 → Registered ports
49152-65535→ Dynamic/ephemeral ports
```

### Port Penting yang Harus Dikenal

```
21   → FTP
22   → SSH
23   → Telnet
25   → SMTP
53   → DNS
80   → HTTP
110  → POP3
143  → IMAP
443  → HTTPS
445  → SMB (Windows file sharing)
3306 → MySQL
3389 → RDP (Remote Desktop)
5432 → PostgreSQL
6379 → Redis
8080 → HTTP alt / proxy
8443 → HTTPS alt
9200 → Elasticsearch
27017→ MongoDB
```

---

## 🔍 DNS

### Cara Kerja DNS

```
Browser → DNS Resolver → Root Server → TLD Server → Authoritative NS → IP
```

### Tipe Record DNS

```
A     → Domain ke IPv4          (example.com → 93.184.216.34)
AAAA  → Domain ke IPv6
CNAME → Alias ke domain lain    (www → example.com)
MX    → Mail server
TXT   → Teks bebas (SPF, DKIM, verifikasi)
NS    → Nameserver
PTR   → Reverse DNS (IP ke domain)
SOA   → Start of Authority
SRV   → Service records
```

### Manual DNS Recon

```bash
# Query record A
dig A target.com

# Query semua record
dig ANY target.com

# Query MX (mail server)
dig MX target.com

# Query TXT (sering bocorkan info)
dig TXT target.com

# Query NS (nameserver)
dig NS target.com

# Reverse DNS lookup
dig -x 93.184.216.34

# Gunakan nameserver spesifik
dig @8.8.8.8 target.com

# Zone transfer (jika misconfigured)
dig AXFR target.com @ns1.target.com

# nslookup alternatif
nslookup target.com
nslookup -type=MX target.com
```

### Informasi dari DNS yang Berguna

```bash
# SPF record — bisa bocorkan IP server email/infra
dig TXT target.com | grep spf
# Contoh: "v=spf1 ip4:203.0.113.42 include:sendgrid.net"
#                    ^^^^ IP asli server

# DMARC
dig TXT _dmarc.target.com

# DKIM
dig TXT default._domainkey.target.com
```

---

## 🌍 HTTP/HTTPS

### HTTP Methods

```
GET     → Ambil resource
POST    → Kirim data
PUT     → Update resource (full)
PATCH   → Update resource (partial)
DELETE  → Hapus resource
HEAD    → Seperti GET tapi tanpa body
OPTIONS → Cek methods yang diizinkan
TRACE   → Debug (sering dimatikan)
```

### HTTP Status Codes

```
200 OK              → Berhasil
201 Created         → Resource dibuat
301 Moved Permanently → Redirect permanen
302 Found           → Redirect sementara
400 Bad Request     → Request tidak valid
401 Unauthorized    → Perlu autentikasi
403 Forbidden       → Akses ditolak
404 Not Found       → Resource tidak ada
405 Method Not Allowed
429 Too Many Requests → Rate limited
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
```

### HTTP Headers Penting

**Request Headers:**
```
Host: target.com
User-Agent: Mozilla/5.0 ...
Cookie: session=abc123
Authorization: Bearer TOKEN
Content-Type: application/json
X-Forwarded-For: 127.0.0.1
Referer: https://target.com/login
Origin: https://target.com
```

**Response Headers (info berharga untuk pentest):**
```
Server: nginx/1.18.0          → versi web server
X-Powered-By: PHP/7.4.3      → teknologi backend
Set-Cookie: ...               → session management
X-Frame-Options: DENY         → clickjacking protection
Content-Security-Policy: ...  → CSP rules
Strict-Transport-Security:    → HSTS
Access-Control-Allow-Origin:  → CORS policy
```

### HTTPS & TLS

```bash
# Cek SSL certificate
openssl s_client -connect target.com:443

# Cek versi TLS yang didukung
nmap --script ssl-enum-ciphers -p 443 target.com

# Cek certificate details
echo | openssl s_client -connect target.com:443 2>/dev/null | openssl x509 -text

# Cek expiry date
echo | openssl s_client -connect target.com:443 2>/dev/null | \
  openssl x509 -noout -dates
```

---

## 🛡️ Firewall, Proxy & CDN

### Firewall

Firewall memfilter traffic berdasarkan rules (IP, port, protocol).

**Teknik Deteksi Firewall:**
```bash
# Port filtered vs closed
# filtered = ada firewall (tidak ada response)
# closed   = tidak ada firewall (RST diterima)
nmap -sS target.com

# Traceroute untuk lihat hop
traceroute target.com
nmap --traceroute target.com
```

**Teknik Bypass Firewall (konsep):**
```bash
# Fragment paket
nmap -f target.com

# Decoy scan (sembunyikan source IP)
nmap -D RND:10 target.com

# Gunakan source port tertentu (bypass rules lama)
nmap --source-port 53 target.com

# Scan via proxy
nmap --proxy socks4://proxy:1080 target.com
```

### Proxy

```bash
# Gunakan proxy di curl
curl --proxy http://proxy:8080 https://target.com

# Proxy dengan auth
curl --proxy http://user:pass@proxy:8080 https://target.com

# SOCKS5 proxy
curl --proxy socks5://proxy:1080 https://target.com

# Set proxy di environment
export http_proxy=http://proxy:8080
export https_proxy=http://proxy:8080
```

### CDN Detection

```bash
# Cek apakah target pakai CDN
dig A target.com
# IP Cloudflare: 104.x.x.x, 172.6x.x.x, 103.x.x.x

# Cek header CF-Ray (Cloudflare)
curl -I https://target.com | grep -i "cf-ray\|server\|x-cache"

# Tools otomatis
httpx -u target.com -cdn
```

---

## 🦈 Wireshark

GUI packet analyzer — capture dan analisis traffic jaringan.

### Dasar Penggunaan

```
1. Pilih network interface (eth0, wlan0, lo)
2. Klik Start Capture
3. Lakukan aktivitas yang ingin dianalisis
4. Klik Stop
5. Analisis packets
```

### Display Filters (Paling Penting)

```
# Filter by protocol
http
dns
tcp
udp
icmp
tls
arp

# Filter by IP
ip.addr == 192.168.1.1
ip.src == 192.168.1.1
ip.dst == 192.168.1.1

# Filter by port
tcp.port == 80
tcp.port == 443
udp.port == 53

# Filter by content
http.request.method == "POST"
http.response.code == 200
dns.qry.name == "target.com"

# Kombinasi filter
ip.addr == 192.168.1.1 && tcp.port == 80
http && ip.src == 192.168.1.100

# Follow TCP stream
# Klik kanan packet → Follow → TCP Stream
```

### Capture Filters (saat capture berlangsung)

```
host 192.168.1.1
port 80
port 80 or port 443
net 192.168.1.0/24
not port 22
tcp and port 8080
```

### Tips Wireshark untuk Pentest

```
- Follow HTTP Stream → lihat request/response lengkap
- Statistics → Protocol Hierarchy → lihat distribusi protokol
- Statistics → Conversations → lihat koneksi antar IP
- Edit → Find Packet → cari string di payload
- File → Export Objects → HTTP → export file yang ditransfer
```

---

## 📟 tcpdump

CLI packet analyzer — ringan, powerful, cocok untuk remote server.

### Syntax Dasar

```bash
# Capture di interface tertentu
tcpdump -i eth0

# Capture dengan resolusi nama (lambat)
tcpdump -i eth0 -n   # -n = no name resolution (lebih cepat)

# Verbose output
tcpdump -i eth0 -v
tcpdump -i eth0 -vv   # lebih verbose

# Simpan ke file .pcap
tcpdump -i eth0 -w capture.pcap

# Baca file .pcap
tcpdump -r capture.pcap

# Batasi jumlah packet
tcpdump -i eth0 -c 100
```

### Filter tcpdump

```bash
# Filter by host
tcpdump -i eth0 host 192.168.1.1
tcpdump -i eth0 src host 192.168.1.1
tcpdump -i eth0 dst host 192.168.1.1

# Filter by port
tcpdump -i eth0 port 80
tcpdump -i eth0 port 80 or port 443

# Filter by protocol
tcpdump -i eth0 tcp
tcpdump -i eth0 udp
tcpdump -i eth0 icmp

# Filter by network
tcpdump -i eth0 net 192.168.1.0/24

# Kombinasi
tcpdump -i eth0 host 192.168.1.1 and port 80
tcpdump -i eth0 not port 22

# Capture HTTP traffic
tcpdump -i eth0 -A port 80

# Capture DNS queries
tcpdump -i eth0 udp port 53
```

### tcpdump untuk Pentest

```bash
# Monitor semua HTTP requests (plaintext)
tcpdump -i eth0 -A -s 0 port 80

# Capture credentials FTP/Telnet (plaintext)
tcpdump -i eth0 -A port 21 or port 23

# Monitor DNS queries (lihat domain yang di-resolve)
tcpdump -i eth0 -n udp port 53

# Capture traffic subnet tertentu
tcpdump -i eth0 -w subnet.pcap net 192.168.1.0/24

# Simpan HTTP untuk dianalisis
tcpdump -i eth0 -w http.pcap port 80
# Buka dengan wireshark
wireshark http.pcap
```

---

## 🛠️ Tools Networking Lainnya

### netcat (nc) — Swiss Army Knife Networking

```bash
# Connect ke port (cek apakah open)
nc -zv target.com 80
nc -zv target.com 1-1000   # scan range port

# Banner grabbing (ambil info service)
nc target.com 80
# Ketik: HEAD / HTTP/1.0
# Enter 2x

# Listen di port (buat listener)
nc -lvnp 4444

# Transfer file
# Sender:
nc -lvnp 4444 < file.txt
# Receiver:
nc target.com 4444 > file.txt

# Simple chat
nc -lvnp 4444   # server
nc target.com 4444  # client
```

### curl — HTTP Client Powerful

```bash
# Request dasar
curl https://target.com

# Tampilkan headers saja
curl -I https://target.com

# Tampilkan headers + body
curl -iv https://target.com

# POST request
curl -X POST -d "username=admin&password=test" https://target.com/login

# POST JSON
curl -X POST -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"test"}' \
  https://target.com/api/login

# Custom headers
curl -H "Authorization: Bearer TOKEN" https://target.com/api

# Cookie
curl -b "session=abc123" https://target.com

# Follow redirects
curl -L https://target.com

# Ignore SSL
curl -k https://target.com

# Proxy via Burp
curl --proxy http://127.0.0.1:8080 https://target.com

# Custom Host header (bypass CDN)
curl -H "Host: target.com" https://IP_ORIGIN
```

### wget

```bash
# Download file
wget https://target.com/file.pdf

# Download seluruh website (mirror)
wget -r -np -k https://target.com

# Spider (hanya lihat links)
wget -r -np --spider https://target.com 2>&1 | grep "200 OK"
```

### traceroute / tracepath

```bash
# Lihat routing path ke target
traceroute target.com

# UDP mode
traceroute -U target.com

# TCP mode (bypass beberapa firewall)
traceroute -T -p 80 target.com

# tracepath (tanpa root)
tracepath target.com
```

### ss / netstat — Lihat Koneksi Aktif

```bash
# Lihat semua port yang listen
ss -tlnp
netstat -tlnp

# Lihat koneksi TCP aktif
ss -tnp
netstat -tnp

# Lihat koneksi UDP
ss -unp

# Lihat semua koneksi
ss -antup
```

### arp — ARP Table

```bash
# Lihat ARP table
arp -a

# ARP scan di subnet
arp-scan -l
arp-scan 192.168.1.0/24
```

---

## 🎯 Manual Tricks untuk Pentest

### Banner Grabbing Manual

```bash
# HTTP banner grabbing
echo -e "HEAD / HTTP/1.0\r\n\r\n" | nc target.com 80

# HTTPS
echo -e "HEAD / HTTP/1.0\r\n\r\n" | openssl s_client -connect target.com:443 -quiet

# FTP banner
nc target.com 21

# SSH banner
nc target.com 22

# SMTP banner
nc target.com 25
```

### HTTP Manual Request

```bash
# GET request manual
echo -e "GET / HTTP/1.1\r\nHost: target.com\r\n\r\n" | nc target.com 80

# POST manual
echo -e "POST /login HTTP/1.1\r\nHost: target.com\r\nContent-Type: application/x-www-form-urlencoded\r\nContent-Length: 27\r\n\r\nusername=admin&password=123" | nc target.com 80
```

### DNS Manual Tricks

```bash
# Cek semua record sekaligus
for type in A AAAA MX NS TXT SOA CNAME; do
  echo "=== $type ==="
  dig $type target.com +short
done

# Brute force subdomain manual
for sub in www mail ftp dev staging api admin test; do
  ip=$(dig +short A "$sub.target.com")
  [ -n "$ip" ] && echo "$sub.target.com → $ip"
done

# Reverse DNS range IP
for i in $(seq 1 254); do
  host 192.168.1.$i 2>/dev/null | grep "domain name" && echo "192.168.1.$i"
done
```

### Deteksi WAF Manual

```bash
# Kirim request dengan payload XSS sederhana
curl -I "https://target.com/?q=<script>alert(1)</script>"
# Jika 403/406 → kemungkinan ada WAF

# Cek header WAF
curl -I https://target.com | grep -i "x-sucuri\|x-firewall\|x-waf\|x-powered-by-imperva\|cf-ray"

# Payload test WAF
curl "https://target.com/?id=1' OR '1'='1"
```

### OS Fingerprinting Manual (TTL)

```bash
# Ping dan lihat TTL
ping -c 1 target.com | grep ttl

# TTL 64  → Linux/Unix
# TTL 128 → Windows
# TTL 255 → Network device/BSD
```

### Cek Redirect Chain

```bash
curl -Lv https://target.com 2>&1 | grep -E "Location:|HTTP/"
```

### HTTP Methods Testing

```bash
# Cek methods yang diizinkan
curl -X OPTIONS https://target.com -I

# Test method berbahaya
curl -X PUT https://target.com/test.txt -d "test"
curl -X DELETE https://target.com/test.txt
curl -X TRACE https://target.com
```

### Virtual Host Discovery

```bash
# Test subdomain via Host header
for sub in admin dev staging api internal; do
  result=$(curl -sk -o /dev/null -w "%{http_code}" -H "Host: $sub.target.com" https://TARGET_IP)
  echo "$sub.target.com → $result"
done
```

---

## 📚 Referensi Belajar

- **Wireshark Official Docs** — wireshark.org/docs
- **tcpdump Manual** — man tcpdump / tcpdump.org
- **Netcat Guide** — nmap.org/ncat/guide
- **HTTP RFC** — tools.ietf.org/html/rfc7230
- **DNS RFC** — tools.ietf.org/html/rfc1035
- **TryHackMe - Pre-Security Path** — tryhackme.com
- **Professor Messer CompTIA N+** — professormesser.com

---

> ⚠️ Gunakan pengetahuan ini secara **legal dan etis** — hanya pada sistem yang Anda miliki izin untuk dianalisis.
