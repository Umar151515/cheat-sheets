# FastAPI + Pydantic + SQLAlchemy 2 + PostgreSQL с нуля до уверенного junior+

Ниже — подробная практическая документация, построенная вокруг реального проекта.

Мы сделаем API для приложения задач:

- регистрация пользователя;
- логин;
- JWT-авторизация;
- получение текущего пользователя;
- создание задач;
- просмотр своих задач;
- обновление задач;
- удаление задач;
- работа с PostgreSQL через SQLAlchemy 2;
- валидация данных через Pydantic v2;
- миграции через Alembic.

Стек:

```text
Python 3.11+
FastAPI
Pydantic v2
SQLAlchemy 2
PostgreSQL
Alembic
JWT
Docker Compose
```

---

# 1. Что такое FastAPI

**FastAPI** — это современный Python-фреймворк для создания API.

API — это интерфейс, через который клиент, например frontend, мобильное приложение или другой сервер, общается с твоим backend.

Пример:

```http
GET /users/1
```

Ответ:

```json
{
  "id": 1,
  "email": "user@example.com"
}
```

FastAPI удобен тем, что:

- автоматически валидирует входные данные;
- автоматически генерирует Swagger-документацию;
- использует type hints Python;
- хорошо работает с Pydantic;
- поддерживает асинхронность;
- быстро работает;
- удобно строить реальные backend-приложения.

---

# 2. Что такое Pydantic

**Pydantic** — библиотека для проверки и преобразования данных.

Например, пользователь отправляет JSON:

```json
{
  "email": "test@example.com",
  "password": "123456"
}
```

Мы хотим проверить:

- email действительно похож на email;
- пароль не пустой;
- пароль минимум 6 символов.

Pydantic позволяет описать схему:

```python
from pydantic import BaseModel, EmailStr, Field


class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(min_length=6)
```

Теперь FastAPI сам проверит данные. Если клиент отправит неправильный email, FastAPI вернёт ошибку `422 Unprocessable Entity`.

---

# 3. Что такое SQLAlchemy

**SQLAlchemy** — ORM и инструмент для работы с базами данных.

ORM означает Object Relational Mapping.

То есть вместо того, чтобы постоянно писать SQL руками:

```sql
SELECT * FROM users WHERE id = 1;
```

Мы можем работать с Python-объектами:

```python
user = await session.get(User, 1)
```

SQLAlchemy превращает Python-код в SQL-запросы.

---

# 4. Что такое PostgreSQL

**PostgreSQL** — популярная реляционная база данных.

Реляционная база хранит данные в таблицах.

Например таблица `users`:

| id | email            | hashed_password |
|----|------------------|-----------------|
| 1  | test@example.com | ...             |

И таблица `todos`:

| id | title        | completed | owner_id |
|----|--------------|-----------|----------|
| 1  | Learn FastAPI| false     | 1        |

---

# 5. Что мы будем делать

Мы создадим backend API:

```text
POST   /auth/register      регистрация
POST   /auth/login         логин
GET    /auth/me            текущий пользователь

POST   /todos              создать задачу
GET    /todos              получить мои задачи
GET    /todos/{todo_id}    получить одну задачу
PATCH  /todos/{todo_id}    обновить задачу
DELETE /todos/{todo_id}    удалить задачу
```

---

# 6. Установка проекта

Создай папку:

```bash
mkdir fastapi_junior_project
cd fastapi_junior_project
```

Создай виртуальное окружение:

```bash
python -m venv venv
```

Активируй его.

Linux/macOS:

```bash
source venv/bin/activate
```

Windows PowerShell:

```bash
venv\Scripts\Activate.ps1
```

---

# 7. Установка зависимостей

Создай файл `requirements.txt`:

```txt
fastapi
uvicorn[standard]

sqlalchemy
asyncpg
alembic

pydantic
pydantic-settings
email-validator

python-dotenv

passlib[bcrypt]
PyJWT
python-multipart
```

Установи зависимости:

```bash
pip install -r requirements.txt
```

Зачем нужны эти библиотеки:

| Библиотека | Зачем нужна |
|---|---|
| fastapi | сам web-фреймворк |
| uvicorn | сервер для запуска FastAPI |
| sqlalchemy | работа с БД |
| asyncpg | async-драйвер PostgreSQL |
| alembic | миграции БД |
| pydantic | схемы и валидация |
| pydantic-settings | настройки через `.env` |
| email-validator | проверка email |
| passlib[bcrypt] | хеширование паролей |
| PyJWT | JWT-токены |
| python-multipart | нужно для OAuth2PasswordRequestForm |

---

# 8. Запуск PostgreSQL через Docker

