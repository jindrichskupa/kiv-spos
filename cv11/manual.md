# Cviceni 11

## PortSentry

```bash
apt-get install portsentry

service portsentry stop|start|restart
```

```
/etc/portsentry/portsentry.conf:
BLOCK_UDP="1",
BLOCK_TCP="1",
```

```
/etc/portsentry/
/var/log/syslog
```
```
nmap -p 1-65535 -T4 -A -v -PE -PS22,25,80 -PA21,23,80 10.228.67.X
```

Kontrola:
```
grep "attackalert" /var/log/syslog
grep -n DENY /etc/hosts.deny
grep -n Blocked /var/lib/portsentry/portsentry.blocked.tcp
grep -n Blocked /var/lib/portsentry/portsentry.history
grep -n Blocked /var/lib/portsentry/portsentry.blocked.udp
netstat -rn | grep "10.228.67.X"
route -n | grep "10.228.67.X"
```

Odblokovat:
* zastavit portsentry
* odstranit zaznam z /etc/hosts.deny
* odstranit zaznamy z portsentry.blocked.tcp, portsentry.history a portsentry.blocked.udp
  * sed -i '/10.228.67.X/d' <jmeno souboru>
* odstrani reject routu:
  * route del -host 10.228.67.X reject
* nastartovat portsentry

## RKhunter

```bash
apt-get install rkhunter

rkhunter --update
rkhunter --propupd
rkhunter --list
rkhunter -c --enable hidden_ports
```

```
/etc/rkhunter.conf

MAIL-ON-WARNING="root@localhost"
MAIL_CMD=mail -s "[rkhunter] Warnings found for ${HOST_NAME}"
SCRIPTWHITELIST="/usr/sbin/adduser"
ALLOWDEVFILE="/dev/.udev/rules.d/root.rules"
ALLOWHIDDENDIR="/dev/.udev"
ALLOWHIDDENFILE="/dev/.blkid.tab"
ALLOW_SSH_ROOT_USER=yes
```

``` bash
rkhunter -C
```

## Cron

```
/etc/crontab
/etc/cron.d
/etc/cron.daily
/etc/cron.hourly
/etc/cron.monthly
/etc/cron.weekly
/var/spool/cron/crontabs/
```

```
crontab -l
crontab -e
crontab -l -u jindra
```

```
15 */4 * * * /usr/bin/rkhunter --cronjob --update --quiet
```

## Rsync

poroz na lomitka na konci :)
```bash
rsync -rav /opt/dir1/ /opt/backup_dir/
rsync -rai --dry-run /etc/ /opt/etc_backup_full/
rsync -a --delete --link-dest=../backup.1 /etc/  backup.0/
```