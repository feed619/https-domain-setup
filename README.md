# https-domain-setup
Этот репозиторий содержит пошаговое руководство по привязке домена к серверу и настройке HTTPS. Здесь вы найдете инструкции по установке SSL-сертификата, настройке веб-сервера (Nginx/Apache) и автоматическому продлению сертификатов с Let’s Encrypt.

1. Подготовка сервера
   - Обнови систему:
```console
sudo apt update && sudo apt upgrade -y
```
2. Установи нужные утилиты:
```console
sudo apt install nginx certbot python3-certbot-nginx -y
```
3. Создайте конфиг
```console
sudo nano /etc/nginx/sites-available/<domain-name>
```
Добавь следующее:
```console
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
4. Активируй конфиг:
```console
sudo ln -s /etc/nginx/sites-available/<domain-name> /etc/nginx/sites-enabled/
sudo systemctl restart nginx
```
5. Подключение HTTPS (Let's Encrypt)
```console
sudo certbot --nginx -d <domain-name>
```
6. Настрой автоматическое обновление сертификата:
```console
sudo certbot renew --dry-run
```
7. Проверить наличие SSL-сертификата
```console
sudo certbot certificates
```
-Подключение ssl
8. Изменяем конфиг
```console
sudo nano /etc/nginx/sites-available/backend
```
```console
server {
    listen 80;
    server_name <domain-name>;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name <domain-name>;
         # сюда нужно вставить пудж до ключей fullchain.pem и privkey.pem из команды sudo certbot certificates
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
9. Перезапускаем Nginx:
```console
sudo systemctl restart nginx
```
10. Проверяем слушает ли Nginx 443 порт:
```console
sudo ss -tlnp | grep nginx
```
11. Запускаем Backend
```console
python main.py
```
