# Cvičení 3: DNS (Domain Name System)

## 1. Úvod

Cílem tohoto cvičení je seznámit se s Domain Name System (DNS) a jeho implementací pomocí serveru BIND 9. Naučíme se konfigurovat DNS zóny, vytvářet různé typy záznamů a ověřovat správnou funkci DNS serveru. Pochopení DNS je klíčové pro správu síťových služeb, jelikož zajišťuje překlad doménových jmen na IP adresy a naopak.

## 2. Použité nástroje a reference

*   **BIND 9 (Berkeley Internet Name Domain):** Nejrozšířenější DNS server software.
    *   ISC Homepage: [www.isc.org/bind/](https://www.isc.org/bind/)
    *   Administrator Reference Manual (ARM): [ISC BIND 9 ARM](https://bind9.readthedocs.io/en/latest/index.html)
*   **BIND Utilities:** Nástroje pro kontrolu konfigurace BIND.
    *   **named-checkconf:** Kontrola syntaxe hlavních konfiguračních souborů BIND.
        *   Reference: [man named-checkconf](https://manpages.debian.org/named-checkconf)
    *   **named-checkzone:** Kontrola syntaxe souborů zón BIND.
        *   Reference: [man named-checkzone](https://manpages.debian.org/named-checkzone)
*   **Nástroje pro dotazování DNS (DNS Query Tools):**
    *   **dig:** Nástroj pro dotazování DNS serverů a diagnostiku DNS problémů.
        *   Reference: [man dig](https://manpages.debian.org/dig)
    *   **nslookup:** Další nástroj pro dotazování DNS.
        *   Reference: [man nslookup](https://manpages.debian.org/nslookup)
    *   **host:** Jednoduchý nástroj pro provádění DNS dotazů.
        *   Reference: [man host](https://manpages.debian.org/host)
*   **Systemd:** Pro správu služby `bind9`.
    *   Reference: [man systemctl](https://www.freedesktop.org/software/systemd/man/systemctl.html)

## 3. Poznámky ke cvičení (How-To)

### Základní pojmy DNS

*   **Doménové jméno (Domain Name):** Člověkem čitelná adresa (např. `example.com`).
*   **IP adresa:** Numerická adresa zařízení v síti (např. `192.168.1.1` nebo `2001:0db8::1`).
*   **Zóna (Zone):** Část jmenného prostoru DNS, za kterou je konkrétní DNS server autoritativní.
*   **Záznam (Resource Record - RR):** Základní stavební blok DNS, který obsahuje informace o doméně.

#### Běžné typy DNS záznamů:

*   **A (Address):** Mapuje doménové jméno na IPv4 adresu.
    *   Příklad: `www.example.com. IN A 192.168.1.100`
*   **AAAA (IPv6 Address):** Mapuje doménové jméno na IPv6 adresu.
    *   Příklad: `www.example.com. IN AAAA 2001:0db8::100`
*   **NS (Name Server):** Definuje autoritativní DNS servery pro doménu nebo subdoménu.
    *   Příklad: `example.com. IN NS ns1.example.com.`
*   **MX (Mail Exchange):** Definuje mailové servery pro doménu a jejich prioritu.
    *   Příklad: `example.com. IN MX 10 mail.example.com.`
*   **CNAME (Canonical Name):** Vytváří alias pro jiné doménové jméno.
    *   Příklad: `ftp.example.com. IN CNAME www.example.com.`
*   **PTR (Pointer):** Používá se pro reverzní DNS (překlad IP adresy na doménové jméno). Nachází se v reverzních zónách (např. `1.168.192.in-addr.arpa`).
    *   Příklad: `100.1.168.192.in-addr.arpa. IN PTR host.example.com.`
*   **SOA (Start of Authority):** Obsahuje administrativní informace o zóně (primární server, email správce, sériové číslo zóny, časy pro refresh, retry, expire a TTL).
    *   Příklad:
        ```
        @ IN SOA ns1.example.com. admin.example.com. (
                                2023010101 ; Serial
                                3600       ; Refresh
                                1800       ; Retry
                                604800     ; Expire
                                86400 )    ; Minimum TTL
        ```
*   **TXT (Text):** Ukládá textové informace, často používané pro SPF, DKIM nebo ověření domény.
    *   Příklad: `example.com. IN TXT "v=spf1 ip4:192.168.1.0/24 -all"`

### Instalace BIND 9

```bash
# Pro Debian/Ubuntu
apt update
apt install bind9 bind9utils bind9-doc
```

### Konfigurace BIND 9

Hlavní konfigurační soubory se obvykle nachází v `/etc/bind/`.
*   `/etc/bind/named.conf`: Hlavní konfigurační soubor, který často importuje další soubory.
*   `/etc/bind/named.conf.options`: Globální nastavení a forwardery.
*   `/etc/bind/named.conf.local`: Definuje lokální zóny (forward i reverse).

**Příklad `named.conf.options` (základní nastavení a forwardery):**

```ini
options {
    directory "/var/cache/bind";
    recursion yes;
    allow-query { any; }; // Povolí dotazy odkudkoli
    forwarders {
        8.8.8.8; // Google Public DNS
        8.8.4.4;
    };
    dnssec-validation auto;
    listen-on { any; };
};
```

**Příklad `named.conf.local` (definice zón):**

```ini
// Forward zóna pro example.com
zone "example.com" {
    type master;
    file "/etc/bind/db.example.com";
    allow-update { none; };
};

// Reverse zóna pro síť 192.168.1.0/24
zone "1.168.192.in-addr.arpa" {
    type master;
    file "/etc/bind/db.192.168.1";
    allow-update { none; };
};
```

**Příklad souboru zóny `/etc/bind/db.example.com` (forward zóna):**

```ini
$TTL 86400
@ IN SOA ns1.example.com. admin.example.com. (
                2023010101 ; Serial
                3600       ; Refresh
                1800       ; Retry
                604800     ; Expire
                86400 )    ; Minimum TTL
@               IN NS       ns1.example.com.
@               IN A        192.168.1.10
ns1             IN A        192.168.1.10
www             IN A        192.168.1.100
mail            IN A        192.168.1.101
ftp             IN CNAME    www
@               IN MX 10    mail
```

**Příklad souboru zóny `/etc/bind/db.192.168.1` (reverse zóna):**

```ini
$TTL 86400
@ IN SOA ns1.example.com. admin.example.com. (
                2023010101 ; Serial
                3600       ; Refresh
                1800       ; Retry
                604800     ; Expire
                86400 )    ; Minimum TTL
@               IN NS       ns1.example.com.
10              IN PTR      example.com.
100             IN PTR      www.example.com.
101             IN PTR      mail.example.com.
```

### Kontrola syntaxe a restart služby

Po každé změně konfiguračních souborů je nutné zkontrolovat jejich syntaxi a restartovat službu BIND.

```bash
# Kontrola syntaxe hlavních konfiguračních souborů
named-checkconf

# Kontrola syntaxe souborů zón
named-checkzone example.com /etc/bind/db.example.com
named-checkzone 1.168.192.in-addr.arpa /etc/bind/db.192.168.1

# Restart služby BIND
systemctl restart bind9
systemctl enable bind9
systemctl status bind9
```

### Testování DNS

```bash
# Dotaz na A záznam
dig www.example.com

# Dotaz na MX záznam
dig example.com MX

# Dotaz na reverzní záznam (PTR)
dig -x 192.168.1.100

# Použití nslookup
nslookup www.example.com
nslookup 192.168.1.100

# Použití host
host www.example.com
host 192.168.1.100
```

## 4. Příklad k procvičení

### Zadání

Nakonfigurujte BIND 9 server pro doménu `mojedomena.local` a reverzní zónu pro síť `10.0.0.0/24`.

1.  **Instalace:** Nainstalujte BIND 9 na váš server.
2.  **Konfigurace `named.conf.options`:**
    *   Povolte rekurzi.
    *   Nastavte forwardery na `1.1.1.1` a `9.9.9.9`.
    *   Povolte dotazy z `any`.
3.  **Konfigurace `named.conf.local`:**
    *   Definujte master zónu pro `mojedomena.local` s názvem souboru `/etc/bind/db.mojedomena.local`.
    *   Definujte master reverzní zónu pro `10.0.0.0/24` (tj. `0.0.10.in-addr.arpa`) s názvem souboru `/etc/bind/db.10.0.0`.
4.  **Vytvoření souboru zóny `/etc/bind/db.mojedomena.local`:**
    *   Nastavte TTL na 3600.
    *   SOA záznam: `ns1.mojedomena.local. root.mojedomena.local.` (s aktuálním sériovým číslem).
    *   NS záznam pro `ns1.mojedomena.local.`.
    *   A záznamy:
        *   `@` na `10.0.0.10`
        *   `ns1` na `10.0.0.10`
        *   `www` na `10.0.0.100`
        *   `mail` na `10.0.0.101`
    *   CNAME záznam: `ftp` jako alias pro `www`.
    *   MX záznam: `mail.mojedomena.local.` s prioritou 10.
5.  **Vytvoření souboru zóny `/etc/bind/db.10.0.0`:**
    *   Nastavte TTL na 3600.
    *   SOA záznam: `ns1.mojedomena.local. root.mojedomena.local.` (s aktuálním sériovým číslem).
    *   NS záznam pro `ns1.mojedomena.local.`.
    *   PTR záznamy:
        *   `10` na `mojedomena.local.`
        *   `100` na `www.mojedomena.local.`
        *   `101` na `mail.mojedomena.local.`
6.  **Ověření:** Zkontrolujte syntaxi a restartujte BIND. Ověřte konfiguraci pomocí `dig`.

### Ověření

```bash
# Kontrola syntaxe BIND konfigurace
named-checkconf
named-checkzone mojedomena.local /etc/bind/db.mojedomena.local
named-checkzone 0.0.10.in-addr.arpa /etc/bind/db.10.0.0

# Kontrola stavu služby
systemctl status bind9

# Ověření A záznamu pro www.mojedomena.local
dig @127.0.0.1 www.mojedomena.local A

# Ověření MX záznamu pro mojedomena.local
dig @127.0.0.1 mojedomena.local MX

# Ověření CNAME záznamu pro ftp.mojedomena.local
dig @127.0.0.1 ftp.mojedomena.local CNAME

# Ověření PTR záznamu pro 10.0.0.100
dig @127.0.0.1 -x 10.0.0.100
```
