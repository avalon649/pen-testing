# Enumeration Cheat Sheet

---

## Host Discovery

```bash
# Ping sweep
nmap -sn 192.168.1.0/24
nmap -sn 10.0.0.0/8

# ARP scan (faster on local network)
arp-scan -l
arp-scan 192.168.1.0/24
netdiscover -r 192.168.1.0/24

# fping
fping -a -g 192.168.1.0/24 2>/dev/null
```

---

## Port Scanning

```bash
# Fast full port scan (all 65535 ports)
nmap -p- --min-rate 5000 -T4 <target_ip>

# Service + version + default scripts on found ports
nmap -sC -sV -p <ports> <target_ip>

# UDP scan (top ports)
nmap -sU --top-ports 100 <target_ip>

# OS detection
nmap -O <target_ip>

# Full aggressive scan
nmap -A -p- <target_ip>

# Output to file
nmap -sC -sV -p- -oN scan.txt <target_ip>
nmap -sC -sV -p- -oA scan <target_ip>   # all formats

# Stealth SYN scan
nmap -sS -p- <target_ip>

# Scan multiple targets
nmap -sC -sV -iL targets.txt
```

---

## Service Enumeration

### FTP (21)

```bash
nmap -sC -sV -p 21 <target_ip>

# Anonymous login
ftp <target_ip>
# user: anonymous  pass: anonymous

# Download all files
wget -r ftp://anonymous:anonymous@<target_ip>/
```

### SSH (22)

```bash
nmap -sC -sV -p 22 <target_ip>

# Check supported auth methods
ssh -v user@<target_ip>

# Banner grab
nc -v <target_ip> 22

# Brute force
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://<target_ip>
hydra -L users.txt -P passwords.txt ssh://<target_ip>
```

### SMTP (25 / 587)

```bash
nmap -sC -sV -p 25 <target_ip>

# Banner grab + manual enum
nc -v <target_ip> 25
EHLO test
VRFY root         # check if user exists
EXPN root         # expand mailing list

# User enumeration
smtp-user-enum -M VRFY -U users.txt -t <target_ip>
```

### DNS (53)

```bash
# Basic query
nslookup <domain> <target_ip>
dig @<target_ip> <domain>

# Zone transfer
dig axfr @<target_ip> <domain>
host -l <domain> <target_ip>

# All record types
dig @<target_ip> <domain> ANY

# Reverse lookup
dig -x <ip> @<target_ip>

# Subdomain bruteforce
gobuster dns -d <domain> -w /usr/share/wordlists/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
dnsx -d <domain> -w subdomains.txt
```

### HTTP / HTTPS (80 / 443)

```bash
# Whatweb — tech stack fingerprint
whatweb http://<target_ip>

# Nikto — web vulnerability scanner
nikto -h http://<target_ip>

# Directory busting
gobuster dir -u http://<target_ip> -w /usr/share/wordlists/dirb/common.txt
gobuster dir -u http://<target_ip> -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -x php,html,txt,bak

feroxbuster -u http://<target_ip> -w /usr/share/wordlists/dirb/common.txt

# ffuf
ffuf -u http://<target_ip>/FUZZ -w /usr/share/wordlists/dirb/common.txt
ffuf -u http://<target_ip>/FUZZ -w wordlist.txt -e .php,.html,.txt -fc 404

# Virtual host / subdomain fuzzing
ffuf -u http://<target_ip> -H "Host: FUZZ.<domain>" -w subdomains.txt -fc 404

# curl
curl -I http://<target_ip>                  # headers only
curl -s http://<target_ip>/robots.txt
curl -s http://<target_ip>/sitemap.xml

# SSL cert info
openssl s_client -connect <target_ip>:443
```

### SMB (139 / 445)

