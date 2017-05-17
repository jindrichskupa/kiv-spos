# Cviceni 4 - vystup

## Zadani

1. Vytvorte v apache dva vhosty, kazdy s vlastnim document rootem, kde index.html obsahuje nazev vhostu
  * www.\<orion-login\>.spos-test.spos
  * server.\<orion-login\>.spos-test.spos
2. Nastavte v hosty na porty 8888 pro rozhrani v siti 192.168.255.0/24 
3. Vytvorte script php_info.php ktery zobrazi PHP info a aktualni datum a cas v titulky webu na www.\<orion-login\>.spos-test.spos
4. Vytvorte HTTPS vhost pro hostname ssl.\<orion-login\>.spos-test.spos a standardni porty, index.html bude obsahovat opet nazev serveru

## Kontrola

Vyuzijte DNS z predchoziho cviceni.

```bash
curl http://www.<orion-login>.spos-test.spos:8888 | grep www.<orion-login>.spos-test.spos && echo OK
curl http://server.<orion-login>.spos-test.spos:8888 | grep server.<orion-login>.spos-test.spos && echo OK
curl -k https://ssl.<orion-login>.spos-test.spos | grep ssl.<orion-login>.spos-test.spos && echo OK
curl http://www.<orion-login>.spos-test.spos:8888/php_info.php | less
```

