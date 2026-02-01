# Vysoká dostupnost

## 1. Úvod
**Cíl:** Seznámení s principy a nástroji pro zajištění vysoké dostupnosti služeb. Cvičení se zaměřuje na techniky jako DNS Round Robin, synchronizaci dat pomocí Rsync a konfiguraci softwarových load balancerů jako jsou Pound a Nginx pro rozložení zátěže a odolnost proti výpadkům.

<h2> 2. Použité nástroje a reference</h2>
<ul>
    <li>
        <strong>DNS (Domain Name System):</strong>
        <a href="https://cs.wikipedia.org/wiki/Domain_Name_System">Wikipedia</a>
    </li>
    <li>
        <strong>Rsync:</strong>
        <a href="https://rsync.samba.org/">Homepage</a>,
        <a href="https://rsync.samba.org/documentation.html">Dokumentace</a>
    </li>
    <li>
        <strong>Pound:</strong>
        <a href="http://www.apsis.ch/pound/">Homepage</a>,
        <a href="http://www.apsis.ch/pound/Pound.html">Dokumentace</a>
    </li>
    <li>
        <strong>Nginx:</strong>
        <a href="https://nginx.org/">Homepage</a>,
        <a href="https://nginx.org/en/docs/">Dokumentace</a>
    </li>
    <li>
        <strong>HAProxy:</strong>
        <a href="https://www.haproxy.com/">Homepage</a>,
        <a href="https://cbonte.github.io/haproxy-dconv/latest/intro.html">Dokumentace</a>
    </li>
    <li>
        <strong>Traefik:</strong>
        <a href="https://traefik.io/traefik/">Homepage</a>,
        <a href="https://doc.traefik.io/traefik/">Dokumentace</a>
    </li>
</ul>

<h2> 3. Poznámky ke cvičení (How-To)</h2>

<h3>DNS Round Robin</h3>
<p>Princip rozložení zátěže na více serverů pomocí definice více A záznamů pro stejné jméno hostitele. DNS server pak klientům vrací tyto záznamy v rotujícím pořadí.</p>

```
www	IN	A	147.228.67.42
www	IN	A	147.228.67.43
www	IN	A	147.228.67.44
```

<h3>Rsync</h3>
<p>Nástroj pro rychlou a efektivní synchronizaci souborů a adresářů, ideální pro udržování konzistentních dat mezi servery.</p>

```bash
rsync -rav /var/www/jindra.spos host2:/var/www/jindra.spos
```
<ul>
    <li><code>-r</code>: rekurzivně</li>
    <li><code>-a</code>: archivní režim (zachovává práva, vlastníky, časy, atd.)</li>
    <li><code>-v</code>: verbose (podrobný výstup)</li>
</ul>

<h3>Pound (Reverse Proxy a Load Balancer)</h3>
<p>Lehký reverse proxy server, který může fungovat jako jednoduchý load balancer.</p>

<h4>Instalace</h4>

```bash
apt-get install pound
```

<h4>Konfigurace (/etc/pound/pound.cfg)</h4>

```
# /etc/pound/pound.cfg
ListenHTTP
    Address 0.0.0.0
    Port    80
End

Service
    HeadRequire "Host:.*www.jindra.spos.*"
    BackEnd
        Address 127.0.0.1
        Port    80
        Priority 5
    End
    BackEnd
        Address 10.228.67.42
        Port    80
        Priority 5
    End
End
```

<h4>Správa Pound</h4>

```bash
poundctl -c /var/run/pound/poundctl.socket
# Změna váhy backendu (např. backend 0, server 0, nová váha 1)
poundctl -c /var/run/pound/poundctl.socket -b 0 0 1
```

<h3>Nginx (Reverse Proxy a Load Balancer)</h3>
<p>Výkonný webový server, který je často používán i jako reverse proxy, load balancer a HTTP cache.</p>

<h4>Instalace</h4>

```bash
apt-get install nginx
```

<h4>Konfigurace (/etc/nginx/sites-available/lb.conf)</h4>

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

<p>Přídavný soubor pro nastavení proxy hlaviček (proxy.include):</p>

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

<h3>Více instancí Apache na jednom serveru</h3>

<h4>Debian-way</h4>
<p>Na Debianu existuje mechanismus pro spouštění více instancí Apache serveru s vlastními konfiguracemi a porty.</p>

```bash
less /usr/share/doc/apache2/README.multiple-insances
/usr/share/doc/apache2/examples/setup-instance oldphp
```

<h4>Docker-way</h4>
<p>Využití kontejnerizace (Docker) pro spouštění více nezávislých instancí Apache, s NginX nebo jiným load balancerem před nimi pro rozhazování zátěže.</p>

<h2> 4. Příklad k procvičení</h2>

<h3>Zadání</h3>

<ol>
    <li>Nasaďte dva vhosty Apache (web01 a web02) s jednoduchým <code>index.html</code> souborem, který bude obsahovat název vhostu.</li>
    <li>
        Nastavte <code>web01</code> na IP adrese <code>192.168.200.&lt;vase_ip&gt;</code> na portu <code>8888</code>.
    </li>
    <li>
        Nastavte <code>web02</code> na IP adrese <code>192.168.100.&lt;vase_ip&gt;</code> na portu <code>8080</code>.
    </li>
    <li>
        Vytvořte load balancer pomocí NginX na IP adrese <code>192.168.253.&lt;vase_ip&gt;</code> na portu <code>8888</code>. Nakonfigurujte ho tak, aby:
        <ul>
            <li>Požadavky na <code>/img</code> byly směřovány pouze na <code>web01</code>.</li>
            <li>Všechny ostatní požadavky byly balancovány mezi <code>web01</code> a <code>web02</code>.</li>
        </ul>
    </li>
    <li>
        Vytvořte load balancer pomocí Pound na IP adrese <code>192.168.253.&lt;vase_ip&gt;</code> na portu <code>8088</code>. Konfigurujte ho tak, aby:
        <ul>
            <li>Všechny požadavky byly balancovány mezi <code>web01</code> a <code>web02</code>.</li>
            <li>Otestujte funkčnost a následně simulujte výpadek <code>web02</code>.</li>
        </ul>
    </li>
</ol>

<h3>Ověření</h3>
<p>Manuální kontrola dostupnosti a chování služeb přes prohlížeč a pomocí nástrojů jako <code>curl</code> nebo <code>telnet</code>. Sledujte logy load balancerů a backend serverů.</p>

<h2> 5. Doplňující materiály</h2>
<p><em>Žádné dodatečné doplňující materiály nejsou k dispozici v původních souborech.</em></p>
