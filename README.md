# SaveTest - Конфигурации для установки

## О SaveTest

**SaveTest** — это полноценная TMS (Test Management System) система для управления тест-кейсами с интеграцией Git репозиториев. Система автоматически парсит файлы из репозиториев и синхронизирует их с базой данных тест-кейсов.

SaveTest позволяет:
- Управлять тест-кейсами и тест-ранами
- Автоматически синхронизировать тесты из Git репозиториев
- Планировать и выполнять тест-раны
- Получать аналитику и отчеты по проектам
- Работать в команде с управлением пользователями и правами доступа
- Использовать плагинную архитектуру для парсинга различных языков и фреймворков

### Скриншоты интерфейса

![Dashboard](screens/dashboard.png)

![Страница проекта](screens/project_page.png)

> **Примечание:** Дополнительные скриншоты интерфейса доступны в директории `screens/`

## Связанные ссылки

- **Демо доступ**: https://demo.save-test.ru/ (tester/tester123)
- **Репозиторий плагинов**: https://github.com/savelinkQA/savetest-plugins
- **Расширение для VS Code (Open VSX)**: https://open-vsx.org/extension/savetest/savetest-tms-plugin
- **Расширение для VS Code (Marketplace)**: https://marketplace.visualstudio.com/items?itemName=SaveTest.savetest-tms-plugin
- **Подробнее о продукте**: https://save-test.ru
- **Приобрести лицензию**: https://save-test.ru

### Лицензирование

По умолчанию доступна **бесплатная лицензия** со следующими ограничениями:
- **1 проект**
- **1 пользователь**
- **10 тест-ранов**

Для расширения возможностей системы необходимо приобрести лицензию на сайте https://save-test.ru

## О репозитории

В этом репозитории представлены примеры конфигурации Docker Compose для установки SaveTest в различных сценариях использования.

## Файлы конфигурации

### Production (продакшн)

#### docker-compose.production.yml

**Назначение:** Конфигурация для продакшн окружения

**Особенности:**
- Использует готовые образы из container registry
- Поддержка переменных окружения через `${VARIABLE:-default}`
- Оптимизирована для стабильной работы

**Использование:**
```bash
docker-compose up -d
```

---

#### docker-compose.production.no-plugins.yml

**Назначение:** Конфигурация для продакшн окружения **БЕЗ ПЛАГИНОВ**

**Особенности:**
- Использует готовые образы из container registry
- Не включает плагины парсеров
- Поддержка переменных окружения через `${VARIABLE:-default}`
- Меньше потребление ресурсов

**Использование:**
```bash
docker-compose up -d
```

---

### Minimal (минимальная)

#### docker-compose.minimal.yml

**Назначение:** Минимальная конфигурация для быстрого старта

**Особенности:**
- Все параметры жестко заданы (без переменных окружения)
- Минимальная конфигурация
- Подходит для быстрого тестирования и демонстрации
- Не требует настройки .env файла

**Использование:**
```bash
docker-compose up -d
```

---

## Параметры установки

### Требования

- Docker 20.10+
- Docker Compose 2.0+
- Минимум 4 CPU (Рекомендовано 8 CPU)
- Минимум 4GB RAM (Рекомендовано 8GB RAM)
- 10GB свободного места на диске

### Быстрая установка

1. **Выберите конфигурацию:**

Для продакшена:
```bash
cp docker-compose.production.yml docker-compose.yml
```

Для продакшена без плагинов:
```bash
cp docker-compose.production.no-plugins.yml docker-compose.yml
```

Для быстрого старта:
```bash
cp docker-compose.minimal.yml docker-compose.yml
```

2. **Настройте переменные окружения (опционально):**

Создайте файл `.env` в корне проекта:

```bash
# .env
POSTGRES_DB=savetest_db
POSTGRES_USER=savetest_user
POSTGRES_PASSWORD=your-secure-password
SECRET_KEY=your-super-secret-key
DEBUG=False
ALLOWED_ORIGINS=http://localhost:8080
UVICORN_WORKERS=4
PLUGIN_URLS=http://python-parser-plugin:8000,http://gherkin-parser-plugin:8000
```

**⚠️ ВАЖНО:** В продакшене обязательно измените `SECRET_KEY` и `POSTGRES_PASSWORD` на надежные значения!

3. **Запустите систему:**

```bash
docker-compose up -d
```

4. **Проверьте статус:**

```bash
docker-compose ps
```

### Доступ к системе

После запуска система будет доступна:

- **Frontend**: http://localhost:8080

## Настройка домена и SSL

Для работы с доменом и SSL сертификатами необходимо настроить внешний Nginx как reverse proxy.

