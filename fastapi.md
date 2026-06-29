# FastAPI + Pydantic + SQLAlchemy + PostgreSQL  
## Полная практическая документация с нуля до уровня уверенного junior+

Ниже мы построим полноценный backend-проект на:

- **FastAPI** — веб-фреймворк для создания API
- **Pydantic v2** — валидация данных, схемы запросов/ответов
- **SQLAlchemy 2.0** — ORM для работы с базой данных
- **PostgreSQL** — настоящая реляционная база данных
- **Alembic** — миграции базы данных
- **Docker Compose** — запуск PostgreSQL
- Немного затронем:
  - структуру проекта
  - CRUD
  - зависимости FastAPI
  - обработку ошибок
  - авторизацию JWT
  - тестирование
  - хорошие практики

---

# 1. Что такое FastAPI

**FastAPI** — это Python-фреймворк для создания HTTP API.

Пример API:

```http
GET /users/1
```

Ответ:

```json
{
  "id": 1,
  "email": "user@example.com",
  "name": "Alex"
}
```

FastAPI позволяет быстро создавать такие эндпоинты.

Простейший пример:

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def hello():
    return {"message": "Hello World"}
```

Запуск:

```bash
uvicorn main:app --reload
```

После запуска:

```text
http://127.0.0.1:8000
```

Документация автоматически:

```text
http://127.0.0.1:8000/docs
```

---

# 2. Что такое Pydantic

**Pydantic** — библиотека для описания и проверки данных.

Например, клиент отправляет JSON:

```json
{
  "email": "test@example.com",
  "age": 20
}
```

Мы хотим проверить:

- email действительно строка
- age число
- age больше 0

Пример:

```python
from pydantic import BaseModel, EmailStr, Field

class UserCreate(BaseModel):
    email: EmailStr
    age: int = Field(gt=0)
```

Если клиент отправит неправильные данные, FastAPI автоматически вернёт ошибку `422`.

---

# 3. Что такое SQLAlchemy

**SQLAlchemy** — это библиотека для работы с базой данных.

Есть два основных способа:

1. **Core** — писать SQL через Python-объекты
2. **ORM** — работать с таблицами как с Python-классами

Мы будем использовать **ORM**.

Таблица `users` в PostgreSQL:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR UNIQUE NOT NULL,
    name VARCHAR NOT NULL
);
```

В SQLAlchemy ORM это будет выглядеть так:

```python
class User(Base):
    __tablename__ = "users"

    id = mapped_column(Integer, primary_key=True)
    email = mapped_column(String, unique=True, nullable=False)
    name = mapped_column(String, nullable=False)
```

---

# 4. Что такое PostgreSQL

**PostgreSQL** — это настоящая реляционная база данных.

Она хранит данные в таблицах.

Пример таблицы `users`:

| id | email            | name |
| -- | ---------------- | ---- |
| 1  | alex@example.com | Alex |
| 2  | bob@example.com  | Bob  |

---

# 5. Что мы будем делать

Мы создадим API для пользователей и постов.

Функциональность:

## Пользователи

- создать пользователя
- получить список пользователей
- получить пользователя по id
- обновить пользователя
- удалить пользователя

## Посты

- создать пост
- получить список постов
- получить пост по id
- обновить пост
- удалить пост
- связь пользователя и постов

## Дополнительно

- настройки через `.env`
- PostgreSQL через Docker
- миграции Alembic
- Pydantic-схемы
- Dependency Injection
- обработка ошибок
- JWT-авторизация

---

# 6. Создание проекта

Создадим папку:

```bash
mkdir fastapi_sqlalchemy_project
cd fastapi_sqlalchemy_project
```

Создадим виртуальное окружение:

```bash
python -m venv venv
```

Активация:

## Windows

```bash
venv\Scripts\activate
```

## Linux / macOS

```bash
source venv/bin/activate
```

---

# 7. Установка зависимостей

```bash
pip install fastapi uvicorn sqlalchemy asyncpg alembic pydantic-settings python-dotenv
```

Дополнительно для email-валидации:

```bash
pip install email-validator
```

Для авторизации:

```bash
pip install passlib[bcrypt] python-jose[cryptography]
```

Для тестов:

```bash
pip install pytest httpx pytest-asyncio
```

Можно создать `requirements.txt`:

```txt
fastapi
uvicorn
sqlalchemy
asyncpg
alembic
pydantic-settings
python-dotenv
email-validator
passlib[bcrypt]
python-jose[cryptography]
pytest
httpx
pytest-asyncio
```

---

# 8. Структура проекта

Сделаем нормальную структуру:

