# Správa databázových systémů

## 1. Úvod
**Cíl:** Seznámení se základními principy relačních databází, správou databázových systémů MySQL/MariaDB a PostgreSQL, a základy jazyka SQL (DML/DDL). Cvičení zahrnuje instalaci, konfiguraci, správu uživatelů a databází, a tvorbu jednoduchých SQL dotazů.

## 2. Použité nástroje a reference
*   **MariaDB (kompatibilní s MySQL):** [Homepage](https://mariadb.org/), [Dokumentace](https://mariadb.com/kb/en/documentation/)
*   **PostgreSQL:** [Homepage](https://www.postgresql.org/), [Dokumentace](https://www.postgresql.org/docs/)
*   **PHP:** [Homepage](https://www.php.net/), [Dokumentace](https://www.php.net/manual/en/)

## 3. Poznámky ke cvičení (How-To)

### MySQL/MariaDB

#### Instalace a nastavení
```bash
apt-get install mariadb-server
```

#### Klíčové konfigurační soubory a adresáře
*   `/etc/mysql/`
*   `/var/lib/mysql`

#### Správa služby
```bash
/etc/init.d/mysql start
/etc/init.d/mysql stop
/etc/init.d/mysql restart
/etc/init.d/mysql status
```

#### Připojení a základní informace
```bash
mysql -uroot -pheslo databaze
SHOW DATABASES;
SHOW TABLES;
USE databaze;
DESC tabulka;
SHOW CREATE TABLE tabulka\G
```

#### Nastavení klienta (pro automatické přihlášení)
```bash
cat ~/.my.cnf
```
```ini
[client]
user=db01
password=password
```

#### Záloha a administrace
```bash
mysqldump
mysqladmin
rsync -rav /var/lib/mysql/ /srv/zaloha-mysql/
```

#### SQL příkazy (uživatel, databáze, tabulka)
```sql
CREATE USER 'db01'@'localhost' IDENTIFIED BY 'password';
CREATE DATABASE db01 DEFAULT CHARACTER SET utf8  DEFAULT COLLATE utf8_general_ci;
GRANT ALL PRIVILEGES ON db01.* to 'db01'@'localhost' IDENTIFIED BY 'password';
CREATE TABLE table01(
   id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
   firstname VARCHAR(30) NOT NULL,
   lastname VARCHAR(30) NOT NULL,
   email VARCHAR(50),
   reg_date TIMESTAMP
);
```

#### PHP Příklad připojení a čtení dat
```php
<?php
$servername = "localhost";
$username = "username";
$password = "password";
$dbname = "myDB";

// Create connection
$conn = new mysqli($servername, $username, $password, $dbname);
// Check connection
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}

$sql = "SELECT id, firstname, lastname FROM MyGuests";
$result = $conn->query($sql);

if ($result->num_rows > 0) {
    // output data of each row
    while($row = $result->fetch_assoc()) {
        echo "id: " . $row["id"]. " - Name: " . $row["firstname"]. " " . $row["lastname"]. "<br>";
    }
} else {
    echo "0 results";
}
$conn->close();
?>
```

### PostgreSQL

#### Instalace a nastavení
```bash
apt-get install postgresql-9.4 # Verze se může lišit
```

#### Klíčové konfigurační soubory a adresáře
*   `/etc/postgresql/9.4/main/`
*   `/var/lib/postgresql/9.4/main/`
*   `pg_hba.conf` (konfigurace autentifikace klientů)

#### Připojení a základní informace
```bash
su - postgres
psql -U root databaze
\l          -- Výpis databází
\c databaze -- Připojení k databázi
\dt         -- Výpis tabulek
\d tabulka  -- Popis tabulky
\x on       -- Rozšířený výpis (pro lepší čitelnost detailů)
\x off      -- Vypnutí rozšířeného výpisu
```

#### Nastavení klienta (pro automatické přihlášení)
```bash
cat ~/.pgpass
```
```
localhost:5432:db01:db01:password
```

#### Záloha a administrace
```bash
pg_dump
pg_restore
createuser
createdb
```

#### SQL příkazy (uživatel, databáze, tabulka)
```sql
CREATE USER db01 WITH PASSWORD 'pasword';
CREATE DATABASE db01 WITH OWNER=db01;
GRANT ALL PRIVILEGES ON DATABASE db01 to db01;
CREATE TABLE table01(
   id SERIAL PRIMARY KEY,
   firstname VARCHAR(30) NOT NULL,
   lastname VARCHAR(30) NOT NULL,
   email VARCHAR(50),
   reg_date TIMESTAMP
);
```

#### PHP Příklad připojení a čtení dat
```php
<?php
// Connecting, selecting database
$dbconn = pg_connect("host=localhost dbname=publishing user=www password=foo")
    or die('Could not connect: ' . pg_last_error());

// Performing SQL query
$query = 'SELECT * FROM authors';
$result = pg_query($query) or die('Query failed: ' . pg_last_error());

// Printing results in HTML
echo "<table>\n";
while ($line = pg_fetch_array($result, null, PGSQL_ASSOC)) {
    echo "\t<tr>\n";
    foreach ($line as $col_value) {
        echo "\t\t<td>$col_value</td>\n";
    }
    echo "\t</tr>\n";
}
echo "</table>\n";

// Free resultset
pg_free_result($result);

// Closing connection
pg_close($dbconn);
?>
```

#### Příklad nastavení `pg_hba.conf`
```
# TYPE  DATABASE        USER            ADDRESS                 METHOD
local   all             postgres                                peer
local   all             all                                     md5
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

## 4. Příklad k procvičení

### Zadání

1.  **(MySQL)**
    *   Vytvořte databázi `spos_test` a dva uživatele:
        *   `spos_test` se všemi právy a přístupem z `localhostu`.
        *   `spos_ro` pouze s právy pro čtení a přístupem z `192.168.255.0/24`.
    *   Ve vytvořené databázi založte tabulku, která bude obsahovat následující sloupce:
        *   `id` (autoincrement a primární klíč)
        *   `name` (varchar)
        *   `description` (text)
        *   `created_at` (timestamp, výchozí `NOW()`)
        *   `price` (float)
        *   Všechny sloupce budou `NOT NULL`.
    *   Modifikujte dodaný PHP script na zobrazení obsahu s možností nastavit řazení podle ceny.
    *   Vygenerujte do tabulky alespoň 50 záznamů.

2.  **(PostgreSQL)**
    *   Vytvořte databázi `spos_test` a uživatele:
        *   `spos_test` jako vlastník, se všemi právy, přístupem z `localhostu` (podle systémového uživatele) a s heslem.
        *   `spos_ro` pouze s právy pro čtení a přístupem z `192.168.255.0/24`.
    *   Ve vytvořené databázi založte tabulku, která bude obsahovat následující sloupce:
        *   `id` (autoincrement a primární klíč)
        *   `name` (varchar)
        *   `description` (text)
        *   `created_at` (timestamp, výchozí `NOW()`)
        *   `price` (float)
        *   Všechny sloupce budou `NOT NULL`.
    *   Modifikujte dodaný PHP script na zobrazení obsahu s možností nastavit řazení podle ceny.
    *   Vygenerujte do tabulky alespoň 50 záznamů.

### Ověření
*   Manuální kontrola vytvořených uživatelů, databází a tabulek.
*   Testování přístupu uživatelů z různých IP adres.
*   Ověření funkčnosti PHP scriptu pro zobrazení a řazení dat.
*   Kontrola počtu záznamů v tabulce.

## 5. Doplňující materiály

### Skript pro generování dat (MySQL/MariaDB)
```bash
for i in `seq 1 50`; do
   echo "INSERT INTO table01 (firstname, lastname, email, reg_date) VALUES ('$(pwgen 5 1)','$(pwgen 10 1)','$(pwgen 5 1)@spos-jindra.spos',now())" | mysql db01
done
```

