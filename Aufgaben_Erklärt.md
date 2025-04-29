# 📘 Dokumentation – Modul 169: Services mit Containern (Docker)

Diese Dokumentation enthält die vollständige und kommentierte Lösung aller Praxisaufgaben aus dem Lehrmittel *Modul 169 – Docker: Services mit Containern bereitstellen*.

---

## 🔹 1. Docker Grundlagen

### 1.1 Container ausführen – Hello-World

```bash
docker run hello-world
```
> Führt einen Testcontainer aus, um zu prüfen, ob Docker korrekt installiert ist. Das Image wird von Docker Hub geladen und zeigt eine Begrüßungsnachricht.

```bash
docker ps -a
```
> Zeigt alle Container an – auch bereits beendete.

```bash
docker rm <container_id>
```
> Entfernt den Container mit der angegebenen ID.

```bash
docker image rm hello-world
```
> Entfernt das heruntergeladene Image.

---

### 1.2 Interaktive Container (z. B. Ubuntu)

```bash
docker run -it ubuntu
```
> Startet ein interaktives Ubuntu-Terminal im Container. `-i` für Eingabe, `-t` für Terminal.

```bash
docker pull ubuntu
```
> Aktualisiert das Ubuntu-Image lokal.

```bash
docker ps -a
```
> Zeigt gestoppte und laufende Container an.

```bash
docker start -ai <container_id>
```
> Startet einen gestoppten Container interaktiv erneut.

```bash
docker run -it --name mein-container ubuntu
```
> Erstellt einen Container mit dem Namen `mein-container`.

> ⚠️ Tools wie `ping` oder `ip addr` funktionieren nicht, da sie im Minimal-Image fehlen. Man kann sie über `apt install` nachinstallieren.

---

### 1.3 Portweiterleitungen – getting-started

```bash
docker run -d -p 8080:80 docker/getting-started
```
> Startet das Docker-Tutorial-Webinterface im Hintergrund und macht es unter http://localhost:8080 erreichbar.

```bash
docker stop <container_id>
docker rm <container_id>
```
> Stoppt und entfernt den Container wieder.

---

## 🔹 1.4 Datenspeicherung in Volumes

### 1.4.1 Benannte Volumes

```bash
docker run -d --name mariadb-container \
  -e MYSQL_ROOT_PASSWORD=geheim \
  -v mariadb-volume:/var/lib/mysql \
  mariadb
```
> Startet MariaDB mit einem benannten Volume, das die Datenbankdaten speichert.

```bash
docker volume inspect mariadb-volume
```
> Zeigt Details zum Volume an.

---

### 1.4.2 Lokale Verzeichnisse

```bash
mkdir -p ~/varlibmysql
```
> Erstellt ein lokales Verzeichnis.

```bash
docker run -d --name mariadb-own-dir \
  -e MYSQL_ROOT_PASSWORD=geheim \
  -v ~/varlibmysql:/var/lib/mysql \
  mariadb
```
> Verwendet das lokale Verzeichnis als Speicherort für Datenbanken.

---

## 🔹 1.5 Kommunikation zwischen Containern

```bash
docker network create mein-netzwerk
```
> Erstellt ein benutzerdefiniertes Netzwerk für Container-Kommunikation.

```bash
docker run -d --name db --network mein-netzwerk \
  -e MYSQL_ROOT_PASSWORD=geheim mariadb
```

```bash
docker run -d --name phpmyadmin --network mein-netzwerk \
  -e PMA_HOST=db -p 8081:80 phpmyadmin/phpmyadmin
```

```bash
docker run -d --name wordpress --network mein-netzwerk \
  -e WORDPRESS_DB_HOST=db \
  -e WORDPRESS_DB_PASSWORD=geheim \
  -p 8080:80 wordpress
```
> Alle Container kommunizieren über ein internes Docker-Netzwerk.

---

## 🔹 1.6 Docker administrieren

```bash
docker system df
```
> Zeigt Speicherverbrauch durch Container, Images und Volumes an.

```bash
docker container prune
docker image prune
docker volume prune
docker system prune
```
> Entfernt ungenutzte Ressourcen.

```bash
docker info
```
> Zeigt Gesamtübersicht zur Docker-Installation.

---

## 🔹 2. Eigene Docker-Images (Dockerfiles)

### 2.2 Beispiel Dockerfile – Ubuntu + joe

```Dockerfile
FROM ubuntu:20.04
LABEL maintainer="dein.name@example.com"
RUN apt-get update && \
    apt-get install -y joe && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
CMD ["/bin/bash"]
```
> Erstellt ein neues Image, das auf Ubuntu basiert und den Editor `joe` enthält.

```bash
docker build -t ubuntu-joe .
docker run -it ubuntu-joe
```

---

### 2.3 Mini-Projekt: Eigener Webserver mit NGINX

Projektstruktur:
```text
webserver/
├── Dockerfile
├── html/
│   └── index.html
├── logs/
```

Dockerfile:
```Dockerfile
FROM nginx:alpine
COPY ./html /usr/share/nginx/html
```

```bash
docker build -t mein-webserver .
docker run -d --name webserver \
  -p 8080:80 \
  -v $(pwd)/html:/usr/share/nginx/html \
  -v $(pwd)/logs:/var/log/nginx \
  mein-webserver
```
> Startet einen eigenen Webserver, die Webseite und Logs liegen lokal.

---

## 🔹 3. Docker Compose

```yaml
version: '3.1'

services:
  db:
    image: mariadb
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: geheim

  wordpress:
    image: wordpress
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_PASSWORD: geheim
```

```bash
docker compose up -d
docker compose down
```
> Verwaltet mehrere Container gleichzeitig in einer Konfiguration.

---

## 🔹 4. Geheimnisse mit Docker (Secrets)

```bash
docker swarm init
echo "geheim123" | docker secret create mariadb_root_pw -
```

```yaml
version: "3.7"

services:
  mariadb:
    image: mariadb
    secrets:
      - mariadb_root_pw
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/mariadb_root_pw

secrets:
  mariadb_root_pw:
    external: true
```

```bash
docker stack deploy -c docker-compose.yml geheimnis-projekt
```
> Nutzt Docker Secrets statt Umgebungsvariablen – sicherer Umgang mit Passwörtern.

---

✅ **Alle Aufgaben aus dem Lehrmittel wurden vollständig und mit Erklärungen gelöst!**