```text
fastapi_sqlalchemy_project/
│
├── app/
│   ├── main.py
│   ├── core/
│   │   ├── config.py
│   │   └── security.py
│   │
│   ├── db/
│   │   ├── database.py
│   │   └── base.py
│   │
│   ├── models/
│   │   ├── user.py
│   │   └── post.py
│   │
│   ├── schemas/
│   │   ├── user.py
│   │   ├── post.py
│   │   └── token.py
│   │
│   ├── crud/
│   │   ├── user.py
│   │   └── post.py
│   │
│   ├── api/
│   │   ├── deps.py
│   │   └── routers/
│   │       ├── users.py
│   │       ├── posts.py
│   │       └── auth.py
│   │
│   └── __init__.py
│
├── alembic/
│
├── alembic.ini
├── docker-compose.yml
├── .env
└── requirements.txt
```

---

# 9. Docker Compose для PostgreSQL

Создай файл `docker-compose.yml`:

```yaml
version: "3.9"

services:
  postgres:
    image: postgres:16
    container_name: fastapi_postgres
    restart: always
    environment:
      POSTGRES_USER: fastapi_user
      POSTGRES_PASSWORD: fastapi_password
      POSTGRES_DB: fastapi_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Запуск PostgreSQL:

```bash
docker compose up -d
```

Проверить:

```bash
docker ps
```

Остановить:

```bash
docker compose down
```

---

# 10. Файл `.env`

Создай `.env`:

```env
DATABASE_URL=postgresql+asyncpg://fastapi_user:fastapi_password@localhost:5432/fastapi_db

SECRET_KEY=super-secret-key-change-me
ALGORITHM=HS256
ACCESS_TOKEN_EXPIRE_MINUTES=30
```

Разберём строку подключения:

```text
postgresql+asyncpg://fastapi_user:fastapi_password@localhost:5432/fastapi_db
```

| Часть | Значение |
|---|---|
| postgresql+asyncpg | PostgreSQL + async-драйвер |
| fastapi_user | пользователь |
| fastapi_password | пароль |
| localhost | сервер |
| 5432 | порт PostgreSQL |
| fastapi_db | база данных |

---

# 11. Настройки проекта через Pydantic Settings

Файл `app/core/config.py`:

```python
from pydantic_settings import BaseSettings, SettingsConfigDict


class Settings(BaseSettings):
    DATABASE_URL: str

    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30

    model_config = SettingsConfigDict(env_file=".env")


settings = Settings()
```

Теперь в любом месте проекта можно импортировать:

```python
from app.core.config import settings

print(settings.DATABASE_URL)
```

---

# 12. Подключение SQLAlchemy к PostgreSQL

Файл `app/db/database.py`:

```python
from sqlalchemy.ext.asyncio import create_async_engine, async_sessionmaker, AsyncSession

from app.core.config import settings


engine = create_async_engine(
    settings.DATABASE_URL,
    echo=True,
)

AsyncSessionLocal = async_sessionmaker(
    bind=engine,
    class_=AsyncSession,
    expire_on_commit=False,
)
```

## Что здесь происходит

```python
create_async_engine(...)
```

Создаёт подключение к базе.

```python
echo=True
```

SQLAlchemy будет показывать SQL-запросы в консоли.  
Для обучения это полезно.

```python
async_sessionmaker(...)
```

Фабрика сессий.

## Что такое Session

`Session` — это объект, через который мы работаем с базой:

- добавить запись
- получить запись
- обновить
- удалить
- выполнить SQL-запрос

---

# 13. Базовый класс моделей

Файл `app/db/base.py`:

```python
from sqlalchemy.orm import DeclarativeBase


class Base(DeclarativeBase):
    pass
```

Все ORM-модели будут наследоваться от `Base`.

---

# 14. Модель User

Файл `app/models/user.py`:

```python
from sqlalchemy import String
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

    name: Mapped[str] = mapped_column(
        String(100),
        nullable=False,
    )

    hashed_password: Mapped[str] = mapped_column(
        String(255),
        nullable=False,
    )

    posts = relationship(
        "Post",
        back_populates="author",
        cascade="all, delete-orphan",
    )
```

## Объяснение

```python
__tablename__ = "users"
```

Имя таблицы в базе данных.

```python
id: Mapped[int] = mapped_column(primary_key=True)
```

Колонка `id`, первичный ключ.

```python
email
```

Уникальный email пользователя.

```python
hashed_password
```

Мы **никогда не храним обычный пароль**.  
Храним только хеш.

```python
posts = relationship(...)
```

Связь пользователя с постами.

Один пользователь может иметь много постов.

---

# 15. Модель Post

Файл `app/models/post.py`:

```python
from sqlalchemy import String, Text, ForeignKey
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db.base import Base


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True, index=True)

    title: Mapped[str] = mapped_column(
        String(200),
        nullable=False,
    )

    content: Mapped[str] = mapped_column(
        Text,
        nullable=False,
    )

    author_id: Mapped[int] = mapped_column(
        ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
    )

    author = relationship(
        "User",
        back_populates="posts",
    )
