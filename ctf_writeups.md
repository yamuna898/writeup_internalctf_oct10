# 1. Sudo Will You Be My.. — 30 Points

**Category:** Linux / Privilege Escalation  
**Target:** `nc 43.204.148.72 43915`  
**Flag Format:** `IIITBh{...}`

## 1. Challenge Overview

The goal was to access a flag file that the current user could not read directly. The challenge demonstrates how an incorrectly configured `sudo` policy can allow privilege escalation.

## 2. Connecting to the Target

Connected to the remote challenge service using Netcat:

```bash
nc 43.204.148.72 43915
```

Listed the files in the current directory:

```bash
ctf@linux:~$ ls
flag.txt
server.py
```

The listing showed `flag.txt` and `server.py`.

## 3. Attempting to Read the Flag

Tried to read the flag directly:

```bash
ctf@linux:~$ cat flag.txt
/bin/cat: /challenge/flag.txt: Permission denied
```

The permission error indicated that the current user did not have sufficient permission to read the file.

## 4. Privilege Escalation with `sudo`

Tried running `cat` with elevated privileges:

```bash
ctf@linux:~$ sudo cat flag.txt
```

The command successfully displayed the flag.

## 5. Result

The flag was retrieved by using `sudo` to read a file that the ordinary user could not access.

## 6. Root Cause and Mitigation

**Likely root cause:** The challenge's `sudoers` configuration allowed the `ctf` user to execute the relevant command with elevated privileges. A `NOPASSWD` rule may have been configured, although the successful command alone does not prove the exact rule.

**Mitigations:**
- Grant `sudo` access only when necessary.
- Restrict permitted commands and arguments.
- Audit `/etc/sudoers` and `/etc/sudoers.d/`.
- Follow the principle of least privilege.

## 7. Key Takeaways

- `cat` reads and prints file contents.
- File permissions determine which users can access a file.
- `sudo` can run an authorized command with elevated privileges.
- Misconfigured privilege rules can expose protected files.

---

# 2. Linuses — 50 Points

**Category:** Linux / Encoding and Text Transformation  
**Challenge:** Linuses

## 1. Challenge Overview

The challenge supplied text that had been transformed through several reversible operations. The objective was to apply the inverse operations in the correct order to recover the original flag.

## 2. Decoding Base64

The first step was to decode the Base64-encoded content:

```bash
base64 -d encoded.txt
```

Base64 represents binary or text data using a restricted set of printable characters. It is an encoding scheme, not encryption, so it does not require a secret key.

## 3. Reversing the Text

The next operation reversed the decoded text:

```bash
rev
```

The `rev` command reverses the characters on each line. This undoes a transformation that had reversed the original text.

## 4. Replacing Characters with `tr`

The text then required character substitutions. The `tr` command translates characters from one set to another.

For example, hyphens can be replaced with underscores, and parentheses can be replaced with curly braces. The exact source and destination character sets must match the challenge's transformations.

```bash
tr '-()' '_{}'
```

*Note:* Depending on the exact input and desired mappings, the character sets may need to be adjusted. When a set begins with `-`, use an option terminator such as `--` where supported, or arrange the sets so the hyphen is not interpreted as an option.

## 5. Decoding ROT13

Finally, ROT13 was reversed using `tr`:

```bash
tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

ROT13 shifts each English letter by 13 positions. Applying it twice restores the original letters, so the same transformation decodes ROT13 text.

## 6. Result

Applying the transformations in the required order recovered the original flag.

**Flag:** Insert the recovered flag here if you have recorded it.

## 7. Key Takeaways

- `base64 -d` decodes Base64 data.
- `rev` reverses characters on each line.
- `tr` performs character-by-character translation.
- ROT13 is a reversible substitution, not secure encryption.
- The order of transformations matters: each operation must undo the corresponding encoding step.

---

# 3. Ping Me! — 50 Points

**Category:** Linux / Command Injection  
**Challenge:** Ping Me!

## 1. Challenge Overview

The challenge accepted an input related to the IP address `8.8.8.8`. The objective was to determine whether the input was passed to a shell in a way that allowed additional commands to run.

## 2. Understanding the Allowed Input

The challenge indicated that `8.8.8.8` was an accepted address. A normal ping command sends network packets to check whether a host is reachable:

```bash
ping 8.8.8.8
```

The `ls` command, by contrast, lists files and directories in the current directory.

## 3. Testing Command Chaining

A semicolon separates commands in many Unix shells. If the application unsafely inserts user input into a shell command, a semicolon may allow a second command to execute.

In the challenge environment, the tested input was:

```bash
8.8.8.8; ls
```

The application processed the ping-related input and the additional command, revealing the directory contents, including `flag.txt`.

## 4. Reading the Flag

After reconnecting to the challenge service, the following input was used:

```bash
8.8.8.8; cat flag.txt
```

The command output revealed the flag.

## 5. Result

The flag was retrieved because the application accepted input that allowed a second shell command to execute.

## 6. Root Cause and Mitigation

**Root cause:** The application likely passed user-controlled input to a shell without safely separating the intended IP address from executable commands.

**Mitigations:**
- Avoid constructing shell commands from raw user input.
- Use a process API with separate arguments instead of invoking a shell.
- Validate input as an IP address using a proper IP-address parser.
- Run the service with minimal permissions.

## 7. Key Takeaways

- `ping` checks network reachability.
- `ls` lists directory contents.
- `cat` prints file contents.
- A semicolon can separate shell commands.
- Unsafe shell input handling can lead to command injection.

---

# 4. Grab Me! — 50 Points

**Category:** Forensics / Archive Analysis  
**Challenge:** Grab Me!

## 1. Challenge Overview

The challenge provided a GitHub link to a ZIP archive. The goal was to download the actual archive, extract its contents, search through the files, and locate the flag.

## 2. Downloading the Archive

The first download attempt used a GitHub web-page URL:

```powershell
Invoke-WebRequest -Uri "https://github.com/ak04M/1st_Internal_CTF/blob/main/Forensics/lost_archive.zip" -OutFile "lost_archive.zip"
```

A GitHub `blob` URL normally serves a webpage around the file rather than the raw file contents. Therefore, the archive was downloaded again using the raw-content URL:

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/ak04M/1st_Internal_CTF/main/Forensics/lost_archive.zip" -OutFile "lost_archive2.zip"
```

## 3. Verifying the Download

Checked the downloaded file's name and size:

```powershell
Get-Item lost_archive2.zip | Select-Object Name, Length
```

Then inspected its first four bytes:

```powershell
[System.IO.File]::ReadAllBytes("lost_archive2.zip")[0..3]
```

The output began with `80`, `75`, `3`, and `4`, corresponding to the ZIP signature bytes `PK\x03\x04`. This was evidence that the file was a ZIP archive.

## 4. Extracting the Archive

Extracted the archive into a directory:

```powershell
Expand-Archive -Path "lost_archive2.zip" -DestinationPath "lost_archive_extracted"
```

Listed the extracted files recursively:

```powershell
Get-ChildItem lost_archive_extracted -Recurse
```

## 5. Searching for the Flag

Searched the extracted text files for the flag prefix:

```powershell
Get-ChildItem -Path "lost_archive_extracted" -Recurse -File |
    Select-String -Pattern "IIITBh\{"
```

The search found the flag inside a deeply nested text file.

## 6. Result

**Flag:** `IIITBh{g0t_my_f1r5t_f14g_1659}`

## 7. Key Takeaways

- GitHub's `blob` URL is intended for viewing a file; raw URLs are suitable for downloading its contents directly.
- File size and magic bytes help verify a downloaded file.
- `Expand-Archive` extracts ZIP files in PowerShell.
- Recursive file searches can locate text hidden deep within nested directories.

---

# 5. WhereShark? — 100 Points

**Category:** Forensics / Packet Analysis  
**Challenge:** WhereShark?

## 1. Challenge Overview

