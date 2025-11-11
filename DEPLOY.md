# Инструкция по деплою бота

## Варианты хостинга

### 1. VPS (Рекомендуется)
- **DigitalOcean** - от $6/месяц
- **Hetzner** - от €4/месяц (лучшее соотношение цена/качество)
- **Timeweb** - от 200₽/месяц (российский хостинг)
- **Selectel** - от 300₽/месяц

### 2. Облачные платформы
- **Railway** - простой деплой, есть бесплатный тариф
- **Render** - бесплатный тариф с ограничениями
- **Fly.io** - хороший бесплатный тариф

---

## Деплой на VPS (Ubuntu/Debian)

### Шаг 1: Подключение к серверу

```bash
ssh root@your-server-ip
```

### Шаг 2: Установка зависимостей

```bash
# Обновление системы
apt update && apt upgrade -y

# Установка Python 3.11
apt install -y python3.11 python3.11-venv python3-pip git

# Установка systemd (обычно уже установлен)
apt install -y systemd
```

### Шаг 3: Создание пользователя для бота (опционально, но рекомендуется)

```bash
adduser --disabled-password --gecos "" botuser
su - botuser
```

### Шаг 4: Клонирование проекта

```bash
cd /home/botuser
git clone <your-repo-url> motiv-bot
cd motiv-bot
```

Или загрузите файлы через SFTP/SCP:

```bash
# С вашего компьютера
scp -r . botuser@your-server-ip:/home/botuser/motiv-bot
```

### Шаг 5: Настройка окружения

```bash
# Создание виртуального окружения
python3.11 -m venv venv
source venv/bin/activate

# Установка зависимостей
pip install -r requirements.txt

# Создание .env файла
nano .env
```

Содержимое `.env`:
```
BOT_TOKEN=your_bot_token_here
ADMIN_ID=your_telegram_user_id
DATABASE_URL=sqlite+aiosqlite:///./data/motiv.db
TIMEZONE=Europe/Moscow
FORECAST_MODEL=SMA
FORECAST_WINDOW=7
FORECAST_COST_BEHAVIOR=extrapolate
CR_ALERT_THRESHOLD=0.0
EPL_DROP_THRESHOLD=0.3
LOG_LEVEL=INFO
```

### Шаг 6: Создание systemd сервиса

```bash
sudo nano /etc/systemd/system/motiv-bot.service
```

Содержимое файла:
```ini
[Unit]
Description=Motiv Bot Telegram Bot
After=network.target

[Service]
Type=simple
User=botuser
WorkingDirectory=/home/botuser/motiv-bot
Environment="PATH=/home/botuser/motiv-bot/venv/bin"
ExecStart=/home/botuser/motiv-bot/venv/bin/python -m app.main
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

### Шаг 7: Запуск сервиса

```bash
# Перезагрузка systemd
sudo systemctl daemon-reload

# Включение автозапуска
sudo systemctl enable motiv-bot

# Запуск бота
sudo systemctl start motiv-bot

# Проверка статуса
sudo systemctl status motiv-bot

# Просмотр логов
sudo journalctl -u motiv-bot -f
```

### Шаг 8: Управление ботом

```bash
# Остановка
sudo systemctl stop motiv-bot

# Перезапуск
sudo systemctl restart motiv-bot

# Просмотр логов
sudo journalctl -u motiv-bot -n 50
```

---

## Деплой через Docker

### Шаг 1: Установка Docker на сервере

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sh get-docker.sh
```

### Шаг 2: Клонирование проекта

```bash
git clone <your-repo-url> motiv-bot
cd motiv-bot
```

### Шаг 3: Создание .env файла

```bash
nano .env
```

### Шаг 4: Запуск через Docker Compose

```bash
docker-compose up -d
```

### Шаг 5: Просмотр логов

```bash
docker-compose logs -f
```

---

## Деплой на Railway

1. Зарегистрируйтесь на [railway.app](https://railway.app)
2. Создайте новый проект
3. Подключите GitHub репозиторий
4. Добавьте переменные окружения в настройках проекта
5. Railway автоматически определит Python проект и запустит его

**Важно:** Railway нужно указать команду запуска:
```
python -m app.main
```

---

## Деплой на Render

1. Зарегистрируйтесь на [render.com](https://render.com)
2. Создайте новый "Web Service"
3. Подключите GitHub репозиторий
4. Настройки:
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `python -m app.main`
5. Добавьте переменные окружения
6. Выберите бесплатный план (бот будет "засыпать" после 15 минут бездействия)

---

## Рекомендации

### Безопасность
- Не коммитьте `.env` файл в Git
- Используйте сильные пароли для SSH
- Настройте firewall (ufw)
- Регулярно обновляйте систему

### Мониторинг
- Настройте логирование в файл
- Используйте `systemd` для автоматического перезапуска
- Настройте уведомления о падении бота

### Резервное копирование
- Регулярно делайте бэкап базы данных (`data/motiv.db`)
- Настройте автоматическое копирование

### Оптимизация
- Используйте PostgreSQL вместо SQLite для продакшена (опционально)
- Настройте логирование уровней

---

## Миграция базы данных

Если нужно перенести базу данных с локального компьютера на сервер:

```bash
# На локальном компьютере
scp data/motiv.db botuser@your-server-ip:/home/botuser/motiv-bot/data/
```

---

## Обновление бота

```bash
# На сервере
cd /home/botuser/motiv-bot
git pull
source venv/bin/activate
pip install -r requirements.txt
sudo systemctl restart motiv-bot
```

Или через Docker:
```bash
docker-compose pull
docker-compose up -d
```

