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
- публикация Docker images;
- GitHub Actions CI/CD;
- SSH deployment;
- migrations/static steps после deploy;
- Telegram notification после успешного workflow.

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
├── .github/workflows/main.yml
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

Workflow GitHub Actions автоматизирует проверки, сборку Docker images и server deployment. Секреты Docker Hub, SSH и Telegram должны храниться в GitHub Actions Secrets, а не в repository files.

## Что показывает проект

Этот репозиторий полезен как доказательство навыков:

- Docker и Compose;
- multi-container application setup;
- PostgreSQL в container environment;
- Nginx routing;
- production configuration;
- CI/CD и deployment automation.

## Статус

Проект выполнен в рамках курса **«Python-разработчик» Яндекс Практикума** и используется как portfolio case по инфраструктуре backend-приложений.

## Автор инфраструктурной реализации

[Николь Журбенко](https://github.com/nikamurkaa)
