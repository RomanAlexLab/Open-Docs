---
title: Установка metabase
---

## **Установка** webmin + nginx + openjdk-11

Ниже представлен код, который необходимо вставить в поле Cloud-init при заказе сервера у облачного провайдера Timeweb Cloud. Операционная система: Ubuntu 22.04.

```
#cloud-config
package_update: true
package_upgrade: true
packages:
  - wget
  - apt-transport-https
  - software-properties-common
  - curl
  - gnupg
  - libhtml-parser-perl
  - nginx
  - git
  - bridge-utils
  - make
  - build-essential
  - libssl-dev
  - zlib1g-dev
  - libbz2-dev
  - libreadline-dev
  - libsqlite3-dev
  - llvm
  - libncurses5-dev
  - libncursesw5-dev
  - xz-utils
  - tk-dev
  - liblzma-dev
  - libffi-dev
  - python3-pip
  - python3.10-venv
  - openjdk-11-jdk

runcmd:
  # Обновление индексов пакетов
  - apt-get update -y

  # Установка обновлений
  - apt-get upgrade -y

  # Установка python-is-python3
  - apt-get install -y python-is-python3
  
  # Установка Webmin
  - curl -o /tmp/webmin-setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repos.sh
  - chmod +x /tmp/webmin-setup-repos.sh
  - /tmp/webmin-setup-repos.sh --force
  - apt-get update
  - apt-get install --install-recommends webmin -y

  # Установка плагина Nginx для Webmin
  - curl -L "https://www.justindhoffman.com/sites/justindhoffman.com/files/nginx-0.10.wbm_.gz" -o /tmp/nginx.wbm.gz
  - gunzip -c /tmp/nginx.wbm.gz > /usr/share/webmin/nginx.wbm
  - /usr/share/webmin/install-module.pl /usr/share/webmin/nginx.wbm
  - rm /tmp/nginx.wbm.gz

# Настройка времени и часового пояса
timezone: Europe/Moscow
```

## **Проверка установки OpenJDK:**

```
java -version
```

## Настройка сервера

-  Настроить язык панели

-  Создать пользователя для панели webmin

-  Создать суперпользователя, которого добавить в группу sudo

-  Добавить сертификаты для входа по SSH

-  Запретить вход для root и вход по паролям

## **Создание системного пользователя metabase**

```
sudo adduser --system --no-create-home --group --disabled-login metabase
```

## **Создание директории metabase**

```
sudo mkdir -p /var/www/metabase
```

## **Установка metabase**

```
sudo wget https://downloads.metabase.com/v0.51.4/metabase.jar -O /var/www/metabase/metabase.jar
sudo chmod +x /var/www/metabase/metabase.jar
sudo chown -R metabase:metabase /var/www/metabase
sudo chmod -R 750 /var/www/metabase
```

## Настройка службы metabase

```
sudo nano /etc/systemd/system/metabase.service
```

```
[Unit]
Description=Metabase Server
After=syslog.target
After=network.target

[Service]
ExecStart=java -jar /var/www/metabase/metabase.jar
WorkingDirectory=/var/www/metabase
User=metabase
Environment=MB_DB_TYPE=postgres
Environment=MB_DB_DBNAME=metabaseappdb
Environment=MB_DB_PORT=5432
Environment=MB_DB_USER=metabaseuser
Environment=MB_DB_PASS=password
Environment=MB_DB_HOST=localhost
Restart=always
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=metabase

[Install]
WantedBy=multi-user.target
```

## **Запуск и остановка службы**

```
sudo systemctl daemon-reload
sudo systemctl start metabase
sudo systemctl enable metabase
```

```
sudo systemctl stop metabase
sudo systemctl disable metabase
```

## **Проверка статуса службы**

```
sudo systemctl status metabase
```

## **Перезапуск службы**

```
sudo systemctl restart metabase
```

## **Логи службы**

```
journalctl -u metabase
```

## **NGINX + HTTPS**

Выпуск сертификатов

```
sudo systemctl stop nginx
sudo certbot certonly --standalone -d domen.ru -d www.domen.ru
sudo systemctl start nginx
sudo systemctl enable nginx
```

Сертификаты будут доступны по пути:

```
/etc/letsencrypt/live/domen.ru/fullchain.pem
/etc/letsencrypt/live/domen.ru/privkey.pem
```

Автоматическое продление сертификатов

```
sudo crontab -e
```

```
0 0 * * * certbot renew --pre-hook "systemctl stop nginx" --post-hook "systemctl start nginx"
```

Конфигурационный файл NGINX

```
server {
    listen 80;
    server_name domen.ru www.domen.ru;

    # Перенаправление на HTTPS
    return 301 https://$host$request_uri;
}

# Настройки для HTTPS
server {
    listen 443 ssl;
    server_name domen.ru www.domen.ru;

    ssl_certificate /etc/letsencrypt/live/domen.ru/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/domen.ru/privkey.pem;

    # Протоколы SSL
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:ECDHE-RSA-AES128-GCM-SHA256';
    ssl_prefer_server_ciphers on;

    # Логирование
    access_log /var/log/nginx/ii_otchety_ssl_access.log;
    error_log /var/log/nginx/ii_otchety_ssl_error.log;

    location / {
        proxy_pass http://localhost:3000;  # Укажите локальный IP и порт
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

	# Защита от доступа к скрытым файлам
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
}
```


