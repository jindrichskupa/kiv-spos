# Cviceni 6

## Mail - Postfix, Dovecot, Mutt

### Postfix

Instalace a informace

```bash
apt-get install postfix
/var/log/mail.log
postqueue -p
postqueue -f
postsuper -d
```

Testovani

```bash
echo "Test" | mail -s "Testovaci mail" jindra@jindra.spos
```

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

Nastaveni a aliasy

```bash
/etc/aliases
newaliases

/etc/postfix/main.cf:
home_mailbox = Maildir/
mailbox_command =
```

### Mutt

```bash
apt-get install mutt
mutt -f ~jindra/Maildir #or
mutt -f /var/spool/mail/jindra
```


### Dovecot

POP3

```bash
apt-get install dovecot-pop3d

telnet localhost 110
# https://tools.ietf.org/html/rfc1939
USER jindra
PASS jindra123
LIST
RETR 1
DELE 1
QUIT
```

IMAP

```bash
apt-get install dovecot-imapd

telnet localhost 143
# https://tools.ietf.org/html/rfc3501
1 LOGIN jindra jindra123
2 LIST "" "*"
3 STATUS INBOX (MESSAGES)
4 EXAMINE INBOX
5 FETCH 1 BODY[]
6 LOGOUT
```

```bash
~/.muttrc
set folder        = imap://jindra:jindra123@localhost:143/
set spoolfile   = imap://jindra:jindra123@localhost:143/
```

## Virtualni maily

Umisteni mailu (dovecot)

```bash
/etc/dovecot/conf.d/10-mail.conf
mail_location = mbox:~/mail:INBOX=/var/mail/%u
mail_location = maildir:~/Maildir
```

Nastaveni uctu pro dorucovani

```bash
/etc/postfix/main.cf
virtual_alias_domains_map = hash:/etc/postfix/virtual_domains
virtual_alias_maps = hash:/etc/postfix/virtual
```

Konfiguracni soubory

```bash
/etc/postfix/virtual_domains
jindra2.spos	OK
jindra3.spos	OK

/etc/postfix/virtual
info@jindra2.spos	jindra
info@jindra3.spos	pepa
```

Kompletni virtualni hosting

```bash
/etc/postfix/main.cf
virtual_mailbox_domains_maps = hash:/etc/postfix/vmailbox_domains
virtual_mailbox_maps = hash:/etc/postfix/vmailbox
virtual_mailbox_base = /home/vmail/vhosts
virtual_minimum_uid = 100
virtual_uid_maps = static:5000
virtual_gid_maps = static:5000

dovecot_destination_recipient_limit = 1
virtual_transport = dovecot


/etc/postfix/vmailbox
info@jindra4.spos    jindra4.spos/info/
group@jindra4.spos   jindra4.spos/group/
@jindra4.spos        jindra4.spos/catchall/

/etc/dovecot/conf.d/auth-passwdfile.conf.ext
passdb {
    driver = passwd-file
    args = scheme=plain username_format=%u /etc/mail.passwd
 }

 userdb  {
    driver = passwd-file
    args = username_format=%u /etc/mail.passwd
 }

/etc/mail.passwd
info@jindra4.spos:{plain}heslo:5000:5000::
	/home/vmail/vhosts/jindra4.spos/info/
group@jindra4.spos:{plain}heslo:5000:5000::
	/home/vmail/vhosts/jindra4.spos/group/
catchall@jindra4.spos:{plain}heslo:5000:5000::
	/home/vmail/vhosts/jindra4.spos/catchall/

/etc/postfix/master.cf
dovecot   unix  -       n       n       -       -       pipe
  flags=DRhu user=vmail:vmail argv=/usr/lib/dovecot/deliver -f ${sender} -d ${recipient}

/etc/dovecot/conf.d/10-auth.conf
#!include auth-system.conf.ext
!include auth-passwdfile.conf.ext
```
