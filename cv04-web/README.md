# Správa webových služeb

## 1. Úvod
Cílem tohoto cvičení je seznámit se se správou webových služeb – základní orientace v protokolu HTTP, správa vybraných webových serverů (Apache, Nginx) a ukázky jednoduchých aplikací, které je možné na nich provozovat. Cvičení zahrnuje konfiguraci virtuálních hostitelů, nastavení PHP a zabezpečení pomocí HTTPS.

## 2. Použité nástroje a reference
*   **Apache HTTP Server:** [Homepage](https://httpd.apache.org/), [Dokumentace](https://httpd.apache.org/docs/2.4/)
*   **Nginx:** [Homepage](https://nginx.org/), [Dokumentace](https://nginx.org/en/docs/)
*   **PHP:** [Homepage](https://www.php.net/), [Dokumentace](https://www.php.net/docs.php)
*   **OpenSSL:** [Homepage](https://www.openssl.org/), [Dokumentace](https://www.openssl.org/docs/)
*   **Dehydrated (Let's Encrypt klient):** [GitHub](https://github.com/dehydrated-io/dehydrated)

## 3. Poznámky ke cvičení (How-To)

### Instalace

Instalace Apache webového serveru:

```bash
apt-get install apache2
```

Instalace PHP pro Apache (verze se může lišit, pro PHP 7+ obvykle `libapache2-mod-php`):

```bash
apt-get install libapache2-mod-php
```

### Konfigurace Apache

#### Moduly a virtuální hostitelé

Povolení/zakázání Apache modulů:

```bash
a2enmod <module_name>
a2dismod <module_name>
```

Povolení/zakázání virtuálních hostitelů (sites):

```bash
a2ensite <site_name>
a2dissite <site_name>
```

Uživatel, pod kterým běží Apache:

```
user: www-data / httpd
```

#### Klíčové konfigurační soubory a adresáře

```
/etc/apache2/sites-*
/etc/apache2/mods-*
/etc/apache2/ports.conf
```

#### Testování konfigurace

Test syntaxe konfigurace a zobrazení nastavených virtuálních hostitelů:

```bash
apachectl -t
apachectl -S
```

### Správa služby Apache

Znovunačtení konfigurace Apache:

```bash
service apache2 reload
```

### Data a logy Apache

Výchozí adresář pro webové soubory (DocumentRoot):

```
/var/www/
```

Log soubory:

```
/var/log/apache2/*access.log
/var/log/apache2/*error.log
```

### Příklad konfigurace VirtualHost

Kompletní příklad VirtualHost konfigurace pro Apache:

```apache
<VirtualHost *:80>
        ServerAdmin admin@jindra.spos
        ServerName www.jindra.spos
        ServerAlias web.jindra.spos

        DocumentRoot /var/www/www.jindra.spos

        Redirect / https://www.jindra.spos/

        Alias /pma /usr/share/phpmyadmin
        
        <Directory />
                Options FollowSymLinks
                AllowOverride None
        </Directory>

        ProxyPass /app1 http://localhost:3333/
        ProxyPassReverse /app1 http://localhost:3333/

        <Directory /var/www/>
                Options Indexes FollowSymLinks MultiViews
                AllowOverride All
                Order allow,deny
                allow from all
        </Directory>

        <Location /wsNS_1.0>
                AuthType Basic
                AuthName "Restricted Access"
                AuthUserFile /etc/apache2/.htpasswd
                Require user app_user1
        </Location>

        # Possible values include: debug, info, notice, warn, error, crit,
        # alert, emerg.
        LogLevel warn

        CustomLog ${APACHE_LOG_DIR}/www.jindra.spos.access.log combined
        ErrorLog ${APACHE_LOG_DIR}/www.jindra.spos.error.log

</VirtualHost>
```

### Konfigurace PHP

#### Adresáře konfigurace PHP

Příklad konfiguračních adresářů pro PHP (verze se může lišit):

```
/etc/php5/apache2
/etc/php5/cli
```

#### Běžná nastavení v `php.ini`

Příklady nastavení, která lze upravit v `php.ini` nebo v konfiguraci VirtualHost:

```ini
upload_max_filesize = 16M
memory_limit = 128M
max_input_time = 60
post_max_size = 16M
max_execution_time = 30

open_basedir = ...
disable_functions = ...
disable_classes = ...
allow_url_fopen = On/Off
allow_url_include = On/Off
display_errors = On/Off
error_reporting = E_ALL
log_errors = On/Off
```

#### Nastavení PHP v `.htaccess` nebo VirtualHostu

Příklad nastavení specifických PHP direktiv:

```apache
php_admin_flag safe_mode On
php_admin_value open_basedir /var/www/www.jindra.spos
php_admin_value upload_tmp_dir /var/www/www.jindra.spos/phptmp
php_admin_value session.save_path /var/www/www.jindra.spos/phptmp
```

#### Zobrazení PHP informací

Jednoduchý PHP skript pro zobrazení detailních informací o PHP konfiguraci (`info.php`):

```php
<?php
  phpinfo();
?>
```

#### Bezpečnostní poznámky (Local File Inclusion - LFI)

Příklad zranitelného PHP skriptu umožňujícího LFI:

```php
<?php
    $page = $_GET['page'];
    $xfile = fopen($page, "r") or die("Unable to open file!");
    echo fgets($xfile);
    fclose($xfile);
?>
```
Příklad zneužití: `http://jindra.spos/index.php?page=/etc/passwd`

### HTTPS/SSL

#### Self-signed certifikát

Generování vlastního (self-signed) SSL certifikátu:

```bash
openssl genrsa -des3 -out server.key 1024
openssl req -new -key server.key -out server.csr
cp server.key server.key.org
openssl rsa -in server.key.org -out server.key
openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
```

#### Let's Encrypt certifikát s Dehydrated

Postup pro získání certifikátu od Let's Encrypt pomocí klienta Dehydrated:

```bash
cd /opt
git clone https://github.com/dehydrated-io/dehydrated
cd dehydrated
cp ./docs/examples/hook.sh /etc/dehydrated/hook.sh
chmod +x /etc/dehydrated/hook.sh
```

Upravte konfigurační soubor `/etc/dehydrated/config` (příklad důležitých nastavení):

```ini
DOMAINS_TXT=/etc/dehydrated/domains
WELLKNOWN="/var/www/dehydrated/.well-known/acme-challenge/"
CERTDIR="/etc/letsencrypt/live/"
CONTACT_EMAIL=skupaj@students.zcu.cz
HOOK=/etc/dehydrated/hook.sh
```

Registrace a získání certifikátu:

```bash
dehydrated --register --accept-terms
```

Konfigurace webového serveru pro Let's Encrypt ověření:

**Apache:**

```apache
Alias /.well-known/acme-challenge/ /var/www/dehydrated/.well-known/acme-challenge/
```

**Nginx:**

```nginx
location /.well-known/acme-challenge {
    alias   /var/www/dehydrated/.well-known/acme-challenge;
}
```

## 4. Příklad k procvičení

### Zadání

1.  Vytvořte v Apache dva virtuální hostitele (vhosty), každý s vlastním `DocumentRoot` adresářem, kde soubor `index.html` obsahuje název vhostu:
    *   `www.<orion-login>.spos-test.spos`
    *   `server.<orion-login>.spos-test.spos`
2.  Nastavte tyto vhosty tak, aby naslouchaly na portu `8888` pro rozhraní v síti `192.168.255.0/24`.
3.  Na vhostu `www.<orion-login>.spos-test.spos` vytvořte skript `php_info.php`, který zobrazí PHP informace a aktuální datum a čas v titulku webové stránky.
4.  Vytvořte HTTPS vhost pro hostname `ssl.<orion-login>.spos-test.spos` a standardní porty (443), jehož `index.html` bude opět obsahovat název serveru.

### Ověření

Pro ověření funkčnosti použijte DNS z předchozího cvičení a následující `curl` příkazy:

```bash
curl http://www.<orion-login>.spos-test.spos:8888 | grep www.<orion-login>.spos-test.spos && echo OK
curl http://server.<orion-login>.spos-test.spos:8888 | grep server.<orion-login>.spos-test.spos && echo OK
curl -k https://ssl.<orion-login>.spos-test.spos | grep ssl.<orion-login>.spos-test.spos && echo OK
curl http://www.<orion-login>.spos-test.spos:8888/php_info.php | less
```

## 5. Doplňující materiály (Volitelné)
Nejsou k dispozici žádné další doplňující materiály.
