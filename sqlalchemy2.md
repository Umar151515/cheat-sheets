# SQLAlchemy 2 + Alembic + PostgreSQL: подробный практический гайд с нуля до Junior+

Ниже — практическая документация, которая даст базовое и уверенное понимание:

- что такое SQLAlchemy 2;
- чем отличается Core от ORM;
- как подключаться к PostgreSQL;
- как писать модели;
- как работать с сессиями;
- как делать CRUD;
- как писать запросы;
- как использовать связи `one-to-many`, `many-to-many`;
- как работать синхронно и асинхронно;
- как делать миграции через Alembic;
- какие ошибки часто бывают у новичков.

---

# 1. Что такое SQLAlchemy

**SQLAlchemy** — это Python-библиотека для работы с базами данных.

Она состоит из двух больших частей:

## 1.1 SQLAlchemy Core

Это низкоуровневый способ писать SQL-запросы на Python.

Пример:

```python
from sqlalchemy import select

stmt = select(users_table).where(users_table.c.id == 1)
```

SQLAlchemy сам превратит это примерно в:

```sql
SELECT * FROM users WHERE id = 1;
```

## 1.2 SQLAlchemy ORM

ORM — Object Relational Mapper.

Это способ работать с таблицами базы как с Python-классами.

Например, есть таблица `users`.

В Python мы создаём класс:

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str]
```

И потом можем писать:

```python
user = User(email="test@example.com")
session.add(user)
session.commit()
```

А SQLAlchemy сам сделает:

```sql
INSERT INTO users (email) VALUES ('test@example.com');
```

---

# 2. Что такое Alembic

**Alembic** — это инструмент миграций для SQLAlchemy.

Миграции нужны, чтобы управлять изменениями структуры базы данных.

Например:

- создать таблицу;
- добавить колонку;
- удалить колонку;
- создать индекс;
- изменить тип поля;
- добавить внешний ключ.

Без Alembic часто делают так:

```python
Base.metadata.create_all(engine)
```

Но это подходит только для простых тестов.

В реальных проектах нужно использовать Alembic.

---

# 3. Установка PostgreSQL через Docker

Создадим файл `docker-compose.yml`:

```yaml
version: "3.9"

services:
  postgres:
    image: postgres:16
    container_name: sqlalchemy_postgres
    restart: always
    environment:
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD: app_password
      POSTGRES_DB: app_db
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Запуск:

```bash
docker compose up -d
```

Проверить контейнеры:

```bash
docker ps
```

---

# 4. Создание проекта

Пример структуры проекта:

```text
project/
│
├── app/
│   ├── __init__.py
│   │
│   ├── db/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── session_sync.py
│   │   ├── session_async.py
│   │
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── post.py
│   │   ├── tag.py
│   │
│   ├── crud/
│   │   ├── __init__.py
│   │   ├── user_sync.py
│   │   ├── user_async.py
│   │
│   ├── main_sync.py
│   ├── main_async.py
│
├── alembic/
│
├── alembic.ini
├── docker-compose.yml
├── requirements.txt
```

---

# 5. Установка зависимостей

```bash
pip install sqlalchemy alembic psycopg[binary] asyncpg
```

Или `requirements.txt`:

```txt
SQLAlchemy>=2.0
alembic>=1.13
psycopg[binary]>=3.1
asyncpg>=0.29
```

---

# 6. Подключение к PostgreSQL

В SQLAlchemy URL подключения выглядит так:

## Синхронный PostgreSQL через psycopg

```text
postgresql+psycopg://app_user:app_password@localhost:5432/app_db
```

## Асинхронный PostgreSQL через asyncpg

```text
postgresql+asyncpg://app_user:app_password@localhost:5432/app_db
```

Разница:

| Версия | Драйвер | URL |
|---|---|---|
| sync | `psycopg` | `postgresql+psycopg://...` |
| async | `asyncpg` | `postgresql+asyncpg://...` |

---

# 7. Базовый класс ORM

Создадим файл:

```python
# app/db/base.py

from sqlalchemy.orm import DeclarativeBase


class Base(DeclarativeBase):
    pass
```

Все ORM-модели будут наследоваться от `Base`.

---

# 8. Первая модель User

Создадим файл:

```python
# app/models/user.py

from datetime import datetime

from sqlalchemy import String, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db.base import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    email: Mapped[str] = mapped_column(
        String(255),
        unique=True,
        index=True,
        nullable=False,
    )

    username: Mapped[str] = mapped_column(
        String(50),
        unique=True,
        nullable=False,
    )

    hashed_password: Mapped[str] = mapped_column(
        String(255),
        nullable=False,
    )

    is_active: Mapped[bool] = mapped_column(
        default=True,
        nullable=False,
    )

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False,
    )
```

Разберём подробно.

---

## 8.1 `__tablename__`

