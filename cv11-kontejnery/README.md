# Kontejnery (Docker, Docker Compose, Traefik)

## 1. Úvod
**Cíl:** Seznámení s kontejnerizačními technologiemi Docker a Docker Compose pro izolaci a správu aplikací. Cvičení zahrnuje práci s Docker obrazy, spouštění kontejnerů, tvorbu vlastních Dockerfile a orchestraci vícekontejnerových aplikací pomocí Docker Compose. Dále se seznámíme s Traefikem jako moderním reverse proxy a load balancerem pro kontejnerizované služby.

## 2. Použité nástroje a reference
*   **Docker:** [Homepage](https://www.docker.com/), [Dokumentace](https://docs.docker.com/)
*   **Docker Compose:** [Homepage](https://docs.docker.com/compose/), [Dokumentace](https://docs.docker.com/compose/overview/)
*   **Traefik:** [Homepage](https://traefik.io/traefik/), [Dokumentace](https://doc.traefik.io/traefik/)

## 3. Poznámky ke cvičení (How-To)

### Docker

#### Instalace
Postup instalace Docker Engine a Docker Compose naleznete v oficiální dokumentaci:
*   [Instalace Docker Engine](https://docs.docker.com/engine/install/debian/)
*   [Instalace Docker Compose](https://docs.docker.com/compose/install/)

#### Základní příkazy
*   **Spuštění kontejneru:**
    ```bash
    docker run hello-world
    docker run --rm docker/whalesay cowsay "It works"
    docker run -it --rm --name debian debian:buster-slim /bin/bash
    ```
    *   `--rm`: Automaticky odstraní kontejner po ukončení.
    *   `-it`: Interaktivní režim s TTY.
    *   `--name`: Pojmenování kontejneru.
*   **Připojení k běžícímu kontejneru:**
    ```bash
    docker exec -it debian /bin/bash
    ```
*   **Správa kontejnerů:**
    ```bash
    docker ps -a       # Seznam všech kontejnerů (běžících i zastavených)
    docker kill debian # Ukončení kontejneru
    docker start debian # Spuštění zastaveného kontejneru
    docker inspect debian # Detailní informace o kontejneru
    docker rm debian   # Odstranění kontejneru
    ```
*   **Spuštění webového serveru s bind mountem:**
    ```bash
    docker run --name apache2 -v "$PWD":/usr/local/apache2/htdocs/ -p 8080:80 -d httpd:2-alpine
    ```
    *   `-v "$PWD":/usr/local/apache2/htdocs/`: Připojí aktuální adresář do adresáře dokumentů Apache uvnitř kontejneru.
    *   `-p 8080:80`: Mapuje port 8080 hostitele na port 80 kontejneru.
    *   `-d`: Spustí kontejner v detached módu (na pozadí).

#### Tvorba a správa vlastních obrazů (Images)
*   **Dockerfile:** Instrukce pro sestavení Docker obrazu.
    ```dockerfile
    FROM debian:buster-slim

    RUN apt update && \
        apt install -y netcat && \
        apt autoremove && \
        apt autoremove -y && \
        apt clean && \
        rm -rf /var/lib/apt/lists

    COPY netcat.sh /usr/local/bin/

    CMD ["/bin/bash", "-lc", "netcat.sh"]
    ```
    *   `FROM`: Základní obraz.
    *   `RUN`: Spuštění příkazu během sestavování obrazu.
    *   `COPY`: Kopírování souborů z hostitele do obrazu.
    *   `CMD`: Výchozí příkaz, který se spustí při startu kontejneru.
*   **Sestavení obrazu:**
    ```bash
    docker build -t path/to/image:tag -f Dockerfile .
    ```
    *   `-t`: Pojmenování a otagování obrazu.
    *   `-f`: Specifikace Dockerfile (pokud není ve výchozím umístění).
    *   `.`: Kontext sestavení (aktuální adresář).
*   **Přihlášení do registru a push:**
    ```bash
    docker login -u user registry.repo.spos
    docker tag path/to/image:tag path/to/image:newtag # Přidání dalšího tagu
    docker push path/to/image:newtag # Nahrání obrazu do registru
    ```
*   **Správa obrazů:**
    ```bash
    docker images          # Seznam stažených obrazů
    docker rmi path/to/image:tag # Odstranění obrazu
    ```

#### Užitečné triky
```bash
docker system prune # Odstraní nepoužívané objekty (kontejnery, obrazy, sítě, volume)
docker ps -a | awk '{print $1}' | xargs docker kill # Zabije všechny běžící kontejnery
docker ps -a | awk '{print $1}' | xargs docker rm   # Odstraní všechny kontejnery
```

### Docker Compose

Docker Compose umožňuje definovat a spouštět vícekontejnerové Docker aplikace. Konfigurace se provádí pomocí souboru `docker-compose.yml`.

#### Příklad docker-compose.yml
```yaml
version: "3"
services:
  app:
    image: path/to/image
    build:
      dockerfile: Dockerfile
      context: .
    volumes:
      - .:/app
    ports:
      - 8080:3000
    environment:
      BSA: false
      SPOS: true
      CONFIG: /app/config.yml
    restart: always
  redis:
    image: redis:latest
```
*   `version`: Verze syntaxe Docker Compose.
*   `services`: Definice jednotlivých služeb (kontejnerů).
*   `image`: Použitý Docker obraz.
*   `build`: Instrukce pro sestavení obrazu (alternativa k `image`).
*   `volumes`: Připojení svazků (volume) nebo bind mountů.
*   `ports`: Mapování portů.
*   `environment`: Proměnné prostředí.
*   `restart`: Restartovací politika kontejneru.

#### Další příklad s Nginx, Redis a MySQL
```yaml
services:
  nginx:
    image: nginx:latest
    volumes:
      - ./:/usr/share/nginx/html
    ports:
      - 8888:80
  redis:
    image: redis
  mysql:
    image: mysql
    volumes:
      - ./mysql-data:/var/lib/mysql
    ports:
      - 3306:3306
```

### Traefik

Traefik je moderní HTTP reverse proxy a load balancer, který se snadno integruje s Dockerem a dalšími orchestrátory.

#### Příklad konfigurace Traefiku (docker-compose.yml)
```yaml
version: '3.3'

services:
  traefik:
    image: traefik
    volumes:
      - ./traefik/traefik.toml:/etc/traefik/traefik.toml
      - ./traefik/conf.d/:/etc/traefik/conf.d/
    ports:
    - target: 80
      protocol: tcp
      published: 80
    - target: 443
      protocol: tcp
      published: 443
    - target: 8080
      protocol: tcp
      published: 8080
```
*   Tento `docker-compose.yml` spouští Traefik a připojuje konfigurační soubory z lokálního adresáře.

#### Hlavní konfigurační soubor Traefiku (traefik.toml)
```toml
################################################################
# Configuration sample for Traefik v2.
################################################################

[global]
  checkNewVersion = true
  sendAnonymousUsage = true

[entryPoints]
  [entryPoints.web]
    address = ":80"

  [entryPoints.websecure]
    address = ":443"

[log]
  level = "DEBUG"

[api]
  insecure = true
  dashboard = true

[ping]

[providers.file]
  directory = "/etc/traefik/conf.d"

[backends]
  [backends.traefik]
    [backends.traefik.servers.server1]
    url = "http://localhost:8080"
    weight = 10

[frontends]
  [frontends.traefikadmin]
  backend = "traefik"
  entrypoints = ["ping"]
    [frontends.traefikadmin.routes.ping]
    rule = "Path:/ping"

[metrics]
  [metrics.prometheus]
```
*   `entryPoints`: Definuje naslouchací porty (HTTP, HTTPS).
*   `api`: Povoluje API a dashboard Traefiku (pro monitoring).
*   `providers.file`: Umožňuje načítat dynamickou konfiguraci ze souborů (např. z `conf.d/`).

#### Dynamická konfigurace Traefiku (www.jindra.spos.toml)
```toml
[http]
  [http.routers]
    [http.routers.www-jindra-spos-static]
      entryPoints = ["web"]
      rule = "Host(`www.jindra.jindra.spos`) && PathPrefix(`/`)"
      service = "www-jindra-spos"

  [http.services]
    [http.services.www-jindra-spos.loadBalancer]
      [[http.services.www-jindra-spos.loadBalancer.servers]]
        url = "http://10.0.1.42:8080/"
      [[http.services.www-jindra-spos.loadBalancer.servers]]
        url = "http://10.0.2.42:8080/"
      [[http.services.www-jindra-spos.loadBalancer.servers]]
        url = "http://10.0.3.42:8080/"

    [http.services.www-jindra-spos]
      [http.services.www-jindra-spos.loadBalancer.sticky.cookie]
        secure = true
        httpOnly = true

    [http.services.www-jindra-spos.loadBalancer.healthCheck]
      path = "/status"
      interval = "10s"
      timeout = "3s"
      port = 8080
      scheme = "http"
```
*   Definuje router pro hosta `www.jindra.spos` a službu `www-jindra-spos` s load balancingem na tři backend servery.
*   Obsahuje nastavení pro sticky sessions a health checks.

## 4. Příklad k procvičení

### Zadání

1.  **Sestavení vlastního obrazu:**
    *   Vytvořte soubor `netcat.sh` s obsahem:
        ```bash
        #!/bin/bash
        echo "Ahoj z dockeru"
        [ -e /bin/nc ] && echo "Mame netcat" || echo "Nemame netcat"
        ```
    *   Sestavte Docker obraz s názvem `my-netcat-app:latest` pomocí souboru `Dockerfile` z části "Tvorba a správa vlastních obrazů".
    *   Spusťte kontejner z tohoto obrazu a ověřte výstup.
2.  **Použití Docker Compose:**
    *   Pomocí souboru `docker-compose2.yml` (Nginx, Redis, MySQL) spusťte všechny služby.
    *   Ověřte, že je Nginx dostupný na portu 8888 a že všechny služby běží (`docker ps`).
3.  **Nastavení Traefiku:**
    *   Pomocí souboru `docker-compose.yml` (obsahující Traefik službu) a konfiguračních souborů v adresáři `traefik/` spusťte Traefik.
    *   Simulujte tři backend servery (např. pomocí Nginx kontejnerů s různým obsahem, běžících na IP adresách `10.0.1.42:8080`, `10.0.2.42:8080`, `10.0.3.42:8080` – tyto IP adresy a porty jsou v `www.jindra.spos.toml`).
    *   Přistupte k `http://www.jindra.spos` (nutno nastavit DNS nebo `/etc/hosts`) a ověřte, že Traefik balancuje požadavky na backendy. Sledujte Traefik dashboard na portu 8080.

### Ověření
*   Kontrola výstupu z `my-netcat-app` kontejneru.
*   Dostupnost Nginxu a stav služeb pomocí `docker ps`.
*   Přístupnost Traefik dashboardu a ověření routování na `www.jindra.spos`.

## 5. Doplňující materiály

### manual.md (kompletní text)
```
# Docker

## Instalace

* https://docs.docker.com/engine/install/debian/
* https://docs.docker.com/compose/install/

## Priklady

```bash
docker run hello-world
docker run --rm docker/whalesay cowsay "It works"
docker run -it --rm  --name debian debian:buster-slim /bin/bash
```

```bash
docker exec -it debian /bin/bash
```

```bash
docker ps -a
docker kill debian
docker start debian
docker inspect debian
docker rm debian
```

```bash
docker run --name apache2 -v "$PWD":/usr/local/apache2/htdocs/ -p 8080:80 -d httpd:2-alpine
```

## Build image

```bash
docker login -u user registry.repo.spos
docker build -t path/to/image:tag -f Dockerfile .
docker tag path/to/image:tag path/to/image:newtag
docker push path/to/image:newtag
```

**Dockerfile**

```dockerfile
FROM debian:buster-slim

RUN apt update && \
    apt install -y netcat && \
    apt autoremove && \
    apt autoremove -y && \
    apt clean && \
    rm -rf /var/lib/apt/lists

COPY netcat.sh /usr/local/bin/

CMD ["/bin/bash", "-lc", "netcat.sh"]
```

**netcat.sh**

```bash
#!/bin/bash

echo "Ahoj z dockeru"
[ -e /bin/nc ] && echo "Mame netcat" || echo "Nemame netcat"

```

```bash
docker images
docker rmi  path/to/image:tag
```

# Triky


```bash
docker system prune
docker ps -a | awk '{print $1}' | xargs docker kill 
docker ps -a | awk '{print $1}' | xargs docker rm
```

### Compose

```
version: "3"
services:
  app:
    image: path/to/image
    build:
      dockerfile: Dockerfile
      context: .
    volumes:
      - .:/app
    ports:
      - 8080:3000
    environment:
      BSA: false
      SPOS: true
      CONFIG: /app/config.yml
    restart: always
  redis:
    image: redis:latest
```
```

### docker-compose.yml
```yaml
version: '3.3'

services:
  traefik:
    image: traefik
    volumes:
      - ./traefik/traefik.toml:/etc/traefik/traefik.toml
      - ./traefik/conf.d/:/etc/traefik/conf.d/
    ports:
    - target: 80
      protocol: tcp
      published: 80
    - target: 443
      protocol: tcp
      published: 443
    - target: 8080
      protocol: tcp
      published: 8080
```

### docker-compose2.yml
```yaml
services:
  nginx:
    image: nginx:latest
    volumes:
      - ./:/usr/share/nginx/html
    ports:
      - 8888:80
  redis:
    image: redis
  mysql:
    image: mysql
    volumes:
      - ./mysql-data:/var/lib/mysql
    ports:
      - 3306:3306
```

### Dockerfile
```dockerfile
FROM debian:stretch

MAINTAINER Ondrej Zalesky <ondrej.zalesky@eman.cz>

ARG APP_NAME
ARG LICENSE_KEY

# Timezone
ENV TZ=Europe/Prague

RUN \
  apt-get update && \
  apt-get install -y \
  wget gnupg apt-transport-https && \
  apt-get autoremove -y && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists 

RUN \
  wget -q https://packages.sury.org/php/apt.gpg -O- | apt-key add - && \
  echo "deb https://packages.sury.org/php/ stretch main" | tee /etc/apt/sources.list.d/php.list

RUN \
  apt-get update && \
  apt-get install -y \
    curl \
    unzip \
    vim \
    git \
    apt-transport-https \
    locales \
    iptables \
    apache2 \
    php5.6 \
    php5.6-mysql \
    php5.6-mcrypt \
    php5.6-mbstring \
    php5.6-gd \
    php5.6-memcache \
    php5.6-zip \
    php5.6-curl \
    php-pear \
    php5.6-apcu \
    php5.6-xml \
    php5.6-pgsql \
    php5.6-cli \
    php5.6-curl \
    php5.6-mcrypt \
    php5.6-sqlite \
    php5.6-intl \
    php5.6-tidy \
    php5.6-imap \
    php5.6-json \
    php5.6-imagick \
    php5.6-common \
    libapache2-mod-php5.6 \
    libapache2-mod-rpaf && \
  a2enmod proxy && \
  a2enmod proxy_http && \
  a2enmod alias && \
  a2enmod dir && \
  a2enmod env && \
  a2enmod mime && \
  a2enmod reqtimeout && \
  a2enmod rewrite && \
  a2enmod status && \
  a2enmod filter && \
  a2enmod deflate && \
  a2enmod setenvif && \
  a2enmod vhost_alias && \
  a2enmod ssl && \
  a2enmod php5.6 && \
  apt-get autoremove -y && \
  apt-get clean && \
  rm -rf /var/lib/apt/lists

# Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
  && dpkg-reconfigure --frontend noninteractive tzdata \
  && echo 'en_US.UTF-8 UTF-8\ncs_CZ.UTF-8 UTF-8' >> /etc/locale.gen \
  && /usr/sbin/locale-gen

RUN ln -sf /dev/stderr /var/log/apache2/error.log

#COPY ./docker/000-default.conf /etc/apache2/sites-enabled/000-default.conf
#COPY ./docker/run.sh /run.sh

#RUN chmod +x /run.sh

#EXPOSE 80

# copy config files
COPY config/sshd_config /etc/ssh/
COPY config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY config/entrypoint.sh /

CMD ["/usr/sbin/apache2ctl", "-D", "FOREGROUND"]
```

### traefik/traefik.toml
```toml
################################################################
#
# Configuration sample for Traefik v2.
#
# For Traefik v1: https://github.com/containous/traefik/blob/v1.7/traefik.sample.toml
#
################################################################

################################################################
# Global configuration
################################################################
[global]
  checkNewVersion = true
  sendAnonymousUsage = true

################################################################
# Entrypoints configuration
################################################################

# Entrypoints definition
#
# Optional
# Default:
[entryPoints]
  [entryPoints.web]
    address = ":80"

  [entryPoints.websecure]
    address = ":443"

################################################################
# Traefik logs configuration
################################################################

# Traefik logs
# Enabled by default and log to stdout
#
# Optional
# Default: "ERROR"
#
[log]

  # Log level
  #
  # Optional
  # Default: "ERROR"
  #
  level = "DEBUG"

  # Sets the filepath for the traefik log. If not specified, stdout will be used.
  # Intermediate directories are created if necessary.
  #
  # Optional
  # Default: os.Stdout
  #
  # filePath = "log/traefik.log"

  # Format is either "json" or "common".
  #
  # Optional
  # Default: "common"
  #
  # format = "json"

################################################################
# Access logs configuration
################################################################

# Enable access logs
# By default it will write to stdout and produce logs in the textual
# Common Log Format (CLF), extended with additional fields.
#
# Optional
#
# [accessLog]

  # Sets the file path for the access log. If not specified, stdout will be used.
  # Intermediate directories are created if necessary.
  #
  # Optional
  # Default: os.Stdout
  #
  # filePath = "/path/to/log/log.txt"

  # Format is either "json" or "common".
  #
  # Optional
  # Default: "common"
  #
  # format = "json"

################################################################
# API and dashboard configuration
################################################################

# Enable API and dashboard
[api]

  # Enable the API in insecure mode
  #
  # Optional
  # Default: true
  #
  insecure = true

  # Enabled Dashboard
  #
  # Optional
  # Default: true
  #
  dashboard = true

################################################################
# Ping configuration
################################################################

# Enable ping
[ping]

  # Name of the related entry point
  #
  # Optional
  # Default: "traefik"
  #
  # entryPoint = "traefik"

################################################################
# Docker configuration backend
################################################################

# Enable Docker configuration backend
#[providers.docker]

  # Docker server endpoint. Can be a tcp or a unix socket endpoint.
  #
  # Required
  # Default: "unix:///var/run/docker.sock"
  #
  # endpoint = "tcp://10.10.10.10:2375"

  # Default host rule.
  #
  # Optional
  # Default: "Host(`{{ normalize .Name }}`)"
  #
  # defaultRule = "Host(`{{ normalize .Name }}.docker.localhost`)"

  # Expose containers by default in traefik
  #
  # Optional
  # Default: true
  #
  # exposedByDefault = false

[providers.file]
  directory = "/etc/traefik/conf.d"

[backends]
  [backends.traefik]
    [backends.traefik.servers.server1]
    url = "http://localhost:8080"
    weight = 10

[frontends]
  [frontends.traefikadmin]
  backend = "traefik"
  entrypoints = ["ping"]
    [frontends.traefikadmin.routes.ping]
    rule = "Path:/ping"

[metrics]
  [metrics.prometheus]
```

### traefik/conf.d/www.jindra.spos.toml
```toml
[http]
  [http.routers]
    [http.routers.www-jindra-spos-static]
      entryPoints = ["web"]
      rule = "Host(`www.jindra.spos`) && PathPrefix(`/`)"
      service = "www-jindra-spos"

  [http.services]
    [http.services.www-jindra-spos.loadBalancer]
      [[http.services.www-jindra-spos.loadBalancer.servers]]
        url = "http://10.0.1.42:8080/"
      [[http.services.www-jindra-spos.loadBalancer.servers]]
        url = "http://10.0.2.42:8080/"
      [[http.services.www-jindra-spos.loadBalancer.servers]]
        url = "http://10.0.3.42:8080/"

    [http.services.www-jindra-spos]
      [http.services.www-jindra-spos.loadBalancer.sticky.cookie]
        secure = true
        httpOnly = true

    [http.services.www-jindra-spos.loadBalancer.healthCheck]
      path = "/status"
      interval = "10s"
      timeout = "3s"
      port = 8080
      scheme = "http"
```