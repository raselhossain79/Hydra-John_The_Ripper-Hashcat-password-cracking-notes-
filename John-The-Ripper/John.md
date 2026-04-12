# 🔓 John the Ripper — Password Cracking Tool

## What is John the Ripper?

John the Ripper (JtR) is an open-source password cracking tool. It is designed to detect weak passwords by cracking password hashes using different attack modes.

It works offline — meaning you need the hash first, then crack it locally on your machine. It never interacts with the live target.

```
Hash file → John → Cracked password
```

---

## Installation

Pre-installed on Kali Linux. Verify:

```bash
john --version
```

If not installed:

```bash
sudo apt install john -y
```

---

## How It Works

Passwords are never stored as plaintext — they are stored as **hashes.**

```
Password → Hash function → Hash (stored)
"password123" → MD5 → 482c811da5d5b4bc6d497ffa98491e38
```

John takes a hash and tries to find the original password by:
1. Hashing wordlist entries and comparing
2. Generating combinations and comparing
3. Using rules to mutate words

---

## Hash Identification

Before cracking, identify the hash type:

```bash
john --list=formats | grep -i md5
```

Or use `hash-identifier`:

```bash
hash-identifier
```

Common hash types:

| Hash Type | Example Length | Used In |
|-----------|---------------|---------|
| MD5 | 32 chars | Old web apps |
| SHA1 | 40 chars | Git, old systems |
| SHA256 | 64 chars | Modern apps |
| bcrypt | 60 chars | WordPress, modern apps |
| NTLM | 32 chars | Windows |
| SHA512crypt | Long | Linux `/etc/shadow` |

---

## Attack Modes

### 1. Wordlist Attack (Most Common)

Takes a wordlist and hashes each word — compares with target hash.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

> `rockyou.txt` contains 14 million real passwords leaked from RockYou breach.

---

### 2. Single Crack Mode

Uses username and related info to generate password guesses. Good for simple passwords based on username.

```bash
john --single hash.txt
```

---

### 3. Incremental Mode (Brute Force)

Tries every possible combination of characters. Slow but thorough.

```bash
john --incremental hash.txt
```

---

### 4. Rules Mode

Applies mutation rules to wordlist entries:

```
password → Password → PASSWORD → p@ssword → passw0rd
```

```bash
john --wordlist=rockyou.txt --rules hash.txt
```

---

## Specifying Hash Format

Always specify format explicitly for accuracy:

```bash
john --format=md5crypt --wordlist=rockyou.txt hash.txt
john --format=sha512crypt --wordlist=rockyou.txt hash.txt
john --format=NT --wordlist=rockyou.txt hash.txt        # Windows NTLM
```

---

## Cracking Linux Password (/etc/shadow)

Linux stores passwords in `/etc/shadow`. Need root to read it.

```bash
# Combine passwd and shadow files
unshadow /etc/passwd /etc/shadow > combined.txt

# Crack
john --wordlist=rockyou.txt combined.txt
```

---

## Cracking ZIP File Password

```bash
# Extract hash from ZIP
zip2john protected.zip > zip.hash

# Crack
john --wordlist=rockyou.txt zip.hash
```

## Cracking PDF Password

```bash
pdf2john protected.pdf > pdf.hash
john --wordlist=rockyou.txt pdf.hash
```

## Cracking SSH Key Password

```bash
ssh2john id_rsa > ssh.hash
john --wordlist=rockyou.txt ssh.hash
```

---

## View Cracked Passwords

```bash
john --show hash.txt
```

John saves cracked passwords in `~/.john/john.pot` — so `--show` reads from there.

---

## Resume Interrupted Session

```bash
john --restore
```

---

## Useful Commands Summary

```bash
# List all supported formats
john --list=formats

# Wordlist attack
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt

# Wordlist + rules
john --wordlist=rockyou.txt --rules hash.txt

# Brute force
john --incremental hash.txt

# Specify format
john --format=md5crypt --wordlist=rockyou.txt hash.txt

# Show cracked
john --show hash.txt

# Resume
john --restore
```

---

## Key Concepts

- **Hash** — one-way function output; cannot be reversed directly
- **Salt** — random value added to password before hashing; prevents identical passwords having same hash
- **Wordlist** — file containing common passwords to try
- **Rules** — patterns applied to wordlist to generate variations
- **Pot file** — `~/.john/john.pot` — stores all previously cracked hashes

---

## ⚠️ Legal Warning

Only use John the Ripper on systems you own or have explicit written permission to test. Unauthorized password cracking is illegal.

---

## Practice Labs

- **DVWA** — extract hashes from database, crack offline
- **HackTheBox** — Linux machines with `/etc/shadow` access after privilege escalation
- **TryHackMe** — dedicated password cracking rooms
