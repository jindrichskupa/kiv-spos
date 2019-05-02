# Cviceni 3

## Ziskani informaci z DNS

```bash
# nastroj host
$ host zcu.cz
$ host -t NS zcu.cz
$ host -t MX zcu.cz localhost

# nastroj dig
$ dig zcu.cz
$ dig -t AAAA zcu.cz
$ dig -t SOA zcu.cz @localhost

# dalsi nastroje
$ nslookup
$ whois
```
 
## Nastaveni DNS
 
```bash
/etc/resolv.conf
/etc/hosts
```

## Bind9

### Instalace a spusteni

```bash
apt-get install bind9 dnsutils
service bind9 start|stop|restart
```

### Konfiguracni soubory a logy

```bash
/etc/bind/named.conf*
/etc/bind/db.*

/var/log/syslog
```

### Naslouchani

``` bash
allow-recursion {10.0.0.0/24;};
listen-on {127.0.0.1;}; 
listen-on-v6 {::1;};
```

### Nastaveni zon

```
zone "jindra.spos." in {
    type master;
    file "/etc/bind/db.jindra.spos";
    allow-transfer {147.228.67.0/24;};
};

zone "lubos.spos." in {
    type slave;
    file "/etc/bind/slave/slave.lubos.spos";
    masters {147.228.67.41;};
};
```

### Zonovy soubor

```
$TTL    604800
@   IN  SOA jindra.spos. root.localhost. (

                  2     ; Serial / YYYYMMDDXX
             604800     ; Refresh / seconds
              86400     ; Retry / seconds
            2419200 ; Expire / seconds
             604800 )   ; Negative Cache TTL / explicitni TTL

@         IN      NS                ns.jindra.spos.
@         IN      NS                lubos.spos.
@         IN      A                 147.228.67.42
@         IN      A                 147.228.67.41
@         IN      AAAA              ::1
mail      IN      A                 147.228.67.42
pop3      IN      CNAME             mail
smtp      IN      CNAME             mail.jindra.spos.
imap      IN      CNAME             mail.jindra.spos
@         IN      MX           10   mail
@         IN      MX           20   mail.lubos.spos.
ns        IN      A                 147.228.67.42
txt       IN      TXT               "ahoj svete"

```

### Pohledy

```
acl local {
   192.168.1.0/24;
   localhost;
};
```

```
view "localnetwork" {
 match-clients { local; 192.168.0.0/24; }; 
  recursion yes;
  zone "jindra.spos." {
    type master;
    file  "/etc/bind/local/db.jindra.spos"
  };
};

view "publicnetwork"{
match-clients {"any"; }; 
  recursion no;
  zone "jindra.spos." {
    type master;
    file  "/etc/bind/public/db.jindra.spos"
  };
};

```

### Forwarding

```
; globalni nastaveni
options {
    directory "/var/named";
    version "not available";
    forwarders {10.0.0.1; 10.0.0.2;};
    forward only;
};
; per zone nastaveni
zone "example.com" IN {
    type forward;
    forwarders {10.0.0.1; 10.0.0.2;};
};
```

### Nastaveni

```
options {
        directory "/var/cache/bind";
        recursion yes; 
        allow-recursion { trusted; };
        listen-on { 10.228.67.42;  147.228.67.42; };
        allow-transfer { none; };
        forwarders {
                8.8.8.8;
                8.8.4.4;
        };
...
};
```
