# Vysoká dostupnost

## 1. Úvod
**Cíl:** Seznámení s principy a nástroji pro zajištění vysoké dostupnosti služeb. Cvičení se zaměřuje na techniky jako DNS Round Robin, synchronizaci dat pomocí Rsync a konfiguraci softwarových load balancerů jako jsou Pound a Nginx pro rozložení zátěže a odolnost proti výpadkům.

## 2. Použité nástroje a reference
*   **DNS (Domain Name System):** [Wikipedia](https://cs.wikipedia.org/wiki/Domain_Name_System)
*   **Rsync:** [Homepage](https://rsync.samba.org/), [Dokumentace](https://rsync.samba.org/documentation.html)

*   **Nginx:** [Homepage](https://nginx.org/), [Dokumentace](https://nginx.org/en/docs/)
*   **HAProxy:** [Homepage](https://www.haproxy.com/), [Dokumentace](https://cbonte.github.io/haproxy-dconv/latest/intro.html)
*   **Traefik:** [Homepage](https://traefik.io/traefik/), [Dokumentace](https://doc.traefik.io/traefik/)

## 3. Poznámky ke cvičení (How-To)

### DNS Round Robin
Princip rozložení zátěže na více serverů pomocí definice více A záznamů pro stejné jméno hostitele. DNS server pak klientům vrací tyto záznamy v rotujícím pořadí.

```
www	IN	A	147.228.67.42
www	IN	A	147.228.67.43
www	IN	A	147.228.67.44
```

### Rsync
Nástroj pro rychlou a efektivní synchronizaci souborů a adresářů, ideální pro udržování konzistentních dat mezi servery.

```bash
rsync -rav /var/www/jindra.spos host2:/var/www/jindra.spos
```
*   `-r`: rekurzivně
*   `-a`: archivní režim (zachovává práva, vlastníky, časy, atd.)
*   `-v`: verbose (podrobný výstup)



### Nginx (Reverse Proxy a Load Balancer)
Výkonný webový server, který je často používán i jako reverse proxy, load balancer a HTTP cache.

#### Instalace

```bash
apt-get install nginx
```

#### Konfigurace (/etc/nginx/sites-available/lb.conf)

```nginx
#/etc/nginx/sites-available/lb.conf
upstream backend  {
        ip_hash; # Zajistí sticky sessions pro klienta
        server www1:8080 max_fails=2 fail_timeout=5s;
        server www2:8080 max_fails=2 fail_timeout=5s;
}

server {
        listen 80;
        server_name www.jindra.spos;

        location / {
                proxy_pass http://backend;
                include proxy.include;
        }
}
```

Přídavný soubor pro nastavení proxy hlaviček (proxy.include):

```nginx
# proxy.include
    proxy_set_header   Host $http_host;
    proxy_set_header   X-Real-IP $remote_addr;
    proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header   X-Forwarded-Proto $scheme;
    proxy_connect_timeout      90;
    proxy_send_timeout         300;
    proxy_read_timeout         300;
```

### Více instancí Apache na jednom serveru

#### Debian-way
Na Debianu existuje mechanismus pro spouštění více instancí Apache serveru s vlastními konfiguracemi a porty.

```bash
less /usr/share/doc/apache2/README.multiple-insances
/usr/share/doc/apache2/examples/setup-instance oldphp
```

#### Docker-way
Využití kontejnerizace (Docker) pro spouštění více nezávislých instancí Apache, s NginX nebo jiným load balancerem před nimi pro rozhazování zátěže.

## 4. Příklad k procvičení

### Zadání

1.  Nasaďte dva vhosty Apache (web01 a web02) s jednoduchým `index.html` souborem, který bude obsahovat název vhostu.
2.  Nastavte `web01` na IP adrese `192.168.200.<vase_ip>` na portu `8888`.
3.  Nastavte `web02` na IP adrese `192.168.100.<vase_ip>` na portu `8080`.
4.  Vytvořte load balancer pomocí NginX na IP adrese `192.168.253.<vase_ip>` na portu `8888`. Nakonfigurujte ho tak, aby:
    *   Požadavky na `/img` byly směřovány pouze na `web01`.
    *   Všechny ostatní požadavky byly balancovány mezi `web01` a `web02`.

### Ověření
Manuální kontrola dostupnosti a chování služeb přes prohlížeč a pomocí nástrojů jako `curl` nebo `telnet`. Sledujte logy load balancerů a backend serverů.

## 5. Doplňující materiály
_Žádné dodatečné doplňující materiály nejsou k dispozici v původních souborech._