```python
__tablename__ = "users"
```

Это имя таблицы в базе данных.

SQLAlchemy создаст таблицу:

```sql
CREATE TABLE users (...);
```

---

## 8.2 `Mapped`

```python
id: Mapped[int]
```

`Mapped[int]` говорит SQLAlchemy:

> Это поле является ORM-полем, которое будет храниться в базе.

В SQLAlchemy 2 желательно использовать именно такой стиль.

---

## 8.3 `mapped_column`

```python
id: Mapped[int] = mapped_column(primary_key=True)
```

`mapped_column()` описывает колонку таблицы.

---

## 8.4 Primary Key

```python
primary_key=True
```

Это первичный ключ.

В PostgreSQL SQLAlchemy обычно создаст что-то вроде:

```sql
id SERIAL PRIMARY KEY
```

или `IDENTITY`, в зависимости от настроек и версии.

---

## 8.5 Unique

```python
unique=True
```

Это значит, что значение должно быть уникальным.

Например, два пользователя не могут иметь одинаковый email.

---

## 8.6 Index

```python
index=True
```

Индекс ускоряет поиск.

Например, если часто ищешь пользователя по email:

```sql
SELECT * FROM users WHERE email = 'test@example.com';
```

индекс поможет базе быстрее найти строку.

---

## 8.7 Nullable

```python
nullable=False
```

Значит поле обязательное.

В SQL это:

```sql
email VARCHAR(255) NOT NULL
```

---

## 8.8 Default и server_default

Есть два типа значений по умолчанию:

### Python default

```python
is_active: Mapped[bool] = mapped_column(default=True)
```

SQLAlchemy сам подставит `True` перед отправкой в БД.

### Server default

```python
created_at: Mapped[datetime] = mapped_column(server_default=func.now())
```

Значение поставит сама база данных.

Для дат создания лучше использовать `server_default=func.now()`.

---

# 9. Импорт моделей

Создадим:

```python
# app/models/__init__.py

from app.models.user import User

__all__ = ["User"]
```

Это важно для Alembic.

Alembic должен увидеть все модели, чтобы автоматически создавать миграции.

---

# 10. Синхронное подключение

```python
# app/db/session_sync.py

from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker, Session

DATABASE_URL = "postgresql+psycopg://app_user:app_password@localhost:5432/app_db"

engine = create_engine(
    DATABASE_URL,
    echo=True,
)

SessionLocal = sessionmaker(
    bind=engine,
    class_=Session,
    autoflush=False,
    autocommit=False,
    expire_on_commit=False,
)
```

---

## 10.1 `create_engine`

```python
engine = create_engine(DATABASE_URL)
```

`engine` — это объект, который управляет подключениями к базе.

Важно:

> Engine — это не одно подключение, а менеджер подключений.

---

## 10.2 `echo=True`

```python
echo=True
```

SQLAlchemy будет печатать SQL-запросы в консоль.

Очень полезно для обучения.

В продакшене обычно ставят `False`.

---

## 10.3 `SessionLocal`

```python
SessionLocal = sessionmaker(...)
```

Это фабрика сессий.

Она не создаёт сессию сразу, а позволяет создавать сессии:

```python
session = SessionLocal()
```

---

# 11. Что такое Session

`Session` — это главный объект для работы с ORM.

Через сессию мы:

- добавляем объекты;
- удаляем объекты;
- делаем запросы;
- коммитим транзакции;
- откатываем транзакции.

Пример:

```python
with SessionLocal() as session:
    user = User(email="a@example.com", username="alex", hashed_password="123")
    session.add(user)
    session.commit()
```

---

# 12. Важные методы Session

## 12.1 `add`

```python
session.add(user)
```

Добавляет объект в сессию.

Но в базу он попадёт не сразу.

---

## 12.2 `flush`

```python
session.flush()
```

Отправляет SQL в базу, но не завершает транзакцию.

Например:

```python
session.add(user)
session.flush()

print(user.id)
```

После `flush()` у пользователя уже появится `id`.

---

## 12.3 `commit`

```python
session.commit()
```

Подтверждает транзакцию.

После `commit()` изменения становятся постоянными.

---

## 12.4 `rollback`

```python
session.rollback()
```

Откатывает транзакцию.

Если произошла ошибка, нужно делать rollback.

---

## 12.5 `refresh`

```python
session.refresh(user)
```

Обновляет объект из базы.

Например, если база сама поставила `created_at`.

---

# 13. CRUD синхронно

CRUD:

- Create — создать;
- Read — прочитать;
- Update — обновить;
- Delete — удалить.

Создадим:

```python
# app/crud/user_sync.py

from sqlalchemy import select
from sqlalchemy.orm import Session

from app.models.user import User


def create_user(
    session: Session,
    email: str,
    username: str,
    hashed_password: str,
) -> User:
    user = User(
        email=email,
        username=username,
        hashed_password=hashed_password,
    )

    session.add(user)
    session.commit()
    session.refresh(user)

    return user


def get_user_by_id(session: Session, user_id: int) -> User | None:
    return session.get(User, user_id)


def get_user_by_email(session: Session, email: str) -> User | None:
    stmt = select(User).where(User.email == email)
    return session.scalar(stmt)


def get_users(session: Session, limit: int = 10, offset: int = 0) -> list[User]:
    stmt = (
        select(User)
        .order_by(User.id)
        .limit(limit)
        .offset(offset)
    )

    return list(session.scalars(stmt).all())


def update_user_username(
    session: Session,
    user_id: int,
    new_username: str,
) -> User | None:
    user = session.get(User, user_id)

    if user is None:
        return None

    user.username = new_username

    session.commit()
    session.refresh(user)

    return user


def delete_user(session: Session, user_id: int) -> bool:
    user = session.get(User, user_id)

    if user is None:
        return False

    session.delete(user)
    session.commit()

    return True
```

---

# 14. Запуск синхронного примера

```python
# app/main_sync.py

from app.db.session_sync import SessionLocal
from app.crud.user_sync import (
    create_user,
    get_user_by_email,
    get_users,
    update_user_username,
    delete_user,
)


def main():
    with SessionLocal() as session:
        user = create_user(
            session=session,
            email="ivan@example.com",
            username="ivan",
            hashed_password="hashed_password",
        )

        print("Created user:", user.id, user.email)

        found_user = get_user_by_email(session, "ivan@example.com")
        print("Found user:", found_user)

        users = get_users(session)
        print("All users:", users)

        updated_user = update_user_username(
            session=session,
            user_id=user.id,
            new_username="ivan_new",
        )
        print("Updated:", updated_user.username)

        deleted = delete_user(session, user.id)
        print("Deleted:", deleted)


if __name__ == "__main__":
    main()
```

---

# 15. Важное замечание про таблицы

Пока мы не создали таблицу `users` в базе.

В учебных целях можно сделать:

```python
Base.metadata.create_all(engine)
```

Но в нормальном проекте так делать не надо.

Правильно использовать Alembic.

Сначала разберём модели дальше, потом Alembic.

---

# 16. Связи между таблицами

Допустим, у нас есть пользователи и посты.

Один пользователь может иметь много постов.

Это связь:

```text
User 1 ---- N Post
```

То есть:

- один User;
- много Post.

---

# 17. Модель Post

```python
# app/models/post.py

from datetime import datetime

from sqlalchemy import String, Text, DateTime, ForeignKey, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db.base import Base


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)

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
        index=True,
    )

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False,
    )

    author: Mapped["User"] = relationship(
        back_populates="posts",
    )
```

Теперь надо обновить User:

```python
# app/models/user.py

from datetime import datetime
from typing import List

from sqlalchemy import String, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db.base import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    email: Mapped[str] = mapped_column(
        String(255),
        unique=True,
        index=True,
        nullable=False,
    )

    username: Mapped[str] = mapped_column(
        String(50),
        unique=True,
        nullable=False,
    )

    hashed_password: Mapped[str] = mapped_column(
        String(255),
        nullable=False,
    )

    is_active: Mapped[bool] = mapped_column(
        default=True,
        nullable=False,
    )

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False,
    )

    posts: Mapped[list["Post"]] = relationship(
        back_populates="author",
        cascade="all, delete-orphan",
    )
```

---

## 17.1 ForeignKey

```python
ForeignKey("users.id")
```

Это внешний ключ.

Он говорит:

> Значение `posts.author_id` должно ссылаться на существующий `users.id`.

---

## 17.2 relationship

```python
author: Mapped["User"] = relationship(back_populates="posts")
```

Это ORM-связь.

Она позволяет писать:

```python
post.author
```

А не вручную делать SQL JOIN.

---

## 17.3 back_populates

В `Post`:

```python
author = relationship(back_populates="posts")
```

В `User`:

```python
posts = relationship(back_populates="author")
```

Эти две стороны связаны.

Теперь можно:

```python
user.posts
```

и

```python
post.author
```

---

## 17.4 cascade

```python
cascade="all, delete-orphan"
```

Это значит:

если удалить пользователя, его посты тоже будут удалены через ORM.

---

# 18. Обновим `models/__init__.py`

```python
# app/models/__init__.py

from app.models.user import User
from app.models.post import Post

__all__ = ["User", "Post"]
```

---

# 19. Создание поста синхронно

```python
from sqlalchemy.orm import Session

from app.models.post import Post


def create_post(
    session: Session,
    author_id: int,
    title: str,
    content: str,
) -> Post:
    post = Post(
        author_id=author_id,
        title=title,
        content=content,
    )

    session.add(post)
    session.commit()
    session.refresh(post)

    return post
```

