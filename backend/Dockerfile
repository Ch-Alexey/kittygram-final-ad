FROM python:3.11-slim-bullseye

# Устанавливаем системные зависимости и корректное время

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    gcc \
    libjpeg-dev \
    zlib1g-dev \
    libtiff-dev \
    libfreetype6-dev \
    liblcms2-dev \
    libwebp-dev \
    libharfbuzz-dev \
    libfribidi-dev \
    libxcb1-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Копируем и устанавливаем зависимости Python

COPY requirements.txt /app/
RUN pip install --upgrade pip && pip install -r requirements.txt

# Копируем код проекта

COPY . /app/

# Создаем директории для статики и медиа
RUN mkdir -p /vol/static /vol/media

# Команда запуска сервера Django
# Миграции и сбор статики будут выполняться при запуске контейнера

CMD sh -c "python manage.py migrate && python manage.py collectstatic --noinput && python manage.py runserver 0.0.0.0:8000"