```

## Объяснение связи

```python
author_id = ForeignKey("users.id")
```

Каждый пост принадлежит пользователю.

В таблице `posts` будет колонка:

```text
author_id
```

Она указывает на `users.id`.

---

# 16. Чтобы Alembic видел модели

В файле `app/db/base.py` можно импортировать модели:

```python
from sqlalchemy.orm import DeclarativeBase


class Base(DeclarativeBase):
    pass


from app.models.user import User  # noqa
from app.models.post import Post  # noqa
```

Это нужно, чтобы Alembic увидел таблицы.

---

# 17. Alembic: миграции базы данных

## Что такое миграции

Миграция — это история изменений базы данных.

Например:

1. Создали таблицу `users`
2. Добавили таблицу `posts`
3. Добавили колонку `created_at`
4. Изменили тип поля

Всё это должно быть под контролем.

---

## Инициализация Alembic

```bash
alembic init alembic
```

Появятся:

```text
alembic/
alembic.ini
```

---

## Настройка `alembic/env.py`

Найди в `alembic/env.py`:

```python
target_metadata = None
```

Замени на:

```python
from app.db.base import Base
from app.core.config import settings

target_metadata = Base.metadata
```

Найди:

```python
config.set_main_option("sqlalchemy.url", ...)
```

И добавь:

```python
config.set_main_option("sqlalchemy.url", settings.DATABASE_URL)
```

Но Alembic по умолчанию работает синхронно, а у нас URL async:

```text
postgresql+asyncpg://...
```

Для Alembic лучше использовать sync URL:

В `.env` добавим:

```env
DATABASE_URL=postgresql+asyncpg://fastapi_user:fastapi_password@localhost:5432/fastapi_db
DATABASE_SYNC_URL=postgresql://fastapi_user:fastapi_password@localhost:5432/fastapi_db
```

В `config.py`:

```python
class Settings(BaseSettings):
    DATABASE_URL: str
    DATABASE_SYNC_URL: str
    SECRET_KEY: str
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30

    model_config = SettingsConfigDict(env_file=".env")
```

В `alembic/env.py`:

```python
from app.core.config import settings
from app.db.base import Base

config.set_main_option("sqlalchemy.url", settings.DATABASE_SYNC_URL)

target_metadata = Base.metadata
```

Для sync PostgreSQL драйвера нужно поставить:

```bash
pip install psycopg2-binary
```

---

## Создание миграции

```bash
alembic revision --autogenerate -m "create users and posts tables"
```

Применение миграции:

```bash
alembic upgrade head
```

Теперь таблицы созданы в PostgreSQL.

---

# 18. Dependency для сессии базы данных

Файл `app/api/deps.py`:

```python
from typing import AsyncGenerator

from sqlalchemy.ext.asyncio import AsyncSession

from app.db.database import AsyncSessionLocal


async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSessionLocal() as session:
        yield session
```

## Что делает `yield`

FastAPI вызовет функцию, получит `session`, отдаст её в endpoint, а после завершения запроса закроет сессию.

Пример использования:

```python
from fastapi import Depends
from sqlalchemy.ext.asyncio import AsyncSession

async def endpoint(db: AsyncSession = Depends(get_db)):
    ...
```

---

# 19. Pydantic-схемы пользователя

Файл `app/schemas/user.py`:

```python
from pydantic import BaseModel, EmailStr, Field, ConfigDict


class UserBase(BaseModel):
    email: EmailStr
    name: str = Field(min_length=2, max_length=100)


class UserCreate(UserBase):
    password: str = Field(min_length=6, max_length=100)


class UserUpdate(BaseModel):
    email: EmailStr | None = None
    name: str | None = Field(default=None, min_length=2, max_length=100)
    password: str | None = Field(default=None, min_length=6, max_length=100)


class UserRead(UserBase):
    id: int

    model_config = ConfigDict(from_attributes=True)
```

## Очень важно

`UserCreate` — схема входных данных при создании пользователя.

Клиент отправляет:

```json
{
  "email": "alex@example.com",
  "name": "Alex",
  "password": "123456"
}
```

`UserRead` — схема ответа.

Мы не возвращаем пароль:

```json
{
  "id": 1,
  "email": "alex@example.com",
  "name": "Alex"
}
```

---

# 20. Pydantic-схемы поста

Файл `app/schemas/post.py`:

```python
from pydantic import BaseModel, Field, ConfigDict


class PostBase(BaseModel):
    title: str = Field(min_length=3, max_length=200)
    content: str = Field(min_length=1)


class PostCreate(PostBase):
    pass


