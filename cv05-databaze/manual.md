# Cviceni 5

Zaklady SQL: INSERT, SELECT, DELETE, UPDATE, TRUNCATE, ALTER... (DML/DDL)

## MySQL

### Instalace a nastaveni

```bash
apt-get install mariadb-server
/etc/init.d/mysql start|stop|restart
/etc/mysql/
/var/lib/mysql
```

### Pripojeni a zakladni informace

```bash
mysql -uroot -pheslo databaze
SHOW DATABASES;
SHOW TABLES;
USE databaze;
DESC tabulka;
SHOW CREATE TABLE tabulka\G
```

### Nastaveni klienta

```
cat ~/.my.cnf
[client]
user=db01
password=password
```

### Zaloha a administrace

```bash
mysqldump 
mysqladmin
rsync -rav /var/lib/mysql/ /srv/zaloha-mysql/
```

### SQL

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
)
```

### PHP Example

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

## PostgreSQL

### Instalace a nastaveni

```bash
apt-get install postgresql-9.4
su - postgres
/etc/postgresql/9.4/main/
/var/lib/postgresql/9.4/main/
pg_hba.conf
```

### Pripojeni a zakladni informace

```bash
psql -U root databaze
\l
\c databaze
\dt 
\d tabulka
\x on | \x off
```

### Nastaveni klienta

```bash
cat ~/.pgpass
localhost:5432:db01:db01:password
```

### Zaloha a administrace

```
pg_dump
pg_restore

createuser
createdb
```

### SQL

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

### PHP Example

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

### Nastaveni pg_hba.conf

```
pg_hba.conf:
local   all         postgres                         peer
local   all         all                              md5
host    all         all             127.0.0.1/32     md5
host    all         all             ::1/128          md5
```


## Generovani dat

```bash
for i in `seq 1 50`; do 
   echo "INSERT INTO table01 (firstname, lastname, email, reg_date) VALUES ('$(pwgen 5 1)','$(pwgen 10 1)','$(pwgen 5 1)@spos-jindra.spos',now())" | mysql db01
done
```
