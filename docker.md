# Docker с нуля до уверенного Junior+ на примере FastAPI + PostgreSQL

Это подробный практический гайд, который объясняет Docker с нуля на реальном примере backend-приложения:

- FastAPI — Python web framework.
- PostgreSQL — база данных.
- Docker — контейнеризация.
- Docker Compose — запуск нескольких контейнеров вместе.
- Volumes — сохранение данных.
- Networks — связь контейнеров.
- Environment variables — конфигурация.
- Миграции БД.
- Debugging.
- Dev/prod подход.
- Best practices.

---

# 1. Что такое Docker простыми словами

## Проблема без Docker

Допустим, у тебя есть FastAPI-приложение.

Оно требует:

- Python 3.12
- PostgreSQL 16
- pip-зависимости
- переменные окружения
- правильные порты
- настройки базы данных

На твоём компьютере всё работает.

Но потом ты отдаёшь проект другому разработчику, и у него:

- другая версия Python;
- нет PostgreSQL;
- PostgreSQL установлен, но другая версия;
- заняты порты;
- не хватает зависимостей;
- приложение падает.

Docker решает эту проблему.

## Идея Docker

Docker позволяет упаковать приложение и всё, что ему нужно, в изолированную среду — контейнер.

Контейнер — это как маленькая виртуальная среда, внутри которой есть:

- приложение;
- Python;
- зависимости;
- переменные окружения;
- настройки;
- файловая система.

И эта среда одинаково запускается:

- у тебя;
- у другого разработчика;
- на сервере;
- в CI/CD;
- в облаке.

---

# 2. Основные понятия Docker

## Image — образ

Docker image — это шаблон для создания контейнера.

Пример:

```bash
python:3.12-slim
postgres:16
nginx:alpine
```

Образ можно сравнить с установочным файлом программы.

Например, `python:3.12-slim` — это готовый образ Linux с установленным Python 3.12.

## Container — контейнер

Container — это запущенный экземпляр образа.

Если image — это класс, то container — это объект.

Один image можно запустить много раз:

```bash
docker run python:3.12-slim
docker run python:3.12-slim
docker run python:3.12-slim
```

Каждый запуск создаст отдельный контейнер.

## Dockerfile

Dockerfile — это инструкция, как собрать свой Docker image.

Например:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install -r requirements.txt

COPY . .

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Docker Compose

Docker Compose нужен, когда приложение состоит из нескольких сервисов.

Например:

- backend FastAPI;
- database PostgreSQL;
- redis;
- nginx.

Все они описываются в одном файле `docker-compose.yml`.

---

# 3. Установка Docker

## Windows / macOS

Установи Docker Desktop:

```text
https://www.docker.com/products/docker-desktop/
```

После установки проверь:

```bash
docker --version
```

Пример:

```bash
Docker version 27.0.3
```

Также проверь Docker Compose:

```bash
docker compose version
```

Важно: сейчас используется команда:

```bash
docker compose
```

А не старая:

```bash
docker-compose
```

---

# 4. Первый запуск контейнера

Запустим простой контейнер:

```bash
docker run hello-world
```

Что происходит:

1. Docker ищет образ `hello-world` локально.
2. Если его нет, скачивает с Docker Hub.
3. Создаёт контейнер.
4. Запускает контейнер.
5. Контейнер выводит сообщение.
6. Контейнер завершается.

---

# 5. Полезные команды Docker

## Посмотреть запущенные контейнеры

```bash
docker ps
```

## Посмотреть все контейнеры, включая остановленные

```bash
docker ps -a
```

## Посмотреть образы

```bash
docker images
```

## Остановить контейнер

```bash
docker stop container_name
```

## Удалить контейнер

```bash
docker rm container_name
```

## Удалить образ

```bash
docker rmi image_name
```

## Посмотреть логи контейнера

```bash
docker logs container_name
```

## Зайти внутрь контейнера

```bash
docker exec -it container_name bash
```

Если внутри нет `bash`, можно использовать:

```bash
docker exec -it container_name sh
```

---

# 6. Что мы будем делать

Мы создадим FastAPI-приложение с PostgreSQL.

Архитектура будет такая:

```text
project/
├── app/
│   ├── main.py
│   ├── database.py
│   ├── models.py
│   └── schemas.py
├── requirements.txt
├── Dockerfile
├── docker-compose.yml
├── .env
└── .dockerignore
```

У нас будет два контейнера:

```text
fastapi_app  -> backend
postgres_db  -> database
```

FastAPI будет подключаться к PostgreSQL внутри Docker-сети.

---

# 7. Создание FastAPI-приложения

Создай папку проекта:

```bash
mkdir fastapi-docker-demo
cd fastapi-docker-demo
```

Создай структуру:

```bash
mkdir app
touch app/main.py app/database.py app/models.py app/schemas.py
touch requirements.txt Dockerfile docker-compose.yml .env .dockerignore
```

---

# 8. requirements.txt

Файл `requirements.txt` хранит зависимости Python.

```txt
fastapi==0.115.6
uvicorn[standard]==0.32.1
sqlalchemy==2.0.36
psycopg2-binary==2.9.10
python-dotenv==1.0.1
```

## Что здесь находится

### FastAPI

```txt
fastapi
```

Фреймворк для создания API.

### Uvicorn

```txt
uvicorn[standard]
```

ASGI-сервер, который запускает FastAPI.

FastAPI — это само приложение.

Uvicorn — это сервер, который его обслуживает.

### SQLAlchemy