class PostUpdate(BaseModel):
    title: str | None = Field(default=None, min_length=3, max_length=200)
    content: str | None = Field(default=None, min_length=1)


class PostRead(PostBase):
    id: int
    author_id: int

    model_config = ConfigDict(from_attributes=True)
```

---

# 21. Безопасность: хеширование паролей

Файл `app/core/security.py`:

```python
from datetime import datetime, timedelta, timezone

from jose import jwt
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


def create_access_token(data: dict) -> str:
    to_encode = data.copy()

    expire = datetime.now(timezone.utc) + timedelta(
        minutes=settings.ACCESS_TOKEN_EXPIRE_MINUTES
    )

    to_encode.update({"exp": expire})

    encoded_jwt = jwt.encode(
        to_encode,
        settings.SECRET_KEY,
        algorithm=settings.ALGORITHM,
    )

    return encoded_jwt
```

---

# 22. CRUD для пользователей

Файл `app/crud/user.py`:

```python
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.models.user import User
from app.schemas.user import UserCreate, UserUpdate
from app.core.security import hash_password


async def get_user_by_id(db: AsyncSession, user_id: int) -> User | None:
    result = await db.execute(
        select(User).where(User.id == user_id)
    )
    return result.scalar_one_or_none()


async def get_user_by_email(db: AsyncSession, email: str) -> User | None:
    result = await db.execute(
        select(User).where(User.email == email)
    )
    return result.scalar_one_or_none()


async def get_users(
    db: AsyncSession,
    skip: int = 0,
    limit: int = 100,
) -> list[User]:
    result = await db.execute(
        select(User)
        .offset(skip)
        .limit(limit)
    )
    return list(result.scalars().all())


async def create_user(db: AsyncSession, user_in: UserCreate) -> User:
    user = User(
        email=user_in.email,
        name=user_in.name,
        hashed_password=hash_password(user_in.password),
    )

    db.add(user)

    await db.commit()
    await db.refresh(user)

    return user


async def update_user(
    db: AsyncSession,
    user: User,
    user_in: UserUpdate,
) -> User:
    update_data = user_in.model_dump(exclude_unset=True)

    if "password" in update_data:
        password = update_data.pop("password")
        update_data["hashed_password"] = hash_password(password)

    for field, value in update_data.items():
        setattr(user, field, value)

    await db.commit()
    await db.refresh(user)

    return user


async def delete_user(db: AsyncSession, user: User) -> None:
    await db.delete(user)
    await db.commit()
```

## Важные моменты

```python
select(User).where(User.id == user_id)
```

SQL-запрос:

```sql
SELECT * FROM users WHERE id = ...
```

```python
db.add(user)
```

Добавляем объект в сессию.

```python
await db.commit()
```

Сохраняем изменения в базе.

```python
await db.refresh(user)
```

Обновляем объект данными из базы, например получаем `id`.

---

# 23. CRUD для постов

Файл `app/crud/post.py`:

```python
from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.models.post import Post
from app.schemas.post import PostCreate, PostUpdate


async def get_post_by_id(db: AsyncSession, post_id: int) -> Post | None:
    result = await db.execute(
        select(Post).where(Post.id == post_id)
    )
    return result.scalar_one_or_none()


async def get_posts(
    db: AsyncSession,
    skip: int = 0,
    limit: int = 100,
) -> list[Post]:
    result = await db.execute(
        select(Post)
        .offset(skip)
        .limit(limit)
    )
    return list(result.scalars().all())


async def create_post(
    db: AsyncSession,
    post_in: PostCreate,
    author_id: int,
) -> Post:
    post = Post(
        title=post_in.title,
        content=post_in.content,
        author_id=author_id,
    )

    db.add(post)
    await db.commit()
    await db.refresh(post)

    return post


async def update_post(
    db: AsyncSession,
    post: Post,
    post_in: PostUpdate,
) -> Post:
    update_data = post_in.model_dump(exclude_unset=True)

    for field, value in update_data.items():
        setattr(post, field, value)

    await db.commit()
    await db.refresh(post)

    return post


async def delete_post(db: AsyncSession, post: Post) -> None:
    await db.delete(post)
    await db.commit()