### Установка Nginx

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install nginx

# CentOS/RHEL
sudo yum install nginx
```

### Конфигурация Nginx

Создайте файл `/etc/nginx/sites-available/savetest` (или `/etc/nginx/conf.d/savetest.conf`):

```nginx
# Map для WebSocket соединений
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

upstream savetest {
    server localhost:8080;
    keepalive 32;
}

# HTTP -> HTTPS редирект
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    
    # Редирект на HTTPS
    return 301 https://$server_name$request_uri;
}

# HTTPS сервер
server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;

    # SSL сертификаты
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    
    # SSL настройки
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;

    # Максимальный размер загружаемого файла
    client_max_body_size 100M;

    # Проксирование всех запросов на 8080
    location / {
        proxy_pass http://savetest;
        proxy_http_version 1.1;
        
        # Заголовки для WebSocket
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        
        # Остальные заголовки
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Увеличенные таймауты для WebSocket
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
        
        # Отключаем буферизацию для WebSocket
        proxy_buffering off;
    }
}
```

### Получение SSL сертификата (Let's Encrypt)

```bash
# Установите certbot
sudo apt-get install certbot python3-certbot-nginx

# Получите сертификат
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Автоматическое обновление (добавится в cron)
sudo certbot renew --dry-run
```

### Активация конфигурации

```bash
# Создайте символическую ссылку (для Ubuntu/Debian)
sudo ln -s /etc/nginx/sites-available/savetest /etc/nginx/sites-enabled/

# Проверьте конфигурацию
sudo nginx -t

# Перезагрузите Nginx
sudo systemctl reload nginx
```

Затем перезапустите контейнер:

```bash
docker-compose restart backend
```

---

## Параметры окружения

Документация по всем переменным окружения, используемым в SaveTest.

### PostgreSQL

| Переменная | Описание | Тип | По умолчанию | Обязательно | Примечания |
|------------|----------|-----|--------------|-------------|------------|
| `POSTGRES_DB` | Имя базы данных PostgreSQL | `string` | `savetest_db` | Нет | |
| `POSTGRES_USER` | Имя пользователя для подключения к PostgreSQL | `string` | `savetest_user` | Нет | |
| `POSTGRES_PASSWORD` | Пароль для подключения к PostgreSQL | `string` | `savetest_password` | Нет | ⚠️ В продакшене обязательно измените на надежный пароль! |
| `POSTGRES_INITDB_ARGS` | Аргументы для инициализации базы данных | `string` | `"--encoding=UTF8 --locale=en_US.UTF-8 --lc-collate=en_US.UTF-8 --lc-ctype=en_US.UTF-8"` | Нет | Обычно не требуется изменять |

### Backend

| Переменная | Описание | Тип | По умолчанию | Обязательно | Примечания |
|------------|----------|-----|--------------|-------------|------------|
| `DATABASE_URL` | URL подключения к базе данных PostgreSQL | `string` | `postgresql://savetest_user:savetest_password@postgres:5432/savetest_db` | Нет | Формат: `postgresql://user:password@host:port/database`. Обычно формируется автоматически из `POSTGRES_USER`, `POSTGRES_PASSWORD`, `POSTGRES_DB` |
| `REDIS_URL` | URL подключения к Redis | `string` | `redis://redis:6379/0` | Нет | Формат: `redis://host:port/database` |
| `SECRET_KEY` | Секретный ключ для подписи JWT токенов и шифрования | `string` | `your-secret-key-here` | Да (в продакшене) | КРИТИЧЕСКИ ВАЖНО: В продакшене обязательно измените на случайную строку (минимум 32 символа). Генерация: `python -c "import secrets; print(secrets.token_urlsafe(32))"` |
| `DEBUG` | Включает режим отладки (подробные логи, трассировка ошибок) | `bool` | `False` | Нет | В продакшене всегда должно быть `False` для безопасности |
| `ALLOWED_ORIGINS` | Список разрешенных источников (origins) для CORS запросов | `string` | `http://localhost:3000,http://127.0.0.1:3000,http://localhost:8080,http://127.0.0.1:8080` | Нет | Формат: список URL через запятую или JSON массив. Для продакшена укажите реальные домены вашего фронтенда |
| `UVICORN_WORKERS` | Количество worker процессов для обработки запросов | `int` | `4` | Нет | Рекомендации: 2-4 для небольших нагрузок, 4-8 для средних, 8-16 для высоких. Формула: `workers = (max_connections - 20) / 2`. Каждый worker создает свой пул соединений к БД |
| `PLUGIN_URLS` | Список URL плагинов парсеров через запятую | `string` | `http://python-parser-plugin:8000,http://gherkin-parser-plugin:8000` | Нет | Формат: `http://plugin1:port,http://plugin2:port`. Альтернативно можно использовать `PLUGIN_1_URL`, `PLUGIN_2_URL`, и т.д. Бекенд автоматически получает конфигурацию плагинов через `/config` endpoint при старте |
| `PLUGIN_TIMEOUT` | Таймаут запросов к плагинам парсеров в секундах | `int` | `30` | Нет | Увеличьте значение для больших файлов или медленных парсеров |
| `LOG_FILE_BACKUP_COUNT` | Количество файлов логов для хранения (дни) | `int` | `30` | Нет | Старые логи автоматически удаляются при превышении лимита |
| `LOG_FLUSH_BATCH_SIZE` | Количество записей перед принудительным flush в файл | `int` | `10` | Нет | Баланс между производительностью и надежностью. Меньше значение = больше надежность, но ниже производительность |
| `APP_HOST` | Хост для привязки сервера | `string` | `0.0.0.0` | Нет | Обычно не требуется изменять, используется внутри контейнера |
| `APP_PORT` | Порт для привязки сервера | `int` | `8000` | Нет | Обычно не требуется изменять, используется внутри контейнера |
| `ACCESS_TOKEN_EXPIRE_MINUTES` | Время жизни access токена в минутах | `int` | `10080` (7 дней) | Нет | |
| `REFRESH_TOKEN_EXPIRE_DAYS` | Время жизни refresh токена в днях | `int` | `14` (2 недели) | Нет | |

