# ⚔ 🛡 fail2ban

global status of fail2ban
```bash
fail2ban-client status
```
status for a jail
```bash
fail2ban-client status sshd
```

ban
```bash
fail2ban-client set [jail] banip [ip]
```
deban
```bash
fail2ban-client set [jail] unbanip [ip]
```
if recidive active
```bash
fail2ban-client set recidive unbanip [ip]
```

## analyser les log d'authentification
### Afficher tous les ban présents dans le fichier de log d'authentification
```bash
zgrep 'Ban' /var/log/fail2ban.log*
```
### Filter lignes concernant ssh, depuis une date spécifique
```bash
awk '$1" "$2 > "Jun 14 17:00:00" && /sshd/ {print}' /var/log/auth.log
```
### Filter connexions ssh réussies, après une date spécifique
```bash
awk '$1" "$2 > "Jun 14 17:00:00" && /sshd/ && /Accepted/ {print}' /var/log/auth.log
```
### Afficher toutes les IP présents dans le log, après une date spécifique
```bash
awk '$1" "$2 > "Jun 14 17:00:00" && /sshd/ {print $NF}' /var/log/auth.log
```
### Extraction de toutes les IP présents dans le fichier de log, après une date spécifique
```bash
awk '$1" "$2 > "Jun 14 17:00:00" && /sshd/ {for (i=1; i<=NF; i++) if ($i ~ /[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+/) print $i}' /var/log/auth.log
```



## Banned IP (check if still banned)
- 195.25.21.145
- 36.110.228.254
- 14.48.241.157
- 31.184.198.71
- 8.213.27.228
- 185.224.128.121
- 121.146.4.161
- 113.219.213.168
- 51.83.43.46


