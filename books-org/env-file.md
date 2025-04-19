---
order: 3
title: Конфигурация BookStack
---

## **Конфигурация BookStack**

Перейти в директорию bookstack

```
cd /var/www/bookstack
```

Отредактировать .env

```
sudo nano .env
```

```
# Language
APP_LANG=ru

# Database details
DB_HOST=your-host
DB_DATABASE=your-batabase
DB_USERNAME=your-username-db
DB_PASSWORD=your-password

# S3
STORAGE_TYPE=s3
STORAGE_S3_KEY=your-service-key
STORAGE_S3_SECRET=your-service-secret-key
STORAGE_S3_BUCKET=your-service-bucket-name
STORAGE_S3_ENDPOINT=https://your-service-base-endpoint.com
STORAGE_URL=https://your-service-base-endpoint.com/your-service-bucket-name
```

## **Настройка HTTPS**

```
sudo apt update
sudo apt install certbot python3-certbot-apache
sudo certbot --apache -d domen.ru
```

Автоматическое обновление

```
sudo crontab -e
```

```
0 3 * * * certbot renew —quiet
```