```txt
sqlalchemy
```

ORM для работы с базой данных.

ORM позволяет работать с таблицами как с Python-классами.

### psycopg2-binary

```txt
psycopg2-binary
```

Драйвер для подключения Python к PostgreSQL.

### python-dotenv

```txt
python-dotenv
```

Позволяет читать переменные из `.env`.

---

# 9. .env

Файл `.env` содержит конфигурацию.

```env
POSTGRES_DB=app_db
POSTGRES_USER=app_user
POSTGRES_PASSWORD=app_password
POSTGRES_HOST=db
POSTGRES_PORT=5432

DATABASE_URL=postgresql://app_user:app_password@db:5432/app_db
```

## Важная деталь

```env
POSTGRES_HOST=db
```

Почему `db`, а не `localhost`?

Потому что FastAPI и PostgreSQL будут находиться в разных контейнерах.

Внутри Docker Compose контейнеры общаются друг с другом по имени сервиса.

Если в `docker-compose.yml` сервис базы называется:

```yaml
db:
```

то FastAPI подключается к нему по адресу:

```text
db
```

А не:

```text
localhost
```

## Почему localhost не подходит

Внутри контейнера `localhost` означает сам этот контейнер.

Если FastAPI-контейнер пытается подключиться к `localhost:5432`, он ищет PostgreSQL внутри себя.

Но PostgreSQL находится в другом контейнере.

Поэтому нужно использовать имя сервиса:

```text
db
```

---

# 10. app/database.py

```python
import os

from sqlalchemy import create_engine
from sqlalchemy.orm import declarative_base, sessionmaker


DATABASE_URL = os.getenv(
    "DATABASE_URL",
    "postgresql://app_user:app_password@db:5432/app_db"
)

engine = create_engine(DATABASE_URL)

SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine
)

Base = declarative_base()


def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

## Разбор кода

### DATABASE_URL

```python
DATABASE_URL = os.getenv(...)
```

Берём строку подключения из переменных окружения.

Пример строки:

```text
postgresql://app_user:app_password@db:5432/app_db
```

Она состоит из:

```text
postgresql://USER:PASSWORD@HOST:PORT/DATABASE
```

То есть:

```text
postgresql://app_user:app_password@db:5432/app_db
```

Означает:

- база: PostgreSQL;
- пользователь: `app_user`;
- пароль: `app_password`;
- хост: `db`;
- порт: `5432`;
- имя базы: `app_db`.

### engine

```python
engine = create_engine(DATABASE_URL)
```

`engine` — объект подключения SQLAlchemy к базе данных.

### SessionLocal

```python
SessionLocal = sessionmaker(...)
```

Создаёт сессии для работы с БД.

Сессия — это объект, через который мы:

- делаем SELECT;
- делаем INSERT;
- делаем UPDATE;
- делаем DELETE;
- сохраняем изменения.

### Base

```python
Base = declarative_base()
```

Базовый класс для моделей.

Все модели таблиц будут наследоваться от `Base`.

### get_db

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

Это dependency для FastAPI.

Он создаёт подключение к базе на время запроса, а после запроса закрывает его.

---

# 11. app/models.py

```python
from sqlalchemy import Column, Integer, String

from app.database import Base


class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    email = Column(String(255), unique=True, index=True, nullable=False)
```

## Что это значит

```python
class User(Base):
```

Создаём модель пользователя.

Эта модель соответствует таблице `users`.

```python
__tablename__ = "users"
```

Имя таблицы в PostgreSQL.

```python
id = Column(Integer, primary_key=True, index=True)
```

Поле `id`:

- целое число;
- primary key;
- индексируется.

```python
name = Column(String(100), nullable=False)
```

Поле `name`:

- строка максимум 100 символов;
- не может быть пустым.

```python
email = Column(String(255), unique=True, index=True, nullable=False)
```

Поле `email`:

- строка максимум 255 символов;
- уникальное;
- индексируется;
- не может быть пустым.

---

# 12. app/schemas.py

```python
from pydantic import BaseModel, EmailStr


class UserCreate(BaseModel):
    name: str
    email: EmailStr


class UserRead(BaseModel):
    id: int
    name: str
    email: EmailStr

    class Config:
        from_attributes = True
```

## Что такое schemas

Schemas — это Pydantic-модели.

Они описывают:

- какие данные принимает API;
- какие данные возвращает API;
- как валидировать данные.

## UserCreate

```python
class UserCreate(BaseModel):
    name: str
    email: EmailStr
```

Используется при создании пользователя.

Клиент отправляет:

```json
{
  "name": "Ivan",
  "email": "ivan@example.com"
}
```

## UserRead

```python
class UserRead(BaseModel):
    id: int
    name: str
    email: EmailStr
```

Используется при ответе API.

Сервер возвращает:

```json
{
  "id": 1,
  "name": "Ivan",
  "email": "ivan@example.com"
}
```

---

# 13. Дополнительная зависимость для EmailStr

Так как мы используем:

```python
EmailStr
```

нужно добавить зависимость:

```txt
email-validator==2.2.0
```

Обнови `requirements.txt`:

```txt
fastapi==0.115.6
uvicorn[standard]==0.32.1
sqlalchemy==2.0.36
psycopg2-binary==2.9.10
python-dotenv==1.0.1
email-validator==2.2.0
```

---

# 14. app/main.py

```python
from fastapi import Depends, FastAPI, HTTPException
from sqlalchemy.orm import Session