---

# 20. Запросы через SQLAlchemy 2

В SQLAlchemy 2 основной стиль запросов:

```python
from sqlalchemy import select

stmt = select(User)
result = session.execute(stmt)
```

Но есть удобные варианты.

---

## 20.1 Получить один объект по primary key

```python
user = session.get(User, 1)
```

Это самый простой способ получить объект по `id`.

---

## 20.2 Получить одного пользователя по email

```python
stmt = select(User).where(User.email == "ivan@example.com")
user = session.scalar(stmt)
```

`scalar()` возвращает первый объект или `None`.

---

## 20.3 Получить список пользователей

```python
stmt = select(User).order_by(User.id)
users = session.scalars(stmt).all()
```

---

## 20.4 Фильтрация

```python
stmt = select(User).where(User.is_active == True)
```

Лучше так:

```python
stmt = select(User).where(User.is_active.is_(True))
```

---

## 20.5 Несколько условий

```python
stmt = select(User).where(
    User.is_active.is_(True),
    User.email.ilike("%gmail.com"),
)
```

Это означает `AND`.

---

## 20.6 OR

```python
from sqlalchemy import or_

stmt = select(User).where(
    or_(
        User.email == "a@example.com",
        User.email == "b@example.com",
    )
)
```

---

## 20.7 IN

```python
stmt = select(User).where(User.id.in_([1, 2, 3]))
```

---

## 20.8 LIKE / ILIKE

```python
stmt = select(User).where(User.email.ilike("%example.com"))
```

`ilike` — поиск без учёта регистра в PostgreSQL.

---

## 20.9 LIMIT / OFFSET

```python
stmt = select(User).limit(10).offset(20)
```

---

## 20.10 ORDER BY

```python
stmt = select(User).order_by(User.created_at.desc())
```

---

# 21. JOIN

Допустим, хотим получить посты вместе с авторами.

```python
stmt = (
    select(Post)
    .join(Post.author)
    .where(User.email == "ivan@example.com")
)

posts = session.scalars(stmt).all()
```

SQLAlchemy понимает связь `Post.author`.

---

# 22. Проблема N+1

Если сделать:

```python
users = session.scalars(select(User)).all()

for user in users:
    print(user.posts)
```

Может получиться так:

1 запрос — получить пользователей.

Потом для каждого пользователя отдельный запрос на посты.

Если пользователей 100, будет 101 запрос.

Это называется проблема N+1.

---

# 23. Решение N+1: selectinload

```python
from sqlalchemy.orm import selectinload

stmt = select(User).options(selectinload(User.posts))

users = session.scalars(stmt).all()

for user in users:
    print(user.posts)
```

Будет примерно 2 запроса:

1. получить пользователей;
2. получить все посты этих пользователей.

Обычно для списков лучше использовать `selectinload`.

---

# 24. joinedload

```python
from sqlalchemy.orm import joinedload

stmt = select(Post).options(joinedload(Post.author))

posts = session.scalars(stmt).all()
```

`joinedload` делает JOIN и загружает связанные объекты сразу.

Для связей many-to-one часто удобно.

---

# 25. Many-to-many

Допустим:

- у поста может быть много тегов;
- у тега может быть много постов.

```text
Post N ---- N Tag
```

Для этого нужна промежуточная таблица.

---

## 25.1 Модель Tag и association table

```python
# app/models/tag.py

from sqlalchemy import String, ForeignKey, Table, Column
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db.base import Base


post_tags = Table(
    "post_tags",
    Base.metadata,
    Column(
        "post_id",
        ForeignKey("posts.id", ondelete="CASCADE"),
        primary_key=True,
    ),
    Column(
        "tag_id",
        ForeignKey("tags.id", ondelete="CASCADE"),
        primary_key=True,
    ),
)


class Tag(Base):
    __tablename__ = "tags"

    id: Mapped[int] = mapped_column(primary_key=True)

    name: Mapped[str] = mapped_column(
        String(50),
        unique=True,
        nullable=False,
    )

    posts: Mapped[list["Post"]] = relationship(
        secondary=post_tags,
        back_populates="tags",
    )
```

Обновим Post:

```python
# app/models/post.py

from datetime import datetime

from sqlalchemy import String, Text, DateTime, ForeignKey, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db.base import Base
from app.models.tag import post_tags


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)

    title: Mapped[str] = mapped_column(String(200), nullable=False)

    content: Mapped[str] = mapped_column(Text, nullable=False)

    author_id: Mapped[int] = mapped_column(
        ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
        index=True,
    )

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False,
    )

    author: Mapped["User"] = relationship(back_populates="posts")

    tags: Mapped[list["Tag"]] = relationship(
        secondary=post_tags,
        back_populates="posts",
    )
```