Создай файл `docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:16
    container_name: fastapi_postgres
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: fastapi_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Запусти PostgreSQL:

```bash
docker compose up -d
```

Проверь контейнеры:

```bash
docker ps
```

---

# 9. Структура проекта

Сделаем такую структуру:

```text
fastapi_junior_project/
│
├── app/
│   ├── main.py
│   │
│   ├── core/
│   │   ├── config.py
│   │   └── security.py
│   │
│   ├── db/
│   │   ├── base.py
│   │   └── session.py
│   │
│   ├── models/
│   │   ├── user.py
│   │   └── todo.py
│   │
│   ├── schemas/
│   │   ├── user.py
│   │   ├── auth.py
│   │   └── todo.py
│   │
│   ├── api/
│   │   ├── deps.py
│   │   └── routes/
│   │       ├── auth.py
│   │       └── todos.py
│   │
│   └── crud/
│       ├── user.py
│       └── todo.py
│
├── alembic/
├── alembic.ini
├── docker-compose.yml
├── requirements.txt
└── .env
```

---

# 10. Переменные окружения

Создай `.env`:

```env
APP_NAME=FastAPI Junior Project
DEBUG=True

DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/fastapi_db

JWT_SECRET_KEY=super-secret-key-change-me
JWT_ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=60
```

Важно:

```text
postgresql+asyncpg://user:password@host:port/db_name
```

`+asyncpg` означает, что SQLAlchemy будет работать асинхронно через драйвер asyncpg.

---

# 11. Настройки приложения

Создай файл `app/core/config.py`:

```python
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    APP_NAME: str
    DEBUG: bool = False

    DATABASE_URL: str

    JWT_SECRET_KEY: str
    JWT_ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 60

    model_config = SettingsConfigDict(
        env_file=".env",
        env_file_encoding="utf-8",
    )


settings = Settings()
```

Что здесь происходит:

```python
class Settings(BaseSettings):
```

Мы создаём класс настроек. Pydantic сам прочитает переменные из `.env`.

Например:

```env
DATABASE_URL=...
```

попадёт сюда:

```python
DATABASE_URL: str
```

Теперь в любом файле можно импортировать:

```python
from app.core.config import settings
```

и использовать:

```python
settings.DATABASE_URL
```

---

# 12. Подключение SQLAlchemy

Создай `app/db/session.py`:

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession

from app.core.config import settings


engine = create_async_engine(
    settings.DATABASE_URL,
    echo=settings.DEBUG,
)

AsyncSessionLocal = async_sessionmaker(
    bind=engine,
    class_=AsyncSession,
    expire_on_commit=False,
)
```

Разберём подробно.

## engine

```python
engine = create_async_engine(...)
```

`engine` — это объект SQLAlchemy, который знает, как подключаться к базе данных.

## echo

```python
echo=settings.DEBUG
```

Если `DEBUG=True`, SQLAlchemy будет показывать SQL-запросы в консоли.

Это удобно при обучении.

## async_sessionmaker

```python
AsyncSessionLocal = async_sessionmaker(...)
```

Это фабрика сессий.

Сессия — это объект, через который мы выполняем запросы:

```python
result = await session.execute(...)
```

---

# 13. Базовая модель SQLAlchemy

Создай `app/db/base.py`:

```python
from sqlalchemy.orm import DeclarativeBase


class Base(DeclarativeBase):
    pass
```

Все модели SQLAlchemy будут наследоваться от `Base`.

Например:

```python
class User(Base):
    ...
```

SQLAlchemy через `Base` понимает, какие таблицы есть в проекте.

---

# 14. Модель пользователя

Создай `app/models/user.py`:

```python
from datetime import datetime

from sqlalchemy import String, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db.base import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)

    email: Mapped[str] = mapped_column(
        String(255),
        unique=True,
        index=True,
        nullable=False,
    )

    hashed_password: Mapped[str] = mapped_column(
        String(255),
        nullable=False,
    )

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False,
    )

    todos = relationship(
        "Todo",
        back_populates="owner",
        cascade="all, delete-orphan",
    )
```

Разберём.

```python
__tablename__ = "users"
```

Имя таблицы в PostgreSQL будет `users`.

```python
id: Mapped[int] = mapped_column(primary_key=True)
```

Поле `id` — первичный ключ.

```python
email: Mapped[str]
```

Поле email будет строкой.

```python
unique=True
```

Два пользователя не могут иметь одинаковый email.

```python
nullable=False
```

Поле обязательно.

```python
todos = relationship(...)
```

У пользователя может быть много задач.

---

# 15. Модель задачи

Создай `app/models/todo.py`:

```python
from datetime import datetime

from sqlalchemy import String, Boolean, DateTime, ForeignKey, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db.base import Base


class Todo(Base):
    __tablename__ = "todos"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)

    title: Mapped[str] = mapped_column(
        String(200),
        nullable=False,
    )

    description: Mapped[str | None] = mapped_column(
        String(1000),
        nullable=True,
    )

    completed: Mapped[bool] = mapped_column(
        Boolean,
        default=False,
        nullable=False,
    )

    owner_id: Mapped[int] = mapped_column(
        ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
        index=True,
    )

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False,
    )

    updated_at: Mapped[datetime | None] = mapped_column(
        DateTime(timezone=True),
        onupdate=func.now(),
    )

    owner = relationship(
        "User",
        back_populates="todos",
    )
```

