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
