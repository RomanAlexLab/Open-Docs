---
title: Установка webmin
---

## **Установка webmin**

После установки панель будет доступна по адресу:

```
https://<IP-адрес_сервера>:10000
```

## **Установка панели**

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

runcmd:
  # Обновление индексов пакетов
  - apt-get update -y
  
  # Установка python-is-python3
  - apt-get install -y python-is-python3
  
  # Установка Webmin
  - curl -o /tmp/webmin-setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repos.sh
  - chmod +x /tmp/webmin-setup-repos.sh
  - /tmp/webmin-setup-repos.sh --force
  - apt-get update
  - apt-get install --install-recommends webmin -y
  
# Настройка времени и часового пояса
timezone: Europe/Moscow 
```

## **Webmin + Docker**

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

runcmd:
  # Обновление индексов пакетов
  - apt-get update -y

  # Установка python-is-python3
  - apt-get install -y python-is-python3
  
  # Установка Webmin
  - curl -o /tmp/webmin-setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repos.sh
  - chmod +x /tmp/webmin-setup-repos.sh
  - /tmp/webmin-setup-repos.sh --force
  - apt-get update
  - apt-get install --install-recommends webmin -y

  # Установка модуля Docker для Webmin
  - curl -L "https://github.com/dave-lang/webmin-docker/releases/latest/download/docker.wbm.gz" -o /tmp/docker.wbm.gz
  - gunzip -c /tmp/docker.wbm.gz > /usr/share/webmin/docker.wbm
  - /usr/share/webmin/install-module.pl /usr/share/webmin/docker.wbm
  - rm /tmp/docker.wbm.gz

  # Установка Docker
  # 1. Установка дополнительных пакетов
  - apt-get install -y curl software-properties-common ca-certificates apt-transport-https

  # 2. Импорт GPG-ключа Docker
  - mkdir -p /etc/apt/keyrings
  - wget -O- https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor | tee /etc/apt/keyrings/docker.gpg > /dev/null

  # 3. Добавление репозитория Docker
  - echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

  # 4. Обновление индексов пакетов после добавления репозитория
  - apt-get update -y

  # 5. Установка Docker CE
  - apt-get install -y docker-ce

  # 6. Проверка статуса Docker
  - systemctl start docker
  - systemctl enable docker

  # Установка Docker Compose
  # 1. Скачивание последней версии Docker Compose
  - curl -L "https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

  # 2. Установка прав на исполнимость
  - chmod +x /usr/local/bin/docker-compose
  
# Настройка времени и часового пояса
timezone: Europe/Moscow
```

## **Webmin + Nginx**

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

runcmd:
  # Обновление индексов пакетов
  - apt-get update -y

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

## **Webmin + Nginx + Docker**

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

runcmd:
  # Обновление индексов пакетов
  - apt-get update -y

  # Установка python-is-python3
  - apt-get install -y python-is-python3
  
  # Установка Webmin
  - curl -o /tmp/webmin-setup-repos.sh https://raw.githubusercontent.com/webmin/webmin/master/webmin-setup-repos.sh
  - chmod +x /tmp/webmin-setup-repos.sh
  - /tmp/webmin-setup-repos.sh --force
  - apt-get update
  - apt-get install --install-recommends webmin -y

  # Установка модуля Docker для Webmin
  - curl -L "https://github.com/dave-lang/webmin-docker/releases/latest/download/docker.wbm.gz" -o /tmp/docker.wbm.gz
  - gunzip -c /tmp/docker.wbm.gz > /usr/share/webmin/docker.wbm
  - /usr/share/webmin/install-module.pl /usr/share/webmin/docker.wbm
  - rm /tmp/docker.wbm.gz

  # Установка плагина Nginx для Webmin
  - curl -L "https://www.justindhoffman.com/sites/justindhoffman.com/files/nginx-0.10.wbm_.gz" -o /tmp/nginx.wbm.gz
  - gunzip -c /tmp/nginx.wbm.gz > /usr/share/webmin/nginx.wbm
  - /usr/share/webmin/install-module.pl /usr/share/webmin/nginx.wbm
  - rm /tmp/nginx.wbm.gz

  # Установка Docker
  # 1. Установка дополнительных пакетов
  - apt-get install -y curl software-properties-common ca-certificates apt-transport-https

  # 2. Импорт GPG-ключа Docker
  - mkdir -p /etc/apt/keyrings
  - wget -O- https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor | tee /etc/apt/keyrings/docker.gpg > /dev/null

  # 3. Добавление репозитория Docker
  - echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

  # 4. Обновление индексов пакетов после добавления репозитория
  - apt-get update -y

  # 5. Установка Docker CE
  - apt-get install -y docker-ce

  # 6. Проверка статуса Docker
  - systemctl start docker
  - systemctl enable docker

  # Установка Docker Compose
  # 1. Скачивание последней версии Docker Compose
  - curl -L "https://github.com/docker/compose/releases/download/v2.12.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

  # 2. Установка прав на исполнимость
  - chmod +x /usr/local/bin/docker-compose

# Настройка времени и часового пояса
timezone: Europe/Moscow
```