Здесь важно:

```python
owner_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
```

Это связь задачи с пользователем.

Одна задача принадлежит одному пользователю.

---

# 16. Импорт моделей для Alembic

Создай или измени `app/models/__init__.py`:

```python
from app.models.user import User
from app.models.todo import Todo

__all__ = ["User", "Todo"]
```

---

# 17. Pydantic-схемы

Важно понять разницу:

## SQLAlchemy-модель

Это описание таблицы базы данных.

```python
class User(Base):
    ...
```

## Pydantic-схема

Это описание данных на входе и выходе API.

```python
class UserCreate(BaseModel):
    ...
```

Обычно не стоит отдавать SQLAlchemy-модель напрямую клиенту.

Например, в таблице есть:

```python
hashed_password
```

Но клиенту нельзя показывать хеш пароля.

Поэтому мы создаём отдельные схемы.

---

# 18. Схемы пользователя

Создай `app/schemas/user.py`:

```python
from datetime import datetime

from pydantic import BaseModel, EmailStr, Field, ConfigDict


class UserCreate(BaseModel):
    email: EmailStr
    password: str = Field(min_length=6, max_length=100)


class UserRead(BaseModel):
    id: int
    email: EmailStr
    created_at: datetime

    model_config = ConfigDict(from_attributes=True)
```

Разберём.

```python
class UserCreate(BaseModel):
```

Эта схема используется при регистрации.

Клиент отправляет:

```json
{
  "email": "user@example.com",
  "password": "123456"
}
```

```python
EmailStr
```

Проверяет, что строка похожа на email.

```python
Field(min_length=6)
```

Пароль минимум 6 символов.

```python
class UserRead(BaseModel):
```

Эта схема используется для ответа клиенту.

Мы отдаём:

```json
{
  "id": 1,
  "email": "user@example.com",
  "created_at": "2025-01-01T10:00:00"
}
```

```python
model_config = ConfigDict(from_attributes=True)
```

Это нужно, чтобы Pydantic мог преобразовывать SQLAlchemy-объекты в JSON.

---

# 19. Схемы авторизации

Создай `app/schemas/auth.py`:

```python
from pydantic import BaseModel


class Token(BaseModel):
    access_token: str
    token_type: str = "bearer"
```

Когда пользователь логинится, мы вернём:

```json
{
  "access_token": "jwt-token-here",
  "token_type": "bearer"
}
```

---

# 20. Схемы задач

Создай `app/schemas/todo.py`:

```python
from datetime import datetime

from pydantic import BaseModel, Field, ConfigDict


class TodoCreate(BaseModel):
    title: str = Field(min_length=1, max_length=200)
    description: str | None = Field(default=None, max_length=1000)


class TodoUpdate(BaseModel):
    title: str | None = Field(default=None, min_length=1, max_length=200)
    description: str | None = Field(default=None, max_length=1000)
    completed: bool | None = None


class TodoRead(BaseModel):
    id: int
    title: str
    description: str | None
    completed: bool
    owner_id: int
    created_at: datetime
    updated_at: datetime | None

    model_config = ConfigDict(from_attributes=True)
```

Разница:

## TodoCreate

Используется при создании задачи.

```json
{
  "title": "Learn FastAPI",
  "description": "Read docs and build project"
}
```

## TodoUpdate

Используется при частичном обновлении.

Например можно отправить только:

```json
{
  "completed": true
}
```

## TodoRead

Используется для ответа клиенту.

---

# 21. Безопасность: хеширование паролей и JWT

Создай `app/core/security.py`:

```python
from datetime import datetime, timedelta, timezone

import jwt
from passlib.context import CryptContext

from app.core.config import settings


pwd_context = CryptContext(
    schemes=["bcrypt"],
    deprecated="auto",
)


def hash_password(password: str) -> str:
    return pwd_context.hash(password)


def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)


def create_access_token(subject: str) -> str:
    expire = datetime.now(timezone.utc) + timedelta(
        minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES
    )

    payload = {
        "sub": subject,
        "exp": expire,
    }

    token = jwt.encode(
        payload,
        settings.JWT_SECRET_KEY,
        algorithm=settings.JWT_ALGORITHM,
    )

    return token


def decode_access_token(token: str) -> dict:
    payload = jwt.decode(
        token,
        settings.JWT_SECRET_KEY,
        algorithms=[settings.JWT_ALGORITHM],
    )

    return payload
```

Разберём.

## Почему нельзя хранить пароль как есть

Плохо:

```text
password = "123456"
```

Если базу украдут, все пароли станут известны.

Правильно:

```text
hashed_password = "$2b$12$..."
```

Хеш нельзя просто так превратить обратно в пароль.

## JWT

JWT — это токен, который клиент хранит у себя и отправляет на защищённые эндпоинты.

Пример заголовка:

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

Внутри токена мы храним:

```json
{
  "sub": "1",
  "exp": "..."
}
```

`sub` — subject, то есть идентификатор пользователя.

---

# 22. Dependency для сессии БД

Создай `app/api/deps.py`:

```python
from typing import AsyncGenerator

import jwt
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.ext.asyncio import AsyncSession

from app.core.security import decode_access_token
from app.db.session import AsyncSessionLocal
from app.models.user import User
from app.crud.user import get_user_by_id


oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/login")


async def get_db_session() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSessionLocal() as session:
        yield session


async def get_current_user(
    token: str = Depends(oauth2_scheme),
    session: AsyncSession = Depends(get_db_session),
) -> User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )

    try:
        payload = decode_access_token(token)
        user_id = payload.get("sub")

        if user_id is None:
            raise credentials_exception

    except jwt.PyJWTError:
        raise credentials_exception

    user = await get_user_by_id(session, int(user_id))

    if user is None:
        raise credentials_exception

    return user
```

Разберём.

## Depends

```python
session: AsyncSession = Depends(get_db_session)
```

FastAPI сам вызовет функцию `get_db_session`, получит сессию и передаст её в эндпоинт.

Это называется dependency injection.

## get_current_user

Эта функция:

1. получает токен из заголовка `Authorization`;
2. расшифровывает JWT;
3. достаёт `user_id`;
4. ищет пользователя в базе;
5. возвращает пользователя.

Если токен неправильный — ошибка `401`.

---

# 23. CRUD для пользователя

Создай `app/crud/user.py`:

```python
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.core.security import hash_password, verify_password
from app.models.user import User
from app.schemas.user import UserCreate


async def get_user_by_id(
    session: AsyncSession,
    user_id: int,
) -> User | None:
    return await session.get(User, user_id)


async def get_user_by_email(
    session: AsyncSession,
    email: str,
) -> User | None:
    stmt = select(User).where(User.email == email)
    result = await session.execute(stmt)
    return result.scalar_one_or_none()


async def create_user(
    session: AsyncSession,
    user_in: UserCreate,
) -> User:
    user = User(
        email=user_in.email,
        hashed_password=hash_password(user_in.password),
    )

    session.add(user)
    await session.commit()
    await session.refresh(user)

    return user


async def authenticate_user(
    session: AsyncSession,
    email: str,
    password: str,
) -> User | None:
    user = await get_user_by_email(session, email)

    if user is None:
        return None

    if not verify_password(password, user.hashed_password):
        return None

    return user
```

Разберём.

## select

```python
stmt = select(User).where(User.email == email)
```

Это SQLAlchemy-запрос.

Примерно он превратится в SQL:

```sql
SELECT * FROM users WHERE email = '...';
```

## scalar_one_or_none

```python
return result.scalar_one_or_none()
```

Возвращает:

- один объект, если найден;
- `None`, если не найден;
- ошибку, если найдено больше одного объекта.

---

# 24. CRUD для задач

Создай `app/crud/todo.py`:

```python
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.models.todo import Todo
from app.schemas.todo import TodoCreate, TodoUpdate


async def create_todo(
    session: AsyncSession,
    todo_in: TodoCreate,
    owner_id: int,
) -> Todo:
    todo = Todo(
        title=todo_in.title,
        description=todo_in.description,
        owner_id=owner_id,
    )

    session.add(todo)
    await session.commit()
    await session.refresh(todo)

    return todo


async def get_todo_by_id(
    session: AsyncSession,
    todo_id: int,
    owner_id: int,
) -> Todo | None:
    stmt = (
        select(Todo)
        .where(Todo.id == todo_id)
        .where(Todo.owner_id == owner_id)
    )

    result = await session.execute(stmt)
    return result.scalar_one_or_none()


async def get_todos(
    session: AsyncSession,
    owner_id: int,
    skip: int = 0,
    limit: int = 20,
) -> list[Todo]:
    stmt = (
        select(Todo)
        .where(Todo.owner_id == owner_id)
        .offset(skip)
        .limit(limit)
        .order_by(Todo.id.desc())
    )

    result = await session.execute(stmt)
    return list(result.scalars().all())


async def update_todo(
    session: AsyncSession,
    todo: Todo,
    todo_in: TodoUpdate,
) -> Todo:
    update_data = todo_in.model_dump(exclude_unset=True)

    for field, value in update_data.items():
        setattr(todo, field, value)

    await session.commit()
    await session.refresh(todo)

    return todo


async def delete_todo(
    session: AsyncSession,
    todo: Todo,
) -> None:
    await session.delete(todo)
    await session.commit()
```

Разберём важное.

## Защита доступа