from app.database import Base, engine, get_db
from app.models import User
from app.schemas import UserCreate, UserRead


Base.metadata.create_all(bind=engine)

app = FastAPI(
    title="FastAPI Docker PostgreSQL Demo",
    version="1.0.0"
)


@app.get("/")
def root():
    return {"message": "Hello from FastAPI inside Docker!"}


@app.get("/health")
def health():
    return {"status": "ok"}


@app.post("/users", response_model=UserRead)
def create_user(user_data: UserCreate, db: Session = Depends(get_db)):
    existing_user = (
        db.query(User)
        .filter(User.email == user_data.email)
        .first()
    )

    if existing_user:
        raise HTTPException(
            status_code=400,
            detail="User with this email already exists"
        )

    user = User(
        name=user_data.name,
        email=user_data.email
    )

    db.add(user)
    db.commit()
    db.refresh(user)

    return user


@app.get("/users", response_model=list[UserRead])
def get_users(db: Session = Depends(get_db)):
    users = db.query(User).all()
    return users


@app.get("/users/{user_id}", response_model=UserRead)
def get_user(user_id: int, db: Session = Depends(get_db)):
    user = (
        db.query(User)
        .filter(User.id == user_id)
        .first()
    )

    if not user:
        raise HTTPException(
            status_code=404,
            detail="User not found"
        )

    return user
```

## Важное замечание про create_all

```python
Base.metadata.create_all(bind=engine)
```

Эта строка автоматически создаёт таблицы.

Для обучения это удобно.

Но в реальных проектах лучше использовать миграции через Alembic.

Позже мы это обсудим.

---

# 15. Первый запуск без Docker

Если хочешь проверить локально, можно создать виртуальное окружение.

Но для нашей темы это необязательно.

Без Docker тебе пришлось бы:

1. установить Python;
2. создать venv;
3. установить зависимости;
4. установить PostgreSQL;
5. создать базу;
6. создать пользователя;
7. настроить переменные окружения;
8. запустить приложение.

С Docker всё будет намного проще.

---

# 16. Dockerfile

Теперь создаём Dockerfile.

```dockerfile
FROM python:3.12-slim

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

COPY requirements.txt .

RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Теперь подробно разберём каждую строку.

---

## FROM

```dockerfile
FROM python:3.12-slim
```

Каждый Dockerfile начинается с базового образа.

Мы говорим:

> Возьми готовый образ Python 3.12 slim.

`slim` означает облегчённую версию образа.

Плюсы:

- меньше размер;
- быстрее скачивается;
- меньше лишнего.

---

## WORKDIR

```dockerfile
WORKDIR /app
```

Создаёт рабочую директорию внутри контейнера.

Все следующие команды будут выполняться внутри `/app`.

То есть внутри контейнера будет такая папка:

```text
/app
```

---

## ENV PYTHONDONTWRITEBYTECODE

```dockerfile
ENV PYTHONDONTWRITEBYTECODE=1
```

Запрещает Python создавать файлы `.pyc`.

Это не обязательно, но часто используется в Docker.

---

## ENV PYTHONUNBUFFERED

```dockerfile
ENV PYTHONUNBUFFERED=1
```

Позволяет сразу видеть логи Python в Docker.

Без этого логи иногда могут буферизоваться и появляться с задержкой.

---

## COPY requirements.txt .

```dockerfile
COPY requirements.txt .
```

Копирует `requirements.txt` с твоего компьютера внутрь контейнера в папку `/app`.

Точка означает текущую директорию.

---

## RUN pip install

```dockerfile
RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir -r requirements.txt
```

Эта команда выполняется во время сборки образа.

Она:

1. обновляет pip;
2. устанавливает зависимости из `requirements.txt`.

### Почему сначала копируем requirements.txt, а потом весь проект?

Потому что Docker использует кэш слоёв.

Если код изменился, но `requirements.txt` нет, Docker не будет заново устанавливать зависимости.

Это ускоряет сборку.

---

## COPY . .

```dockerfile
COPY . .
```

Копирует весь проект внутрь контейнера.

Первая точка — текущая папка на твоём компьютере.

Вторая точка — текущая папка внутри контейнера, то есть `/app`.

---

## EXPOSE

```dockerfile
EXPOSE 8000
```

Документирует, что приложение внутри контейнера слушает порт `8000`.

Важно: `EXPOSE` сам по себе не открывает порт наружу.

Чтобы открыть порт, нужно сделать mapping в Docker Compose:

```yaml
ports:
  - "8000:8000"
```

---

## CMD

```dockerfile
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Команда, которая запускается при старте контейнера.

Разбор:

```text
uvicorn
```

запускает сервер.

```text
app.main:app
```

означает:

- файл `app/main.py`;
- переменная `app` внутри него.

```text
--host 0.0.0.0
```

Очень важно.

Если написать:

```bash
--host 127.0.0.1
```

приложение будет доступно только внутри контейнера.

А нам нужно, чтобы оно принимало запросы извне контейнера.

Поэтому используем:

```bash
0.0.0.0
```

```text
--port 8000
```

порт внутри контейнера.

---

# 17. .dockerignore

Файл `.dockerignore` нужен, чтобы не копировать лишние файлы в образ.

```dockerignore
__pycache__
*.pyc
*.pyo
*.pyd
.Python

.env
.venv
venv
env

.git
.gitignore

.idea
.vscode

Dockerfile
docker-compose.yml

