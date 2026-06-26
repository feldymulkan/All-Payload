# Dokumentasi FFuf - Web Fuzzing Tools

**FFuf** adalah tools web fuzzing modern yang digunakan untuk:
- Mencari direktori & file tersembunyi
- Enumerasi subdomain
- Brute-force parameter
- Deteksi virtual host
- Automation reconnaissance

---

## 1. Fuzzing Direktori & File

### Mencari Direktori Aktif (Directory Discovery)

```bash
ffuf -u http://10.10.10.10/FUZZ -w common.txt
```

**Fungsi:** Mengganti `FUZZ` dengan setiap kata dalam wordlist untuk menemukan direktori aktif (misal: `/admin`, `/backup`, `/api`)

---

### Mencari Berkas dengan Ekstensi Spesifik

```bash
ffuf -u http://10.10.10.10/FUZZ -w common.txt -e .php,.txt,.html
```

**Parameter `-e`:** Kombinasikan setiap kata dengan ekstensi yang didefinisikan
- Contoh: `config` → `config.php`, `config.txt`, `config.html`

---

## 2. Manajemen Output & Penyaringan (Filtering)

### Menampilkan Kode Status HTTP Tertentu (Match Code)

```bash
ffuf -u http://10.10.10.10/FUZZ -w common.txt -mc 200,301,302
```

**Parameter `-mc`:** Hanya tampilkan respon dengan status code 200 (OK), 301 (redirect), atau 302 (redirect sementara)

**Kode Status HTTP Umum:**
- `200` = OK (found)
- `301/302` = Redirect
- `403` = Forbidden
- `404` = Not Found

---

### Menyembunyikan Ukuran Konten Tertentu (Filter Size)

```bash
ffuf -u http://10.10.10.10/FUZZ -w common.txt -fs 4242
```

**Parameter `-fs`:** Sembunyikan semua respon dengan ukuran tepat 4242 bytes (biasanya halaman error default)

**Contoh Real Case:**
```bash
ffuf -u http://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200,301,302 -fs 1512
```

---

### Kombinasi Filter (Match & Hide)

```bash
ffuf -u http://10.10.10.10/FUZZ -w common.txt -mc 200,301,302 -fs 4242 -fw 0
```

**Parameter `-fw`:** Filter respon berdasarkan jumlah words/kata dalam response body

---

## 3. Enumerasi Subdomain & Virtual Host (Vhost)

### Fuzzing Subdomain (DNS Resolution)

```bash
ffuf -u http://FUZZ.target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

**Fungsi:** Cari subdomain aktif dengan mengganti `FUZZ` di bagian subdomain

**Wordlist Tersedia di Kali:**
```bash
ls /usr/share/seclists/Discovery/DNS/
```

---

### Fuzzing Virtual Host (Vhost via HTTP Header)

```bash
ffuf -u http://target.com -H "Host: FUZZ.target.com" -w subdomains.txt -fs 1512
```

**Fungsi:** Menguji vhost tanpa mengubah DNS publik (berguna saat DNS belum propagate)

**Contoh Praktis:**
```bash
ffuf -u http://10.10.10.10 -H "Host: FUZZ.challenge.local" -w subdomains.txt -mc 200
```

---

## 4. Pengujian Parameter (Parameter Fuzzing)

### Menemukan Parameter GET Tersembunyi

```bash
ffuf -u http://10.10.10.10/index.php?FUZZ=test -w common.txt
```

**Fungsi:** Cari nama parameter query string yang valid (misal: `?id=`, `?file=`, `?search=`)

---

### Fuzzing Nilai Parameter (untuk LFI/SQLi)

```bash
ffuf -u "http://10.10.10.10/view.php?page=FUZZ" -w /usr/share/seclists/Fuzzing/LFI/linux-lfi-wordlist.txt
```

**Use Case:** 
- Local File Inclusion (LFI)
- SQL Injection (SQLi) enumeration
- Path traversal

---

### Brute-Force Login dengan Data POST

```bash
ffuf -u http://10.10.10.10/login.php -X POST -d "username=admin&password=FUZZ" -w passwords.txt -mc 302
```

**Parameter Penjelasan:**
- `-X POST` = Ubah method HTTP ke POST
- `-d "..."` = Data body yang dikirim
- `-mc 302` = Filter redirect (indikasi login berhasil)

---

### Fuzzing Multiple Parameters

```bash
ffuf -u "http://10.10.10.10/search.php?user=FUZZ&role=FUZZ" -w users.txt -w roles.txt
```

---

## 5. Optimasi Kecepatan & Stealth

### Mengatur Jumlah Thread (Kecepatan)

```bash
ffuf -u http://10.10.10.10/FUZZ -w common.txt -t 100
```

**Parameter `-t`:** 
- Default = 40 threads
- `100` = sangat cepat (cocok untuk CTF)
- `200+` = sangat aggressive (risk blocked by WAF)

---

### Menambah Delay Antar Request (Stealth)

```bash
ffuf -u http://10.10.10.10/FUZZ -w common.txt -p 0.5
```

**Parameter `-p`:** Delay 0.5 detik antar request (untuk hindari rate limiting)

---

### Menggunakan Proxy (Burp Suite Integration)

```bash
ffuf -u http://10.10.10.10/FUZZ -w common.txt -x http://127.0.0.1:8080
```

**Fungsi:** Redirect semua traffic melalui Burp Suite untuk logging & analisis mendalam

---

## 6. Rekursi (Recursive Fuzzing)

### Fuzzing Direktori Secara Rekursif

```bash
ffuf -u http://10.10.10.10/FUZZ -w common.txt -recursion -recursion-depth 2
```

**Fungsi:** Otomatis scan subdirektori yang ditemukan (misal: `/admin` → `/admin/panel` → `/admin/panel/users`)

**Contoh:**
```bash
ffuf -u http://10.10.10.10/FUZZ -w common.txt -recursion -recursion-depth 3 -e .php
```

---

## 7. Referensi Cepat (Cheat Sheet)

| Parameter | Fungsi | Contoh |
|-----------|--------|--------|
| `-u` | URL target dengan FUZZ | `-u http://target.com/FUZZ` |
| `-w` | Wordlist path | `-w common.txt` |
| `-e` | Ekstensi file | `-e .php,.html,.txt` |
| `-mc` | Match status code | `-mc 200,301,302` |
| `-fs` | Filter size (bytes) | `-fs 4242` |
| `-fw` | Filter words count | `-fw 0` |
| `-H` | Custom header | `-H "User-Agent: Custom"` |
| `-X` | HTTP method | `-X POST` |
| `-d` | POST data | `-d "param=value"` |
| `-t` | Thread count | `-t 100` |
| `-p` | Delay antar request | `-p 0.5` |
| `-x` | Proxy | `-x http://127.0.0.1:8080` |
| `-recursion` | Rekursi scan | `-recursion -recursion-depth 3` |