```python
.where(Todo.owner_id == owner_id)
```

Мы получаем только задачи текущего пользователя.

Иначе пользователь мог бы посмотреть чужую задачу по ID.

## Частичное обновление

```python
todo_in.model_dump(exclude_unset=True)
```

Если клиент отправил:

```json
{
  "completed": true
}
```

то мы обновим только поле `completed`.

---

# 25. Роуты авторизации

Создай `app/api/routes/auth.py`:

```python
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm
from sqlalchemy.ext.asyncio import AsyncSession

from app.api.deps import get_db_session, get_current_user
from app.core.security import create_access_token
from app.crud.user import create_user, get_user_by_email, authenticate_user
from app.models.user import User
from app.schemas.auth import Token
from app.schemas.user import UserCreate, UserRead


router = APIRouter(prefix="/auth", tags=["Auth"])


@router.post(
    "/register",
    response_model=UserRead,
    status_code=status.HTTP_201_CREATED,
)
async def register(
    user_in: UserCreate,
    session: AsyncSession = Depends(get_db_session),
):
    existing_user = await get_user_by_email(session, user_in.email)

    if existing_user is not None:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="User with this email already exists",
        )

    user = await create_user(session, user_in)

    return user


@router.post(
    "/login",
    response_model=Token,
)
async def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    session: AsyncSession = Depends(get_db_session),
):
    user = await authenticate_user(
        session=session,
        email=form_data.username,
        password=form_data.password,
    )

    if user is None:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect email or password",
            headers={"WWW-Authenticate": "Bearer"},
        )

    access_token = create_access_token(subject=str(user.id))

    return Token(access_token=access_token)


@router.get(
    "/me",
    response_model=UserRead,
)
async def get_me(
    current_user: User = Depends(get_current_user),
):
    return current_user
```

Важно.

## Почему login использует username

`OAuth2PasswordRequestForm` стандартно отправляет поля:

```text
username
password
```

Мы используем email как username.

В Swagger будет форма, где в поле `username` нужно ввести email.

---

# 26. Роуты задач

Создай `app/api/routes/todos.py`:

```python
from fastapi import APIRouter, Depends, HTTPException, Query, Response, status
from sqlalchemy.ext.asyncio import AsyncSession

from app.api.deps import get_db_session, get_current_user
from app.crud.todo import (
    create_todo,
    get_todo_by_id,
    get_todos,
    update_todo,
    delete_todo,
)
from app.models.user import User
from app.schemas.todo import TodoCreate, TodoRead, TodoUpdate


router = APIRouter(prefix="/todos", tags=["Todos"])


@router.post(
    "",
    response_model=TodoRead,
    status_code=status.HTTP_201_CREATED,
)
async def create_my_todo(
    todo_in: TodoCreate,
    session: AsyncSession = Depends(get_db_session),
    current_user: User = Depends(get_current_user),
):
    todo = await create_todo(
        session=session,
        todo_in=todo_in,
        owner_id=current_user.id,
    )

    return todo


@router.get(
    "",
    response_model=list[TodoRead],
)
async def read_my_todos(
    skip: int = Query(default=0, ge=0),
    limit: int = Query(default=20, ge=1, le=100),
    session: AsyncSession = Depends(get_db_session),
    current_user: User = Depends(get_current_user),
):
    todos = await get_todos(
        session=session,
        owner_id=current_user.id,
        skip=skip,
        limit=limit,
    )

    return todos


@router.get(
    "/{todo_id}",
    response_model=TodoRead,
)
async def read_my_todo(
    todo_id: int,
    session: AsyncSession = Depends(get_db_session),
    current_user: User = Depends(get_current_user),
):
    todo = await get_todo_by_id(
        session=session,
        todo_id=todo_id,
        owner_id=current_user.id,
    )

    if todo is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Todo not found",
        )

    return todo


@router.patch(
    "/{todo_id}",
    response_model=TodoRead,
)
async def update_my_todo(
    todo_id: int,
    todo_in: TodoUpdate,
    session: AsyncSession = Depends(get_db_session),
    current_user: User = Depends(get_current_user),
):
    todo = await get_todo_by_id(
        session=session,
        todo_id=todo_id,
        owner_id=current_user.id,
    )

    if todo is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Todo not found",
        )

    updated_todo = await update_todo(
        session=session,
        todo=todo,
        todo_in=todo_in,
    )

    return updated_todo


@router.delete(
    "/{todo_id}",
    status_code=status.HTTP_204_NO_CONTENT,
)
async def delete_my_todo(
    todo_id: int,
    session: AsyncSession = Depends(get_db_session),
    current_user: User = Depends(get_current_user),
):
    todo = await get_todo_by_id(
        session=session,
        todo_id=todo_id,
        owner_id=current_user.id,
    )

    if todo is None:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Todo not found",
        )

    await delete_todo(session=session, todo=todo)

    return Response(status_code=status.HTTP_204_NO_CONTENT)
```

