# «Что посмотреть» (what_to_watch)

Веб-сервис мнений о фильмах: на главной странице случайным образом показывается чужое мнение — чтобы решить, какой фильм посмотреть сегодня. Пользователь может добавлять собственные мнения (с названием фильма, текстом, ссылкой на обзор и картинками), делиться прямой ссылкой на конкретное мнение, а также пользоваться полноценным REST API.

Учебный проект Яндекс.Практикума (спринт 19).

## Возможности

- **Случайное мнение** на главной — помощь в выборе фильма.
- **Добавление мнения** через форму: название фильма, текст, ссылка на источник, загрузка нескольких изображений (jpg, jpeg, png, gif, bmp) — картинки асинхронно отправляются в Dropbox, в БД сохраняются их публичные ссылки.
- **Прямая ссылка на мнение** (`/opinion/<id>`) — для отправки друзьям.
- **REST API** (`/api/opinions/`) — список, создание, получение, частичное обновление и удаление мнений, а также отдельный эндпоинт случайного мнения (`/api/get-random-opinion/`). Ошибки отдаются в виде JSON.
- **Консольная команда** `flask load_opinions` — массовая загрузка мнений из CSV-файла.
- **Миграции БД** через Alembic (Flask-Migrate).
- Обработка ошибок 404 и 500 с кастомными страницами.

## Технологии

- Python 3.11, Flask 3, Flask-SQLAlchemy 3 (SQLite), Flask-Migrate (Alembic)
- Flask-WTF — формы с CSRF-защитой и валидацией загрузки файлов
- aiohttp — асинхронная загрузка изображений в Dropbox API
- python-dotenv — конфигурация через переменные окружения
- Bootstrap 5, кастомные стили, шрифт Montserrat

## Структура проекта

```
what_to_watch/
├── settings.py             # класс Config: DATABASE_URI, SECRET_KEY, DROPBOX_TOKEN
├── opinions_app/           # пакет приложения
│   ├── __init__.py         # создание app, db, migrate
│   ├── models.py           # модель Opinion (title, text, source, images, added_by, timestamp)
│   ├── views.py            # веб-роуты: /, /add, /opinion/<id>
│   ├── api_views.py        # REST API /api/opinions/
│   ├── forms.py            # OpinionForm (Flask-WTF)
│   ├── dropbox.py          # асинхронная загрузка картинок в Dropbox
│   ├── cli_commands.py     # команда flask load_opinions (из opinions.csv)
│   ├── error_handlers.py   # InvalidAPIUsage, обработчики 404/500
│   ├── templates/          # Jinja2-шаблоны
│   └── static/             # css, img, шрифты
├── migrations/             # миграции Alembic
├── requirements.txt
└── .env                    # переменные окружения (не входит в репозиторий)
```

## Установка и запуск

Клонировать репозиторий и перейти в него:

```bash
git clone https://github.com/linfir21/what_to_watch-1.git
cd what_to_watch-1
```

Создать и активировать виртуальное окружение:

```bash
python -m venv venv
# Linux/macOS:
source venv/bin/activate
# Windows:
venv\Scripts\activate
```

Установить зависимости:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

Создать файл `.env` в корне проекта:

```
DATABASE_URI=sqlite:///db.sqlite3
SECRET_KEY=<случайная строка>
DROPBOX_TOKEN=<токен Dropbox API с правами files.content.write и sharing.write>
```

> Без `DROPBOX_TOKEN` добавление мнений с картинками не будет работать. Токен можно получить на [developers.dropbox.com](https://www.dropbox.com/developers) — для загрузки файлов в личный Dropbox достаточно токена с ограниченным доступом.

Применить миграции (БД появится в `instance/db.sqlite3`):

```bash
flask db upgrade
```

Опционально наполнить базу из CSV (файл `opinions.csv` с колонками `title, text, source` в корне проекта):

```bash
flask load_opinions
```

Запустить приложение:

```bash
flask run
```

## API

| Метод | Эндпоинт | Описание |
|-------|----------|----------|
| GET | `/api/opinions/` | список всех мнений |
| POST | `/api/opinions/` | создать мнение (обязательны `title` и `text`, текст должен быть уникальным) |
| GET | `/api/opinions/<id>/` | получить мнение по id |
| PATCH | `/api/opinions/<id>/` | частично обновить мнение |
| DELETE | `/api/opinions/<id>/` | удалить мнение |
| GET | `/api/get-random-opinion/` | случайное мнение (404, если база пуста) |

Пример создания:

```bash
curl -X POST http://localhost:5000/api/opinions/ \
  -H "Content-Type: application/json" \
  -d '{"title": "Интерстеллар", "text": "Эпичная история про любовь и гравитацию.", "source": "https://example.com/review"}'
```

При ошибках API возвращает JSON вида `{"message": "..."}` с соответствующим HTTP-кодом.
