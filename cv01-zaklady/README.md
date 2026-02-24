# Cvičení 1: Základy správy serveru

## 1. Úvod

Cílem tohoto cvičení je seznámit se se základy správy Linuxového serveru. Zaměříme se na tři klíčové oblasti:

*   **Zabezpečený vzdálený přístup:** Nastavení a používání SSH klíčů pro bezpečné přihlášení.
*   **Základní firewall:** Omezení přístupu k serveru pomocí `iptables` a jeho moderního nástupce `nftables`.
*   **Prevence proti útokům:** Instalace a základní pochopení funkce `Fail2Ban` pro automatické blokování útočníků.

## 2. Použité nástroje a reference

*   **OpenSSH:** Standard pro zabezpečený vzdálený přístup.
    *   Homepage: [www.openssh.com](https://www.openssh.com/)
    *   Dokumentace: [Man pages](https://www.openssh.com/manual.html)
*   **Netfilter / iptables:** Starší, ale stále velmi rozšířený framework pro firewall.
    *   Homepage: [www.netfilter.org](https://www.netfilter.org/)
    *   Dokumentace: [Oficiální dokumentace](https://www.netfilter.org/documentation/index.html)
*   **Netfilter / nftables:** Moderní nástupce `iptables`, poskytující jednodušší syntaxi a lepší výkon.
    *   Homepage: [www.netfilter.org](https://www.netfilter.org/projects/nftables/)
    *   Dokumentace: [Wiki](https://wiki.nftables.org/)
*   **Fail2Ban:** Nástroj pro skenování logů a blokování IP adres, které vykazují známky útoku.
    *   Homepage: [www.fail2ban.org](https://www.fail2ban.org/)
    *   GitHub: [https://github.com/fail2ban/fail2ban](https://github.com/fail2ban/fail2ban)
*   **UFW (Uncomplicated Firewall):** Zjednodušená správa firewallu pro Linux (frontend pro `nftables`/`iptables`).
    *   Homepage: [https://wiki.ubuntu.com/UncomplicatedFirewall](https://wiki.ubuntu.com/UncomplicatedFirewall)
*   **Systemd:** Systém a správce služeb pro operační systémy Linux.
    *   Homepage: [https://systemd.io/](https://systemd.io/)
    *   Dokumentace: [Freedesktop.org](https://www.freedesktop.org/wiki/Software/systemd/)
*   **APT (Advanced Package Tool):** Systém pro správu balíčků v Debianu a Ubuntu.
    *   Homepage: [https://wiki.debian.org/Apt](https://wiki.debian.org/Apt)
    *   Dokumentace: [Manpages](https://manpages.debian.org/apt)
*   **Základní správa uživatelů a skupin:** Příkazy jako `useradd`, `groupadd`, `passwd`, `usermod`, `userdel`, `groupdel`, `getent`.
    *   Reference: [man useradd](https://manpages.debian.org/useradd), [man groupadd](https://manpages.debian.org/groupadd), [man passwd](https://manpages.debian.org/passwd), [man usermod](https://manpages.debian.org/usermod)
*   **journalctl:** Nástroj pro prohlížení a správu logů spravovaných `systemd-journald`.
    *   Reference: [man journalctl](https://www.freedesktop.org/software/systemd/man/journalctl.html)
*   **Síťové diagnostické nástroje:** Příkazy jako `netstat`, `ss`, `ip`.
    *   Reference: [man netstat](https://manpages.debian.org/netstat), [man ss](https://manpages.debian.org/ss), [man ip](https://manpages.debian.org/ip)
*   **Nástroje pro monitorování systému:** Příkazy jako `top`, `htop`, `free`.
    *   Reference: [man top](https://manpages.debian.org/top), [man free](https://manpages.debian.org/free), [htop.dev](https://htop.dev/man_page.html)


## 3. Poznámky ke cvičení (How-To)

### Správa služeb
Většinu systémových služeb lze spravovat pomocí následujících příkazů. Moderní distribuce Linuxu (jako Debian, Ubuntu, CentOS 7+, Fedora) používají **systemd**.

```bash
# System V (starší, už se moc nepoužívá, ale pro znalost)
service <nazev_sluzby> start|stop|restart|status
/etc/init.d/<nazev_sluzby> start|stop|restart|status

# systemd (moderní a preferovaný způsob)
# Základní operace
systemctl start <nazev_sluzby>   # Spustí službu
systemctl stop <nazev_sluzby>    # Zastaví službu
systemctl restart <nazev_sluzby> # Restartuje službu
systemctl status <nazev_sluzby>  # Zobrazí stav služby

# Povolení/zakázání služby při startu systému
systemctl enable <nazev_sluzby>  # Povolí automatické spuštění
systemctl disable <nazev_sluzby> # Zakáže automatické spuštění

```

### SSH (Secure Shell)

#### Autentizace pomocí klíčů
Místo hesla je výrazně bezpečnější používat pro přihlášení pár SSH klíčů (veřejný a soukromý).

```bash
# 1. Vygenerování páru klíčů (pokud ještě nemáte)
# -t rsa - typ klíče (dnes se doporučuje ed25519)
# -b 4096 - délka klíče v bitech
# -C "komentář" - komentář pro identifikaci klíče
ssh-keygen -t ed25519 -C "vas_email@example.com"

# 2. Nahrání veřejného klíče na server (nahraďte user a server.adresa)
# Pokud nemáte ssh-copy-id, můžete to udělat ručně:
# cat ~/.ssh/id_ed25519.pub | ssh user@server.adresa "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
ssh-copy-id user@server.adresa

# Obsah veřejných klíčů autorizovaných pro přihlášení na serveru
cat ~/.ssh/authorized_keys
```

#### Konfigurace SSH klienta (`~/.ssh/config`)
Pro usnadnění práce s více servery a automatizaci nastavení je vhodné použít konfigurační soubor klienta.

**Příklad: `~/.ssh/config`**
```ini
Host myserver
    Hostname server.adresa.cz
    User mujuzivatel
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ForwardAgent yes # Povolí předávání SSH agenta
    # Další užitečné volby:
    # StrictHostKeyChecking no
    # UserKnownHostsFile /dev/null
    # ControlMaster auto
    # ControlPath ~/.ssh/sockets/%r@%h:%p
    # ControlPersist 600
```
Poté se můžete připojit jednoduše `ssh myserver`.

#### Konfigurace SSH serveru (`/etc/ssh/sshd_config`)
Hlavní konfigurační soubor serveru je `/etc/ssh/sshd_config`. Po úpravě je potřeba restartovat službu `sshd`.

**Důležité direktivy pro bezpečnost a hardening:**
```ini
# Zakáže přihlašování pomocí hesla (povolí pouze klíče)
PasswordAuthentication no
ChallengeResponseAuthentication no

# Zakáže přihlášení uživateli root, ale pouze pomocí klíče (starší doporučení)
PermitRootLogin without-password

# Úplně zakáže přihlášení uživatelem root (silně doporučeno!)
# PermitRootLogin no

# Omezí, kteří uživatelé se mohou přihlásit (Whitelist)
# Příklad: Pouze uživatelé 'admin' a 'operator'
AllowUsers admin operator

# Nebo omezí skupiny
# AllowGroups sshusers

# Změna výchozího SSH portu (pro snížení "šumu" od skenovacích botů)
# POZOR: Musíte zajistit, aby firewall tento port povoloval!
Port 2222
```

Po změně `/etc/ssh/sshd_config` vždy restartujte službu:
```bash
systemctl restart sshd
```

### Firewall

#### Iptables (starší metoda)
`iptables` je nástroj pro konfiguraci linuxového firewallu. Pravidla se zpracovávají sekvenčně v řetězcích (chains).

```bash
# Zobrazení aktuálních pravidel v řetězci INPUT
iptables -nvL INPUT

# Povolení SSH (port 22) pouze ze sítě 147.228.0.0/16
iptables -A INPUT -s 147.228.0.0/16 -p tcp --dport 22 -j ACCEPT

# Zahození všech ostatních pokusů o připojení na SSH
iptables -A INPUT -p tcp --dport 22 -j DROP

# Uložení pravidel (aby přetrvala restart)
# Debian/Ubuntu:
iptables-save > /etc/iptables/rules.v4
```
Pro automatické načtení pravidel po startu systému se často používá balíček `iptables-persistent`.

#### nftables (moderní metoda)
`nftables` je nástupce `iptables`. Používá strukturovanější syntaxi s tabulkami, řetězci a pravidly.

```bash
# Zobrazení aktuální sady pravidel
nft list ruleset

# Vytvoření základní struktury (pokud neexistuje)
nft add table inet filter
nft add chain inet filter input { type filter hook input priority 0 \; policy accept \; }

# Povolení SSH (port 22) pouze ze sítě 147.228.0.0/16
nft add rule inet filter input ip saddr 147.228.0.0/16 tcp dport 22 accept

# Zahození všech ostatních pokusů o připojení na SSH (pokud je politika accept)
nft add rule inet filter input tcp dport 22 drop

# Pro zahození veškeré ostatní příchozí komunikace je bezpečnější nastavit politiku řetězce na drop
# a povolovat jen konkrétní služby.
# nft flush chain inet filter input # Smazání stávajících pravidel
# nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
# nft add rule inet filter input iif lo accept # Povolení loopback komunikace
# nft add rule inet filter input ct state established,related accept # Povolení navázaných spojení
# nft add rule inet filter input ip saddr 147.228.0.0/16 tcp dport 22 accept

# Uložení pravidel (aby přetrvala restart)
# Pravidla se ukládají do /etc/nftables.conf. Služba nftables.service je automaticky načte.
nft list ruleset > /etc/nftables.conf
systemctl enable nftables
```



### Fail2Ban
Fail2Ban monitoruje logy a na základě neúspěšných pokusů o přihlášení automaticky vytváří firewallová pravidla pro blokování útočících IP adres. Funguje s `iptables` i `nftables`.

```bash
# Instalace
apt-get install fail2ban
```
Konfigurace se provádí v souborech `.local` v adresáři `/etc/fail2ban/`, aby se předešlo přepsání při aktualizacích. Např. `jail.local`.

**Příklad `jail.local` pro SSH:**
```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5
banaction = iptables-multiport

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
```

**Správa a kontrola Fail2Ban:**
```bash
# Zobrazení statusu všech jailů
fail2ban-client status

# Zobrazení statusu konkrétního jailu (např. sshd)
fail2ban-client status sshd

# Ruční zabanování IP adresy
fail2ban-client set sshd banip 1.2.3.4

# Ruční odbanování IP adresy
fail2ban-client set sshd unbanip 1.2.3.4

# Restart služby Fail2Ban po úpravách konfigurace
systemctl restart fail2ban
```

### Správa balíčků (Debian/Ubuntu)
Pro správu softwarových balíčků v Debianu a Ubuntu se používá systém `APT` (Advanced Package Tool).

```bash
# Aktualizace seznamu dostupných balíčků z repozitářů
apt update

# Aktualizace všech nainstalovaných balíčků na nejnovější verzi
apt upgrade

# Instalace nového balíčku
apt install <nazev_balicku>

# Odstranění balíčku (bez konfiguračních souborů)
apt remove <nazev_balicku>

# Úplné odstranění balíčku (včetně konfiguračních souborů)
apt purge <nazev_balicku>

# Vyčištění lokálního repozitáře stažených balíčku
apt clean

# Odstranění nepotřebných balíčků, které byly nainstalovány jako závislosti
apt autoremove

# Vyhledání balíčku
apt search <klíčové_slovo>

# Zobrazení informací o balíčku
apt show <nazev_balicku>
```

### Správa uživatelů a skupin
Základní příkazy pro správu uživatelských účtů a skupin na Linuxu.

```bash
# Vytvoření nového uživatele (s domovským adresářem, skořápkou a skupinou)
useradd -m -s /bin/bash <jmeno_uzivatele>

# Nastavení/změna hesla pro uživatele
passwd <jmeno_uzivatele>

# Přidání uživatele do existující skupiny
usermod -aG <nazev_skupiny> <jmeno_uzivatele>

# Změna primární skupiny uživatele
usermod -g <nazev_skupiny> <jmeno_uzivatele>

# Změna shellu uživatele
usermod -s /bin/zsh <jmeno_uzivatele>

# Smazání uživatele (včetně domovského adresáře)
userdel -r <jmeno_uzivatele>

# Vytvoření nové skupiny
groupadd <nazev_skupiny>

# Smazání skupiny
groupdel <nazev_skupiny>

# Zobrazení informací o uživateli (skupiny, UID, GID atd.)
id <jmeno_uzivatele>

# Zobrazení členů skupiny
getent group <nazev_skupiny>

# Konfigurace (přidání uživatele do skupiny 'sudo' nebo 'wheel' je časté)
# soubor /etc/sudoers by se měl editovat příkazem 'visudo'
# Příklad řádku v /etc/sudoers pro povolení skupiny 'sudo' bez hesla:
# %sudo ALL=(ALL) NOPASSWD: ALL
```

### Logování
Důležité nástroje pro prohlížení a správu systémových logů.

```bash
# Prohlížení logů pomocí journalctl (pro systemd)
# Zobrazení všech logů
journalctl

# Zobrazení logů od posledního bootu
journalctl -b

# Zobrazení logů pro konkrétní službu (např. sshd)
journalctl -u sshd

# Sledování logů v reálném čase (jako tail -f)
journalctl -f

# Zobrazení logů od určitého času
journalctl --since "2023-01-01 10:00:00"

# Filtrování podle priority (např. error, critical)
journalctl -p err

# Klasické prohlížení textových logů
tail -f /var/log/syslog     # Obecný systémový log
tail -f /var/log/auth.log   # Logy autentizace (SSH, sudo atd.)
tail -f /var/log/kern.log   # Logy jádra
```

## 4. Příklad k procvičení

### Zadání
Následující kroky provedete na vašem testovacím Linuxovém serveru. Cílem je zvýšit bezpečnost a správu přístupu.

1.  **Vytvoření nového uživatele:**
    *   Vytvořte nového uživatele s názvem `admin_spos`.
    *   Nastavte mu silné heslo.
2.  **SSH Autentizace:**
    *   Vygenerujte si nový SSH klíčový pár na svém lokálním stroji (pokud ještě nemáte) s typem `ed25519`.
    *   Nahrajte veřejný klíč uživateli `admin_spos` na server tak, aby se mohl přihlašovat pomocí klíče.
3.  **Zabezpečení SSH serveru:**
    *   Zakázat přihlašování pomocí hesla (`PasswordAuthentication no`).
    *   Úplně zakázat přímé přihlášení uživatele `root` (`PermitRootLogin no`).
    *   Povolit přihlášení pouze uživatelům ze skupiny `sudo` (nebo konkrétně `admin_spos`).
    *   Změnit výchozí SSH port z 22 na 2222.
    *   Po úpravách `sshd_config` restartujte službu `sshd`.
    *   **Otestujte:** Zkuste se přihlásit jako `admin_spos` na nový port pomocí SSH klíče. Zkuste se přihlásit pomocí hesla (mělo by selhat). Zkuste se přihlásit jako `root` (mělo by selhat).
4.  **Konfigurace Firewallu (iptables/nftables):**
    *   Nastavte výchozí politiku firewallu tak, aby zakazovala veškerou příchozí komunikaci.
    *   Povolte příchozí komunikaci na novém SSH portu (2222) pouze z vaší lokální sítě (např. `192.168.1.0/24`).
    *   Povolte příchozí komunikaci pro HTTP (port 80) a HTTPS (port 443) ze všech IP adres.
    *   **Otestujte:** Ověřte status firewallu. Zkuste se připojit přes SSH z povolené/zakázané IP adresy.

### Ověření

Správnost nastavení si můžete ověřit pomocí následujících příkazů spuštěných na vašem serveru, případně z vašeho lokálního stroje pro testování SSH konektivity.

```bash
# 1. Ověření uživatele a sudo práv
id admin_spos

# 2. Ověření SSH klíče (na serveru)
grep "ssh-ed25519" /home/admin_spos/.ssh/authorized_keys

# 3. Ověření konfigurace SSH serveru
grep -E '^(PasswordAuthentication|PermitRootLogin|Port|AllowUsers|AllowGroups)' /etc/ssh/sshd_config

# Test připojení SSH (z lokálního stroje):
# ssh -p 2222 admin_spos@your_server_ip
# (zkuste bez klíče, s klíčem, jako root)

# 4. Ověření konfigurace Firewallu

# Pro UFW:
ufw status verbose

# Pro iptables:
iptables -nvL INPUT

# Pro nftables:
nft list ruleset

# Test přístupu na web (pokud máte nainstalovaný webserver)
# curl -I http://your_server_ip
```

## 5. Doplňující materiály

### 5.1 Užitečné diagnostické příkazy

```bash
# Informace o síťových spojeních, portech a procesech
netstat -tupln
ss -tupln

# Informace o přihlášených uživatelích
who
w

# Informace o systému
uname -a
hostname
cat /etc/debian_version

# Informace o síťovém nastavení
ip addr show
ip route show

# Informace o využití paměti a CPU
free -h
top
htop
```

### 5.2 Správa Služeb pomocí systemd (Rozšířené příkazy)

```bash
# Další užitečné systemctl příkazy
systemctl is-active <nazev_sluzby> # Zjistí, zda je služba aktivní
systemctl is-enabled <nazev_sluzby> # Zjistí, zda je služba povolena při startu
systemctl daemon-reload # Znovu načte konfiguraci systemd po úpravách unit souborů
systemctl list-units --type=service # Seznam všech service jednotek
systemctl list-units --type=target # Seznam všech target jednotek
```

**Vytvoření vlastní systemd služby (jednoduchý příklad):**

Pro spuštění vlastního skriptu jako služby můžete vytvořit `.service` soubor v `/etc/systemd/system/`.

**Příklad: `/etc/systemd/system/mojeaplikace.service`**

```ini
[Unit]
Description=Moje ukázková aplikace
After=network.target

[Service]
ExecStart=/usr/local/bin/moje_aplikace_skript.sh
Restart=always
User=mojeuzivatel
Group=mojeskupina

[Install]
WantedBy=multi-user.target
```

Po vytvoření souboru:

```bash
systemctl daemon-reload           # Načtení nové definice služby
systemctl enable mojeaplikace.service # Povolení služby při startu
systemctl start mojeaplikace.service  # Spuštění služby
systemctl status mojeaplikace.service # Kontrola stavu
```

### 5.3 Zjednodušená správa firewallu (UFW)
UFW zjednodušuje správu firewallu. Funguje jako nadstavba nad `nftables` nebo `iptables`.

```bash
# Povolení UFW
ufw enable

# Zakázání veškeré příchozí komunikace jako výchozí
ufw default deny incoming

# Povolení SSH (port 22)
ufw allow ssh

# Povolení specifického portu
ufw allow 80/tcp

# Zobrazení stavu firewallu
ufw status verbose

# Zakázání pravidla
ufw deny from 192.168.1.1 to any port 22
```