Обновим `models/__init__.py`:

```python
from app.models.user import User
from app.models.post import Post
from app.models.tag import Tag, post_tags

__all__ = ["User", "Post", "Tag", "post_tags"]
```

---

## 25.2 Добавление тегов к посту

```python
from app.models.post import Post
from app.models.tag import Tag


def add_tag_to_post(session, post_id: int, tag_name: str):
    post = session.get(Post, post_id)

    if post is None:
        return None

    tag = session.scalar(
        select(Tag).where(Tag.name == tag_name)
    )

    if tag is None:
        tag = Tag(name=tag_name)

    post.tags.append(tag)

    session.commit()
    session.refresh(post)

    return post
```

---

# 26. Асинхронный SQLAlchemy

Асинхронная версия нужна, если приложение async.

Например:

- FastAPI;
- aiohttp;
- aiogram;
- асинхронные воркеры.

Важно:

> Нельзя использовать обычный `Session` внутри async-кода как будто он async.

Для async нужны:

- `create_async_engine`;
- `AsyncSession`;
- `async_sessionmaker`;
- `await`.

---

# 27. Асинхронное подключение

```python
# app/db/session_async.py

from sqlalchemy.ext.asyncio import (
    create_async_engine,
    async_sessionmaker,
    AsyncSession,
)

DATABASE_URL = "postgresql+asyncpg://app_user:app_password@localhost:5432/app_db"

async_engine = create_async_engine(
    DATABASE_URL,
    echo=True,
)

AsyncSessionLocal = async_sessionmaker(
    bind=async_engine,
    class_=AsyncSession,
    autoflush=False,
    expire_on_commit=False,
)
```

---

# 28. AsyncSession

Синхронно:

```python
with SessionLocal() as session:
    ...
```

Асинхронно:

```python
async with AsyncSessionLocal() as session:
    ...
```

Синхронно:

```python
session.commit()
```

Асинхронно:

```python
await session.commit()
```

---

# 29. CRUD асинхронно

```python
# app/crud/user_async.py

from sqlalchemy import select
from sqlalchemy.ext.asyncio import AsyncSession

from app.models.user import User


async def create_user(
    session: AsyncSession,
    email: str,
    username: str,
    hashed_password: str,
) -> User:
    user = User(
        email=email,
        username=username,
        hashed_password=hashed_password,
    )

    session.add(user)

    await session.commit()
    await session.refresh(user)

    return user


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


async def get_users(
    session: AsyncSession,
    limit: int = 10,
    offset: int = 0,
) -> list[User]:
    stmt = (
        select(User)
        .order_by(User.id)
        .limit(limit)
        .offset(offset)
    )

    result = await session.execute(stmt)

    return list(result.scalars().all())


async def update_user_username(
    session: AsyncSession,
    user_id: int,
    new_username: str,
) -> User | None:
    user = await session.get(User, user_id)

    if user is None:
        return None

    user.username = new_username

    await session.commit()
    await session.refresh(user)

    return user


async def delete_user(
    session: AsyncSession,
    user_id: int,
) -> bool:
    user = await session.get(User, user_id)

    if user is None:
        return False

    await session.delete(user)
    await session.commit()

    return True
```

---

# 30. Запуск async-примера

```python
# app/main_async.py

import asyncio

from app.db.session_async import AsyncSessionLocal
from app.crud.user_async import (
    create_user,
    get_user_by_email,
    get_users,
)


async def main():
    async with AsyncSessionLocal() as session:
        user = await create_user(
            session=session,
            email="async@example.com",
            username="async_user",
            hashed_password="hashed_password",
        )

        print("Created:", user.id, user.email)

        found_user = await get_user_by_email(
            session=session,
            email="async@example.com",
        )

        print("Found:", found_user)

        users = await get_users(session)
        print("Users:", users)


if __name__ == "__main__":
    asyncio.run(main())
```

---

# 31. Важная разница sync и async

| Действие | Sync | Async |
|---|---|---|
| Engine | `create_engine` | `create_async_engine` |
| Session | `Session` | `AsyncSession` |
| session context | `with` | `async with` |
| commit | `session.commit()` | `await session.commit()` |
| execute | `session.execute()` | `await session.execute()` |
| refresh | `session.refresh(obj)` | `await session.refresh(obj)` |
| delete | `session.delete(obj)` | `await session.delete(obj)` |

---

# 32. Транзакции

Транзакция — это группа операций, которые должны выполниться полностью или не выполниться вообще.

Например:

1. Создать пользователя.
2. Создать профиль.
3. Создать настройки.

Если на шаге 3 ошибка — надо откатить всё.

---

## 32.1 Sync transaction