---

# 27. Главный файл приложения

Создай `app/main.py`:

```python
from fastapi import FastAPI

from app.api.routes.auth import router as auth_router
from app.api.routes.todos import router as todos_router
from app.core.config import settings


app = FastAPI(
    title=settings.APP_NAME,
    debug=settings.DEBUG,
)


@app.get("/")
async def root():
    return {
        "message": "FastAPI Junior Project is running"
    }


app.include_router(auth_router)
app.include_router(todos_router)
```

---

# 28. Запуск приложения

Запусти сервер:

```bash
uvicorn app.main:app --reload
```

Открой:

```text
http://127.0.0.1:8000
```

Swagger-документация:

```text
http://127.0.0.1:8000/docs
```

ReDoc:

```text
http://127.0.0.1:8000/redoc
```

---

# 29. Миграции Alembic

Сейчас у нас есть модели SQLAlchemy, но таблицы в базе ещё не созданы.

Для этого нужен Alembic.

Инициализируй Alembic:

```bash
alembic init alembic
```

Появятся:

```text
alembic/
alembic.ini
```

---

## 29.1. Настрой alembic/env.py

Открой `alembic/env.py`.

Импортируй настройки, Base и модели.

Пример рабочего `env.py`:

```python
from logging.config import fileConfig

from sqlalchemy import pool
from sqlalchemy.engine import Connection
from sqlalchemy.ext.asyncio import async_engine_from_config

from alembic import context

from app.core.config import settings
from app.db.base import Base
from app.models import User, Todo


config = context.config

config.set_main_option("sqlalchemy.url", settings.DATABASE_URL)

if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata


def run_migrations_offline() -> None:
    url = config.get_main_option("sqlalchemy.url")

    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()


def do_run_migrations(connection: Connection) -> None:
    context.configure(
        connection=connection,
        target_metadata=target_metadata,
    )

    with context.begin_transaction():
        context.run_migrations()


async def run_async_migrations() -> None:
    configuration = config.get_section(config.config_ini_section)

    connectable = async_engine_from_config(
        configuration,
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)

    await connectable.dispose()


def run_migrations_online() -> None:
    import asyncio

    asyncio.run(run_async_migrations())


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

Почему импортируем модели:

```python
from app.models import User, Todo
```

Alembic должен увидеть модели, чтобы создать таблицы.

---

## 29.2. Создание миграции

Выполни:

```bash
alembic revision --autogenerate -m "create users and todos tables"
```

Alembic создаст файл в:

```text
alembic/versions/
```

---

## 29.3. Применение миграции

```bash
alembic upgrade head
```

После этого в PostgreSQL появятся таблицы:

```text
users
todos
alembic_version
```

---

# 30. Проверка API через Swagger

Открой:

```text
http://127.0.0.1:8000/docs
```

## 30.1. Регистрация

Endpoint:

```http
POST /auth/register
```

Body:

```json
{
  "email": "user@example.com",
  "password": "123456"
}
```

Ответ:

```json
{
  "id": 1,
  "email": "user@example.com",
  "created_at": "2025-01-01T12:00:00.000Z"
}
```

---

## 30.2. Логин

Endpoint:

```http
POST /auth/login
```

В Swagger нажми `Try it out`.

Введи:

```text
username: user@example.com
password: 123456
```

Ответ:

```json
{
  "access_token": "...",
  "token_type": "bearer"
}
```

---

## 30.3. Авторизация в Swagger

Нажми кнопку `Authorize`.

Вставь:

```text
Bearer твой_токен
```

Иногда Swagger сам добавляет `Bearer`, тогда нужно вставить только токен.

---

## 30.4. Получение текущего пользователя

```http
GET /auth/me
```

Если токен правильный:

```json
{
  "id": 1,
  "email": "user@example.com",
  "created_at": "..."
}
```

---

## 30.5. Создание задачи

```http
POST /todos
```

Body:

```json
{
  "title": "Learn FastAPI",
  "description": "Build real API project"
}
```

Ответ:

```json
{
  "id": 1,
  "title": "Learn FastAPI",
  "description": "Build real API project",
  "completed": false,
  "owner_id": 1,
  "created_at": "...",
  "updated_at": null
}
```

---

# 31. Главные концепции FastAPI

## 31.1. Endpoint

Endpoint — это функция, которая обрабатывает HTTP-запрос.

```python
@app.get("/")
async def root():
    return {"message": "Hello"}
```

Здесь:

```python
@app.get("/")
```

означает:

```text
GET /
```

---

## 31.2. HTTP-методы

| Метод | Назначение |
|---|---|
| GET | получить данные |
| POST | создать данные |
| PUT | полностью заменить данные |
| PATCH | частично обновить данные |
| DELETE | удалить данные |

---

## 31.3. Path parameters

```python
@app.get("/users/{user_id}")
async def get_user(user_id: int):
    return {"user_id": user_id}