*.log
```

## Зачем нужен .dockerignore

Если не использовать `.dockerignore`, Docker будет копировать всё:

- `.git`;
- виртуальное окружение;
- кэш;
- IDE-файлы;
- логи;
- мусор.

Это:

- увеличивает размер образа;
- замедляет сборку;
- может случайно добавить секреты.

## Важный момент про .env

Мы добавили:

```dockerignore
.env
```

Это хорошо для безопасности.

Переменные окружения мы будем передавать через Docker Compose.

---

# 18. docker-compose.yml

Теперь главный файл для запуска нескольких контейнеров.

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: fastapi_app
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      - db
    volumes:
      - .:/app
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  db:
    image: postgres:16
    container_name: postgres_db
    ports:
      - "5432:5432"
    env_file:
      - .env
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Теперь подробно.

---

# 19. Разбор docker-compose.yml

## services

```yaml
services:
```

Здесь описываются контейнеры.

Каждый сервис — это будущий контейнер.

У нас два сервиса:

```yaml
app:
db:
```

---

## Сервис app

```yaml
app:
```

Это FastAPI-приложение.

---

## build

```yaml
build:
  context: .
  dockerfile: Dockerfile
```

Означает:

> Собери образ для этого сервиса из Dockerfile.

```yaml
context: .
```

Контекст сборки — текущая папка проекта.

```yaml
dockerfile: Dockerfile
```

Использовать файл с именем `Dockerfile`.

---

## container_name

```yaml
container_name: fastapi_app
```

Явно задаёт имя контейнера.

Без этого Docker Compose создаст имя автоматически, например:

```text
fastapi-docker-demo-app-1
```

С `container_name` будет проще писать команды:

```bash
docker logs fastapi_app
```

---

## ports

```yaml
ports:
  - "8000:8000"
```

Проброс портов.

Формат:

```text
HOST_PORT:CONTAINER_PORT
```

То есть:

```text
8000:8000
```

означает:

- порт 8000 на твоём компьютере;
- порт 8000 внутри контейнера.

Теперь ты можешь открыть:

```text
http://localhost:8000
```

и попасть в FastAPI внутри контейнера.

---

## env_file

```yaml
env_file:
  - .env
```

Передаёт переменные окружения из `.env` внутрь контейнера.

То есть внутри контейнера будут доступны:

```env
DATABASE_URL
POSTGRES_DB
POSTGRES_USER
POSTGRES_PASSWORD
```

---

## depends_on

```yaml
depends_on:
  - db
```

Говорит Docker Compose:

> Сначала запусти контейнер `db`, потом `app`.

Важно: `depends_on` не гарантирует, что PostgreSQL уже полностью готов принимать подключения.

Он гарантирует только порядок запуска контейнеров.

Иногда FastAPI может стартовать раньше, чем PostgreSQL будет готов.

Позже мы добавим healthcheck.

---

## volumes для app

```yaml
volumes:
  - .:/app
```

Это bind mount.

Он связывает папку на твоём компьютере с папкой внутри контейнера.

```text
.:/app
```

означает:

- текущая папка на компьютере;
- папка `/app` внутри контейнера.

Зачем это нужно?

Для разработки.

Ты меняешь код на компьютере — изменения сразу появляются внутри контейнера.

А благодаря:

```yaml
command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

сервер автоматически перезапускается.

---

## command

```yaml
command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

Переопределяет `CMD` из Dockerfile.

В Dockerfile у нас:

```dockerfile
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

А в compose для разработки мы добавили:

```bash
--reload
```

`--reload` нужен, чтобы приложение перезапускалось при изменении кода.

---

# 20. Сервис db

```yaml
db:
  image: postgres:16
```

Мы не собираем свой образ PostgreSQL.

Мы используем готовый официальный образ:

```text
postgres:16
```

---

## ports для db

```yaml
ports:
  - "5432:5432"
```

Пробрасывает PostgreSQL наружу.

Теперь можно подключиться к базе с компьютера, например через:

- DBeaver;
- DataGrip;
- pgAdmin;
- psql.

Параметры подключения:

```text
Host: localhost
Port: 5432
Database: app_db
User: app_user
Password: app_password
```

Но внутри Docker FastAPI подключается не к `localhost`, а к `db`.

---

## env_file для db

```yaml
env_file:
  - .env
```

PostgreSQL-образ использует переменные:

```env
POSTGRES_DB
POSTGRES_USER
POSTGRES_PASSWORD
```

При первом запуске контейнера он:

- создаст базу `app_db`;
- создаст пользователя `app_user`;
- задаст пароль `app_password`.

---

## volumes для db

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

Это named volume.

Он сохраняет данные PostgreSQL вне контейнера.

Почему это важно?

Контейнер можно удалить:

```bash
docker rm postgres_db
```

Но данные останутся в volume.

Если volume не использовать, данные могут потеряться при пересоздании контейнера.

---

## volumes внизу файла

```yaml
volumes:
  postgres_data:
```

Здесь объявляется named volume.

Docker сам создаст его и будет хранить данные.

---

# 21. Запуск проекта

Выполни:

```bash
docker compose up --build
```

Что делает команда:

```bash
docker compose up
```

запускает сервисы из `docker-compose.yml`.

```bash
--build
```

говорит пересобрать image перед запуском.

После запуска открой:

```text
http://localhost:8000
```

Ожидаемый ответ:

```json
{
  "message": "Hello from FastAPI inside Docker!"
}
```

Документация FastAPI:

```text
http://localhost:8000/docs
```

---

# 22. Проверка API

## Создать пользователя

POST:

```text
http://localhost:8000/users
```

Body:

```json
{
  "name": "Ivan",
  "email": "ivan@example.com"
}
```

Ответ:

```json
{
  "id": 1,
  "name": "Ivan",
  "email": "ivan@example.com"
}
```

## Получить пользователей

GET:

```text
http://localhost:8000/users
```

Ответ:

```json
[
  {
    "id": 1,
    "name": "Ivan",
    "email": "ivan@example.com"
  }
]
```

---

# 23. Основные команды Docker Compose

## Запустить проект

```bash
docker compose up
```

## Запустить с пересборкой

```bash
docker compose up --build
```

## Запустить в фоне

```bash
docker compose up -d
```

`-d` означает detached mode.

Контейнеры работают в фоне.

## Остановить контейнеры

```bash
docker compose down
```

## Остановить и удалить volumes

```bash
docker compose down -v
```

Осторожно: это удалит данные PostgreSQL.

## Посмотреть логи

```bash
docker compose logs
```

Логи конкретного сервиса:

```bash
docker compose logs app
```

```bash
docker compose logs db
```

Следить за логами:

```bash
docker compose logs -f app
```

## Выполнить команду внутри контейнера

```bash
docker compose exec app bash
```

Если bash нет:

```bash
docker compose exec app sh
```

## Выполнить Python внутри контейнера

```bash
docker compose exec app python
```

## Выполнить psql внутри контейнера PostgreSQL

```bash
docker compose exec db psql -U app_user -d app_db
```

---

# 24. Проверка базы данных

Зайди в PostgreSQL:

```bash
docker compose exec db psql -U app_user -d app_db
```

Посмотреть таблицы:

```sql
\dt
```

Посмотреть пользователей:

```sql
SELECT * FROM users;
```

Выйти:

```sql
\q
```

---

# 25. Docker image подробно

Когда ты выполняешь:

```bash
docker compose up --build
```

Docker собирает image для FastAPI.

Посмотреть images:

```bash
docker images
```

Ты увидишь что-то вроде:

```text
REPOSITORY                 TAG       IMAGE ID       SIZE
fastapi-docker-demo-app    latest    abc123         180MB
postgres                   16        def456         430MB
python                     3.12-slim ghi789         120MB
```

## Образ состоит из слоёв

Каждая инструкция Dockerfile создаёт слой:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install ...
COPY . .
```

Docker кэширует слои.

Если поменялся только `main.py`, Docker не будет заново устанавливать зависимости.

---

# 26. Container подробно

Container — это запущенный image.

Посмотреть контейнеры:

```bash
docker ps
```

Пример:

```text
CONTAINER ID   IMAGE              NAME          PORTS
abc123         demo-app           fastapi_app   0.0.0.0:8000->8000/tcp
def456         postgres:16        postgres_db   0.0.0.0:5432->5432/tcp
```

## Жизненный цикл контейнера

Контейнер может быть:

- created;
- running;
- stopped;
- exited;
- restarting.

Остановить:

```bash
docker stop fastapi_app
```

Запустить снова:

```bash
docker start fastapi_app
```

Удалить:

```bash
docker rm fastapi_app
```

---

# 27. Почему данные БД не должны жить внутри контейнера

Контейнеры считаются временными.

Их можно:

- остановить;
- удалить;
- пересоздать;
- обновить.

Если данные базы хранятся только внутри контейнера, они могут быть потеряны.

Поэтому для PostgreSQL используют volume:

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

---

# 28. Виды volumes

## 1. Named volume

```yaml
volumes:
  - postgres_data:/var/lib/postgresql/data
```

Docker сам управляет этим volume.

Подходит для баз данных.

## 2. Bind mount

```yaml
volumes:
  - .:/app
```

Связывает папку хоста с контейнером.

Подходит для разработки.

## Главное отличие

Named volume:

- управляется Docker;
- хорошо для данных;
- путь на хосте не важен.

Bind mount:

- управляется тобой;
- удобно для кода;
- зависит от структуры файлов на хосте.

---

# 29. Docker networks

Docker Compose автоматически создаёт сеть для сервисов.

Например:

```text
fastapi-docker-demo_default
```

Все сервисы внутри compose находятся в одной сети.

Поэтому `app` может обращаться к `db` по имени:

```text
db
```

Пример:

```env
DATABASE_URL=postgresql://app_user:app_password@db:5432/app_db
```

## Внутри Docker-сети

```text
app -> db:5432
```

## Снаружи, с твоего компьютера

```text
localhost:5432 -> db:5432
```

Это очень важно понимать.

---

# 30. Улучшаем docker-compose: healthcheck

Как мы говорили, `depends_on` не ждёт полной готовности PostgreSQL.

Добавим healthcheck.

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: fastapi_app
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - .:/app
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  db:
    image: postgres:16
    container_name: postgres_db
    ports:
      - "5432:5432"
    env_file:
      - .env
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app_user -d app_db"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

## Разбор healthcheck

```yaml
healthcheck:
```

Проверка состояния контейнера.

```yaml
test: ["CMD-SHELL", "pg_isready -U app_user -d app_db"]
```

Команда проверяет, готов ли PostgreSQL принимать подключения.

```yaml
interval: 5s
```

Проверять каждые 5 секунд.

```yaml
timeout: 5s
```

Ждать ответ максимум 5 секунд.

```yaml
retries: 5
```

Если 5 раз подряд не получилось — контейнер считается unhealthy.

---

# 31. Alembic: миграции базы данных

В учебном примере мы использовали:

```python
Base.metadata.create_all(bind=engine)
```

Но в реальных проектах так делать плохо.

Почему?

Потому что `create_all`:

- создаёт таблицы;
- но плохо управляет изменениями;
- не умеет нормально изменять существующие таблицы;
- не хранит историю изменений БД.

Для реального проекта используют Alembic.

## Добавим Alembic

В `requirements.txt` добавь:

```txt
alembic==1.14.0
```

Полный файл:

```txt
fastapi==0.115.6
uvicorn[standard]==0.32.1
sqlalchemy==2.0.36
psycopg2-binary==2.9.10
python-dotenv==1.0.1
email-validator==2.2.0
alembic==1.14.0
```

Пересобери контейнер:

```bash
docker compose up --build
```

В другом терминале выполни:

```bash
docker compose exec app alembic init alembic
```

Появится структура:

```text
alembic/
├── versions/
├── env.py
├── README
└── script.py.mako