```bash
nmap -sC -sV -p 139,445 <target_ip>
nmap --script smb-enum-shares,smb-enum-users -p 445 <target_ip>

# Null session enum
smbclient -L //<target_ip> -N
smbclient //<target_ip>/share -N

# enum4linux
enum4linux -a <target_ip>
enum4linux-ng -A <target_ip>

# CrackMapExec
crackmapexec smb <target_ip>
crackmapexec smb <target_ip> -u '' -p '' --shares
crackmapexec smb <target_ip> -u guest -p '' --shares

# Mount share
mount -t cifs //<target_ip>/share /mnt/smb -o username=user,password=pass
```

### LDAP (389 / 636)

```bash
nmap -sC -sV -p 389 <target_ip>

# Anonymous bind
ldapsearch -x -h <target_ip> -b "dc=domain,dc=local"
ldapsearch -x -h <target_ip> -b "dc=domain,dc=local" "(objectclass=*)"

# Enum users
ldapsearch -x -h <target_ip> -b "dc=domain,dc=local" "(objectclass=user)" sAMAccountName

# With credentials
ldapsearch -x -h <target_ip> -D "user@domain.local" -w password -b "dc=domain,dc=local"
```

### SNMP (161 UDP)

```bash
nmap -sU -p 161 <target_ip>
nmap -sU -p 161 --script snmp-info <target_ip>

# snmpwalk
snmpwalk -c public -v1 <target_ip>
snmpwalk -c public -v2c <target_ip>

# onesixtyone — community string brute force
onesixtyone -c /usr/share/wordlists/SecLists/Discovery/SNMP/snmp.txt <target_ip>

# snmp-check
snmp-check <target_ip> -c public
```

### RDP (3389)

```bash
nmap -sC -sV -p 3389 <target_ip>
nmap -p 3389 --script rdp-enum-encryption <target_ip>

# Check for BlueKeep (CVE-2019-0708)
nmap -p 3389 --script rdp-vuln-ms12-020 <target_ip>

# Connect
xfreerdp /u:user /p:password /v:<target_ip>
rdesktop <target_ip>

# Brute force
hydra -l administrator -P passwords.txt rdp://<target_ip>
```

### MySQL (3306)

```bash
nmap -sC -sV -p 3306 <target_ip>

# Connect
mysql -u root -p -h <target_ip>
mysql -u root --password='' -h <target_ip>

# Enum
show databases;
show tables;
select * from users;

# File read (if FILE privilege)
select load_file('/etc/passwd');
```

### MSSQL (1433)

```bash
nmap -sC -sV -p 1433 <target_ip>

# impacket
mssqlclient.py sa:password@<target_ip>
mssqlclient.py -windows-auth domain/user:password@<target_ip>

# Enable xp_cmdshell
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';
```

### NFS (2049)

```bash
nmap -sC -sV -p 2049 <target_ip>

# Show exports
showmount -e <target_ip>

# Mount
mkdir /mnt/nfs
mount -t nfs <target_ip>:/export /mnt/nfs
ls -la /mnt/nfs
```

### WinRM (5985 / 5986)

```bash
nmap -sC -sV -p 5985,5986 <target_ip>

# evil-winrm
evil-winrm -i <target_ip> -u administrator -p password
evil-winrm -i <target_ip> -u administrator -H <NTLM_HASH>

# CrackMapExec
crackmapexec winrm <target_ip> -u user -p password
```

---

## Web Application Enumeration

```bash
# Robots / sitemap
curl http://<target_ip>/robots.txt
curl http://<target_ip>/sitemap.xml

# Common interesting paths
/.git/
/.env
/admin
/api
/backup
/config
/wp-admin
/wp-config.php
/phpmyadmin
/server-status

# Tech fingerprinting
whatweb http://<target_ip>
wappalyzer (browser extension)

# Parameter fuzzing
ffuf -u "http://<target_ip>/page.php?FUZZ=value" -w params.txt
ffuf -u "http://<target_ip>/page.php?id=FUZZ" -w /usr/share/wordlists/SecLists/Fuzzing/LFI/LFI-Jhaddix.txt

# CMS scanners
wpscan --url http://<target_ip> --enumerate u,p,t    # WordPress
wpscan --url http://<target_ip> -U users.txt -P rockyou.txt   # brute force
droopescan scan drupal -u http://<target_ip>          # Drupal
joomscan -u http://<target_ip>                        # Joomla
```

