# SSL Сертификаты

Для работы HTTPS необходимо разместить SSL сертификаты в этой директории:

- `cert.pem` - SSL сертификат
- `key.pem` - приватный ключ

## Получение сертификатов

### Вариант 1: Let's Encrypt (рекомендуется для production)

```bash
# Установите certbot
sudo apt-get update
sudo apt-get install certbot

# Получите сертификат
sudo certbot certonly --standalone -d your-domain.com

# Скопируйте сертификаты
sudo cp /etc/letsencrypt/live/your-domain.com/fullchain.pem ./ssl/cert.pem
sudo cp /etc/letsencrypt/live/your-domain.com/privkey.pem ./ssl/key.pem
sudo chmod 644 ./ssl/cert.pem
sudo chmod 600 ./ssl/key.pem
```

### Вариант 2: Самоподписанный сертификат (для разработки)

```bash
# Создайте самоподписанный сертификат
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout ssl/key.pem \
  -out ssl/cert.pem \
  -subj "/C=RU/ST=State/L=City/O=Organization/CN=localhost"
```

### Вариант 3: Использование существующих сертификатов

Просто скопируйте ваши сертификаты в эту директорию с именами `cert.pem` и `key.pem`.