```

---

# 24. Роутер пользователей

Файл `app/api/routers/users.py`:

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

from app.api.deps import get_db
from app.crud import user as user_crud
from app.schemas.user import UserCreate, UserRead, UserUpdate


router = APIRouter(
    prefix="/users",
    tags=["Users"],
)


@router.post(
    "",
    response_model=UserRead,
    status_code=status.HTTP_201_CREATED,
)
async def create_user(
    user_in: UserCreate,
    db: AsyncSession = Depends(get_db),
):
    existing_user = await user_crud.get_user_by_email(db, user_in.email)

    if existing_user:
        raise HTTPException(
            status_code=status.HTTP_400_BAD_REQUEST,
            detail="User with this email already exists",
        )

    user = await user_crud.create_user(db, user_in)

    return user


@router.get(
    "",
    response_model=list[UserRead],
)
async def read_users(
    skip: int = 0,
    limit: int = 100,
    db: AsyncSession = Depends(get_db),
):
    users = await user_crud.get_users(db, skip=skip, limit=limit)
    return users


@router.get(
    "/{user_id}",
    response_model=UserRead,
)
async def read_user(
    user_id: int,
    db: AsyncSession = Depends(get_db),
):
    user = await user_crud.get_user_by_id(db, user_id)

    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found",
        )

    return user


@router.patch(
    "/{user_id}",
    response_model=UserRead,
)
async def update_user(
    user_id: int,
    user_in: UserUpdate,
    db: AsyncSession = Depends(get_db),
):
    user = await user_crud.get_user_by_id(db, user_id)

    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found",
        )

    updated_user = await user_crud.update_user(db, user, user_in)

    return updated_user


@router.delete(
    "/{user_id}",
    status_code=status.HTTP_204_NO_CONTENT,
)
async def delete_user(
    user_id: int,
    db: AsyncSession = Depends(get_db),
):
    user = await user_crud.get_user_by_id(db, user_id)

    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found",
        )

    await user_crud.delete_user(db, user)

    return None
```

---

# 25. Роутер постов

Файл `app/api/routers/posts.py`:

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

from app.api.deps import get_db
from app.crud import post as post_crud
from app.crud import user as user_crud
from app.schemas.post import PostCreate, PostRead, PostUpdate


router = APIRouter(
    prefix="/posts",
    tags=["Posts"],
)


@router.post(
    "",
    response_model=PostRead,
    status_code=status.HTTP_201_CREATED,
)
async def create_post(
    author_id: int,
    post_in: PostCreate,
    db: AsyncSession = Depends(get_db),
):
    author = await user_crud.get_user_by_id(db, author_id)

    if not author:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Author not found",
        )

    post = await post_crud.create_post(
        db=db,
        post_in=post_in,
        author_id=author_id,
    )

    return post


@router.get(
    "",
    response_model=list[PostRead],
)
async def read_posts(
    skip: int = 0,
    limit: int = 100,
    db: AsyncSession = Depends(get_db),
):
    posts = await post_crud.get_posts(db, skip=skip, limit=limit)
    return posts


@router.get(
    "/{post_id}",
    response_model=PostRead,
)
async def read_post(
    post_id: int,
    db: AsyncSession = Depends(get_db),
):
    post = await post_crud.get_post_by_id(db, post_id)

    if not post:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Post not found",
        )

    return post


@router.patch(
    "/{post_id}",
    response_model=PostRead,
)
async def update_post(
    post_id: int,
    post_in: PostUpdate,
    db: AsyncSession = Depends(get_db),
):
    post = await post_crud.get_post_by_id(db, post_id)

    if not post:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Post not found",
        )

    updated_post = await post_crud.update_post(db, post, post_in)

    return updated_post


@router.delete(
    "/{post_id}",
    status_code=status.HTTP_204_NO_CONTENT,
)
async def delete_post(
    post_id: int,
    db: AsyncSession = Depends(get_db),
):
    post = await post_crud.get_post_by_id(db, post_id)

    if not post:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Post not found",
        )

    await post_crud.delete_post(db, post)

    return None
```

---

# 26. Главный файл приложения

Файл `app/main.py`:

```python
from fastapi import FastAPI

from app.api.routers import users, posts


app = FastAPI(
    title="FastAPI SQLAlchemy PostgreSQL Project",
    description="Учебный backend-проект на FastAPI",
    version="1.0.0",
)


@app.get("/")
async def root():
    return {
        "message": "API is running"
    }


app.include_router(users.router)
app.include_router(posts.router)
```

Запуск:

```bash
uvicorn app.main:app --reload
```

Документация:

```text
http://127.0.0.1:8000/docs
```

---

# 27. Что такое endpoint

Endpoint — это конкретный URL + HTTP-метод.

Например:

```python
@router.get("/users")
async def read_users():
    ...
```

Это endpoint:

```http
GET /users
```

---

# 28. HTTP-методы

| Метод | Для чего |
|---|---|
| GET | получить данные |
| POST | создать данные |
| PUT | полностью заменить данные |
| PATCH | частично обновить данные |
| DELETE | удалить данные |

Примеры:

```http
GET /users
POST /users
GET /users/1
PATCH /users/1
DELETE /users/1
```

---

# 29. HTTP-статусы

| Статус | Значение |
|---|---|
| 200 | успешно |
| 201 | создано |
| 204 | успешно, без тела ответа |
| 400 | плохой запрос |
| 401 | не авторизован |
| 403 | запрещено |
| 404 | не найдено |
| 422 | ошибка валидации |
| 500 | ошибка сервера |

Пример:

```python
raise HTTPException(
    status_code=404,
    detail="User not found",
)
```

Ответ:

```json
{
  "detail": "User not found"
}
```

---

# 30. Query, Path и Body параметры

## Path parameter

```python
@router.get("/users/{user_id}")
async def get_user(user_id: int):
    ...