```

Запрос:

```http
GET /users/5
```

FastAPI передаст:

```python
user_id = 5
```

Если отправить:

```http
GET /users/abc
```

будет ошибка, потому что `user_id: int`.

---

## 31.4. Query parameters

```python
@app.get("/todos")
async def get_todos(skip: int = 0, limit: int = 20):
    return {"skip": skip, "limit": limit}
```

Запрос:

```http
GET /todos?skip=10&limit=5
```

---

## 31.5. Request body

```python
from pydantic import BaseModel


class TodoCreate(BaseModel):
    title: str


@app.post("/todos")
async def create_todo(todo: TodoCreate):
    return todo
```

Клиент отправляет JSON:

```json
{
  "title": "Learn FastAPI"
}
```

---

## 31.6. Response model

```python
@app.get("/users/{user_id}", response_model=UserRead)
async def get_user(user_id: int):
    return user
```

`response_model` говорит FastAPI:

1. какие поля вернуть клиенту;
2. как преобразовать данные;
3. что показать в Swagger.

---

## 31.7. HTTPException

```python
from fastapi import HTTPException


raise HTTPException(
    status_code=404,
    detail="User not found",
)
```

FastAPI вернёт:

```json
{
  "detail": "User not found"
}
```

---

# 32. Главные концепции Pydantic

## 32.1. BaseModel

```python
from pydantic import BaseModel


class Product(BaseModel):
    title: str
    price: float
```

Pydantic проверит:

```json
{
  "title": "Phone",
  "price": 999.99
}
```

---

## 32.2. Field

```python
from pydantic import Field


class Product(BaseModel):
    title: str = Field(min_length=2, max_length=100)
    price: float = Field(gt=0)
```

`gt=0` значит greater than 0.

---

## 32.3. Optional поля

В Python 3.10+:

```python
description: str | None = None
```

Это значит:

- может быть строкой;
- может быть `None`.

---

## 32.4. model_dump

```python
data = schema.model_dump()
```

Превращает Pydantic-модель в словарь.

Пример:

```python
todo_in.model_dump()
```

Результат:

```python
{
    "title": "Learn FastAPI",
    "description": "..."
}
```

---

## 32.5. exclude_unset

```python
todo_in.model_dump(exclude_unset=True)
```

Если клиент отправил только:

```json
{
  "completed": true
}
```

то получим:

```python
{
    "completed": True
}
```

Это полезно для PATCH.

---

# 33. Главные концепции SQLAlchemy 2

## 33.1. Модель

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255))
```

Это описание таблицы.

---

## 33.2. Добавление записи

```python
user = User(email="test@example.com", hashed_password="...")

session.add(user)
await session.commit()
await session.refresh(user)
```

Что происходит:

```python
session.add(user)
```

говорит SQLAlchemy: этот объект нужно сохранить.

```python
await session.commit()
```

отправляет изменения в базу.

```python
await session.refresh(user)
```

обновляет объект из базы, например получает `id`.

---

## 33.3. Получение по primary key

```python
user = await session.get(User, 1)
```

---

## 33.4. SELECT

```python
stmt = select(User).where(User.email == email)
result = await session.execute(stmt)
user = result.scalar_one_or_none()
```

---

## 33.5. Обновление

```python
user.email = "new@example.com"
await session.commit()
await session.refresh(user)
```

---

## 33.6. Удаление

```python
await session.delete(user)
await session.commit()
```

---

# 34. Частые ошибки новичков

## Ошибка 1. Забыли await

Неправильно:

```python
session.commit()
```

Правильно:

```python
await session.commit()
```

Если работаешь с async SQLAlchemy, почти все операции с БД требуют `await`.

---

## Ошибка 2. Отдаёшь пароль клиенту

Плохо:

```python
response_model=User
```

Лучше:

```python
response_model=UserRead
```

В `UserRead` нет `hashed_password`.

---

## Ошибка 3. Не проверяешь владельца ресурса

Плохо:

```python
todo = await session.get(Todo, todo_id)
```

Так пользователь может получить чужую задачу.

Лучше:

```python
select(Todo).where(
    Todo.id == todo_id,
    Todo.owner_id == current_user.id,
)
```

---

## Ошибка 4. Не используешь миграции

Плохо вручную создавать таблицы.

Правильно использовать Alembic:

```bash
alembic revision --autogenerate -m "message"
alembic upgrade head
```

---

## Ошибка 5. Хранишь SECRET_KEY в коде

Плохо:

```python
JWT_SECRET_KEY = "secret"
```

Лучше:

```env
JWT_SECRET_KEY=...
```

---

# 35. Минимальный порядок разработки нового endpoint

Допустим, ты хочешь добавить новую сущность `Category`.

Порядок:

