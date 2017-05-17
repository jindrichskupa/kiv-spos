# Cviceni 10 - vystup

## Zadani

1. Nastavte v apache dva vhosty web01 a web02, kde index.html bude obsahovat nazev vhostu
2. web01:
  * ip `192.168.200.<vase_ip>` port 8888
3. web02:
  * ip `192.168.100.<vase_ip>` port 8080

1. Vytvorte loadbalancer na ip `192.168.253.<vase\_ip>` na portu 8888 pomoci NginX, nastavte tak ze `/img` se bere pouze z web01 a vse ostatni z web01 i web02
2. Vytvorte loadbalancer na ip `192.168.253.<vase\_ip>` na portu 8088 pomoci Pound, vse se bude balancovat mezi web01 a web02, vyzkousejte a nasledne vypnete web02

## Kontrola

Manualni.