```python
with SessionLocal() as session:
    try:
        user = User(
            email="tx@example.com",
            username="tx_user",
            hashed_password="123",
        )

        session.add(user)

        post = Post(
            title="Hello",
            content="Content",
            author=user,
        )

        session.add(post)

        session.commit()

    except Exception:
        session.rollback()
        raise
```

---

## 32.2 Более красивый sync-вариант

```python
with SessionLocal() as session:
    with session.begin():
        user = User(
            email="begin@example.com",
            username="begin_user",
            hashed_password="123",
        )

        session.add(user)

        post = Post(
            title="Post",
            content="Text",
            author=user,
        )

        session.add(post)
```

`session.begin()` сам сделает commit, если ошибок нет.

Если ошибка есть — rollback.

---

## 32.3 Async transaction

```python
async with AsyncSessionLocal() as session:
    async with session.begin():
        user = User(
            email="async_tx@example.com",
            username="async_tx",
            hashed_password="123",
        )

        session.add(user)

        post = Post(
            title="Async post",
            content="Content",
            author=user,
        )

        session.add(post)
```

---

# 33. Alembic с нуля

Теперь самое важное: миграции.

---

# 34. Инициализация Alembic

В корне проекта:

```bash
alembic init alembic
```

Появится:

```text
alembic/
│
├── versions/
├── env.py
├── script.py.mako
│
alembic.ini
```

---

# 35. Настройка `alembic.ini`

Найди строку:

```ini
sqlalchemy.url = driver://user:pass@localhost/dbname
```

Замени на:

```ini
sqlalchemy.url = postgresql+psycopg://app_user:app_password@localhost:5432/app_db
```

Для Alembic проще использовать синхронный драйвер `psycopg`, даже если приложение async.

То есть приложение может быть async:

```text
postgresql+asyncpg://...
```

А Alembic может быть sync:

```text
postgresql+psycopg://...
```

Это нормально.

---

# 36. Настройка Alembic `env.py`

Открой:

```text
alembic/env.py
```

Нужно импортировать `Base` и модели.

Пример:

```python
# alembic/env.py

from logging.config import fileConfig

from sqlalchemy import engine_from_config
from sqlalchemy import pool

from alembic import context

from app.db.base import Base
from app.models import User, Post, Tag  # важно импортировать модели

config = context.config

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


def run_migrations_online() -> None:
    connectable = engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            compare_type=True,
            compare_server_default=True,
        )

        with context.begin_transaction():
            context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

---

## 36.1 Зачем `target_metadata`

```python
target_metadata = Base.metadata
```

Alembic смотрит на metadata и понимает:

- какие таблицы есть в моделях;
- какие колонки;
- какие индексы;
- какие внешние ключи.

---

## 36.2 Зачем импортировать модели

Если ты не импортируешь модели:

```python
from app.models import User, Post, Tag
```

то `Base.metadata` может быть пустым.

Alembic не увидит таблицы.

---

## 36.3 `compare_type=True`

```python
compare_type=True
```

Alembic будет замечать изменения типов колонок.

Например:

```python
String(50)
```

изменили на:

```python
String(100)
```

---

## 36.4 `compare_server_default=True`

Alembic будет сравнивать server default.

Например:

```python
server_default=func.now()
```

---

# 37. Первая миграция

```bash
alembic revision --autogenerate -m "create users posts tags tables"
```

Появится файл:

```text
alembic/versions/xxxx_create_users_posts_tags_tables.py
```

Примерно такой:

```python
"""create users posts tags tables

Revision ID: abc123
Revises:
Create Date: 2025-01-01 12:00:00
"""

from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa


revision: str = "abc123"
down_revision: Union[str, None] = None
branch_labels: Union[str, Sequence[str], None] = None
depends_on: Union[str, Sequence[str], None] = None


def upgrade() -> None:
    op.create_table(
        "users",
        sa.Column("id", sa.Integer(), nullable=False),
        sa.Column("email", sa.String(length=255), nullable=False),
        sa.Column("username", sa.String(length=50), nullable=False),
        sa.Column("hashed_password", sa.String(length=255), nullable=False),
        sa.Column("is_active", sa.Boolean(), nullable=False),
        sa.Column(
            "created_at",
            sa.DateTime(timezone=True),
            server_default=sa.text("now()"),
            nullable=False,
        ),
        sa.PrimaryKeyConstraint("id"),
    )
    op.create_index(op.f("ix_users_email"), "users", ["email"], unique=True)
    op.create_unique_constraint(None, "users", ["username"])

    op.create_table(
        "tags",
        sa.Column("id", sa.Integer(), nullable=False),
        sa.Column("name", sa.String(length=50), nullable=False),
        sa.PrimaryKeyConstraint("id"),
        sa.UniqueConstraint("name"),
    )

    op.create_table(
        "posts",
        sa.Column("id", sa.Integer(), nullable=False),
        sa.Column("title", sa.String(length=200), nullable=False),
        sa.Column("content", sa.Text(), nullable=False),
        sa.Column("author_id", sa.Integer(), nullable=False),
        sa.Column(
            "created_at",
            sa.DateTime(timezone=True),
            server_default=sa.text("now()"),
            nullable=False,
        ),
        sa.ForeignKeyConstraint(
            ["author_id"],
            ["users.id"],
            ondelete="CASCADE",
        ),
        sa.PrimaryKeyConstraint("id"),
    )
    op.create_index(op.f("ix_posts_author_id"), "posts", ["author_id"], unique=False)

    op.create_table(
        "post_tags",
        sa.Column("post_id", sa.Integer(), nullable=False),
        sa.Column("tag_id", sa.Integer(), nullable=False),
        sa.ForeignKeyConstraint(["post_id"], ["posts.id"], ondelete="CASCADE"),
        sa.ForeignKeyConstraint(["tag_id"], ["tags.id"], ondelete="CASCADE"),
        sa.PrimaryKeyConstraint("post_id", "tag_id"),
    )


