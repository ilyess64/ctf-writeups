# OverTheWire — Bandit CTF Writeups

![Platform](https://img.shields.io/badge/Platform-OverTheWire-blueviolet)
![Levels](https://img.shields.io/badge/Levels%20Completed-0--14-brightgreen)
![Category](https://img.shields.io/badge/Category-Linux%20%7C%20Networking%20%7C%20Cryptography-blue)

> Personal writeups for the Bandit wargame by OverTheWire.
> Each level introduces a core Linux or security concept used in real CTF competitions.

---

## Table of Contents

| Level | Concept | Status |
|-------|---------|--------|
| [Level 0](#level-0) | Remote Login & Navigation | Done |
| [Level 1](#level-1) | Input Streams vs File Paths | Done |
| [Level 2](#level-2) | Filenames with Spaces | Done |
| [Level 3](#level-3) | Hidden Files | Done |
| [Level 4](#level-4) | Wildcard Matching | Done |
| [Level 5](#level-5) | File Searching & Size Filtering | Done |
| [Level 6](#level-6) | System-Wide Search & Error Hiding | Done |
| [Level 7](#level-7) | Keyword Matching with grep | Done |
| [Level 8](#level-8) | Sorting & Duplicate Detection | Done |
| [Level 9](#level-9) | Binary Text Extraction | Done |
| [Level 10](#level-10) | Base64 Encoding | Done |
| [Level 11](#level-11) | ROT13 Substitution Cipher | Done |
| [Level 12](#level-12) | Nested Compression Layers | Done |
| [Level 13](#level-13) | SSH Private Key Authentication | Done |
| [Level 14](#level-14) | Network Ports & Netcat | Done |
| [Level 15](#level-15) | SSL/TLS Encrypted Connections | Done |
| [Level 16](#level-16) | Port Scanning & SSL Private Key | Done |
| [Level 17](#level-17) | File Diffing | Done |
| [Level 18](#level-18) | Non-Interactive SSH Commands | Done |
| [Level 19](#level-19) | SUID Binaries & Privilege Escalation | Done |
| [Level 20](#level-20) | Netcat Listener & IPC | Done |

---

## Level 0

**Concept:** Remote Login & Basic Navigation

**Objective:** Log into the game server and find the first password.

**Strategy:**
Logged in using the default credentials on port 2220, listed the files in the home directory, and opened the readme file.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
ls
cat readme
```

---

## Level 1

**Concept:** Input Streams vs File Paths

**Objective:** Read the password from a file named `-` (a single dash).

**Strategy:**
Typing a plain dash tells Linux to read from keyboard input (stdin), not a file. Fixed this by adding `./` in front, forcing the terminal to treat it as a regular file path.

```bash
cat ./-
```

> **Key insight:** In Linux, `-` is a special symbol for stdin. Always use `./` when a filename starts with a dash.

---

## Level 2

**Concept:** Filenames with Spaces

**Objective:** Open a file that has spaces in its name.

**Strategy:**
The terminal treats spaces as separators between different commands or arguments. Solved this by wrapping the entire filename in double quotes so Linux reads it as one single argument.

```bash
cat "./spaces in this filename"
```

> **Key insight:** Two ways to handle spaces in filenames — quotes: `"name with spaces"`, or escape each space: `name\ with\ spaces`.

---

## Level 3

**Concept:** Hidden Files & Detailed Listing

**Objective:** Find and open a hidden file inside a folder.

**Strategy:**
Files starting with a dot (`.`) are hidden from standard `ls` output. Switched into the directory and used the `-la` flags to force the terminal to show everything, including hidden files and their full details.

```bash
cd inhere
ls -la
cat ./.hidden
```

> **Key insight:** `-l` shows long format (permissions, size, date), `-a` shows all files including hidden ones. These flags are used constantly in CTF challenges.

---

## Level 4

**Concept:** Wildcard Matching & Content Scanning

**Objective:** Find the only human-readable text file inside a group of binary files.

**Strategy:**
Used the `*` wildcard to open all `file0X` files at once, then scanned the output on screen to identify the one line that looked like a readable password rather than binary garbage.

```bash
cd inhere
cat ./file0*
```

> **Key insight:** The `file` command (`file ./*`) is a cleaner approach — it identifies the type of each file without opening them all.

---

## Level 5

**Concept:** File Searching & Size Filtering

**Objective:** Find a 1033-byte, non-executable, human-readable file buried in nested folders.

**Strategy:**
Used `find` with precise filters: regular files only (`-type f`), exact byte size (`-size 1033c`), readable by current user (`-readable`), and not executable (`! -executable`).

```bash
cd inhere/
find . -type f -size 1033c ! -executable -readable
cat ./maybehere07/.file2
```

> **Key insight:** The `c` suffix in `-size 1033c` means bytes. Other units: `k` = kilobytes, `M` = megabytes, `G` = gigabytes.

---

## Level 6

**Concept:** System-Wide Search & Error Suppression

**Objective:** Find a 33-byte file owned by user `bandit7` and group `bandit6` anywhere on the system.

**Strategy:**
Searched from the root directory `/` with ownership and size filters. Redirected all `Permission denied` errors to `/dev/null` to keep the output clean and readable.

```bash
find / -type f -size 33c -group bandit6 -user bandit7 2>/dev/null
```

> **Key insight:** `2>/dev/null` is one of the most useful CTF tricks. `2>` redirects stderr (error messages), and `/dev/null` is a black hole that discards anything sent to it.

---

## Level 7

**Concept:** Text Searching with grep

**Objective:** Extract the password sitting on the same line as a specific word inside a large file.

**Strategy:**
Used `grep` to scan through the massive `data.txt` file and instantly return only the line containing the keyword `millionth`.

```bash
grep "millionth" data.txt
```

> **Key insight:** `grep` is the most-used text search tool in CTF. Key flags: `-i` (case-insensitive), `-r` (recursive search in folders), `-n` (show line numbers).

---

## Level 8

**Concept:** Sorting & Duplicate Detection

**Objective:** Find the only line of text that appears exactly once in a file full of duplicates.

**Strategy:**
The `uniq` tool only detects duplicates on adjacent lines. First sorted the file to group identical lines together, then piped the result to `uniq -c` to count occurrences and find the line with a count of 1.

```bash
sort data.txt | uniq -c
```

> **Key insight:** `sort | uniq -u` is a cleaner version — `-u` prints only the unique lines directly without the count. The pipe `|` is what makes these one-liners possible.

---

## Level 9

**Concept:** Binary Text Extraction

**Objective:** Find a human-readable password inside a file filled with binary garbage.

**Strategy:**
The file was mostly unreadable machine code but contained a few ASCII strings preceded by `=` signs. Used `strings` to filter out all non-printable characters, then piped to `grep` to find the relevant lines.

```bash
strings data.txt | grep "=="
```

> **Key insight:** `strings` is essential for CTF forensics and binary challenges. It extracts any sequence of printable characters from a file — works on executables, images, and binary files.

---

## Level 10

**Concept:** Base64 Encoding & Decoding

**Objective:** Decode a file that contains a Base64-encoded password.

**Strategy:**
Identified the data as Base64 text by its format (letters, numbers, `+`, `/`, ending with `=` padding). Used the `-d` flag with the `base64` command to instantly decode it back to plaintext.

```bash
base64 -d data.txt
```

> **Key insight:** Base64 is the most common encoding in CTF crypto challenges. Recognizable by its character set and `=` padding at the end. Python equivalent: `import base64; base64.b64decode(data)`.

---

## Level 11

**Concept:** ROT13 Substitution Cipher

**Objective:** Decrypt a password where all letters have been shifted by 13 positions.

**Strategy:**
Recognized the scrambling pattern as ROT13 — a Caesar cipher where each letter is shifted exactly 13 positions forward. Since the alphabet has 26 letters, applying ROT13 twice returns the original text.

```bash
cat data.txt
# Then decoded using an online ROT13 decoder
```

> **Key insight:** ROT13 can also be decoded directly in the terminal with `tr`:
> ```bash
> cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
> ```

---

## Level 12

**Concept:** Nested Compression & File Signatures

**Objective:** Recover a text file buried inside multiple layers of compressed archives.

**Strategy:**
Moved to `/tmp` to have write permissions. Converted the hex dump back into binary using `xxd -r`. Then repeatedly used `file` to detect each layer's compression format, renamed the file with the correct extension, and unpacked it. The decompression chain was: `Gzip → Bzip2 → Gzip → Tar → Tar → Bzip2 → Tar → Gzip → plaintext`.

```bash
mkdir /tmp/something && cp data.txt /tmp/something && cd /tmp/something

# Convert hex dump back to binary
xxd -r data.txt > name

# Repeat this cycle until you reach plaintext:
file name                  # identify current format
mv name name.gz            # rename with correct extension
gzip -d name.gz            # decompress (or bunzip2 / tar -xf)
```

> **Key insight:** Always check file type with `file` before trying to decompress. The extension on a file means nothing — the magic bytes at the start of the file reveal the real format.

---

## Level 13

**Concept:** SSH Private Key Authentication

**Objective:** Log into the next level using a private SSH key instead of a password.

**Strategy:**
The server blocks internal connections to localhost. Copied the private key content, exited back to the local WSL2 terminal, saved it as a `.key` file, and set strict permissions with `chmod 600` (SSH refuses to use a key file that others can read). Then logged in directly using the `-i` flag.

```bash
# On the bandit13 server:
cat sshkey.private    # copy the full output

# Back on local WSL2:
nano bandit14.key     # paste the key, save with Ctrl+X
chmod 600 bandit14.key
ssh -i bandit14.key bandit14@bandit.labs.overthewire.org -p 2220
```

> **Key insight:** `chmod 600` means only the file owner can read/write it. SSH enforces this — if permissions are too open, it refuses the key with "WARNING: UNPROTECTED PRIVATE KEY FILE".

---

## Level 14

**Concept:** Network Ports & Raw Data Transfer with Netcat

**Objective:** Submit the current level's password to a specific port to receive the next one.

**Strategy:**
Read the stored password for `bandit14` from the system password file, then used `nc` (netcat) to open a raw TCP connection to port 30000 on localhost and pipe the password directly to it.

```bash
cat /etc/bandit_pass/bandit14 | nc localhost 30000
```

> **Key insight:** Netcat (`nc`) is the "Swiss army knife" of networking. It can open raw TCP/UDP connections, send data, listen on ports, and transfer files. Used in almost every CTF networking challenge.

---

## Level 15

**Concept:** SSL/TLS Encrypted Connections

**Objective:** Submit the current level's password to port 30001 on localhost using SSL encryption.

**Strategy:**
Regular `nc` sends plaintext — this port requires an encrypted SSL connection. Used `openssl s_client` to open an SSL/TLS tunnel to the port, then typed the password once connected. The server validated it and returned the next password.

```bash
openssl s_client -connect localhost:30001
# Once connected, type the bandit15 password and press Enter
```

> **Key insight:** `openssl s_client` is the command-line equivalent of your browser connecting to an HTTPS site. Any time a CTF challenge says "connect using SSL" or runs on port 443, this is the tool. The `-quiet` flag suppresses the certificate handshake output if you want cleaner results.

---

## Level 16

**Concept:** Port Scanning & SSL Private Key Retrieval

**Objective:** Find which port in the range 31000–32000 speaks SSL and will return a private key when given the correct password.

**Strategy:**
The goal is to find the right port without making unnecessary noise. The two-phase approach is better practice than a single loud scan:

- Phase 1: fast scan to find which ports are even open
- Phase 2: version detection only on those specific ports

```bash
# Phase 1 — find open ports in the range (fast, quiet)
nmap -p 31000-32000 localhost

# Phase 2 — run service/version detection only on the ports that responded
nmap -sV 31046,31518,31691,31790,31960 localhost

# Connect to whichever port shows SSL, submit the bandit16 password
openssl s_client -connect localhost:31790
```

The server returned an RSA private key. Save it, set permissions, and use it to log into level 17:

```bash
nano bandit17.key     # paste the private key
chmod 600 bandit17.key
ssh -i bandit17.key bandit17@bandit.labs.overthewire.org -p 2220
```

> **Key insight:** Running `-sV` (version detection) against all 1000 ports at once is loud and slow — it sends many more packets and is easier to detect. In real engagements, the two-phase approach (open port discovery first, then targeted version scan) is standard practice. The first scan uses SYN packets only; the second digs deeper on a small set of confirmed targets.

---

## Level 17

**Concept:** File Diffing

**Objective:** Find the one line that changed between two password files.

**Strategy:**
Two files exist: `passwords.old` and `passwords.new`. The password for the next level is the only line present in `.new` that is not in `.old`. Used `diff` to compare both files and immediately spot the change.

```bash
diff passwords.old passwords.new
```

The line marked with `>` is the new password.

> **Key insight:** `diff` output uses `<` for lines only in the first file and `>` for lines only in the second. In CTF forensics challenges, diffing two versions of a config file or binary dump is a fast way to find what was modified.

---

## Level 18

**Concept:** Non-Interactive SSH Command Execution

**Objective:** Read a file from a server that immediately logs you out the moment you connect.

**Strategy:**
The `.bashrc` file has been modified to run `exit` on login, so an interactive session closes instantly. The fix is to pass the command directly to SSH as an argument — it executes the command and returns the output without ever starting an interactive shell.

```bash
# First confirm the file exists
ssh bandit18@bandit.labs.overthewire.org -p 2220 "ls -la"

# Then read it directly
ssh bandit18@bandit.labs.overthewire.org -p 2220 "cat readme"
```

> **Key insight:** SSH accepts a command as a final argument and runs it remotely without an interactive session. This bypasses any shell initialization files like `.bashrc` or `.bash_profile`. Useful in CTF and real-world scenarios where a shell is restricted or immediately closed.

---

## Level 19

**Concept:** SUID Binaries & Privilege Escalation

**Objective:** Use a SUID binary to read a file you do not have permission to access directly.

**Strategy:**
A binary called `bandit20-do` is sitting in the home directory. Checking its permissions with `ls -la` shows the SUID bit is set — meaning it runs as its owner (`bandit20`) regardless of who executes it. Used it to read the bandit20 password file directly.

```bash
ls -la ./bandit20-do
# Notice the 's' in the permissions: -rwsr-x---

./bandit20-do cat /etc/bandit_pass/bandit20
```

> **Key insight:** SUID (Set User ID) is one of the most important privilege escalation concepts in Linux. When a binary has the SUID bit set, it executes with the file owner's privileges, not the caller's. In real penetration testing, finding SUID binaries owned by root is a primary privesc vector. The command to find all of them on a system: `find / -perm -4000 2>/dev/null`.

---

## Level 20

**Concept:** Netcat Listener & Inter-Process Communication

**Objective:** Use a SUID binary that connects to a port you control, reads a password you send, and returns the next one if it matches.

**Strategy:**
The binary `suconnect` connects to a port on localhost, reads a password, and if it matches the bandit20 password it sends back the bandit21 password. Two terminals are needed — one to listen and send the password, one to run the binary.

```bash
# Terminal 1 — set up a listener that sends the bandit20 password
echo "bandit20_password_here" | nc -l -p 12345

# Terminal 2 — run suconnect pointing at your listener
./suconnect 12345
```

`suconnect` connects to port 12345, receives the password from your listener, validates it, and sends back the next level's password to Terminal 1.

> **Key insight:** This level introduces a fundamental concept in networking CTFs — you control both sides of a connection. Setting up your own listener with `nc -l -p PORT` is used constantly in reverse shell challenges, where you make a target machine connect back to a port you are listening on.

---

*Writeups by Ilyess · Learning cybersecurity · [CTF Write-ups repo](../)*