---

## Active Directory Enumeration

```bash
# BloodHound (remote)
bloodhound-python -u user -p password -d domain.local -dc <DC_IP> -c all

# Enum4linux-ng
enum4linux-ng -A -u user -p password <DC_IP>

# LDAP enum
ldapdomaindump -u "domain\user" -p password <DC_IP>

# CrackMapExec AD enum
crackmapexec smb <DC_IP> -u user -p password --users
crackmapexec smb <DC_IP> -u user -p password --groups
crackmapexec smb <DC_IP> -u user -p password --pass-pol
crackmapexec smb <DC_IP> -u user -p password --shares

# Kerberos user enum (no credentials needed)
kerbrute userenum -d domain.local --dc <DC_IP> users.txt

# GetADUsers
GetADUsers.py -all -dc-ip <DC_IP> domain/user:password
```

---

## Password Attacks

```bash
# Hydra — online brute force
hydra -l admin -P /usr/share/wordlists/rockyou.txt http-post-form "/login:user=^USER^&pass=^PASS^:Invalid" -V
hydra -L users.txt -P passwords.txt ssh://<target_ip>
hydra -l admin -P rockyou.txt ftp://<target_ip>

# Medusa
medusa -h <target_ip> -u admin -P rockyou.txt -M http

# Hashcat — offline cracking
hashcat -m 0 hash.txt rockyou.txt           # MD5
hashcat -m 100 hash.txt rockyou.txt         # SHA1
hashcat -m 1000 hash.txt rockyou.txt        # NTLM
hashcat -m 1800 hash.txt rockyou.txt        # sha512crypt
hashcat -m 13100 hash.txt rockyou.txt       # Kerberoast (TGS)
hashcat -m 18200 hash.txt rockyou.txt       # AS-REP

# John the Ripper
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
john hash.txt --format=NT --wordlist=rockyou.txt

# Hash identification
hash-identifier
hashid <hash>
```

---

## Wordlists

```bash
# Common locations
/usr/share/wordlists/rockyou.txt
/usr/share/wordlists/dirb/common.txt
/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
/usr/share/wordlists/SecLists/

# Generate custom wordlist
cewl http://<target_ip> -m 5 -w custom.txt         # scrape words from site
crunch 8 8 abcdefghijklmnopqrstuvwxyz -o 8char.txt  # generate by length
```

---

## Automated Scanners

```bash
# Nessus (GUI — runs on localhost:8834)
# OpenVAS
gvm-start
# access at https://localhost:9392

# Nuclei — template-based scanner
nuclei -u http://<target_ip>
nuclei -l targets.txt -t /root/nuclei-templates/

# Metasploit auxiliary scanners
use auxiliary/scanner/portscan/tcp
use auxiliary/scanner/smb/smb_ms17_010
use auxiliary/scanner/http/dir_scanner
```

---

## Quick Reference — Common Ports

| Port | Service |
|------|---------|
| 21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 110 | POP3 |
| 135 | RPC |
| 139/445 | SMB |
| 143 | IMAP |
| 389 | LDAP |
| 443 | HTTPS |
| 636 | LDAPS |
| 1433 | MSSQL |
| 1521 | Oracle |
| 2049 | NFS |
| 3306 | MySQL |
| 3389 | RDP |
| 5432 | PostgreSQL |
| 5985 | WinRM HTTP |
| 5986 | WinRM HTTPS |
| 6379 | Redis |
| 8080 | HTTP Alt |
| 27017 | MongoDB |

---

## Resources

- **SecLists** — github.com/danielmiessler/SecLists
- **HackTricks** — book.hacktricks.xyz
- **GTFOBins** — gtfobins.github.io
- **Exploit-DB** — exploit-db.com
- **NVD CVE** — nvd.nist.gov
