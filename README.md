## motiv-bot

Телеграм-бот на `aiogram` для CPA-арбитражника: ведёт учёт лидов и апрувов, расходов, считает метрики (CR, ROI, EPL, чистая прибыль), строит прогноз, поддерживает импорт/экспорт и уведомления.

### Запуск

1. Создай `.env` (см. ниже).
2. Установи зависимости:
   ```
   poetry install
   ```
3. Применяй миграции (или `init_db` при первом запуске), затем:
   ```
   poetry run python -m app.main
   ```

Пример `.env`:
```
BOT_TOKEN=...
ADMIN_ID=...
DATABASE_URL=sqlite+aiosqlite:///./data/motiv.db
TIMEZONE=Europe/Moscow
FORECAST_MODEL=SMA
FORECAST_WINDOW=7
FORECAST_COST_BEHAVIOR=extrapolate
CR_ALERT_THRESHOLD=0.0
EPL_DROP_THRESHOLD=0.3
LOG_LEVEL=INFO
```

### Структура

- `app/core` — конфиги, DI, логирование, планировщик.
- `app/models` — ORM-модели.
- `app/repositories` — доступ к данным.
- `app/services` — бизнес-логика.
- `app/bot` — aiogram-роутеры, клавиатуры, FSM.
- `app/jobs` — фоновые задачи.
- `docs` — архитектура и план.

### Лицензия

MIT.


