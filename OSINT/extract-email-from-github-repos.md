```bash
curl -s "https://api.github.com/repos/avalon649/cheat-sheets/commits" | grep -Eio '([[:alnum:]_.]+@[[:alnum:]_]+?\.[[:alpha:].]{2,6})'
```