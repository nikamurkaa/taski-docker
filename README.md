# Taski Docker

Taski Docker - учебный проект Яндекс Практикума, в котором готовое Django + React приложение было подготовлено к контейнерному запуску, работе с PostgreSQL, Nginx и автоматическому деплою через GitHub Actions.

Основная цель проекта - не разработка самого todo-приложения с нуля, а упаковка приложения в Docker-инфраструктуру, настройка взаимодействия нескольких контейнеров и подготовка проекта к запуску на сервере.

---

## Что реализовано

В проекте настроена инфраструктура для запуска приложения Taski:

- Dockerfile для backend-приложения на Django;
- Dockerfile для frontend-приложения на React;
- отдельный Dockerfile для gateway-контейнера с Nginx;
- запуск backend через Gunicorn;
- подключение PostgreSQL вместо SQLite;
- хранение данных PostgreSQL в Docker volume;
- общий volume для статических файлов backend и frontend;
- маршрутизация запросов через Nginx;
- локальный запуск проекта через Docker Compose;
- production-конфигурация `docker-compose.production.yml`;
- сборка и публикация Docker-образов в Docker Hub;
- CI/CD workflow на GitHub Actions;
- автоматический деплой на сервер по SSH;
- отправка Telegram-уведомления после успешного деплоя.

---

## Функциональность приложения

Taski — небольшое приложение для управления задачами.

Пользователь может:

- просматривать список задач;
- создавать новые задачи;
- редактировать задачи;
- отмечать задачи выполненными;
- удалять задачи.

Backend предоставляет REST API, а frontend отображает пользовательский интерфейс для работы с задачами.

---

## Технологии

### Backend

- Python 3.12
- Django 5.1.1
- Django REST Framework
- Gunicorn
- PostgreSQL
- psycopg2-binary
- django-cors-headers

### Frontend

- JavaScript
- React 18
- Axios
- Bootstrap
- Reactstrap

### Инфраструктура

- Docker
- Docker Compose
- Docker Hub
- Nginx
- GitHub Actions
- SSH deploy
- Telegram notifications

---

## Архитектура проекта

Проект состоит из нескольких контейнеров:

| Контейнер | Назначение |
|---|---|
| `backend` | Django API, запущенное через Gunicorn |
| `frontend` | React-приложение, которое собирает production-статику |
| `db` | PostgreSQL database |
| `gateway` | Nginx, единая точка входа в приложение |

Схема работы:

1. Пользователь отправляет запрос на `gateway`.
2. Nginx обрабатывает запрос.
3. Запросы к `/api/` и `/admin/` перенаправляются в backend-контейнер.
4. Остальные запросы обрабатываются как frontend-статика.
5. Backend обращается к PostgreSQL-контейнеру по имени сервиса `db`.
6. Данные базы и статические файлы хранятся в Docker volumes.

---

## Структура проекта

```text
.
├── .github/
│   └── workflows/
│       └── main.yml
├── backend/
│   ├── api/
│   ├── backend/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── manage.py
│   └── requirements.txt
├── frontend/
│   ├── public/
│   ├── src/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── package.json
│   └── package-lock.json
├── gateway/
│   ├── Dockerfile
│   └── nginx.conf
├── docker-compose.yml
├── docker-compose.production.yml
├── setup.cfg
├── LICENSE
└── README.md
```

---

## Переменные окружения

Для запуска проекта нужен файл `.env` в корневой директории.

Пример `.env`:

```env
POSTGRES_USER=django_user
POSTGRES_PASSWORD=django_password
POSTGRES_DB=django_db
DB_HOST=db
DB_PORT=5432
```

Файл `.env` содержит чувствительные данные и не должен попадать в публичный репозиторий.

---

## Локальный запуск через Docker Compose

Убедитесь, что Docker запущен.

Клонируйте репозиторий:

```bash
git clone https://github.com/kindarufy/taski-docker.git
cd taski-docker
```

Создайте файл `.env` в корне проекта и добавьте в него переменные окружения из примера выше.

Соберите и запустите контейнеры:

```bash
docker compose up --build
```

В отдельном окне терминала выполните миграции:

```bash
docker compose exec backend python manage.py migrate
```

Соберите статику backend-приложения:

```bash
docker compose exec backend python manage.py collectstatic --noinput
```

