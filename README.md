# ⚔ 🛡 fail2ban

> Configurations files are presents under [`files/etc/fail2ban`](files/etc/fail2ban/) folder

## Install
```sh
apt install fail2ban
```
- Create file `/etc/fail2ban/jail.local`, and put content of [jail.local](files/etc/fail2ban/jail.local)
  
- Create file `/etc/fail2ban/filter.d/proxmox.conf`, and put content of [proxmox.conf](files/etc/fail2ban/filter.d/proxmox.conf)
  
- Restart fail2ban
	```sh
	systemctl restart fail2ban
	```

- Check if iptables rules has been correctly created :
	```sh
	iptables -L -n -v
	```
	```sh
	Chain INPUT (policy DROP 1 packets, 52 bytes)
	pkts bytes target     prot opt in     out     source               destination
	92820 5199K f2b-sshd   tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            multiport dports 22

	Chain f2b-sshd (1 references)
	pkts bytes target     prot opt in     out     source               destination
	90937 5058K RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0
	```

## Configuration
There are multiple configuration files
```sh
ls -l /etc/fail2ban/
--
drwxr-xr-x 2 root root    65 Jun  4  2020 action.d
-rw-r--r-- 1 root root  2817 Jan 11  2020 fail2ban.conf
drwxr-xr-x 2 root root     2 Mar  2  2020 fail2ban.d
drwxr-xr-x 3 root root    90 Jun  4  2020 filter.d
-rw-r--r-- 1 root root 25740 Jan 11  2020 jail.conf
drwxr-xr-x 2 root root     4 Jun  4  2020 jail.d
-rw-r--r-- 1 root root   645 Jan 11  2020 paths-arch.conf
-rw-r--r-- 1 root root  2827 Jan 11  2020 paths-common.conf
-rw-r--r-- 1 root root   573 Jan 11  2020 paths-debian.conf
-rw-r--r-- 1 root root   738 Jan 11  2020 paths-opensuse.conf
```

| Dossiers/fichiers | Description |
| - | - |
| action.d | Les actions à effectuer lorsqu’un filtre est positif| 
| fail2ban.conf | Les paramètres généraux de fail2ban| 
| fail2ban.d | L’utilisateur peut créer un fichier de configuration pour personnaliser les réglages de fail2ban| 
| filter.d | Les fichiers de configuration des filtres
jail.conf | Le fichier de configuration des prisons| 
| jail.d | C’est là où place nos fichiers de configuration pour activer les prisons configurés dans jail.conf ou ajoutés par vous même| 

Les services à protéger et sécuriser se définissant jail.d.
Ce dernier utilisent les filtres et les actions à effectuer.
Il ne faut en aucun cas modifier jail.conf car vous risquez de perdre son contenu à la prochaine mise à jour de la distribution.
Ajoutez à chaque fois un fichier de configuration dans jail.d


### **Default configuration**
Le paramétrage par défaut est visible dans `/etc/fail2ban/jail.conf`. Pour surcharger cette configuration, créer un fichier `/etc/fail2ban/jail.d/custom.conf` :
```sh
[DEFAULT]
ignoreip = 127.0.0.1 124.32.5.48
findtime = 10m
bantime = 24h
maxretry = 3
```
- port: Define the service name or service port.
- logpath: Define the name of the log file fail2ban checks for.
- bantime: Define the number of seconds a host will be blocked by fail2ban.
- maxretry: Define the maximum number of failed login attempts a host is allowed before it is banned.
- ignoreip: Define the IP addresses that fail2ban will ignore.

### **Jail configuration**
Create a jail config file in `/etc/fail2ban/jail.d`, for example `ssh` :
```sh
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 120
ignoreip = whitelist-IP
```
- port: Define the service name or service port.
- logpath: Define the name of the log file fail2ban checks for.
- bantime: Define the number of seconds a host will be blocked by fail2ban.
- maxretry: Define the maximum number of failed login attempts a host is allowed before it is banned.
- ignoreip: Define the IP addresses that fail2ban will ignore.

### **basic configuration of fail2ban** (content of [jail.local](files/etc/fail2ban/jail.local))
find explanations in [jail.conf](files/etc/fail2ban/jail.conf)
```sh
[DEFAULT]
bantime.increment = true
bantime.rndtime = 10
bantime.factor = 1
bantime.formula = ban.Time * (1<<(ban.Count if ban.Count<20 else 20)) * banFactor
bantime.multipliers = 1 2 4 8 16 32 64

ignoreip = 127.0.0.1/8 ::1
bantime  = 10m
findtime  = 10m
maxretry = 5
maxmatches = %(maxretry)s

enabled = false
mode = normal

#
# ACTIONS
#
destemail = root@localhost
sender = root@<fq-hostname>
mta = sendmail

#
# JAILS
#

# Jail for more extended banning of persistent abusers
[recidive]
logpath  = /var/log/fail2ban.log
banaction = %(banaction_allports)s
bantime  = 1w
findtime = 1d

# SSH servers
[sshd]
#mode   = normal
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s


# Mail servers
[sendmail-auth]
port    = submission,465,smtp
logpath = %(syslog_mail)s
backend = %(syslog_backend)s

[sendmail-reject]
#mode    = normal
port     = smtp,465,submission
logpath  = %(syslog_mail)s
backend  = %(syslog_backend)s


# Miscellaneous
[gitlab]
port    = http,https
logpath = /var/log/gitlab/gitlab-rails/application.log

[traefik-auth]
# to use 'traefik-auth' filter you have to configure your Traefik instance,
# see `filter.d/traefik-auth.conf` for details and service example.
port    = http,https
logpath = /var/log/traefik/access.log

[phpmyadmin-syslog]
port    = http,https
logpath = %(syslog_authpriv)s
backend = %(syslog_backend)s
```


## Manage fail2ban
Check if fail2ban works :
```sh
systemctl status fail2ban
```
Reload fail2ban configuration
```sh
systemctl reload fail2ban
```
Check fail2ban logs
```sh
tail -f /var/log/fail2ban.log
```

### **fail2ban-client**
Check actives jails :
```sh
fail2ban-client status
```
Check if SSH jail is up :
```sh
fail2ban-client status sshd
```
```sh
Status for the jail: sshd
|- Filter
|  |- Currently failed: 8
|  |- Total failed:     2134
|  `- File list:        /var/log/auth.log
`- Actions
   |- Currently banned: 0
   |- Total banned:     223
   `- Banned IP list:
```

## Usefull commands
global status of fail2ban
```bash
fail2ban-client status
```
status for a jail
```bash
fail2ban-client status sshd
```

In `etc/fail2ban/jail.conf`, add IPs to white list :
```sh
ignoreip = 127.0.0.1/8 10.128.31.48
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