The challenge provided a link to a file named `flag.pcap`. The objective was to inspect the captured network traffic and find the flag in a packet payload.

## 2. Downloading the Capture

Downloaded the file directly from the GitHub raw-content URL:

```powershell
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/ak04M/1st_Internal_CTF/main/Forensics/flag.pcap" -OutFile "flag.pcap"
```

Checked the file's name and size:

```powershell
Get-Item flag.pcap | Select-Object Name, Length
```

The file was reported to be 887 bytes. That size alone does not establish whether a capture is valid, so the content needed further inspection.

## 3. Inspecting the Contents

Used the following command to inspect the file:

```powershell
Get-Content flag.pcap
```

Although a PCAP file is binary and may display garbled characters in a text terminal, some packet payloads contain readable text. The output showed an HTTP exchange, including lines resembling:

```text
GET / HTTP/1.1
Host: example.com
HTTP/1.1 200 OK
```

The flag was visible in plaintext in the captured packet payload.

For more reliable analysis, a packet analyzer such as Wireshark or `tshark` can decode the capture and display protocol fields and packet data.

## 4. Result

**Flag:** `IIITBh{w1r3_5h4rk_bruhh!_2184}`

## 5. Key Takeaways

- PCAP files store captured network packets.
- Network captures may contain readable application-layer data, including HTTP requests and responses.
- A small capture can still contain useful evidence.
- Wireshark and `tshark` are more reliable than viewing binary capture files as plain text.

---

# 6. Squil You! — 50 Points

**Category:** Web / SQL Injection  
**Target:** `http://172.168.168.114:5000`  
**Flag Format:** `IIITBh{...}`

## 1. Challenge Overview

The application exposed an input field that appeared to interact with a database. The challenge was to determine whether user input was inserted into a SQL query unsafely and whether this could bypass the intended access checks.

## 2. Reconnaissance

Opened the challenge application:

```text
http://172.168.168.114:5000
```

Inspected the available form and identified the field that accepted user input.

## 3. Testing SQL Injection

A common SQL injection test payload is:

```sql
' OR '1'='1
```

The payload attempts to close a quoted string and add a condition that is always true. Its effect depends on the original query structure, the database, and how the application handles the input.

For illustration, unsafe query construction might look like:

```sql
SELECT * FROM users
WHERE username = '<input>' AND password = '<password>';
```

If untrusted input is concatenated directly into SQL, the resulting query may have a different meaning from the developer's intended query. However, the payload above does not guarantee that every query becomes true; SQL operator precedence and the exact placement of the input matter.

## 4. Result

The challenge write-up indicates that the vulnerable input was used to bypass the intended application logic and access the flag. The exact flag value was not recorded in the supplied notes.

**Flag:** `IIITBh{...}` — replace this placeholder with the flag obtained during the challenge.

## 5. Root Cause and Mitigation

**Root cause:** The likely vulnerability was unsafe construction of SQL queries using untrusted input.

**Mitigations:**
- Use parameterized queries or prepared statements.
- Use an ORM correctly, without building raw SQL from untrusted strings.
- Give the database account only the permissions it needs.
- Treat input validation and web application firewalls as additional layers, not replacements for parameterization.

## 6. Key Takeaways

- SQL injection changes the meaning of a database query through crafted input.
- A tautology is a condition that is always true, such as `1=1`.
- The effect of a payload depends on the complete query and SQL operator precedence.
- Parameterized queries keep input data separate from SQL instructions.

---

# 7. Excess and Access — 150 Points

**Category:** Web / Cross-Site Scripting (XSS)  
**Challenge:** Excess and Access

## 1. Challenge Overview

The challenge archive contained a binary named `IIITBh-XSS`, which served a vulnerable web application locally. The objective was to exploit an XSS issue in the challenge environment and access a restricted page.

## 2. Setting Up the Application

After extracting the archive, made the binary executable and ran it:

```bash
chmod +x IIITBh-XSS
./IIITBh-XSS
```

This started the local challenge application.

## 3. Understanding the XSS Flow

