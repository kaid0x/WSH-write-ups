# WSH'26 (intraschool) — CTF Challenge Writeups

> **Westminster School Hackathon 2026**
> *Build. Hack. Break. Learn.*

Official solution writeups for all 30+ challenges across 7 categories from WSH'26 — the first intraschool CTF & Hackathon at Westminster School Dubai. Challenges were designed to teach real-world attack techniques in a controlled, beginner-to-advanced environment.

---

## 📂 Categories

| Category | Challenges | Difficulty Range |
|---|---|---|
| [Web Exploitation](#-web-exploitation) | 5 | Beginner → Advanced |
| [Cryptography](#-cryptography) | 6 | Beginner → Advanced |
| [Reverse Engineering](#-reverse-engineering) | 5 | Intermediate → Advanced |
| [Forensics](#-forensics) | 4 | Intermediate → Advanced |
| [Steganography](#-steganography) | 4 | Intermediate → Advanced |
| [OSINT](#-osint) | 5 | Beginner → Advanced |
| [Misc](#-misc) | 5 | Beginner → Advanced |

---

## 🌐 Web Exploitation

### WEB-01 — Open Sesame! *(100 pts)*
**Concept:** Hidden HTML Comments

A QuantBit employee portal login page is given. No credentials are provided.

**Solution:**
- View the page source (`Ctrl+U` / `Cmd+U`)
- Locate the hidden comment in the `<head>` section:
```html
<!-- Flag{y0u_f0und_the_s3cr3t} -->
```

**FLAG:** `Flag{y0u_f0und_the_s3cr3t}`

---

### WEB-02 — Debug Mode *(160 pts)*
**Concept:** Parameter Tampering

A QuantBit knowledge base search page is given. The URL bar shows `debug=false`.

**Solution:**
- Modify the URL parameter from `debug=false` to `debug=true`:
```
quantbit-solutions.io/kb?search=quantum&debug=true
```
- A debug panel appears revealing the flag

**FLAG:** `Flag{d3bug_m0d3_expos3d}`

---

### WEB-03 — Left Behind *(160 pts)*
**Concept:** Exposed Backup Files

A QuantBit staff portal login page is given. No credentials are provided.

**Solution:**
- View page source — spot the developer comment:
```html
<!-- NOTE TO DEV: removed old auth, backup at login.php.bak -->
```
- Navigate to `/login.php.bak` — hardcoded credentials are revealed:
  - Username: `sysadmin`
  - Password: `Qu@ntB1t_2026!`
- Log in with those credentials

**FLAG:** `Flag{b4ckup_f1l3s_4r3_d4ng3r0us}`

---

### WEB-04 — Hidden in Plain Sight *(200 pts)*
**Concept:** Directory Enumeration

A live Flask developer portal is running. A wordlist file is provided.

**Solution:**
```bash
gobuster dir -u http://[TARGET]:8080 -w wordlist.txt
```
- Gobuster discovers `/dev-console` returning HTTP 200
- Navigate to `/dev-console/internal` — flag is displayed on the page

**FLAG:** `Flag{g0bust3r_f0und_m3}`

> **Lesson:** Security through obscurity is not security. Hidden endpoints are discoverable through directory enumeration.

---

### WEB-05 — Intercept Me *(200 pts)*
**Concept:** Broken Access Control / Parameter Tampering

A live Flask secure portal is running. Credentials provided: `staff / staff123`.

**Solution:**
- Open Burp Suite → Proxy → Intercept On
- Log in with the provided credentials
- Burp intercepts the POST request — inspect the body:
```
username=staff&password=staff123&role=user
```
- Modify `role=user` to `role=admin`
- Forward the request — admin dashboard loads with the flag

**FLAG:** `Flag{burp_1nt3rc3pt_pr1v3sc}`

> **Lesson:** Never trust client-supplied values. Role and privilege decisions must always be enforced server-side. This is OWASP's #1 web security risk — Broken Access Control.

---

## 🔐 Cryptography

### CRYPTO-01 — Lost in Translation *(50 pts)*
**Concept:** Base64 Decoding

**Given:** `RmxhZ3tiNHMzXzY0XzFzXzNhc3l9`

**Solution:**
- Open [CyberChef](https://gchq.github.io/CyberChef)
- Apply "From Base64" recipe

**FLAG:** `Flag{b4s3_64_1s_3asy}`

---

### CRYPTO-02 — Rotated *(50 pts)*
**Concept:** ROT13

**Given:** `Synt{e0g13_1f_a0g_f3phe3}`

**Solution:**
- Apply ROT13 decode on [CyberChef](https://gchq.github.io/CyberChef) or [dcode.fr](https://dcode.fr/rot-13-cipher)

**FLAG:** `Flag{r0t13_1s_n0t_s3cur3}`

---

### CRYPTO-03 — XOR'd *(150 pts)*
**Concept:** Single-Byte XOR Brute Force

**Given:** `xord_flag.bin`

**Solution:**
```python
with open('xord_flag.bin', 'rb') as f:
    data = f.read()

for key in range(256):
    decrypted = ''.join(chr(b ^ key) for b in data)
    if 'Flag{' in decrypted:
        print(f"Key: {key} -> {decrypted}")
```
Key is `0x4B`.

**FLAG:** `Flag{s1ngl3_byt3_x0r_crack3d}`

---

### CRYPTO-04 — Weak Hash *(200 pts)*
**Concept:** MD5 Hash Cracking

**Given:** `d8578edf8458ce06fbc5bb76a58c5ca4`

**Solution:**
- Paste the hash into [crackstation.net](https://crackstation.net)
- Instantly found — plaintext is `qwerty`

**FLAG:** `Flag{qwerty}`

---

### CRYPTO-05 — Hashcat Time *(250 pts)*
**Concept:** SHA256 Dictionary Attack

**Given:** `000c285457fc971f862a79b786476c78812c8897063c6fa9c045f579a3b2d63f`

**Solution:**
```bash
echo "000c285457fc971f862a79b786476c78812c8897063c6fa9c045f579a3b2d63f" > hash.txt
hashcat -m 1400 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```
Plaintext: `monkey`

**FLAG:** `Flag{monkey}`

---

### CRYPTO-06 — Broken RSA *(350 pts)*
**Concept:** Weak RSA — Small Modulus Factorization

**Given:** `n = 223453581498057202499787747366276011917`, `e = 65537`, `c = 97550082622071417773854378262669973112`

**Solution:**
```python
from sympy import factorint
n = 223453581498057202499787747366276011917
factors = factorint(n)
# p = 17558207245877303381, q = 12726446291976903257

p, q = 17558207245877303381, 12726446291976903257
e = 65537
phi = (p-1)*(q-1)
d = pow(e, -1, phi)

c = 97550082622071417773854378262669973112
m = pow(c, d, n)
flag = m.to_bytes((m.bit_length()+7)//8, byteorder='big')
print(flag.decode())
```

**FLAG:** `Flag{rsa_pwn3d}`

---

## 🔧 Reverse Engineering

### RE-01 — Strings Attached *(120 pts)*
**Concept:** Binary String Extraction

**Solution:**
```bash
chmod +x strings_attached
strings strings_attached
```
Flag appears in plaintext in the output.

**FLAG:** `Flag{str1ngs_4r3_p0w3rful}`

---

### RE-02 — What's in the Hex? *(120 pts)*
**Concept:** Hex Dump Analysis

**Solution:**
```bash
xxd mystery.bin
```
Flag appears in the ASCII column at offset `0x57`.

**FLAG:** `Flag{h3x_dump_r3v34l5}`

---

### RE-03 — Patch Me *(250 pts)*
**Concept:** Static Binary Analysis with radare2

**Solution:**
```bash
chmod +x patch_me
r2 ./patch_me
> aaa
> afl
> pdf @sym.check_password
```
The string `Qu4ntumB1t!` is loaded before the `strcmp` call. Run the binary and enter it when prompted.

**FLAG:** `Flag{r4d4r3_4n4lys1s}`

---

### RE-04 — Java Secrets *(280 pts)*
**Concept:** Java Decompilation + XOR Decoding

**Solution:**
```bash
jadx FlagChecker.jar
cat FlagChecker/sources/default-package/FlagChecker.java
```
The `validateKey` method contains an XOR-encoded array with key `7`:
```python
keyBytes = [86, 114, 51, 105, 115, 114, 106, 69, 54, 115, 38]
print(''.join(chr(b ^ 7) for b in keyBytes))
# Output: Qu4ntumB1t!
```

**FLAG:** `Flag{j4dx_d3c0mp1l3d}`

---

### RE-05 — Ghidra Time *(350 pts)*
**Concept:** Stripped Binary Reverse Engineering with Ghidra

**Solution:**
- Open binary in Ghidra → Analyze
- Locate the validate function containing a hardcoded XOR byte array
- Reverse the XOR transformation (key: `13 + i`):
```python
expected = [0x4b,0x62,0x6e,0x77,0x6a,0x75,0x7b,0x25,
            0x71,0x64,0x23,0x47,0x6b,0x29,0x6d,0x2f,
            0x6f,0x6d,0x2c,0x44,0x5c]
print(''.join(chr(b ^ ((13+i)%256)) for i,b in enumerate(expected)))
```

**FLAG:** `Flag{gh1dr4_r3v3rs3d}`

---

## 🔍 Forensics

### FORENSICS-01 — Metadata in Plain Sight *(Beginner)*
**Concept:** EXIF Metadata Analysis

**Solution:**
```bash
exiftool vacation_photo.jpg
```
Flag is in the `Comment` field.

**FLAG:** `Flag{metadata_never_lies}`

---

### FORENSICS-02 — Suspicious Log File Analysis *(Intermediate)*
**Concept:** Log Analysis / Brute Force Detection

**Solution:**
```bash
grep "401" access.log | awk '{print $1}' | sort | uniq -c | sort -rn
grep "185.220.101.47" access.log
```
One IP has 25 failed logins. Their successful login is followed by a GET request to `/admin/config_backup.php?token=<FLAG>`.

**FLAG:** `Flag{brute_force_left_a_trail}`

---

### FORENSICS-03 — DNS Tunneling *(Advanced)*
**Concept:** Covert Channel Analysis in Wireshark

**Solution:**
- Open `dns_challenge.pcap` in Wireshark → filter: `dns`
- Spot queries to `*.updates.coolweathersite.com` with hex-encoded subdomains
- Filter: `dns.qry.name contains "coolweathersite"`
- Concatenate hex chunks in sequence order and decode:
```python
hex_str = "4354467b646e735f68696465735f696e5f706c61696e5f73696768747d"
print(bytes.fromhex(hex_str).decode())
```

**FLAG:** `Flag{dns_hides_in_plain_sight}`

---

### FORENSICS-04 — The Poison Packet *(350 pts)*
**Concept:** TTL-based Data Exfiltration

**Solution:**
- Open `poison_packet.pcap` in Wireshark → filter: `ip.src == 172.16.0.99 && icmp`
- Note unusual TTL values from sequential ICMP packets
- Convert TTL values to ASCII:
```python
ttls = [70,108,97,103,123,116,116,108,95,51,120,102,
        49,108,116,114,52,116,49,48,110,125]
print(''.join(chr(t) for t in ttls))
```

**FLAG:** `Flag{ttl_3xf1ltr4t10n}`

---

## 🖼️ Steganography

### STEGO-01 — Audio Spectrogram *(Beginner)*
**Concept:** Spectrogram Analysis

- Upload `solvemewsh.wav` to a spectrogram analyzer
- The flag is visually encoded in the frequency spectrum

**FLAG:** `Flag{stenographycongrats!}`

---

### STEGO-02 — Hidden Text File *(Intermediate)*
**Concept:** Zero-Width Character Steganography

**Solution:**
```python
with open('hollow_archive.txt', 'r', encoding='utf-8') as f:
    content = f.read()

zw_map = {'\u200b': '0', '\u200c': '1'}
bits = ''.join(zw_map[c] for c in content if c in zw_map)
chars = [chr(int(bits[i:i+8], 2)) for i in range(0, len(bits), 8)]
print(''.join(chars))
```

**FLAG:** `Flag{Excecutillize}`

---

### STEGO-03 — Nested Zip Archives *(Advanced)*
**Concept:** Password-Protected Zip Cracking

**Solution:**
```bash
fcrackzip -u -D -p wordlist.txt challenge.zip
# Outer password: dragon
unzip -P dragon challenge.zip
unzip -P nestedflag2024 inner.zip
cat flag.txt
```

**FLAG:** `Flag{double_zip_double_trouble}`

---

### STEGO-04 — Stacked Secrets *(200 pts)*
**Concept:** LSB Steganography

**Solution:**
```bash
zsteg stacked_secrets.png
```
Or manually with Python using PIL to extract least significant bits from pixel data.

**FLAG:** `Flag{lsb_st3g0_unv31l3d}`

---

## 🔎 OSINT

### OSINT-01 — Where Am I? *(75 pts)*
**Concept:** Reverse Image Search

- Run a reverse image search on the provided photo
- Identifies the landmark as being in **Mostar, Bosnia and Herzegovina**

**FLAG:** `Flag{Mostar}`

---

### OSINT-02 — Contact the Registrant *(80 pts)*
**Concept:** WHOIS Lookup

- Look up `Ebf.edu.mx` on [whois.domaintools.com](https://whois.domaintools.com)
- Find the registrant's name in the WHOIS record

**FLAG:** `Flag{HectorMedinaPalma}`

---

### OSINT-03 — Pixels Don't Lie *(150 pts)*
**Concept:** Image Metadata / EXIF Analysis

```bash
exiftool challenge.jpg
```
The `Image Description` field contains the flag. GPS coordinates (25°N, 55°E) also confirm Dubai.

**FLAG:** `Flag{25N_55E_burjkhalifa_2019}`

---

### OSINT-04 — Follow The Hire *(150 pts)*
**Concept:** Phone Keypad Cipher

- A job posting lists a recruitment hotline: `+971-2255387`
- Map digits to phone keypad letters: `2255387` → `CALLUS`

**FLAG:** `Flag{callus}`

---

### OSINT-05 — The Reorder *(180 pts)*
**Concept:** Supply Chain / Manufacturer OSINT

- Key specs from the email: SODIMM, DDR4, 3200MHz, Industrial, Series prefix KT, Pingshan District Shenzhen
- Search leads to **Kimtigo (Shenzhen TIGO Semiconductor)**
- Navigate to their industrial memory products → `KT-B900 SODIMM DDR4 3200MHz`

**FLAG:** `Flag{KT-B900-DDR4-3200}`

---

## 🎲 Misc

### MISC-01 — ASCII Art *(50 pts)*
- Open the text file — read the large blocky ASCII letters
- They spell: `ASCIIMASTER`

**FLAG:** `Flag{asciimaster}`

---

### MISC-02 — Morse Code *(50 pts)*
**Given:** `-- --- .-. ... . -.-. --- -.. . ..--- ----- ..--- -....`

- Decode using [morsecode.world](https://morsecode.world) → `MORSECODE2026`

**FLAG:** `Flag{morsecode2026}`

---

### MISC-03 — Binary Whisper *(150 pts)*
- Paste the binary groups into CyberChef → "From Binary"

**FLAG:** `Flag{b1n4ry_wh1sp3r}`

---

### MISC-04 — The Riddle *(150 pts)*
*"I speak without a mouth and hear without ears. I have no body, but I come alive with wind."*

Answer: **echo**

**FLAG:** `Flag{echo}`

---

### MISC-05 — Reading the Bits *(300 pts)*
**Concept:** Magic Bytes / File Type Identification

```bash
file mystery_track.mp3
# Output: PNG image data, 500 x 200, 8-bit/color RGB
mv mystery_track.mp3 mystery_track.png
```
Open the renamed image — flag is displayed as text.

**FLAG:** `Flag{m4g1c_byt3s_m4tt3r}`

---

### MISC-06 — Git Exposed *(200 pts)*
**Concept:** Git History Forensics

```bash
unzip quantbit-portal.zip && cd quantbit-portal
git log --oneline
git show 41ea212
```
The original `src/auth.py` contains a hardcoded secret before it was "fixed."

**FLAG:** `Flag{g1t_h1st0ry_n3v3r_l13s}`

---

## 🛠️ Tools Used Across Challenges

| Tool | Used For |
|---|---|
| CyberChef | Base64, ROT13, Binary, Hex decoding |
| Burp Suite | HTTP interception, parameter tampering |
| Gobuster | Directory enumeration |
| ExifTool | Image metadata extraction |
| Wireshark | Packet capture analysis |
| Ghidra | Binary reverse engineering |
| radare2 | Disassembly and analysis |
| Hashcat | Password hash cracking |
| zsteg | LSB steganography detection |
| fcrackzip | ZIP password cracking |
| jadx | Java decompilation |
| Scapy | Packet manipulation and analysis |

---

## 📋 Event Details

**WSH'26 — Westminster School Hackathon**
*Build. Hack. Break. Learn.*
Intraschool CTF & Hackathon — Westminster School Dubai, 2026

**Total Challenges:** 30+ across 7 categories

**Infrastructure:** CTFd self-hosted on Google Cloud Platform → [Deployment Repo](https://github.com/kaid0x/CTFd-deployment-on-GCP-with-Docker)

**Web Challenges:** Custom Flask apps → [Challenge Repo](https://github.com/kaid0x/wsh26-web-challenges)

---

## ⚠️ Disclaimer

These writeups are published after the event for educational purposes. All challenge files and vulnerable applications were deployed in an isolated, controlled environment.