---

## 8. Skenario Real-World CTF

### Scenario 1: Web App Directory Discovery + Extension Fuzzing

```bash
ffuf -u http://challenge.ctf/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -e .php,.html,.txt,.bak \
  -mc 200,301,302 \
  -fs 4242 \
  -t 100
```

---

### Scenario 2: Subdomain Enumeration

```bash
ffuf -u http://FUZZ.challenge.ctf \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -mc 200,301,302 \
  -t 150
```

---

### Scenario 3: Virtual Host Discovery (saat DNS belum setup)

```bash
ffuf -u http://10.10.10.10 \
  -H "Host: FUZZ.challenge.local" \
  -w /usr/share/seclists/Discovery/DNS/subdomains-common.txt \
  -mc 200 \
  -fs 1512
```

---

### Scenario 4: Parameter Fuzzing untuk LFI/Path Traversal

```bash
ffuf -u "http://challenge.ctf/view.php?FUZZ=/etc/passwd" \
  -w /usr/share/seclists/Discovery/Web-Content/web-methods.txt \
  -mc 200
```

---

### Scenario 5: Login Brute-Force

```bash
ffuf -u http://challenge.ctf/login \
  -X POST \
  -d "username=admin&password=FUZZ" \
  -w /usr/share/seclists/Passwords/Leaked-Databases/rockyou-100.txt \
  -mc 302 \
  -t 50
```

---

## 9. Tips & Tricks

### ✅ Best Practices

1. **Mulai dengan filter `-mc 200,301,302`** untuk fokus pada hasil yang meaningful
2. **Gunakan `-fs` untuk filter false positive** (halaman 404 custom dengan size konsisten)
3. **Increase threads (-t 100+)** saat CTF untuk speed, kurangi saat target punya WAF
4. **Combine `-recursion`** untuk deep directory discovery
5. **Selalu gunakan `/usr/share/seclists`** wordlist yang tersedia di Kali

### ⚠️ Hindari

- Jangan fuzzing tanpa filter `-mc` (hasil berantakan)
- Jangan `-t 500+` di network shared (risk blocked)
- Jangan lupa `-recursion-depth limit` (infinite loops)

---

## 10. Wordlist Paths di Kali Linux

```bash
# Directory discovery
/usr/share/seclists/Discovery/Web-Content/common.txt
/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt

# Subdomain enumeration
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Password lists
/usr/share/seclists/Passwords/Leaked-Databases/rockyou-10.txt

# LFI/Path traversal
/usr/share/seclists/Fuzzing/LFI/linux-lfi-wordlist.txt
```

---

**Last Updated:** 2026
**For:** CTF Challenges & Penetration Testing
