# Správa emailových služeb

## 1. Úvod
**Cíl:** Seznámení se základními komponentami emailových služeb (MTA, MDA, MUA), instalací a konfigurací Postfixu jako Mail Transfer Agent, Dovecotu jako IMAP/POP3 serveru a Mutt jako textového emailového klienta. Cvičení zahrnuje konfiguraci aliasů, virtuálních domén a uživatelů pro emailové schránky.

## 2. Použité nástroje a reference
*   **Postfix:** Mail Transfer Agent (MTA).
    *   Homepage: [http://www.postfix.org/](http://www.postfix.org/)
    *   Dokumentace: [Postfix Documentation](http://www.postfix.org/documentation.html)
    *   **Postfix Utilities:**
        *   **postqueue:** Správa fronty pošty Postfixu.
            *   Reference: [man postqueue](http://www.postfix.org/postqueue.1.html)
        *   **postsuper:** Provádění údržbových úkolů ve frontě Postfixu.
            *   Reference: [man postsuper](http://www.postfix.org/postsuper.1.html)
        *   **newaliases:** Přestavba aliasové databáze Postfixu.
            *   Reference: [man newaliases](http://www.postfix.org/newaliases.1.html)
*   **Dovecot:** IMAP a POP3 server (Mail Delivery Agent - MDA).
    *   Homepage: [https://www.dovecot.org/](https://www.dovecot.org/)
    *   Dokumentace: [Dovecot Documentation](https://doc.dovecot.org/)
*   **Mutt:** Textový emailový klient (Mail User Agent - MUA).
    *   Homepage: [http://www.mutt.org/](http://www.mutt.org/)
    *   Dokumentace: [Mutt Documentation](http://www.mutt.org/doc.html)
*   **Systemd:** Pro správu služeb Postfix a Dovecot.
    *   Reference: [man systemctl](https://www.freedesktop.org/software/systemd/man/systemctl.html)
*   **mail:** Jednoduchá utilita pro odesílání pošty z příkazové řádky (součást `mailutils` nebo `bsd-mailx`).
    *   Reference: [man mail](https://manpages.debian.org/mail)
*   **netcat (nc):** Nástroj pro čtení a zápis dat přes síťová připojení, často používaný pro testování síťových služeb.
    *   Reference: [man nc](https://manpages.debian.org/nc)

## 3. Poznámky ke cvičení (How-To)

### Postfix

#### Instalace a informace
```bash
apt-get install postfix
```

#### Logy a fronta pošty
*   Log soubor: `/var/log/mail.log`
*   Zobrazení fronty: `postqueue -p`
*   Vynucené zpracování fronty: `postqueue -f`
*   Smazání zprávy z fronty (dle ID): `postsuper -d <message_id>`

#### Testování odesílání emailů
```bash
echo "Test" | mail -s "Testovaci mail" jindra@jindra.spos
```

#### Testování pomocí Telnetu
```bash
telnet localhost 25
HELO jindra.spos
MAIL FROM: pepa@jindra.spos
RCPT TO: jindra@jindra.spos
DATA
Subject: Testovaci
Ahoj svete
.
QUIT
```

#### Nastavení a aliasy
*   Konfigurace aliasů v `/etc/aliases`. Po změně je nutné spustit `newaliases`.
```bash
newaliases
```

*   Nastavení umístění lokální schránky v `/etc/postfix/main.cf`:
```ini
# /etc/postfix/main.cf
home_mailbox = Maildir/
mailbox_command =
```

#### Virtuální maily
*   Nastavení pro doručování virtuálních mailů v `/etc/postfix/main.cf`:
```ini
# /etc/postfix/main.cf
virtual_alias_domains_map = hash:/etc/postfix/virtual_domains
virtual_alias_maps = hash:/etc/postfix/virtual
```
*   Příklady konfiguračních souborů:
    *   `/etc/postfix/virtual_domains`
    ```
    jindra2.spos	OK
    jindra3.spos	OK
    ```
    *   `/etc/postfix/virtual`
    ```
    info@jindra2.spos	jindra
    info@jindra3.spos	pepa
    ```
*   **Kompletní virtuální hosting (příklad s `vmail` uživatelem):**
    *   `/etc/postfix/main.cf`
    ```ini
    # /etc/postfix/main.cf
    virtual_mailbox_domains_maps = hash:/etc/postfix/vmailbox_domains
    virtual_mailbox_maps = hash:/etc/postfix/vmailbox
    virtual_mailbox_base = /home/vmail/vhosts
    virtual_minimum_uid = 100
    virtual_uid_maps = static:5000
    virtual_gid_maps = static:5000

    dovecot_destination_recipient_limit = 1
    virtual_transport = dovecot
    ```
    *   `/etc/postfix/vmailbox`
    ```
    info@jindra4.spos    jindra4.spos/info/
    group@jindra4.spos   jindra4.spos/group/
    @jindra4.spos        jindra4.spos/catchall/
    ```
    *   `/etc/dovecot/conf.d/auth-passwdfile.conf.ext`
    ```ini
    # /etc/dovecot/conf.d/auth-passwdfile.conf.ext
    passdb {
        driver = passwd-file
        args = scheme=plain username_format=%u /etc/mail.passwd
     }

     userdb  {
        driver = passwd-file
        args = username_format=%u /etc/mail.passwd
     }
    ```
    *   `/etc/mail.passwd` (příklad uživatelských hesel pro Dovecot)
    ```
    info@jindra4.spos:{plain}heslo:5000:5000::/home/vmail/vhosts/jindra4.spos/info/
    group@jindra4.spos:{plain}heslo:5000:5000::/home/vmail/vhosts/jindra4.spos/group/
    catchall@jindra4.spos:{plain}heslo:5000:5000::/home/vmail/vhosts/jindra4.spos/catchall/
    ```
    *   `/etc/postfix/master.cf`
    ```
    # /etc/postfix/master.cf
    dovecot   unix  -       n       n       -       -       pipe
      flags=DRhu user=vmail:vmail argv=/usr/lib/dovecot/deliver -f ${sender} -d ${recipient}
    ```
    *   `/etc/dovecot/conf.d/10-auth.conf`
    ```ini
    # /etc/dovecot/conf.d/10-auth.conf
    #!include auth-system.conf.ext
    !include auth-passwdfile.conf.ext
    ```

### Mutt

#### Instalace
```bash
apt-get install mutt
```

#### Použití
```bash
mutt -f ~jindra/Maildir # pro Maildir
mutt -f /var/spool/mail/jindra # pro mbox
```

### Dovecot

#### Instalace POP3 a IMAP serveru
```bash
apt-get install dovecot-pop3d dovecot-imapd
```

#### Testování POP3 pomocí Telnetu
```bash
telnet localhost 110
USER jindra
PASS jindra123
LIST
RETR 1
DELE 1
QUIT
```

#### Testování IMAP pomocí Telnetu
```bash
telnet localhost 143
1 LOGIN jindra jindra123
2 LIST "" "*"
3 STATUS INBOX (MESSAGES)
4 EXAMINE INBOX
5 FETCH 1 BODY[]
6 LOGOUT
```

#### Konfigurace pro Mutt (IMAP)
```bash
~/.muttrc
```
```ini
set folder        = imap://jindra:jindra123@localhost:143/
set spoolfile   = imap://jindra:jindra123@localhost:143/
```

#### Umístění mailů (např. Maildir)
*   Nastavení v `/etc/dovecot/conf.d/10-mail.conf`:
```ini
# /etc/dovecot/conf.d/10-mail.conf
# Pro mbox:
mail_location = mbox:~/mail:INBOX=/var/mail/%u
# Pro Maildir:
mail_location = maildir:~/Maildir
```

## 4. Příklad k procvičení

### Zadání

1.  Vytvořte virtuální domény `<orion-login>1.spos-test.spos` a `<orion-login>2.spos-test.spos`.
2.  Nastavte ukládání mailů do Maildiru.
3.  Vytvořte systémové uživatele `mail001` a `mail002` bez možnosti přihlášení na server.
4.  Nastavte alias `info@<orion-login>1.spos-test.spos` na `mail001`.
5.  Nastavte alias `kontakt@<orion-login>2.spos-test.spos` na `mail002`.
6.  Odešlete mail na vytvořené aliasy z `kontrola@spos-test.spos` a zkontrolujte doručení pomocí `mutt`.
7.  Nastavte `mutt` na prohlížení schránky přes IMAP uživatele `mail001`.

### Ověření
Pomocí `mutt` zobrazte doručené maily (lokálně a přes IMAP).

## 5. Doplňující materiály
_Žádné dodatečné doplňující materiály nejsou k dispozici v původních souborech._