Cross-Site Scripting (XSS) occurs when an application includes attacker-controlled content in a page in a way that allows it to execute as JavaScript.

The challenge flow involved an admin context visiting or processing attacker-controlled content. The supplied notes describe capturing the admin session cookie and then using that session to access a protected page.

A session cookie may act as a bearer credential: anyone who possesses a valid session token may be able to act as the associated user, depending on the application's controls.

## 4. Accessing the Restricted Page

The challenge was solved by using the session information obtained through the XSS flow to access the protected/admin page. The application then revealed the flag.

## 5. Result

**Flag:** `IIITBh{X55_1s_c001_4487}`

## 6. Root Cause and Mitigation

**Likely root cause:** The application allowed untrusted content to execute as JavaScript in a privileged browsing context. The exact XSS type and cookie configuration would need to be verified from the application implementation.

**Mitigations:**
- Apply context-appropriate output encoding.
- Sanitize user-generated HTML where HTML is intentionally allowed.
- Set session cookies with `HttpOnly`, `Secure`, and appropriate `SameSite` attributes.
- Use a strict Content Security Policy as defense in depth.
- Avoid exposing privileged admin sessions to untrusted content.

`HttpOnly` prevents JavaScript from reading a cookie directly, but it does not by itself prevent XSS from performing actions as the user within the vulnerable page.

## 7. Key Takeaways

- XSS allows injected JavaScript to execute in another user's browser context.
- Session tokens must be protected because they can represent authenticated identity.
- `HttpOnly` limits cookie theft through JavaScript but does not eliminate all XSS impact.
- Output encoding and safe rendering are core XSS defenses.

---

# 8. Python & Ladder — 100 Points

**Category:** Coding  
**Challenge:** Python & Ladder  
**Flag:** `IIITBh{c0d1ng_0n_70p_3386}`

## 1. Challenge Overview

The challenge required a Python solution that printed exactly 550 dashes (`-`) using a loop. The solution had to be saved in `sol.txt` and supplied to the provided executable.

## 2. Inspecting the Executable

Identified the executable type:

```bash
file dash_challenge_linux
```

The output identified it as a 64-bit Linux ELF executable.

## 3. Writing the Python Solution

Created the solution file:

```bash
nano sol.txt
```

Added the following code:

```python
for i in range(550):
    print("-", end="")
```

The code works as follows:

- `range(550)` produces 550 loop iterations.
- Each iteration prints one dash.
- `end=""` prevents `print()` from adding a newline after every dash.

The challenge's submission format also required an `END` marker to indicate the end of the submitted code. That marker is part of the challenge input format, not a Python statement. Include it in the submission file only if required by the executable's instructions.

## 4. Running the Solution

Made the executable runnable and passed the file through standard input:

```bash
chmod +x dash_challenge_linux
./dash_challenge_linux < sol.txt
```

The `<` operator redirects the contents of `sol.txt` to the program's standard input.

## 5. Result

The executable accepted the solution and printed:

```text
[+] Correct! Flag: IIITBh{c0d1ng_0n_70p_3386}
```

## 6. Key Takeaways

- Python loops automate repetitive operations.
- `range(550)` performs 550 iterations, from 0 through 549.
- `print("-", end="")` prints consecutive dashes without line breaks.
- Shell input redirection sends a file's contents to a program's standard input.

---

# 9. Hash and Smash! — 100 Points

**Category:** Cryptography / Hash Cracking  
**Challenge:** Hash and Smash!

## 1. Challenge Overview

The challenge provided an MD5 hash and a custom wordlist. The goal was to find a candidate whose MD5 digest matched the supplied hash.

**Given hash:**

```text
4d76e589bd414beb6ded4697cfe951b2
```

## 2. Choosing a Dictionary Attack

Because a wordlist was provided, a dictionary attack was a sensible first approach. The tool tests each candidate word by hashing it and comparing the result with the target digest.

MD5 is a cryptographic hash function, not encryption. Hashing is designed to be one-way; cracking attempts to find an input that produces the known hash.