```

Запрос:

```http
GET /users/5
```

`user_id = 5`

---

## Query parameter

```python
@router.get("/users")
async def get_users(skip: int = 0, limit: int = 100):
    ...
```

Запрос:

```http
GET /users?skip=10&limit=20
```

---

## Body parameter

```python
@router.post("/users")
async def create_user(user_in: UserCreate):
    ...
```

Тело запроса:

```json
{
  "email": "alex@example.com",
  "name": "Alex",
  "password": "123456"
}
```

---

# 31. Dependency Injection в FastAPI

Dependency — это функция, которую FastAPI вызывает автоматически.

Пример:

```python
async def get_db():
    ...
```

Использование:

```python
db: AsyncSession = Depends(get_db)
```

FastAPI:

1. вызывает `get_db`
2. получает сессию
3. передаёт её в endpoint
4. после запроса закрывает сессию

Это удобно для:

- базы данных
- текущего пользователя
- проверки прав
- подключения настроек

---

# 32. Подробно про Pydantic

## Базовая модель

```python
from pydantic import BaseModel

class Product(BaseModel):
    title: str
    price: float
```

Правильные данные:

```python
Product(title="Phone", price=1000)
```

Неправильные данные:

```python
Product(title="Phone", price="abc")
```

Будет ошибка валидации.

---

## Field

```python
from pydantic import Field

class Product(BaseModel):
    title: str = Field(min_length=3, max_length=100)
    price: float = Field(gt=0)
```

Ограничения:

| Параметр | Значение |
|---|---|
| min_length | минимальная длина строки |
| max_length | максимальная длина строки |
| gt | больше чем |
| ge | больше или равно |
| lt | меньше чем |
| le | меньше или равно |

---

## Optional поля

```python
class UserUpdate(BaseModel):
    name: str | None = None
```

Это значит:

- поле можно не передавать
- если передали, оно должно быть строкой или `null`

---

## model_dump

```python
data = user_in.model_dump()
```

Преобразует Pydantic-модель в словарь.

Пример:

```python
UserCreate(
    email="a@example.com",
    name="Alex",
    password="123456"
).model_dump()
```

Результат:

```python
{
    "email": "a@example.com",
    "name": "Alex",
    "password": "123456"
}
```

---

## exclude_unset=True

Очень важно для PATCH:

```python
update_data = user_in.model_dump(exclude_unset=True)
```

Если клиент отправил:

```json
{
  "name": "New name"
}
```

То будет:

```python
{
  "name": "New name"
}
```

А не:

```python
{
  "email": None,
  "name": "New name",
  "password": None
}
```

---

## from_attributes=True

```python
model_config = ConfigDict(from_attributes=True)
```

Нужно, чтобы Pydantic мог превращать SQLAlchemy-объект в JSON.

SQLAlchemy объект:

```python
user.email
user.name
```

Pydantic сможет прочитать эти атрибуты.

---

# 33. Авторизация JWT

Сейчас у нас можно создавать пользователей, но нет логина.

Добавим:

- `/auth/login`
- получение токена
- dependency `get_current_user`
- создание поста от текущего пользователя

---

## Схема токена

Файл `app/schemas/token.py`:

```python
from pydantic import BaseModel


class Token(BaseModel):
    access_token: str
    token_type: str = "bearer"
```

---

## Dependency текущего пользователя

Обновим `app/api/deps.py`:

```python
from typing import AsyncGenerator

from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from jose import JWTError, jwt
from sqlalchemy.ext.asyncio import AsyncSession

from app.core.config import settings
from app.db.database import AsyncSessionLocal
from app.crud.user import get_user_by_id
from app.models.user import User


oauth2_scheme = OAuth2PasswordBearer(tokenUrl="/auth/login")


async def get_db() -> AsyncGenerator[AsyncSession, None]:
    async with AsyncSessionLocal() as session:
        yield session


async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: AsyncSession = Depends(get_db),
) -> User:
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )

    try:
        payload = jwt.decode(
            token,
            settings.SECRET_KEY,
            algorithms=[settings.ALGORITHM],
        )

        user_id: str | None = payload.get("sub")

        if user_id is None:
            raise credentials_exception

    except JWTError:
        raise credentials_exception

    user = await get_user_by_id(db, int(user_id))

    if user is None:
        raise credentials_exception

    return user
