# ⚡ Hashcat — GPU-Accelerated Password Cracking

## What is Hashcat?

Hashcat is the world's fastest password cracking tool. Unlike John the Ripper which uses CPU, Hashcat uses the **GPU (graphics card)** — making it significantly faster for large-scale cracking.

```
Hash → Hashcat (GPU) → Cracked password
```

It is purely offline — needs the hash first, then cracks it locally.

---

## Installation

Pre-installed on Kali Linux. Verify:

```bash
hashcat --version
```

If not installed:

```bash
sudo apt install hashcat -y
```

---

## Why GPU?

| Method | Speed (MD5 example) |
|--------|-------------------|
| CPU (John) | ~100 million hashes/sec |
| GPU (Hashcat) | ~10 billion hashes/sec |

GPU has thousands of cores that can work in parallel — ideal for trying billions of password combinations.

---

## Hash Modes (-m)

Every hash type has a mode number in Hashcat:

| Mode | Hash Type | Used In |
|------|-----------|---------|
| 0 | MD5 | Old web apps |
| 100 | SHA1 | Git, old systems |
| 1400 | SHA256 | Modern apps |
| 1800 | SHA512crypt | Linux shadow |
| 3200 | bcrypt | WordPress, modern apps |
| 1000 | NTLM | Windows |
| 500 | MD5crypt | Linux old shadow |
| 1500 | DES | Old Unix |

Full list:

```bash
hashcat --help | grep -i "Hash modes" -A 200
```

---

## Attack Modes (-a)

| Mode | Type | Description |
|------|------|-------------|
| 0 | Wordlist | Try passwords from a file |
| 1 | Combination | Combine two wordlists |
| 3 | Brute Force (Mask) | Try all combinations using a pattern |
| 6 | Hybrid Wordlist + Mask | Wordlist + appended pattern |
| 7 | Hybrid Mask + Wordlist | Pattern + wordlist |

---

## Basic Syntax

```bash
hashcat -m <hash-mode> -a <attack-mode> <hash-file> <wordlist>
```

---

## Attack Examples

### Wordlist Attack

```bash
hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

> `-m 0` = MD5, `-a 0` = wordlist attack

---

### Wordlist + Rules

Rules mutate wordlist entries:
```
password → Password → P@ssword → passw0rd
```

```bash
hashcat -m 0 -a 0 hash.txt rockyou.txt -r /usr/share/hashcat/rules/best64.rule
```

Common rule files in Kali:

```bash
ls /usr/share/hashcat/rules/
```

| Rule File | Description |
|-----------|-------------|
| `best64.rule` | 64 most effective rules |
| `rockyou-30000.rule` | Large rule set |
| `OneRuleToRuleThemAll.rule` | Community favorite |

---

### Brute Force / Mask Attack

Mask defines the pattern to try:

| Mask | Meaning |
|------|---------|
| `?l` | lowercase letter (a-z) |
| `?u` | uppercase letter (A-Z) |
| `?d` | digit (0-9) |
| `?s` | special character |
| `?a` | all characters |

Example — crack 6 character all-lowercase password:

```bash
hashcat -m 0 -a 3 hash.txt ?l?l?l?l?l?l
```

Example — 8 char password: uppercase + 6 lowercase + digit:

```bash
hashcat -m 0 -a 3 hash.txt ?u?l?l?l?l?l?l?d
```

---

### Combination Attack

Combines two wordlists:

```bash
hashcat -m 0 -a 1 hash.txt wordlist1.txt wordlist2.txt
```

Example output combinations:
```
apple + 123 → apple123
pass + word → password
```

---

## Show Cracked Passwords

```bash
hashcat -m 0 hash.txt --show
```

Hashcat saves cracked passwords in `~/.local/share/hashcat/hashcat.potfile`

---

## Benchmark (Test GPU Speed)

```bash
hashcat -b
```

Shows how many hashes per second your GPU can process for each hash type.

---

## Useful Options

```bash
--force              # ignore warnings (use in VM)
--status             # show live progress
--runtime=60         # stop after 60 seconds
-o output.txt        # save cracked passwords to file
--increment          # try increasing lengths (brute force)
--increment-min=6    # start from length 6
--increment-max=8    # stop at length 8
```

---

## Example: Crack Linux Shadow Hash

```bash
# Extract hash from shadow file
sudo cat /etc/shadow | grep username

# Crack with hashcat
hashcat -m 1800 -a 0 shadow.hash rockyou.txt
```

---

## Useful Commands Summary

```bash
# Wordlist attack (MD5)
hashcat -m 0 -a 0 hash.txt rockyou.txt

# Wordlist + rules
hashcat -m 0 -a 0 hash.txt rockyou.txt -r best64.rule

# Brute force 6-char lowercase
hashcat -m 0 -a 3 hash.txt ?l?l?l?l?l?l

# Show cracked
hashcat -m 0 hash.txt --show

# Benchmark
hashcat -b

# List hash modes
hashcat --help | grep -i md5
```

---

## Key Concepts

- **GPU cracking** — thousands of parallel cores make GPU exponentially faster than CPU
- **Mask** — pattern defining what characters to try in brute force
- **Rules** — transformation patterns applied to wordlist entries
- **Potfile** — stores all previously cracked hashes; avoids re-cracking same hash
- **Salted hash** — harder to crack; same password produces different hash each time

---

## John vs Hashcat

| Feature | John the Ripper | Hashcat |
|---------|----------------|---------|
| Processing | CPU | GPU |
| Speed | Moderate | Very fast |
| Ease of use | Easier | More options |
| Auto hash detect | ✅ Yes | ❌ Manual (-m) |
| Best for | Beginners, variety | Speed, large scale |

---

## ⚠️ Legal Warning

Only use Hashcat on hashes you own or have explicit written permission to crack. Unauthorized password cracking is illegal.

---

## Practice Labs

- **HackTheBox** — crack hashes found during privilege escalation
- **TryHackMe** — dedicated Hashcat rooms
- **CrackStation** — online hash lookup (for learning only)