1. Создать SQLAlchemy-модель `Category`.
2. Импортировать её в `models/__init__.py`.
3. Создать Pydantic-схемы:
   - `CategoryCreate`;
   - `CategoryUpdate`;
   - `CategoryRead`.
4. Создать CRUD-функции.
5. Создать router.
6. Подключить router в `main.py`.
7. Сделать миграцию Alembic.
8. Проверить через Swagger.

---

# 36. Как думать как junior backend-разработчик

Когда ты пишешь endpoint, всегда задавай вопросы:

## 1. Кто может вызвать этот endpoint?

Если только авторизованный пользователь:

```python
current_user: User = Depends(get_current_user)
```

## 2. Какие данные приходят?

Создай Pydantic-схему:

```python
class TodoCreate(BaseModel):
    title: str
```

## 3. Какие данные возвращаются?

Создай response schema:

```python
class TodoRead(BaseModel):
    id: int
    title: str
```

## 4. Есть ли доступ к чужим данным?

Всегда фильтруй по владельцу:

```python
.where(Todo.owner_id == current_user.id)
```

## 5. Какие ошибки возможны?

Например:

```python
if todo is None:
    raise HTTPException(status_code=404, detail="Todo not found")
```

---

# 37. Пример полного запроса через curl

## Регистрация

```bash
curl -X POST http://127.0.0.1:8000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"user@example.com","password":"123456"}'
```

## Логин

```bash
curl -X POST http://127.0.0.1:8000/auth/login \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=user@example.com&password=123456"
```

## Создание задачи

```bash
curl -X POST http://127.0.0.1:8000/todos \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Learn FastAPI","description":"Practice every day"}'
```

---

# 38. Что нужно знать на уровне уверенного junior+

Тебе нужно уверенно понимать:

## FastAPI

- `FastAPI()`;
- routers;
- path/query/body parameters;
- `Depends`;
- `HTTPException`;
- `response_model`;
- status codes;
- Swagger;
- авторизация через Bearer token.

## Pydantic

- `BaseModel`;
- `Field`;
- `EmailStr`;
- optional поля;
- `model_dump`;
- `exclude_unset`;
- `ConfigDict(from_attributes=True)`;
- схемы для create/read/update.

## SQLAlchemy 2

- `DeclarativeBase`;
- `Mapped`;
- `mapped_column`;
- `relationship`;
- `ForeignKey`;
- `AsyncSession`;
- `select`;
- `session.add`;
- `session.commit`;
- `session.refresh`;
- `session.delete`.

## PostgreSQL

- таблицы;
- primary key;
- foreign key;
- unique index;
- nullable;
- связи one-to-many.

## Alembic

- `alembic init`;
- `revision --autogenerate`;
- `upgrade head`;
- зачем нужны миграции.

## Безопасность

- нельзя хранить обычные пароли;
- нужен bcrypt;
- JWT должен иметь срок жизни;
- секреты хранятся в `.env`;
- пользователь не должен получать чужие данные.

---

# 39. Итоговая схема работы приложения

Когда пользователь создаёт задачу:

1. Клиент отправляет запрос:

```http
POST /todos
Authorization: Bearer token
```

2. FastAPI вызывает:

```python
get_current_user()
```

3. `get_current_user` проверяет JWT.

4. Если токен правильный, достаёт пользователя из БД.

5. FastAPI валидирует body через `TodoCreate`.

6. Вызывается CRUD-функция:

```python
create_todo(...)
```

7. SQLAlchemy сохраняет задачу в PostgreSQL.

8. FastAPI возвращает ответ по схеме `TodoRead`.

---

# 40. Что делать дальше

После этого проекта стоит изучить:

1. `PUT` vs `PATCH`;
2. роли пользователей: admin/user;
3. refresh tokens;
4. pagination с total count;
5. фильтрацию и сортировку;
6. тестирование через pytest;
7. Dockerfile для приложения;
8. CI/CD;
9. логирование;
10. обработку ошибок централизованно;
11. repository/service architecture;
12. загрузку файлов;
13. Redis;
14. Celery;
15. WebSocket.

---

# 41. Краткий чеклист запуска проекта

```bash
docker compose up -d
```

```bash
python -m venv venv
source venv/bin/activate
```

или Windows:

```bash
venv\Scripts\Activate.ps1
```

```bash
pip install -r requirements.txt
```

```bash
alembic revision --autogenerate -m "create tables"
```

```bash
alembic upgrade head
```

```bash
uvicorn app.main:app --reload
```

Открыть:

```text
http://127.0.0.1:8000/docs
```

---

# 42. Самая важная мысль

FastAPI-приложение обычно состоит из слоёв:

```text
HTTP request
    ↓
Router / Endpoint
    ↓
Pydantic validation
    ↓
Dependencies
    ↓
CRUD / Service logic
    ↓
SQLAlchemy
    ↓
PostgreSQL
```

Если ты понимаешь этот путь данных, ты уже близок к уровню junior backend developer.
