## Установка и запуск

### Предварительные требования

- Docker и Docker Compose
- Git
- Минимум 2GB свободной оперативной памяти

### Локальный запуск

1. **Клонируйте репозиторий:**

```bash
git clone https://github.com/your-username/kittygram-final-ad.git
cd kittygram-final-ad
```

2. **Создайте файл `.env` в корне проекта:**

```bash
cp .env.example .env
nano .env
```

3. **Заполните переменные окружения в `.env`:**

```env
# Настройки PostgreSQL
POSTGRES_DB=kittygram                    # Название базы данных
POSTGRES_USER=kittygram_user             # Имя пользователя БД
POSTGRES_PASSWORD=your_secure_password   # Пароль для БД (смените на свой!)
DB_HOST=db                               # Хост БД (для Docker оставьте 'db')
DB_PORT=5432                             # Порт PostgreSQL

# Настройки Django
SECRET_KEY=your-secret-key-here          # Секретный ключ Django (сгенерируйте новый!)
ALLOWED_HOSTS=localhost,127.0.0.1,your-domain.com  # Разрешённые хосты
DEBUG=False                              # Режим отладки (False для продакшн)
```

**Важно:**
- `SECRET_KEY` — сгенерируйте уникальный ключ:
  ```bash
  python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'
  ```
- `POSTGRES_PASSWORD` — используйте надёжный пароль
- `DEBUG=False` для production, `DEBUG=True` только для разработки
- `ALLOWED_HOSTS` — укажите ваш домен или IP-адрес сервера

4. **Запустите проект:**

```bash
# Соберите и запустите контейнеры
docker compose up -d --build

# Выполните миграции
docker compose exec backend python manage.py migrate

# Соберите статику
docker compose exec backend python manage.py collectstatic --no-input

# Создайте суперпользователя (опционально)
docker compose exec backend python manage.py createsuperuser
```

5. **Откройте приложение:**

- Фронтенд: http://localhost:9000
- API: http://localhost:9000/api/
- Админ-панель: http://localhost:9000/admin/

### Остановка проекта

```bash
# Остановить контейнеры
docker compose down

# Остановить и удалить volumes (далит все данные!)
docker compose down -v
```

## Production деплой

### На удалённом сервере

1. **Подготовьте сервер:**

```bash
# Обновите систему
sudo apt update && sudo apt upgrade -y

# Установите Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Установите Docker Compose
sudo apt install docker-compose-plugin
```

2. **Скопируйте файлы на сервер:**

```bash
# На локальной машине
scp docker-compose.production.yml user@your-server:/home/user/kittygram/
scp .env user@your-server:/home/user/kittygram/
```

3. **Запустите на сервере:**

```bash
cd /home/user/kittygram
docker compose -f docker-compose.production.yml up -d
docker compose -f docker-compose.production.yml exec backend python manage.py migrate
docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic --no-input
```

### CI/CD с GitHub Actions

Проект имеет два варианта workflow:

#### Вариант 1: Без автоматического деплоя (main.yml)

Используется для разработки. При каждом push:
- Запускаются тесты на Python 3.10, 3.11, 3.12
- Проверяется форматирование кода (ruff)
- Выполняется линтинг
- При push в `main`/`master` образы публикуются на Docker Hub
- Отправляется уведомление в Telegram

**Настройте секреты в GitHub:**

`Settings → Secrets and variables → Actions → New repository secret`

| Секрет | Описание |
|--------|----------|
| `DOCKER_USERNAME` | Логин на Docker Hub |
| `DOCKER_PASSWORD` | Пароль или токен Docker Hub |
| `TELEGRAM_TO` | ID вашего Telegram аккаунта |
| `TELEGRAM_TOKEN` | Токен Telegram бота для уведомлений |

#### Вариант 2: С автоматическим деплоем (main.yml.production)

Используется для production. Дополнительно к тестам выполняет деплой на сервер.

**Дополнительные секреты для production:**

| Секрет | Описание |
|--------|----------|
| `HOST` | IP-адрес вашего сервера |
| `USER` | Имя пользователя на сервере |
| `SSH_KEY` | Приватный SSH ключ (`cat ~/.ssh/id_ed25519`) |
| `SSH_PASSPHRASE` | Пароль от SSH ключа (если задавали) |

**Переключение между вариантами:**

