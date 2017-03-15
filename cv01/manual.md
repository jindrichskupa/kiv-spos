# Cviceni 1

## Zakladni prikazy

```bash
ps auxf
ls | ls -1 | ls -al
mkdir | mkdir -p
touch | chmod | chown
man | info
cd | cd - | cd ~ | pwd
netstat | netstat -tupln
w | who | id
find
grep | grep -R
cat | sort | mc |cut
nano | vim | for | while | if | read
sed | awk
```

## SSH

### Autentizace klici

```bash
ssh-keygen -t rsa -b 4096 -C "jindra@spos"
ssh-agent | eval `ssh-agent`
ssh-add
ssh-copy-id
cat ~/.ssh/authorized_keys
```
### Authorized keys

```bash
cat ~/.ssh/authorized_keys
command="./run-this",no-port-forwarding,no-x11-forwarding,no-agent-forwarding ssh-dss KEY user@machine
```

### Prihlaseni 

```bash
ssh -i ~/.ssh/id_dsa  -l user machine
ssh -l user machine.local
ssh user@machine.local
ssh user@machine "uname -a"
```

### Nastaveni klienta

```
cat ~/.ssh/config

Host spos
   HostName 147.228.67.42
   User root
   Port 22
```

### Zakladni nastaveni serveru

```
cat /etc/ssh/sshd_config

...
AllowUsers root
PasswordAuthentication no
PermitRootLogin without-password
...

```

## Fail2ban

```bash
apt-get install fail2ban
```

## Iptables
 
```bash
# omezene pristupu na SSH jen z univerzity
iptables-nvL
iptables -A INPUT -s  147.228.0.0/16  -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -p tcp --dport 22  -j DROP

# ulozeni a obnoveni nastaveni
iptables-save > /etc/network/iptables
iptables-restore /etc/network/iptables

# nastaveni po nahozeni interface
/etc/network/interfaces
	post-up /sbin/iptables-restore /etc/network/iptables

```

## TCPWrappers

```bash
# overeni podpory
ldd /usr/sbin/sshd | grep libwrap

# nastaveni povolenych/zakazanych hostu
/etc/hosts.allow
	sshd : 147.228.0.0/255.255.0.0
/etc/hosts.deny

# kontrola nastaveni
tcpdmatch sshd localhost
tcpdchk
```

## Informace o serveru a nastaveni:

```bash
/etc/
/etc/resolv.conf
/etc/hostname
/etc/network/interfaces
/etc/fstab
/etc/hosts
/etc/passwd | /etc/group | /etc/shadow
/etc/debian_version | /etc/redhat-release

hostname
uname
iptables
ifconfig | route | ip
dpkg | aptitude

free
cat /proc/cpuinfo | cat /proc/meminfo
mount
netstat -tupln
crontab -l
top
htop
```

## Sprava sluzeb

```bash
# start | stop | restart | status | ...
service fail2ban restart
/etc/init.d/fail2ban restart
systemctl restart fail2ban.service
```