```

---

## Роутер авторизации

Файл `app/api/routers/auth.py`:

```python
from fastapi import APIRouter, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm
from sqlalchemy.ext.asyncio import AsyncSession

from app.api.deps import get_db
from app.crud.user import get_user_by_email
from app.core.security import verify_password, create_access_token
from app.schemas.token import Token


router = APIRouter(
    prefix="/auth",
    tags=["Auth"],
)


@router.post(
    "/login",
    response_model=Token,
)
async def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    db: AsyncSession = Depends(get_db),
):
    user = await get_user_by_email(db, form_data.username)

    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect email or password",
        )

    if not verify_password(form_data.password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect email or password",
        )

    access_token = create_access_token(
        data={"sub": str(user.id)}
    )

    return {
        "access_token": access_token,
        "token_type": "bearer",
    }
```

Важно: `OAuth2PasswordRequestForm` ожидает поля:

```text
username
password
```

Даже если у нас email, мы передаём email в поле `username`.

---

## Подключаем auth router

В `app/main.py`:

```python
from fastapi import FastAPI

from app.api.routers import users, posts, auth


app = FastAPI(
    title="FastAPI SQLAlchemy PostgreSQL Project",
    version="1.0.0",
)


@app.get("/")
async def root():
    return {"message": "API is running"}


app.include_router(auth.router)
app.include_router(users.router)
app.include_router(posts.router)
```

---

# 34. Создание поста только авторизованным пользователем

Обновим endpoint создания поста.

Файл `app/api/routers/posts.py`:

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

from app.api.deps import get_db, get_current_user
from app.crud import post as post_crud
from app.models.user import User
from app.schemas.post import PostCreate, PostRead, PostUpdate


router = APIRouter(
    prefix="/posts",
    tags=["Posts"],
)


@router.post(
    "",
    response_model=PostRead,
    status_code=status.HTTP_201_CREATED,
)
async def create_post(
    post_in: PostCreate,
    db: AsyncSession = Depends(get_db),
    current_user: User = Depends(get_current_user),
):
    post = await post_crud.create_post(
        db=db,
        post_in=post_in,
        author_id=current_user.id,
    )

    return post
```

Теперь для создания поста нужен токен.

---

# 35. Как пользоваться авторизацией в Swagger

1. Создай пользователя:

```http
POST /users
```

```json
{
  "email": "alex@example.com",
  "name": "Alex",
  "password": "123456"
}
```

2. Открой `/docs`

3. Найди `/auth/login`

4. Нажми `Try it out`

5. Введи:

```text
username: alex@example.com
password: 123456
```

6. Получишь token

7. Вверху нажми кнопку **Authorize**

8. Вставь токен

Теперь защищённые эндпоинты будут работать.

---

# 36. Частые ошибки

## 1. ModuleNotFoundError: No module named app

Запускай из корня проекта:

```bash
uvicorn app.main:app --reload
```

Не из папки `app`.

---

## 2. Таблицы не создаются

Проверь:

- импортированы ли модели в `app/db/base.py`
- правильно ли настроен Alembic
- применил ли миграцию:

```bash
alembic upgrade head
```

---

## 3. Ошибка asyncpg

Проверь, что URL:

```env
DATABASE_URL=postgresql+asyncpg://...
```

А не:

```env
postgresql://...
```

---

## 4. Ошибка psycopg2 при Alembic

Для Alembic sync URL нужен драйвер:

```bash
pip install psycopg2-binary
```

---

## 5. 422 Unprocessable Entity

Это ошибка валидации.

Например, схема требует:

```json
{
  "email": "test@example.com",
  "name": "Alex",
  "password": "123456"
}
```

А ты отправил:

```json
{
  "email": "wrong"
}
```

FastAPI вернёт `422`.

---

# 37. Хорошие практики junior+

## 1. Разделяй слои

Плохо:

```python
@router.post("/users")
async def create_user(...):
    # SQL прямо здесь
```

Лучше:

```text
routers -> crud -> models
```

Роутер отвечает за HTTP.  
CRUD отвечает за работу с базой.  
Модель описывает таблицу.

---

## 2. Никогда не возвращай пароль

Даже хешированный пароль лучше не возвращать.

---

## 3. Используй миграции

Не создавай таблицы вручную в продакшене.

---

## 4. Используй response_model

```python
@router.get("/users/{user_id}", response_model=UserRead)
```

Это:

- фильтрует лишние поля
- документирует API
- проверяет ответ

---

## 5. Используй статусы

```python
status_code=status.HTTP_201_CREATED
```

Это читаемо и правильно.

---

## 6. Не делай огромный `main.py`

Структурируй проект:

```text
api/
crud/
models/
schemas/
core/
db/
```

---

## 7. Для PATCH используй exclude_unset

```python
model_dump(exclude_unset=True)
```

Иначе можно случайно затереть поля на `None`.

---

# 38. Минимальный полный порядок запуска

```bash
docker compose up -d
```

```bash
alembic revision --autogenerate -m "create users and posts"
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

# 39. Что ты должен понимать на уровне junior+

После изучения этого проекта ты должен уметь:

## FastAPI

- создавать приложение
- создавать эндпоинты
- использовать path/query/body параметры
- использовать `APIRouter`
- использовать `Depends`
- возвращать правильные HTTP-статусы
- обрабатывать ошибки через `HTTPException`
- читать Swagger-документацию

## Pydantic

- создавать схемы
- валидировать входные данные
- использовать `Field`
- использовать `EmailStr`
- делать разные схемы:
  - Create
  - Update
  - Read
- понимать `model_dump`
- понимать `exclude_unset`
- понимать `from_attributes=True`

## SQLAlchemy

- создавать модели
- понимать таблицы и колонки
- понимать связи `relationship`
- делать `select`
- делать `add`
- делать `commit`
- делать `refresh`
- делать `delete`
- работать с `AsyncSession`

## PostgreSQL

- понимать базу данных
- понимать таблицы
- понимать связи по `ForeignKey`
- запускать PostgreSQL через Docker
- подключаться через URL

## Alembic

- создавать миграции
- применять миграции
- понимать зачем они нужны

## Auth

- хешировать пароль
- проверять пароль
- создавать JWT
- защищать endpoint через `get_current_user`

---

# 40. Итоговая логика проекта

Когда клиент создаёт пользователя:

```http
POST /users
```

FastAPI:

1. принимает JSON
2. проверяет его через `UserCreate`
3. вызывает endpoint
4. endpoint проверяет email
5. вызывает CRUD
6. CRUD хеширует пароль
7. сохраняет пользователя в PostgreSQL
8. возвращает пользователя через `UserRead`

Когда клиент логинится:

```http
POST /auth/login
```

FastAPI:

1. принимает email и пароль
2. ищет пользователя
3. проверяет пароль
4. создаёт JWT
5. возвращает токен

Когда клиент создаёт пост:

```http
POST /posts
Authorization: Bearer TOKEN
```

FastAPI:

1. достаёт токен
2. проверяет JWT
3. получает текущего пользователя
4. валидирует пост через `PostCreate`
5. создаёт пост с `author_id=current_user.id`
6. сохраняет в базу
7. возвращает `PostRead`

---

# 41. Что учить дальше

Чтобы стать уверенным junior+ backend-разработчиком, дальше изучай:

1. **SQL глубже**
   - JOIN
   - GROUP BY
   - индексы
   - транзакции

2. **SQLAlchemy глубже**
   - eager loading
   - lazy loading
   - joinedload/selectinload
   - транзакции
   - сложные фильтры

3. **FastAPI глубже**
   - middleware
   - exception handlers
   - background tasks
   - lifespan events
   - file upload
   - WebSocket

4. **Авторизация**
   - refresh token
   - roles/permissions
   - OAuth2
   - scopes

5. **Тестирование**
   - pytest
   - test database
   - dependency override

6. **DevOps**
   - Dockerfile
   - Docker Compose для всего приложения
   - Nginx
   - CI/CD

7. **Архитектура**
   - service layer
   - repository pattern
   - clean architecture
   - domain-driven design basics

---

# 42. Очень короткая шпаргалка

## FastAPI endpoint

```python
@router.get("/items/{item_id}")
async def get_item(item_id: int):
    return {"item_id": item_id}
```

## Pydantic schema

```python
class ItemCreate(BaseModel):
    title: str = Field(min_length=3)
```

## SQLAlchemy model

```python
class Item(Base):
    __tablename__ = "items"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(100))
```

## Получить из базы

```python
result = await db.execute(select(Item).where(Item.id == item_id))
item = result.scalar_one_or_none()
```

## Создать в базе

```python
item = Item(title="Test")
db.add(item)
await db.commit()
await db.refresh(item)
```

## Обновить

```python
item.title = "New title"
await db.commit()
await db.refresh(item)
```

## Удалить

```python
await db.delete(item)
await db.commit()
```

---

# 43. Главная мысль

FastAPI-приложение обычно строится так:

```text
HTTP request
    ↓
FastAPI router
    ↓
Pydantic validation
    ↓
Dependency injection
    ↓
CRUD / Service logic
    ↓
SQLAlchemy ORM
    ↓
PostgreSQL
    ↓
Response model
    ↓
JSON response
```

Если ты понимаешь этот путь данных — ты уже понимаешь основу backend-разработки на FastAPI.