## 3. Running Hashcat

Saved the target hash in `hash.txt`, then ran:

```bash
hashcat -m 0 -a 0 hash.txt /home/daddy/Downloads/wordlist.txt
```

Command breakdown:

- `-m 0`: selects MD5.
- `-a 0`: selects a straight dictionary attack.
- `hash.txt`: contains the target hash.
- `wordlist.txt`: contains candidate plaintext values.

The path to the wordlist must be changed if the file is stored elsewhere.

## 4. Result

Hashcat found a matching candidate that represented the challenge flag.

**Flag:** `IIITBh{R41nb0w_MD5_1s}`

## 5. Security Notes and Mitigation

MD5 is unsuitable for password storage because it is fast and has known cryptographic weaknesses. Unsalted hashes are especially vulnerable to dictionary and precomputation attacks.

For password storage, use a password-hashing algorithm designed to be slow and configurable, such as Argon2id, scrypt, or bcrypt, with unique salts.

## 6. Key Takeaways

- A dictionary attack tests likely inputs from a wordlist.
- Hashcat supports multiple hash types and attack modes.
- MD5 does not decrypt; a cracking tool searches for a matching input.
- Strong password storage uses dedicated password-hashing algorithms rather than fast general-purpose hashes.

---

# 10. Pawns — 200 Points

**Category:** Binary Exploitation  
**Challenge:** Pawns  
**Flag:** `IIITBh{574CKED_4ND_H4CK3D_3232}`

## 1. Challenge Overview

The challenge involved a Linux ELF executable with unsafe input handling. The goal was to inspect the binary, identify a possible stack-based buffer overflow, and understand how execution could be redirected to an existing function.

## 2. Initial Reconnaissance

Identified the executable and searched its printable strings:

```bash
file ./vuln1x86
strings ./vuln1x86
```

The binary was identified as a 64-bit Linux ELF executable. The strings included `gets` and a message suggesting that a hidden function was intended to be reached during exploitation.

The flag also appeared in the binary's readable strings, according to the challenge notes. In real analysis, strings output is a useful clue, but it does not replace understanding the program's control flow.

## 3. Inspecting the Disassembly

Used GDB to inspect the executable's code:

```gdb
info files
disassemble 0x1001650, 0x1001788
```

The disassembly showed instructions consistent with reserving a 64-byte stack buffer and passing its address to `gets`:

```asm
sub $0x40, %rsp
lea -0x40(%rbp), %rdi
call gets@plt
```

`gets()` reads input without checking the destination buffer's size. Excessive input can overwrite adjacent stack data, potentially including the saved return address.

## 4. Checking Binary Protections

Checked the executable's protections:

```bash
checksec --file=./vuln1x86
```

The notes reported:

- No stack canary
- No PIE
- NX enabled
- No RELRO

These protections have different purposes. For example, no PIE generally means the executable's main code addresses are not randomized, while NX prevents execution from memory regions marked non-executable. Neither fact alone proves that a particular exploit will work.

## 5. Testing the Overflow Hypothesis

Tested whether a long input caused a crash:

```bash
python3 -c 'print("A"*72)' | ./vuln1x86
```

The program reportedly terminated with a segmentation fault. This supports the buffer-overflow hypothesis, but does not establish the precise offset to the saved return address.

The suspected approach was **ret2win**: redirect execution to an existing function that produces the intended success result. A reliable exploit would require verifying the exact offset, target address, calling convention, and runtime behavior.

## 6. Result

The challenge notes identify the flag as:

```text
IIITBh{574CKED_4ND_H4CK3D_3232}
```

## 7. Key Takeaways

- `file` identifies executable formats and architectures.
- `strings` reveals printable text embedded in binaries.
- GDB can inspect disassembly and runtime behavior.
- `gets()` is unsafe because it does not limit input length.
- A crash is evidence of a possible memory-safety bug, not proof of a complete exploit.
- NX, PIE, stack canaries, and RELRO mitigate different classes of exploitation.