alembic.ini
```

---

## Настройка alembic.ini

Найди строку:

```ini
sqlalchemy.url = driver://user:pass@localhost/dbname
```

Можно заменить на:

```ini
sqlalchemy.url = postgresql://app_user:app_password@db:5432/app_db
```

Но лучше брать из переменной окружения.

---

## Настройка alembic/env.py

В `alembic/env.py` нужно импортировать metadata моделей.

Примерно найди:

```python
target_metadata = None
```

Замени на:

```python
from app.database import Base
from app.models import User

target_metadata = Base.metadata
```

Также можно настроить DATABASE_URL из env.

---

## Создать миграцию

```bash
docker compose exec app alembic revision --autogenerate -m "create users table"
```

Применить миграции:

```bash
docker compose exec app alembic upgrade head
```

После этого можно убрать из `main.py`:

```python
Base.metadata.create_all(bind=engine)
```

---

# 32. Dev и Prod режимы

Для разработки мы используем:

```yaml
volumes:
  - .:/app
command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

Это удобно, потому что:

- код обновляется без пересборки;
- `--reload` перезапускает сервер.

Но для production это плохо.

Почему?

- bind mount не нужен;
- `--reload` тратит ресурсы;
- лучше запускать несколько workers;
- не стоит монтировать исходники;
- не стоит открывать PostgreSQL наружу.

---

# 33. Пример docker-compose.prod.yml

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: fastapi_app_prod
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
    restart: unless-stopped
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000

  db:
    image: postgres:16
    container_name: postgres_db_prod
    env_file:
      - .env
    volumes:
      - postgres_data_prod:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app_user -d app_db"]
      interval: 5s
      timeout: 5s
      retries: 5
    restart: unless-stopped

volumes:
  postgres_data_prod:
```

## Отличия от dev

В production:

- нет `volumes: .:/app`;
- нет `--reload`;
- у базы нет `ports: "5432:5432"`;
- добавлен `restart: unless-stopped`.

## Почему у базы нет ports

Если PostgreSQL не нужен снаружи сервера, не открывай его наружу.

FastAPI всё равно сможет подключиться к PostgreSQL через Docker-сеть:

```text
db:5432
```

А извне база будет недоступна.

Это безопаснее.

---

# 34. restart policies

```yaml
restart: unless-stopped
```

Означает:

> Перезапускать контейнер, если он упал, кроме случая, когда его остановили вручную.

Другие варианты:

```yaml
restart: "no"
```

Не перезапускать.

```yaml
restart: always
```

Всегда перезапускать.

```yaml
restart: on-failure
```

Перезапускать только при ошибке.

---

# 35. Переменные окружения

Переменные окружения нужны, чтобы не хардкодить настройки.

Плохо:

```python
DATABASE_URL = "postgresql://app_user:app_password@db:5432/app_db"
```

Лучше:

```python
DATABASE_URL = os.getenv("DATABASE_URL")
```

Почему?

Потому что в разных окружениях могут быть разные настройки:

- dev database;
- test database;
- prod database.

---

# 36. Безопасность .env

Нельзя коммитить реальные секреты в Git.

В `.gitignore` обычно добавляют:

```gitignore
.env
```

А рядом создают пример:

```text
.env.example
```

Пример `.env.example`:

```env
POSTGRES_DB=app_db
POSTGRES_USER=app_user
POSTGRES_PASSWORD=change_me
POSTGRES_HOST=db
POSTGRES_PORT=5432

