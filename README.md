# HTTPS Domain Setup

Этот репозиторий содержит пошаговое руководство по привязке домена к серверу и настройке HTTPS. Здесь вы найдете инструкции по установке SSL-сертификата, настройке веб-сервера (Nginx/Apache) и автоматическому продлению сертификатов с Let’s Encrypt.

## 1. Подготовка сервера
Обновите систему:
```sh
sudo apt update && sudo apt upgrade -y
```

Установите необходимые утилиты:
```sh
sudo apt install nginx certbot python3-certbot-nginx -y
```

## 2. Создание конфигурации
Создайте конфигурационный файл:
```sh
sudo nano /etc/nginx/sites-available/<domain-name>
```
Добавьте следующий код:
```nginx
server {
    listen 80;
    server_name <domain-name>;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Активируйте конфигурацию и перезапустите Nginx:
```sh
sudo ln -s /etc/nginx/sites-available/<domain-name> /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```

## 3. Подключение HTTPS (Let's Encrypt)

Запустите Certbot для получения SSL-сертификата:
```sh
sudo certbot --nginx -d <domain-name>
```

Настройте автоматическое обновление сертификата:
```sh
sudo certbot renew --dry-run
```

Проверьте наличие SSL-сертификата:
```sh
sudo certbot certificates
```

## 4. Подключение SSL

Редактируем конфигурацию:
```sh
sudo nano /etc/nginx/sites-available/<domain-name>
```

Добавляем поддержку HTTPS:
```nginx
server {
    listen 80;
    server_name <domain-name>;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name <domain-name>;
    
    ssl_certificate /etc/letsencrypt/live/<domain-name>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain-name>/privkey.pem;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Перезапустите Nginx:
```sh
sudo systemctl restart nginx
```

Проверьте, слушает ли Nginx 443 порт:
```sh
sudo ss -tlnp | grep nginx
```

## 5. Запуск Backend
```sh
python main.py
```

