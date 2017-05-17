# Cviceni 5 - vystup

## Zadani

1. (MySQL) Vytvorte databazi `spos_test` a dva uzivatele 
  * `spos_test`, vsechna prava, pristup z localhostu
  * `spos_ro`, pouze read-only, pristup z 192.168.255.0/24
2. Ve vytvoreni databazi zalozte tabulku, ktera bude obsahovat nasledujici sloupce: `id` (autoincrement a primarni klic), `name` (varchar), `description` (text), `created_at` (time stamp, vychozi NOW) a `price` (float), vsechny sloupce budou `NOT NULL`
3. Modifikujte dodany PHP script na zobrazeni obsahu s moznosti nastavit razeni podle ceny.
4. Vygenerujte do tabulky alespon 50 zaznamu.

1. (PostgreSQL) Vytvorte databazi `spos_test`
  * `spos_test`, vlastnik, vsechna prava, pristup z local podle systemoveho username, z localhostu s heslem
  * `spos_ro`, pouze read-only, pristup z 192.168.255.0/24
2. Ve vytvoreni databazi zalozte tabulku, ktera bude obsahovat nasledujici sloupce: `id` (autoincrement a primarni klic), `name` (varchar), `description` (text), `created_at` (time stamp, vychozi NOW) a `price` (float), vsechny sloupce budou `NOT NULL`
3. Modifikujte dodany PHP script na zobrazeni obsahu s moznosti nastavit razeni podle ceny.
4. Vygenerujte do tabulky alespon 50 zaznamu.

## Kontrola

Manualni