DATABASE_URL=postgresql://app_user:change_me@db:5432/app_db
```

В репозиторий коммитят `.env.example`, но не `.env`.

---

# 37. Сборка образа вручную

Можно собрать image без Compose:

```bash
docker build -t fastapi-demo .
```

Разбор:

```bash
docker build
```

собрать образ.

```bash
-t fastapi-demo
```

дать имя образу.

```bash
.
```

контекст сборки — текущая папка.

Запустить:

```bash
docker run -p 8000:8000 fastapi-demo
```

Но база данных не запустится автоматически.

Поэтому для проектов с несколькими сервисами удобнее Docker Compose.

---

# 38. docker run: важные флаги

## -p

```bash
docker run -p 8000:8000 fastapi-demo
```

Проброс портов.

## -d

```bash
docker run -d fastapi-demo
```

Запуск в фоне.

## --name

```bash
docker run --name my_app fastapi-demo
```

Имя контейнера.

## --env

```bash
docker run -e DATABASE_URL=postgresql://... fastapi-demo
```

Передать переменную окружения.

## --env-file

```bash
docker run --env-file .env fastapi-demo
```

Передать env-файл.

## -v

```bash
docker run -v .:/app fastapi-demo
```

Смонтировать volume.

---

# 39. Частые ошибки и решения

## Ошибка 1: connection refused

Пример:

```text
connection refused
```

Причины:

1. PostgreSQL ещё не успел запуститься.
2. Неправильный host.
3. Неправильный порт.
4. База упала.

Решения:

- добавь healthcheck;
- проверь `DATABASE_URL`;
- внутри Docker используй `db`, а не `localhost`;
- проверь логи:

```bash
docker compose logs db
```

---

## Ошибка 2: password authentication failed

```text
password authentication failed for user
```

Причины:

- неправильный пароль;
- volume уже создан со старым паролем.

Важно: PostgreSQL использует `POSTGRES_USER` и `POSTGRES_PASSWORD` только при первом создании базы.

Если ты потом поменял `.env`, старый volume уже содержит старые настройки.

Решение для dev:

```bash
docker compose down -v
docker compose up --build
```

Осторожно: это удалит данные базы.

---

## Ошибка 3: port is already allocated

```text
Bind for 0.0.0.0:5432 failed: port is already allocated
```

Означает, что порт уже занят.

Например, у тебя локально установлен PostgreSQL.

Решение:

В `docker-compose.yml` поменять порт хоста:

```yaml
ports:
  - "5433:5432"
```

Теперь с компьютера подключение будет:

```text
localhost:5433
```

Но внутри Docker всё равно:

```text
db:5432
```

---

## Ошибка 4: module not found

```text
ModuleNotFoundError: No module named 'app'
```

Причины:

- неправильный WORKDIR;
- неправильный путь в uvicorn;
- нет `__init__.py`, если требуется;
- проект скопирован не туда.

Проверь:

```dockerfile
WORKDIR /app
COPY . .
CMD ["uvicorn", "app.main:app", ...]
```

---

## Ошибка 5: изменения кода не применяются

Причины:

- нет bind mount;
- нет `--reload`;
- контейнер нужно пересобрать.

Для dev должно быть:

```yaml
volumes:
  - .:/app
command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

---

# 40. Debugging Docker

## Посмотреть логи

```bash
docker compose logs -f app
```

## Зайти внутрь app

```bash
docker compose exec app bash
```

Внутри можно проверить:

```bash
pwd
ls -la
env
python --version
pip list
```

## Проверить переменные окружения

```bash
docker compose exec app env
```

Или конкретную:

```bash
docker compose exec app printenv DATABASE_URL
```

## Проверить сеть

Зайти в app:

```bash
docker compose exec app bash
```

Попробовать подключиться к db:

```bash
python
```

```python
import socket
socket.gethostbyname("db")
```

Если вернулся IP — имя `db` резолвится.

---

# 41. Очистка Docker

Со временем Docker занимает много места.

## Удалить остановленные контейнеры

```bash
docker container prune
```

## Удалить неиспользуемые образы

```bash
docker image prune
```

## Удалить неиспользуемые volumes

```bash
docker volume prune
```

Осторожно: можно удалить данные БД.

## Полная очистка

```bash
docker system prune -a
```

Очень осторожно.

Удаляет много всего неиспользуемого.

---

# 42. Best practices Dockerfile

## 1. Используй slim-образы

Хорошо:

```dockerfile
FROM python:3.12-slim
```

Плохо:

```dockerfile
FROM python:latest
```

Почему плохо `latest`?

Потому что сегодня это одна версия, завтра другая.

Проект может неожиданно сломаться.

---

## 2. Сначала копируй requirements.txt

Хорошо:

```dockerfile
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

Плохо:

```dockerfile
COPY . .
RUN pip install -r requirements.txt
```

Первый вариант лучше использует кэш.

---

## 3. Используй .dockerignore

Чтобы не тащить мусор в image.

---

## 4. Не храни секреты в Dockerfile

Плохо:

```dockerfile
ENV POSTGRES_PASSWORD=my_super_secret_password
```

Лучше использовать `.env` или секреты в CI/CD.

---

## 5. Не запускай dev-команды в production

Для production не используй:

```bash
--reload
```

---

# 43. Более правильный Dockerfile для production

Можно улучшить Dockerfile:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

RUN addgroup --system appgroup && adduser --system --group appuser

COPY requirements.txt .

RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir -r requirements.txt

COPY . .

USER appuser

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## Что изменилось

```dockerfile
RUN addgroup --system appgroup && adduser --system --group appuser
```

Создаём отдельного пользователя.

```dockerfile
USER appuser
```

Запускаем приложение не от root.

Это безопаснее.

---

# 44. Что должен понимать Junior+ по Docker

К уровню уверенного Junior+ ты должен понимать:

## Базовые понятия

- что такое image;
- что такое container;
- что такое Dockerfile;
- что такое Docker Compose;
- что такое volume;
- что такое network;
- что такое port mapping;
- что такое environment variables.

## Dockerfile

Ты должен уметь объяснить:

- `FROM`;
- `WORKDIR`;
- `COPY`;
- `RUN`;
- `ENV`;
- `EXPOSE`;
- `CMD`;
- разницу между `CMD` и `RUN`;
- зачем нужен `.dockerignore`;
- как работает кэш слоёв.

## Compose

Ты должен понимать:

- `services`;
- `build`;
- `image`;
- `ports`;
- `volumes`;
- `env_file`;
- `depends_on`;
- `healthcheck`;
- `restart`;
- named volumes;
- bind mounts.

## Практика

Ты должен уметь:

- собрать image;
- запустить контейнер;
- посмотреть логи;
- зайти внутрь контейнера;
- выполнить команду внутри контейнера;
- поднять FastAPI + PostgreSQL;
- подключиться к базе;
- понять ошибку `connection refused`;
- понять проблему с `localhost`;
- удалить контейнеры и volumes;
- отличать dev compose от prod compose.

---

# 45. Финальная версия файлов

## requirements.txt

```txt
fastapi==0.115.6
uvicorn[standard]==0.32.1
sqlalchemy==2.0.36
psycopg2-binary==2.9.10
python-dotenv==1.0.1
email-validator==2.2.0
alembic==1.14.0
```

## .env

```env
POSTGRES_DB=app_db
POSTGRES_USER=app_user
POSTGRES_PASSWORD=app_password
POSTGRES_HOST=db
POSTGRES_PORT=5432