### Frontend

| Переменная | Описание | Тип | По умолчанию | Обязательно | Примечания |
|------------|----------|-----|--------------|-------------|------------|
| `FRONTEND_PORT` | Порт для проброса фронтенда на хост | `int` | `8080` | Нет | Используется только в docker-compose для маппинга портов |
| `BACKEND_PORT` | Порт для проброса бекенда на хост | `int` | `8001` | Нет | Используется только в docker-compose для маппинга портов |

### Плагины парсеров

| Переменная | Описание | Тип | По умолчанию | Обязательно | Примечания |
|------------|----------|-----|--------------|-------------|------------|
| `PLUGIN_NAME` | Имя плагина (для логирования) | `string` | Зависит от плагина (`python-parser`, `gherkin-parser`) | Нет | |
| `PLUGIN_LOG_LEVEL` | Уровень логирования плагина | `string` | `INFO` | Нет | Возможные значения: `DEBUG`, `INFO`, `WARNING`, `ERROR`, `CRITICAL` |

### Redis

Redis не требует дополнительных переменных окружения в docker-compose, так как используется стандартный образ `redis:7-alpine` с настройками по умолчанию.

### Примеры конфигураций

#### Минимальная конфигурация (для быстрого старта)

```yaml
backend:
  environment:
    - DATABASE_URL=postgresql://savetest_user:savetest_password@postgres:5432/savetest_db
    - REDIS_URL=redis://redis:6379/0
    - SECRET_KEY=your-super-secret-key-change-in-production
    - DEBUG=False
    - ALLOWED_ORIGINS=http://localhost:8080
    - PLUGIN_URLS=http://python-parser-plugin:8000,http://gherkin-parser-plugin:8000
```

#### Продакшн конфигурация

```yaml
backend:
  environment:
    - DATABASE_URL=postgresql://${POSTGRES_USER}:${POSTGRES_PASSWORD}@postgres:5432/${POSTGRES_DB}
    - REDIS_URL=redis://redis:6379/0
    - SECRET_KEY=${SECRET_KEY}  # Из секретов
    - DEBUG=False
    - ALLOWED_ORIGINS=https://app.example.com,https://www.example.com
    - UVICORN_WORKERS=8
    - PLUGIN_URLS=http://python-parser-plugin:8000,http://gherkin-parser-plugin:8000
    - PLUGIN_TIMEOUT=60
    - LOG_FILE_BACKUP_COUNT=90
```

### Использование .env файла

Рекомендуется использовать `.env` файл для хранения переменных окружения:

```bash
# .env
POSTGRES_DB=savetest_db
POSTGRES_USER=savetest_user
POSTGRES_PASSWORD=your-secure-password
SECRET_KEY=your-super-secret-key
DEBUG=False
ALLOWED_ORIGINS=https://app.example.com
UVICORN_WORKERS=4
PLUGIN_URLS=http://python-parser-plugin:8000,http://gherkin-parser-plugin:8000
```

Затем в docker-compose.yml:

```yaml
services:
  backend:
    env_file:
      - .env
```