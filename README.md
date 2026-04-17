# onboard

AI-driven Telegram-бот для обучения сотрудников новому материалу и автоматического тестирования.

Бот работает полностью через LLM:
- объясняет материал простыми сообщениями;
- отвечает на вопросы сотрудника по заданной теме;
- сам решает, когда можно переходить к тесту;
- задает вопросы по одному;
- проверяет ответы по материалу;
- сохраняет результат теста в Postgres.

## Что умеет бот

- попросить имя сотрудника;
- провести обучение по `TRAINING_TOPIC` и `TRAINING_MATERIAL`;
- провести тест из `QUIZ_QUESTION_COUNT` вопросов;
- посчитать количество верных ответов и процент;
- сохранить результат в таблицу `training_results`.

## Как работает

1. Пользователь отправляет `/start`.
2. Бот просит имя сотрудника.
3. После этого AI-наставник начинает обучение по заданному материалу.
4. Когда сотрудник готов, AI переходит к тестированию.
5. После завершения теста бот сохраняет результат в Postgres.

## Структура результата в БД

Таблица `training_results` содержит:
- `employee_name`
- `telegram_user_id`
- `telegram_chat_id`
- `topic`
- `total_questions`
- `correct_answers`
- `score_percent`
- `final_summary`
- `created_at`

## Переменные окружения

```env
BOT_TOKEN=your_telegram_bot_token
OPENAI_API_KEY=your_openai_api_key
OPENAI_MODEL=gpt-5.4-mini
OPENAI_BASE_URL=https://api.openai.com/v1
TRAINING_TOPIC=Регламент работы команды
TRAINING_MATERIAL=Команда использует асинхронную коммуникацию по умолчанию. Все задачи фиксируются в трекере. Блокеры нужно эскалировать в течение 30 минут. Перед релизом обязательны code review, зеленые тесты и запись в changelog.
QUIZ_QUESTION_COUNT=5
LOG_LEVEL=INFO
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_DB=bot_db
POSTGRES_USER=user
POSTGRES_PASSWORD=your_postgres_password
DATABASE_URL=postgresql+asyncpg://user:your_postgres_password@postgres:5432/bot_db
```
`DATABASE_URL` теперь хранится в `.env` и подхватывается контейнером через `env_file` в `docker-compose.yml`.

## Локальный запуск

```powershell
cd C:\Users\daniil\Desktop\prompt_cases\onboard
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
Copy-Item .env.example .env
python main.py
```

## Запуск через Docker

Важно: текущий билд программы актуален только для сервера, где уже существует и запущена PostgreSQL. Эта PostgreSQL ранее использовалась и была проверена на других проектах на том же сервере.

1. Создайте `.env` из шаблона:
   - `cp .env.example .env`
2. Заполните значения в `.env` (минимум: `BOT_TOKEN`, `OPENAI_API_KEY`, `POSTGRES_PASSWORD`, `DATABASE_URL`)
3. Выполните:

```powershell
docker compose up --build
```

## Пример сценария

```text
Пользователь: /start
Бот: Напишите имя сотрудника, которого нужно обучить.
Пользователь: Иван Петров
Бот: Сегодня разберем материал по регламенту команды...
Пользователь: А когда нужно эскалировать блокер?
Бот: В течение 30 минут. Готовы перейти к тесту?
Пользователь: Да
Бот: Вопрос 1. Где должны фиксироваться задачи?
...
Бот: Тест завершен. Результат сохранен в Postgres.
```
