# Installare docker su AWS

```bash
sudo apt update
sudo apt upgrade

wget https://get.docker.com -O ./get-docker.sh 
sudo sh ./get-docker.sh
sudo groupadd docker
sudo usermod -aG docker $USER
```

Eserguire il reboot della macchina:

```bash
sudo reboot 0
```

## Controlliamo che funzioni

```bash
docker run hello-world
```

## Directory e file necessari

Creare una directory per il progetto e le sottodirectory **mariadb** e **src**<br>
creare anche all'interno della directory del progetto i file: **docker-compose.yml** **default.conf** **./src/index.php**


## docker-compose.yml

```yaml
version: '3.9'

services:
  db:
    image: mariadb:latest
    container_name: db
    restart: unless-stopped
    environment:
      MARIADB_ROOT_PASSWORD: **PASSWORD**
    volumes:
      - ./mariadb:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin
    restart: unless-stopped
    environment:
      PMA_HOST: db
      MYSQL_ROOT_PASSWORD: **PASSWORD**
    ports:
      - 8080:8080

  web:
    image: nginx:latest
    ports:
      - 80:80
    volumes:
      - ./src:/var/www/html
      - ./default.conf:/etc/nginx/conf.d/default.conf

  php-fpm:
    image: php:8-fpm
    volumes:
      - ./src:/var/www/html
```

## default.conf

```nginx
server {
    index index.php index.html;
    server_name miosito.local;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    root /var/www/html;
    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass php-fpm:9000;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }
}
```

## index.php

```php
<?php
echo phpinfo();
?>
```

## Avviare il tutto

```bash
docker-compose up
```

**OPPURE** per avviare come demone

```bash
docker-compose up -d
```