# Bezpečnost serverů

## 1. Úvod
**Cíl:** Seznámení se základními nástroji a postupy pro zvyšování bezpečnosti Linuxových serverů. Cvičení zahrnuje konfiguraci detekce portscanů (PortSentry), detekci rootkitů (RKhunter), automatizaci bezpečnostních úloh pomocí Cronu a bezpečné zálohování dat pomocí Rsync.

## 2. Použité nástroje a reference
*   **PortSentry:** Nástroj pro detekci a blokování portscanů.
    *   Homepage (starší projekt): [http://www.psionic.com/products/portsentry.html](http://www.psionic.com/products/portsentry.html)
    *   Dokumentace (např. z Debian man pages): [man portsentry](https://manpages.debian.org/testing/portsentry/portsentry.8.en.html)
*   **RKhunter (Rootkit Hunter):** Nástroj pro detekci rootkitů, backdooorů a lokálních exploitů.
    *   Homepage: [http://rkhunter.sourceforge.net/](http://rkhunter.sourceforge.net/)
    *   Dokumentace: [RKhunter Documentation](http://rkhunter.sourceforge.net/documentation.html)
*   **Cron:** Plánovač úloh pro spouštění příkazů v pravidelných intervalech.
    *   Wikipedia: [https://cs.wikipedia.org/wiki/Cron](https://cs.wikipedia.org/wiki/Cron)
    *   Crontab guru (pomocník pro syntaxi): [https://crontab.guru/](https://crontab.guru/)
*   **Rsync:** Nástroj pro rychlou a efektivní synchronizaci souborů a adresářů.
    *   Homepage: [https://rsync.samba.org/](https://rsync.samba.org/)
    *   Dokumentace: [Rsync Documentation](https://rsync.samba.org/documentation.html)
*   **Systemd:** Pro správu služeb (např. PortSentry).
    *   Reference: [man systemctl](https://www.freedesktop.org/software/systemd/man/systemctl.html)
*   **nmap:** Nástroj pro průzkum sítě a bezpečnostní audit. (Pro testování PortSentry).
    *   Homepage: [https://nmap.org/](https://nmap.org/)
    *   Dokumentace: [nmap Man Page](https://nmap.org/man/nmap.html)
*   **grep, sed, netstat, route:** Základní Linuxové utility pro zpracování textu, síťovou diagnostiku a správu rout.
    *   Reference: [man grep](https://manpages.debian.org/grep), [man sed](https://manpages.debian.org/sed), [man netstat](https://manpages.debian.org/netstat), [man route](https://manpages.debian.org/route)
*   **pg_dump:** (Pokud se vztahuje na zálohu PostgreSQL) Nástroj pro zálohování databází PostgreSQL.
    *   Reference: [man pg_dump](https://www.postgresql.org/docs/current/app-pgdump.html)

## 3. Poznámky ke cvičení (How-To)

### PortSentry (Detekce portscanů)

#### Instalace
```bash
apt-get install portsentry
```

#### Správa služby
```bash
service portsentry stop
service portsentry start
service portsentry restart
```

#### Konfigurace (`/etc/portsentry/portsentry.conf`)
*   Zapnutí blokování TCP a UDP skenů:
```ini
# /etc/portsentry/portsentry.conf
BLOCK_UDP="1"
BLOCK_TCP="1"
```

#### Klíčové soubory a logy
*   Konfigurační adresář: `/etc/portsentry/`
*   Logy: `/var/log/syslog` (hledejte "attackalert")
*   Seznam zablokovaných IP adres:
    *   `/var/lib/portsentry/portsentry.blocked.tcp`
    *   `/var/lib/portsentry/portsentry.blocked.udp`
*   Historie skenů: `/var/lib/portsentry/portsentry.history`

#### Testování (příklad s nmap)
```bash
nmap -p 1-65535 -T4 -A -v -PE -PS22,25,80 -PA21,23,80 10.228.67.X # Nahraďte X IP adresou cíle
```

#### Kontrola zablokování
```bash
grep "attackalert" /var/log/syslog
grep -n DENY /etc/hosts.deny
grep -n Blocked /var/lib/portsentry/portsentry.blocked.tcp
grep -n Blocked /var/lib/portsentry/portsentry.history
grep -n Blocked /var/lib/portsentry/portsentry.blocked.udp
netstat -rn | grep "10.228.67.X" # Kontrola aktivních rout
route -n | grep "10.228.67.X"    # Alternativní kontrola rout
```

#### Odblokování IP adresy
1.  Zastavte PortSentry: `service portsentry stop`
2.  Odstraňte záznam z `/etc/hosts.deny`.
3.  Odstraňte záznamy z `/var/lib/portsentry/portsentry.blocked.tcp`, `/var/lib/portsentry/portsentry.history` a `/var/lib/portsentry/portsentry.blocked.udp`. Příklad:
    ```bash
    sed -i '/10.228.67.X/d' /etc/hosts.deny
    sed -i '/10.228.67.X/d' /var/lib/portsentry/portsentry.blocked.tcp
    # ... a další soubory
    ```
4.  Odstraňte reject routu: `route del -host 10.228.67.X reject`
5.  Spusťte PortSentry: `service portsentry start`

### RKhunter (Rootkit Hunter)

#### Instalace
```bash
apt-get install rkhunter
```

#### Základní použití
```bash
rkhunter --update   # Aktualizace databáze
rkhunter --propupd  # Aktualizace databáze hashů souborů
rkhunter --list     # Výpis dostupných testů
rkhunter -c --enable hidden_ports # Spuštění kontroly, včetně skrytých portů
```

#### Konfigurace (`/etc/rkhunter.conf`)
Příklady nastavení:
```ini
# /etc/rkhunter.conf
MAIL-ON-WARNING="root@localhost" # Kam posílat upozornění
MAIL_CMD=mail -s "[rkhunter] Warnings found for ${HOST_NAME}" # Příkaz pro odesílání emailu
SCRIPTWHITELIST="/usr/sbin/adduser" # Povolení scriptů, které mohou měnit soubory
ALLOWDEVFILE="/dev/.udev/rules.d/root.rules" # Povolení konkrétních souborů v /dev
ALLOWHIDDENDIR="/dev/.udev" # Povolení skrytých adresářů
ALLOWHIDDENFILE="/.blkid.tab" # Povolení skrytých souborů
ALLOW_SSH_ROOT_USER=yes # Povolení přihlášení roota přes SSH (doporučeno NE)
```

#### Spuštění kontroly v Cronu
```bash
rkhunter -C # Spustí všechny kontroly
```

### Cron (Plánovač úloh)

#### Konfigurační soubory
*   Systémový crontab: `/etc/crontab`
*   Crontaby pro balíčky: `/etc/cron.d/`
*   Adresáře pro denní/měsíční/týdenní/hodinové úlohy:
    *   `/etc/cron.daily/`
    *   `/etc/cron.hourly/`
    *   `/etc/cron.monthly/`
    *   `/etc/cron.weekly/`
*   Uživatelské crontaby: `/var/spool/cron/crontabs/` (editace pomocí `crontab -e`)

#### Správa uživatelských crontabů
```bash
crontab -l          # Výpis aktuálního uživatelského crontabu
crontab -e          # Editace aktuálního uživatelského crontabu
crontab -l -u jindra # Výpis crontabu uživatele jindra
```

#### Příklad záznamu v crontabu
```
15 */4 * * * /usr/bin/rkhunter --cronjob --update --quiet
```
Syntaxe: `minuta hodina den_v_měsíci měsíc den_v_týdnu příkaz`

### Rsync (Zálohování)

#### Důležitost lomítek na konci
*   `rsync -rav /opt/dir1/ /opt/backup_dir/` : Zkopíruje obsah `dir1` do `backup_dir`.
*   `rsync -rav /opt/dir1 /opt/backup_dir/` : Zkopíruje `dir1` (adresář) do `backup_dir` (výsledek `/opt/backup_dir/dir1/`).

#### Příklady zálohování
```bash
rsync -rai --dry-run /etc/ /opt/etc_backup_full/ # Dry run pro ověření změn
rsync -a --delete --link-dest=../backup.1 /etc/ backup.0/ # Inkrementální záloha
```

## 4. Příklad k procvičení

### Zadání

1.  Vyzkoušejte si nastavení cronu pro následující případy (např. pro spuštění jednoduchého echo příkazu do souboru):
    *   Každých 10 minut v pracovní době od 9:00 do 17:00.
    *   Každou neděli ve 22:30.
    *   V 6:30 ráno jednou za dva dny.
    *   První pondělí v měsíci.
    *   V 12:05, 12:20, 12:35, 12:50 každý den.
2.  Vytvořte backup script na zálohu PostgreSQL (`pg_dump`) všech databází, kdy každá databáze bude v samostatném dumpu.
3.  Vytvořte script, který bude pomocí `rsync` zálohovat adresář `/var/www/web1` do `/srv/backups/<den_v_tydnu>/` přes SSH.

### Ověření
*   Manuální kontrola záznamů v log souborech po spuštění cron úloh.
*   Kontrola vytvořených záložních souborů a adresářů.
*   Ověření správnosti rsync příkazů.

## 5. Doplňující materiály
_Žádné dodatečné doplňující materiály nejsou k dispozici v původních souborech._