```bash
# Для разработки (без деплоя)
cp .github/workflows/main.yml.backup .github/workflows/main.yml

# Для production (с деплоем)
cp .github/workflows/main.yml.production .github/workflows/main.yml
git add .github/workflows/main.yml
git commit -m "Enable production deployment"
git push
```

После настройки секретов каждый push в ветку `main` будет автоматически:
1. Запускать тесты на нескольких версиях Python
2. Форматировать и проверять код
3. Собирать Docker образы
4. Публиковать образы на Docker Hub
5. **(Production)** Деплоить на сервер
6. Отправлять уведомление в Telegram

## Структура проекта

```
kittygram-final-ad/
├── backend/                    # Django приложение
│   ├── cats/                   # Приложение для работы с котиками
│   ├── kittygram_backend/      # Настройки проекта
│   ├── Dockerfile              # Образ для backend
│   ├── manage.py
│   └── requirements.txt        # Python зависимости
├── frontend/                   # React приложение
│   ├── public/
│   ├── src/
│   ├── Dockerfile              # Образ для frontend
│   └── package.json            # Node.js зависимости
├── nginx/                      # Конфигурация Nginx
│   ├── nginx.conf              # Настройки прокси-сервера
│   └── Dockerfile              # Образ для Nginx
├── .github/
│   └── workflows/
│       └── main.yml            # CI/CD конфигурация
├── docker-compose.yml          # Локальная разработка
├── docker-compose.production.yml  # Production конфигурация
├── .env.example                # Пример переменных окружения
├── .pre-commit-config.yaml     # Pre-commit hooks
└── README.md                   # Этот файл
```

## API Endpoints

### Аутентификация

```
POST   /api/auth/token/login/    # Получить токен
POST   /api/auth/token/logout/   # Выйти
POST   /api/auth/users/          # Регистрация
GET    /api/auth/users/me/       # Текущий пользователь
```

### Котики

```
GET    /api/cats/                # Список всех котиков
POST   /api/cats/                # Создать котика
GET    /api/cats/{id}/           # Получить котика
PUT    /api/cats/{id}/           # Обновить котика
PATCH  /api/cats/{id}/           # Частично обновить
DELETE /api/cats/{id}/           # Удалить котика
```

### Достижения

```
GET    /api/achievements/        # Список достижений
POST   /api/achievements/        # Создать достижение
```

## Тестирование

```bash
# Backend тесты
docker compose exec backend python manage.py test

# Frontend тесты
docker compose exec frontend npm run test

# Линтинг
docker compose exec backend ruff check backend/
docker compose exec backend ruff format --check backend/

# Pre-commit hooks (локально)
pre-commit run --all-files
```

## Переменные окружения

### Обязательные переменные

| Переменная | Описание | Пример |
|------------|----------|--------|
| `POSTGRES_DB` | Название БД | `kittygram` |
| `POSTGRES_USER` | Пользователь БД | `kittygram_user` |
| `POSTGRES_PASSWORD` | Пароль БД | `secure_password_123` |
| `DB_HOST` | Хост БД | `db` (Docker) / `localhost` |
| `DB_PORT` | Порт БД | `5432` |
| `SECRET_KEY` | Django секретный ключ | `your-secret-key` |
| `ALLOWED_HOSTS` | Разрешённые хосты | `localhost,127.0.0.1` |

### Опциональные переменные

| Переменная | Описание | По умолчанию |
|------------|----------|--------------|
| `DEBUG` | Режим отладки | `False` |

## Разработка

### Настройка окружения разработчика

```bash
# Создайте виртуальное окружение
python -m venv venv
source venv/bin/activate  # Linux/Mac
# или
venv\Scripts\activate     # Windows

# Установите зависимости
pip install -r backend/requirements.txt

# Установите pre-commit hooks
pip install pre-commit
pre-commit install
```

### Внесение изменений

1. Создайте новую ветку: `git checkout -b feature/your-feature`
2. Внесите изменения
3. Отформатируйте код: `ruff format backend/`
4. Проверьте линтером: `ruff check backend/`
5. Закоммитьте: `git commit -m "Add feature"`
6. Отправьте: `git push origin feature/your-feature`
7. Создайте Pull Request


## Автор

**Ваше Имя**
- GitHub: [@1Oblivious1](https://github.com/1Oblivious1)
- Email: vitalikmerzlyackov@yandex.ru | merzlyakov_vitaliy_05@mail.ru

## Благодарности

- Yandex Practicum за образовательную программу
- Сообщество Django и React за отличную документацию

---

Если проект был полезен, поставьте звезду на GitHub!