def downgrade() -> None:
    op.drop_table("post_tags")
    op.drop_index(op.f("ix_posts_author_id"), table_name="posts")
    op.drop_table("posts")
    op.drop_table("tags")
    op.drop_index(op.f("ix_users_email"), table_name="users")
    op.drop_table("users")
```

---

# 38. Применить миграцию

```bash
alembic upgrade head
```

Это создаст таблицы в базе.

---

# 39. Проверить текущую миграцию

```bash
alembic current
```

---

# 40. История миграций

```bash
alembic history
```

---

# 41. Откат миграции

Откатить на одну миграцию назад:

```bash
alembic downgrade -1
```

Откатить всё:

```bash
alembic downgrade base
```

Снова применить всё:

```bash
alembic upgrade head
```

---

# 42. Как правильно менять модели

Допустим, хотим добавить пользователю поле `bio`.

В модель User добавляем:

```python
bio: Mapped[str | None] = mapped_column(
    String(500),
    nullable=True,
)
```

Теперь создаём миграцию:

```bash
alembic revision --autogenerate -m "add bio to users"
```

Alembic создаст:

```python
def upgrade() -> None:
    op.add_column(
        "users",
        sa.Column("bio", sa.String(length=500), nullable=True),
    )


def downgrade() -> None:
    op.drop_column("users", "bio")
```

Применяем:

```bash
alembic upgrade head
```

---

# 43. Важное правило Alembic

После команды:

```bash
alembic revision --autogenerate
```

всегда открывай файл миграции и проверяй его.

Alembic помогает, но он не идеален.

---

# 44. Alembic async-вариант

Если хочешь полностью async Alembic:

```bash
alembic init -t async alembic
```

Тогда `env.py` будет async.

Пример:

```python
# alembic/env.py

import asyncio
from logging.config import fileConfig

from sqlalchemy import pool
from sqlalchemy.engine import Connection
from sqlalchemy.ext.asyncio import async_engine_from_config

from alembic import context

from app.db.base import Base
from app.models import User, Post, Tag

config = context.config

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
        compare_type=True,
        compare_server_default=True,
    )

    with context.begin_transaction():
        context.run_migrations()


def do_run_migrations(connection: Connection) -> None:
    context.configure(
        connection=connection,
        target_metadata=target_metadata,
        compare_type=True,
        compare_server_default=True,
    )

    with context.begin_transaction():
        context.run_migrations()


