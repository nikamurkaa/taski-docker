# Taski Docker

**Taski Docker** — учебный инфраструктурный проект Яндекс Практикума: готовое Django + React приложение подготовлено к работе в Docker, PostgreSQL, Nginx и CI/CD.

> Репозиторий является fork исходного учебного проекта `yandex-praktikum/taski-docker`. Моя работа здесь — контейнеризация, инфраструктурная конфигурация и deployment pipeline, а не исходная бизнес-логика todo-приложения.

## Мой вклад

- Dockerfile для Django backend;
- Dockerfile для React frontend;
- отдельный Nginx gateway container;
- запуск backend через Gunicorn;
- PostgreSQL вместо локальной SQLite-конфигурации;
- Docker volumes для database и static;
- `docker-compose.yml` для локального запуска;
- `docker-compose.production.yml` для server deployment;
- GitHub Actions CI для backend/frontend и Docker build;
- отдельный ручной workflow для публикации Docker images и SSH deployment;
- migrations/static steps после deploy;
- Telegram notification после успешного production workflow.

## Стек

**Backend:** Python, Django, Django REST Framework, Gunicorn, PostgreSQL  
**Frontend:** JavaScript, React 18, Axios, Bootstrap  
**Infrastructure:** Docker, Docker Compose, Nginx, Docker Hub, GitHub Actions, SSH

## Архитектура

```text
Client
  │
  ▼
Nginx gateway
  ├──► React static
  └──► Django REST API
           │
           ▼
       PostgreSQL
```

| Container | Назначение |
| --- | --- |
| `backend` | Django API / Gunicorn |
| `frontend` | React production build |
| `gateway` | Nginx entry point |
| `db` | PostgreSQL |

## Структура

```text
.
├── backend/
├── frontend/
├── gateway/
├── .github/workflows/
│   ├── main.yml              # CI: tests + Docker build
│   └── deploy.yml            # manual production deployment
├── docker-compose.yml
├── docker-compose.production.yml
├── setup.cfg
└── README.md
```

## Локальный запуск

Клонировать этот fork:

```bash
git clone https://github.com/nikamurkaa/taski-docker.git
cd taski-docker
```

Создать `.env` с PostgreSQL-настройками:

```env
POSTGRES_USER=django_user
POSTGRES_PASSWORD=django_password
POSTGRES_DB=django_db
DB_HOST=db
DB_PORT=5432
```

Запуск:

```bash
docker compose up --build
```

Миграции:

```bash
docker compose exec backend python manage.py migrate
```

Сборка backend static:

```bash
docker compose exec backend python manage.py collectstatic --noinput
```

## CI/CD

`.github/workflows/main.yml` запускается на push и pull request в `main`. Он проверяет backend, frontend и локальную сборку Docker images без production-секретов.

`.github/workflows/deploy.yml` запускается вручную через `workflow_dispatch`. Он публикует images в Docker Hub и выполняет SSH deployment только для настроенного production environment.

Docker Hub namespace не зашит в repository files: `docker-compose.production.yml` использует переменную `DOCKER_USERNAME`, а GitHub Actions — secret с тем же именем.

Секреты Docker Hub, SSH и Telegram хранятся в GitHub Actions Secrets и не должны попадать в repository files.

## Что демонстрирует проект

- Docker и Compose;
- multi-container application setup;
- PostgreSQL в container environment;
- Nginx routing;
- production configuration;
- CI/CD и deployment automation.

## Статус

Проект выполнен в рамках курса **«Python-разработчик» Яндекс Практикума** и демонстрирует инфраструктуру backend-приложения: контейнеризацию, CI и контролируемый production deployment.

## Автор инфраструктурной реализации

[Николь Журбенко](https://github.com/nikamurkaa)
