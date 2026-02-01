# Cviceni 1 - vystup

## Zadani

1. vygenerovani klicu + pridani na server
2. zakaz prihlasovani heslem na ssh
3. povoleni SSH pouze z univerzitnich adres
4. zakazani SSH z dalsich adres

## Kontrola

*bude premisteno do souboru check.sh*

```bash
# pridani klicu
wc -l .ssh/authorized_keys | awk '{if ($1 > 2) print "SSH KEYS OK"; else print "SSH KEYS ERR"}'

# zakaz prihlasovani heslem
grep '^PasswordAuthentication no' /etc/ssh/sshd_config &>/dev/null && echo SSHD AUTH OK || echo SSHD AUTH ERR

# povoleni SSH z university
iptables -nvL | grep 147.228 | grep dpt:22 |grep ACCEPT &>/dev/null && echo IPTABLES UNI ALLLOW OK || echo IPTABLES UNI ALLLOW ERR

# zakaz SSH z ostatnich
iptables -nvL | grep -v 147.228 | grep dpt:22 | grep DROP &>/dev/null && echo IPTABLES ALL DROP OK || echo IPTABLES ALL DROP ERR
```