async def run_async_migrations() -> None:
    connectable = async_engine_from_config(
        config.get_section(config.config_ini_section, {}),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    async with connectable.connect() as connection:
        await connection.run_sync(do_run_migrations)

    await connectable.dispose()


def run_migrations_online() -> None:
    asyncio.run(run_async_migrations())


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

В `alembic.ini` тогда:

```ini
sqlalchemy.url = postgresql+asyncpg://app_user:app_password@localhost:5432/app_db
```

Но новичку проще начинать с sync Alembic.

---

# 45. Полезные типы колонок

```python
from sqlalchemy import (
    String,
    Text,
    Integer,
    BigInteger,
    Boolean,
    DateTime,
    Date,
    Numeric,
    JSON,
)
```

Примеры:

```python
name: Mapped[str] = mapped_column(String(100))
description: Mapped[str | None] = mapped_column(Text, nullable=True)
age: Mapped[int] = mapped_column(Integer)
price: Mapped[Decimal] = mapped_column(Numeric(10, 2))
is_active: Mapped[bool] = mapped_column(Boolean, default=True)
data: Mapped[dict] = mapped_column(JSON)
```

Для PostgreSQL можно использовать:

```python
from sqlalchemy.dialects.postgresql import UUID, JSONB, ARRAY
```

---

# 46. UUID primary key

```python
import uuid

from sqlalchemy.dialects.postgresql import UUID


id: Mapped[uuid.UUID] = mapped_column(
    UUID(as_uuid=True),
    primary_key=True,
    default=uuid.uuid4,
)
```

---

# 47. Индексы и ограничения

## 47.1 Простое unique

```python
email: Mapped[str] = mapped_column(unique=True)
```

## 47.2 Составной unique

Например, один пользователь не может иметь два поста с одинаковым title:

```python
from sqlalchemy import UniqueConstraint


class Post(Base):
    __tablename__ = "posts"

    __table_args__ = (
        UniqueConstraint("author_id", "title", name="uq_author_title"),
    )
```

## 47.3 Индекс

```python
from sqlalchemy import Index


class Post(Base):
    __tablename__ = "posts"

    __table_args__ = (
        Index("ix_posts_title_created_at", "title", "created_at"),
    )
```

---

# 48. Частые ошибки новичков

## Ошибка 1: забыли импортировать модели в Alembic

Alembic создаёт пустую миграцию.

Решение:

```python
from app.models import User, Post, Tag
target_metadata = Base.metadata
```

---

## Ошибка 2: используют `create_all` вместе с Alembic

Не надо в нормальном проекте:

```python
Base.metadata.create_all(engine)
```

Если используешь Alembic — структуру базы меняет Alembic.

---

## Ошибка 3: забыли `await` в async

Неправильно:

```python
session.commit()
```

Правильно:

```python
await session.commit()
```

---

## Ошибка 4: используют sync-драйвер в async engine

Неправильно:

```python
postgresql+psycopg://...
```

для:

```python
create_async_engine()
```

Правильно:

```python
postgresql+asyncpg://...
```

---

## Ошибка 5: одна Session на всё приложение

Нельзя делать глобальную сессию:

```python
session = SessionLocal()
```

и использовать её везде.

Правильно:

```python
with SessionLocal() as session:
    ...
```

или в FastAPI через dependency.

---

# 49. Как мыслить при работе с SQLAlchemy

Обычно алгоритм такой:

1. Создал или изменил модель.
2. Создал миграцию Alembic.
3. Проверил миграцию.
4. Применил миграцию.
5. Написал CRUD.
6. Написал запросы.
7. Проверил SQL через `echo=True`.

---

# 50. Мини-шпаргалка

## Sync

```python
with SessionLocal() as session:
    user = User(
        email="a@example.com",
        username="alex",
        hashed_password="123",
    )

    session.add(user)
    session.commit()
    session.refresh(user)
```

## Async

```python
async with AsyncSessionLocal() as session:
    user = User(
        email="a@example.com",
        username="alex",
        hashed_password="123",
    )

    session.add(user)
    await session.commit()
    await session.refresh(user)
```

## Select sync

```python
stmt = select(User).where(User.email == "a@example.com")
user = session.scalar(stmt)
```

## Select async

```python
stmt = select(User).where(User.email == "a@example.com")
result = await session.execute(stmt)
user = result.scalar_one_or_none()
```

## Alembic

```bash
alembic revision --autogenerate -m "message"
alembic upgrade head
alembic downgrade -1
```

---

# 51. Что нужно знать на уровень уверенного Junior+

Ты должен уверенно понимать:

1. Что такое ORM-модель.
2. Что такое `Base`.
3. Что такое `Engine`.
4. Что такое `Session`.
5. Разницу между `flush`, `commit`, `rollback`, `refresh`.
6. Как писать `select`.
7. Как фильтровать данные.
8. Как делать пагинацию через `limit` и `offset`.
9. Как делать `relationship`.
10. Разницу между `ForeignKey` и `relationship`.
11. Что такое lazy loading и проблема N+1.
12. Когда использовать `selectinload`.
13. Как создавать миграции Alembic.
14. Почему миграции надо проверять.
15. Чем sync SQLAlchemy отличается от async.
16. Почему нельзя использовать одну Session глобально.
17. Почему нельзя полагаться на `create_all` в реальном проекте.

---

# 52. Практическое задание

Чтобы закрепить материал, сделай мини-проект блог.

## Таблицы

### users

- id
- email
- username
- hashed_password
- is_active
- created_at

### posts

- id
- title
- content
- author_id
- created_at

### tags

- id
- name

### post_tags

- post_id
- tag_id

## Функции

Синхронные и асинхронные:

1. Создать пользователя.
2. Найти пользователя по email.
3. Создать пост.
4. Получить все посты пользователя.
5. Добавить тег к посту.
6. Получить посты по тегу.
7. Обновить title поста.
8. Удалить пост.
9. Получить пользователей вместе с постами через `selectinload`.
10. Сделать миграции через Alembic.

Если ты это сделаешь без подсказок — у тебя уже будет хорошая база Junior+ по SQLAlchemy 2 и Alembic.