Скопируйте собранную backend-статику в общий volume:

```bash
docker compose exec backend cp -r /app/collected_static/. /backend_static/static/
```

После запуска приложение будет доступно по адресу:

```text
http://localhost:8000/
```

Админ-панель Django:

```text
http://localhost:8000/admin/
```

API задач:

```text
http://localhost:8000/api/tasks/
```

> Контейнер `frontend` после сборки и копирования статики может завершить работу. Это нормальное поведение: его задача — собрать frontend и положить файлы в общий volume.

---

## Основные API-эндпоинты

| Метод | URL | Описание |
|---|---|---|
| `GET` | `/api/tasks/` | Получить список задач |
| `POST` | `/api/tasks/` | Создать задачу |
| `GET` | `/api/tasks/{id}/` | Получить задачу по id |
| `PUT` / `PATCH` | `/api/tasks/{id}/` | Обновить задачу |
| `DELETE` | `/api/tasks/{id}/` | Удалить задачу |

Пример запроса на создание задачи:

```bash
curl -X POST http://localhost:8000/api/tasks/ \
  -H "Content-Type: application/json" \
  -d '{"title": "Изучить Docker", "description": "Запустить проект через Docker Compose"}'
```

---

## Production-запуск

Для production используется отдельный файл:

```text
docker-compose.production.yml
```

В production-конфигурации контейнеры запускаются не из локальной сборки, а из готовых Docker-образов:

- `kindarufy/taski_backend`
- `kindarufy/taski_frontend`
- `kindarufy/taski_gateway`

Запуск на сервере:

```bash
sudo docker compose -f docker-compose.production.yml up -d
```

После запуска необходимо выполнить миграции и подготовить статику:

```bash
sudo docker compose -f docker-compose.production.yml exec backend python manage.py migrate
sudo docker compose -f docker-compose.production.yml exec backend python manage.py collectstatic --noinput
sudo docker compose -f docker-compose.production.yml exec backend cp -r /app/collected_static/. /backend_static/static/
```

Проверить состояние контейнеров:

```bash
sudo docker compose -f docker-compose.production.yml ps
```

Посмотреть логи:

```bash
sudo docker compose -f docker-compose.production.yml logs
```

Остановить контейнеры:

```bash
sudo docker compose -f docker-compose.production.yml down
```

---

## CI/CD

В проекте настроен workflow GitHub Actions: `.github/workflows/main.yml`.

Pipeline выполняет следующие шаги:

1. Запускает backend-проверки:
   - установка Python-зависимостей;
   - запуск flake8;
   - запуск Django tests;
   - поднятие PostgreSQL service для тестов.
2. Запускает frontend-проверки:
   - установка Node.js-зависимостей;
   - запуск React tests.
3. Собирает Docker-образы backend, frontend и gateway.
4. Публикует образы в Docker Hub.
5. Копирует `docker-compose.production.yml` на сервер.
6. Подключается к серверу по SSH.
7. Обновляет Docker-образы и перезапускает контейнеры.
8. Выполняет миграции и сборку статики.
9. Отправляет уведомление в Telegram об успешном деплое.

Для работы workflow в GitHub Secrets должны быть добавлены переменные:

```text
DOCKER_USERNAME
DOCKER_PASSWORD
HOST
USER
SSH_KEY
TELEGRAM_TO
TELEGRAM_TOKEN
```

---

## Тестирование

Backend-проверки:

```bash
python -m flake8 backend/
cd backend
python manage.py test
```

Frontend-проверки:

```bash
cd frontend
npm ci
CI=true npm run test
```

В GitHub Actions эти проверки запускаются автоматически при push в ветку `main`.

---

## Особенности реализации

- Backend работает через Gunicorn, а не через Django development server.
- PostgreSQL вынесен в отдельный контейнер.
- Данные PostgreSQL сохраняются в Docker volume и не пропадают при пересоздании контейнеров.
- Frontend собирается в production-статику и передаёт её в общий volume.
- Nginx выступает единой точкой входа и маршрутизирует запросы между frontend и backend.
- Production-конфигурация использует готовые Docker-образы из Docker Hub.
- Деплой автоматизирован через GitHub Actions и SSH.


Главный фокус проекта — контейнеризация, настройка Docker Compose, подключение PostgreSQL, работа с Nginx и автоматизация деплоя.
