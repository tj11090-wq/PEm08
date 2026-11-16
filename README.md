# Анализатор конкурентов для сети пекарен

AI-ассистент для мониторинга конкурентов: анализирует текст, изображения и сайты конкурентов (цены, граммовки, оформление) и выдаёт структурированные выводы в JSON. Сделан по проекту урока PEm08 курса «Промпт-инжиниринг 3.0» (Zerocoder) и доработан под мой кейс: сеть пекарен в Екатеринбурге, продукция — пироги и мелкоштучная выпечка.

## Что умеет

- **Анализ текста** (`POST /analyze_text`): сильные и слабые стороны, уникальные предложения, рекомендации, ценовой сегмент (`price_position`) и цены с граммовками (`weight_notes`).
- **Анализ изображения** (`POST /analyze_image`): описание, инсайты, оценка дизайна `design_score` (0-10), аппетитность подачи `appetite_appeal` (0-10), цены и граммовки на картинке.
- **Парсинг сайта** (`POST /parse_demo`): Selenium открывает сайт в Chrome, делает скриншот (сохраняется в `data/screenshots/`), забирает видимый текст страницы, а ИИ анализирует скриншот и текст вместе.
- **Конкуренты в `backend/config.py`** (`competitor_urls`): Семейная пекарня, Хлебничная, Бисквитный двор. В интерфейсе под них есть кнопки (`GET /competitors`).
- **История** последних 10 запросов (`GET /history`).
- **Desktop-приложение** на PyQt6 (`desktop/`), собирается в `.exe` через `desktop/build.py`.

## Установка

```bash
python -m venv venv
venv\Scripts\activate
pip install -r requirements.txt
```

Если `pip install -r` обрывается по таймауту, ставьте пакеты по одному (`pip install fastapi`, `pip install uvicorn` и т.д.).

Для парсинга нужен установленный Google Chrome (драйвер скачивается автоматически через `webdriver-manager`).

## Ключ API

Скопируйте `env.example.txt` в `.env` и впишите ключ (`OPENAI_API_KEY` или `PROXY_API_KEY`). Файл `.env` в `.gitignore`, в репозиторий он не попадает. По умолчанию используется модель `gpt-4.1` (текст и изображения).

## Запуск

```bash
python run.py
```

или

```bash
python -m uvicorn backend.main:app --reload --port 8000
```

Интерфейс: http://localhost:8000, документация API: http://localhost:8000/docs

## Desktop-версия

```bash
cd desktop
pip install -r requirements.txt
python build.py
```

Готовый `competitor_monitor.exe` появится в `desktop/dist/`. Backend (`python run.py`) должен быть запущен: приложение обращается к `http://localhost:8000`.

## Структура

```
backend/            FastAPI: main.py, config.py, models/, services/ (openai, parser, history)
frontend/           HTML, CSS, JS
desktop/            PyQt6-приложение и сборка .exe
data/               материалы по конкурентам (цены, скриншоты)
env.example.txt     пример .env
```

## Ограничения

- Цены на сайтах конкурентов могут отличаться от цен в магазинах, а у некоторых сайтов цены подгружаются динамически: тогда ИИ видит только то, что попало на скриншот и в текст страницы.
- Ответы ИИ нужно проверять: модель может ошибиться в цене или весе, поэтому цифры сверяйте с сайтом.