DATABASE_URL=postgresql://app_user:app_password@db:5432/app_db
```

## Dockerfile

```dockerfile
FROM python:3.12-slim

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

COPY requirements.txt .

RUN pip install --no-cache-dir --upgrade pip \
    && pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

## docker-compose.yml

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: fastapi_app
    ports:
      - "8000:8000"
    env_file:
      - .env
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - .:/app
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

  db:
    image: postgres:16
    container_name: postgres_db
    ports:
      - "5432:5432"
    env_file:
      - .env
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app_user -d app_db"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

## .dockerignore

```dockerignore
__pycache__
*.pyc
*.pyo
*.pyd
.Python

.env
.venv
venv
env

.git
.gitignore

.idea
.vscode

*.log
```

## app/database.py

```python
import os

from sqlalchemy import create_engine
from sqlalchemy.orm import declarative_base, sessionmaker


DATABASE_URL = os.getenv(
    "DATABASE_URL",
    "postgresql://app_user:app_password@db:5432/app_db"
)

engine = create_engine(DATABASE_URL)

SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine
)

Base = declarative_base()


def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

## app/models.py

```python
from sqlalchemy import Column, Integer, String

from app.database import Base


class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String(100), nullable=False)
    email = Column(String(255), unique=True, index=True, nullable=False)
```

## app/schemas.py

```python
from pydantic import BaseModel, EmailStr


class UserCreate(BaseModel):
    name: str
    email: EmailStr


class UserRead(BaseModel):
    id: int
    name: str
    email: EmailStr

    class Config:
        from_attributes = True
```

## app/main.py

```python
from fastapi import Depends, FastAPI, HTTPException
from sqlalchemy.orm import Session

from app.database import Base, engine, get_db
from app.models import User
from app.schemas import UserCreate, UserRead


Base.metadata.create_all(bind=engine)

app = FastAPI(
    title="FastAPI Docker PostgreSQL Demo",
    version="1.0.0"
)


@app.get("/")
def root():
    return {"message": "Hello from FastAPI inside Docker!"}


@app.get("/health")
def health():
    return {"status": "ok"}


@app.post("/users", response_model=UserRead)
def create_user(user_data: UserCreate, db: Session = Depends(get_db)):
    existing_user = (
        db.query(User)
        .filter(User.email == user_data.email)
        .first()
    )

    if existing_user:
        raise HTTPException(
            status_code=400,
            detail="User with this email already exists"
        )

    user = User(
        name=user_data.name,
        email=user_data.email
    )

    db.add(user)
    db.commit()
    db.refresh(user)

    return user


@app.get("/users", response_model=list[UserRead])
def get_users(db: Session = Depends(get_db)):
    users = db.query(User).all()
    return users


@app.get("/users/{user_id}", response_model=UserRead)
def get_user(user_id: int, db: Session = Depends(get_db)):
    user = (
        db.query(User)
        .filter(User.id == user_id)
        .first()
    )

    if not user:
        raise HTTPException(
            status_code=404,
            detail="User not found"
        )

    return user
```

---

# 46. Главная шпаргалка

```bash
docker compose up --build
```

Запустить проект с пересборкой.

```bash
docker compose up -d
```

Запустить в фоне.

```bash
docker compose down
```

Остановить проект.

```bash
docker compose down -v
```

Остановить и удалить данные volumes.

```bash
docker compose logs -f app
```

Смотреть логи FastAPI.

```bash
docker compose logs -f db
```

Смотреть логи PostgreSQL.

```bash
docker compose exec app bash
```

Зайти в FastAPI-контейнер.

```bash
docker compose exec db psql -U app_user -d app_db
```

Зайти в PostgreSQL.

```bash
docker ps
```

Посмотреть запущенные контейнеры.

```bash
docker images
```

Посмотреть образы.

```bash
docker volume ls
```

Посмотреть volumes.

---

# 47. Самая важная идея Docker

Docker не заменяет приложение.

Docker создаёт предсказуемую среду, в которой приложение запускается одинаково везде.

Если коротко:

```text
Dockerfile описывает, как собрать приложение.
Image — собранный шаблон.
Container — запущенный image.
Docker Compose запускает несколько контейнеров вместе.
Volume хранит данные.
Network соединяет контейнеры.
.env передаёт настройки.
```

Если ты это понял и можешь поднять FastAPI + PostgreSQL, посмотреть логи, зайти внутрь контейнера, исправить ошибку подключения и объяснить разницу между `localhost` и `db`, ты уже владеешь Docker на хорошем junior/junior+ уровне.
