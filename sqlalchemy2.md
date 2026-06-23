Ниже — **практическая документация/учебник по SQLAlchemy 2 + Alembic + PostgreSQL** с нуля до уровня уверенного junior+.

Будем использовать **строго современный SQLAlchemy 2**:

- `DeclarativeBase`
- `Mapped[...]`
- `mapped_column(...)`
- `select(...)`
- `Session.execute(...)`
- `Session.scalars(...)`
- `relationship(...)`
- Alembic migrations

Без старого стиля:

```python
session.query(User).filter(...)
```

В SQLAlchemy 2 так писать уже не нужно.

---

# 1. Что такое SQLAlchemy

**SQLAlchemy** — это библиотека для работы с базами данных в Python.

Она состоит из двух больших частей:

## 1.1 SQLAlchemy Core

Это низкоуровневый слой.

Он позволяет писать SQL-запросы через Python-объекты:

```python
from sqlalchemy import select

stmt = select(users_table).where(users_table.c.email == "test@example.com")
```

Core ближе к чистому SQL.

## 1.2 SQLAlchemy ORM

ORM — Object Relational Mapper.

Он позволяет работать с таблицами как с Python-классами.

Например, есть таблица `users`.

В ORM мы описываем её как класс:

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str]
```

И потом можем создавать пользователя так:

```python
user = User(email="test@example.com")
session.add(user)
session.commit()
```

ORM сам преобразует Python-объект в SQL:

```sql
INSERT INTO users (email) VALUES ('test@example.com');
```

---

# 2. Что такое Alembic

**Alembic** — это инструмент миграций для SQLAlchemy.

Миграции нужны, чтобы управлять изменениями структуры базы данных.

Например:

- создать таблицу `users`
- добавить колонку `age`
- удалить колонку `name`
- создать индекс
- добавить внешний ключ
- изменить тип данных

Без Alembic тебе пришлось бы вручную писать SQL:

```sql
ALTER TABLE users ADD COLUMN age INTEGER;
```

С Alembic это делается через миграции:

```python
def upgrade():
    op.add_column("users", sa.Column("age", sa.Integer(), nullable=True))


def downgrade():
    op.drop_column("users", "age")
```

---

# 3. Подготовка проекта

Создадим проект.

```bash
mkdir sqlalchemy_alembic_project
cd sqlalchemy_alembic_project
```

Создадим виртуальное окружение:

```bash
python -m venv .venv
```

Активируем.

Linux / macOS:

```bash
source .venv/bin/activate
```

Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

Установим зависимости:

```bash
pip install sqlalchemy alembic psycopg[binary] python-dotenv
```

Проверить версии:

```bash
python -c "import sqlalchemy; print(sqlalchemy.__version__)"
```

Желательно SQLAlchemy 2.x:

```text
2.0.x
```

---

# 4. Поднимаем PostgreSQL через Docker

Создай файл `docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:16
    container_name: sqlalchemy_postgres
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

Проверить контейнер:

```bash
docker ps
```

Подключиться внутрь PostgreSQL:

```bash
docker exec -it sqlalchemy_postgres psql -U app_user -d app_db
```

Выйти:

```sql
\q
```

---

# 5. Структура проекта

Сделаем такую структуру:

```text
sqlalchemy_alembic_project/
│
├── app/
│   ├── __init__.py
│   ├── config.py
│   ├── database.py
│   ├── models.py
│   └── main.py
│
├── alembic/
│   ├── versions/
│   ├── env.py
│   ├── README
│   └── script.py.mako
│
├── alembic.ini
├── docker-compose.yml
└── .env
```

Создадим папку `app`:

```bash
mkdir app
touch app/__init__.py
```

На Windows можно создать файлы вручную.

---

# 6. Настройки подключения

Создадим файл `.env`:

```env
DATABASE_URL=postgresql+psycopg://app_user:app_password@localhost:5432/app_db
```

Разберём строку:

```text
postgresql+psycopg://app_user:app_password@localhost:5432/app_db
```

Где:

| Часть | Значение |
|---|---|
| `postgresql` | тип базы данных |
| `psycopg` | драйвер подключения |
| `app_user` | пользователь |
| `app_password` | пароль |
| `localhost` | хост |
| `5432` | порт PostgreSQL |
| `app_db` | имя базы данных |

Теперь `app/config.py`:

```python
from pydantic_settings import BaseSettings, SettingsConfigDict
```

Но мы не устанавливали `pydantic-settings`. Чтобы не усложнять, сделаем проще через `python-dotenv`.

`app/config.py`:

```python
import os
from dotenv import load_dotenv

load_dotenv()


DATABASE_URL = os.getenv("DATABASE_URL")

if DATABASE_URL is None:
    raise RuntimeError("DATABASE_URL is not set")
```

---

# 7. Подключение SQLAlchemy

Создадим `app/database.py`.

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker

from app.config import DATABASE_URL


engine = create_engine(
    DATABASE_URL,
    echo=True,
)

SessionLocal = sessionmaker(
    bind=engine,
    autoflush=False,
    expire_on_commit=False,
)


class Base(DeclarativeBase):
    pass
```

Разберём подробно.

---

## 7.1 `create_engine`

```python
engine = create_engine(DATABASE_URL, echo=True)
```

`Engine` — это главный объект SQLAlchemy Core, который знает:

- куда подключаться
- через какой драйвер
- как управлять пулом соединений
- как выполнять SQL

`echo=True` означает:

> SQLAlchemy будет печатать SQL-запросы в консоль.

Это очень полезно для обучения.

В продакшене обычно ставят:

```python
echo=False
```

---

## 7.2 `Session`

```python
SessionLocal = sessionmaker(bind=engine)
```

`Session` — это объект ORM.

Через него мы:

- добавляем объекты
- удаляем объекты
- выполняем запросы
- фиксируем транзакции
- откатываем транзакции

Важно:

> `Session` — это не просто соединение с БД. Это рабочая область ORM.

Она хранит объекты, следит за их изменениями и решает, какие SQL-запросы нужно выполнить.

---

## 7.3 `Base`

```python
class Base(DeclarativeBase):
    pass
```

`Base` — базовый класс для всех ORM-моделей.

Все модели будут наследоваться от него:

```python
class User(Base):
    ...
```

SQLAlchemy собирает информацию обо всех таблицах в:

```python
Base.metadata
```

Именно `Base.metadata` потом понадобится Alembic.

---

# 8. Первая модель

Создадим `app/models.py`.

```python
from datetime import datetime

from sqlalchemy import String, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column

from app.database import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    email: Mapped[str] = mapped_column(
        String(255),
        unique=True,
        nullable=False,
        index=True,
    )

    username: Mapped[str] = mapped_column(
        String(100),
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

    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        onupdate=func.now(),
        nullable=False,
    )
```

---

# 9. Разбор модели

## 9.1 Класс модели

```python
class User(Base):
```

Мы создаём Python-класс, который соответствует таблице в БД.

---

## 9.2 Имя таблицы

```python
__tablename__ = "users"
```

Это имя таблицы в PostgreSQL.

SQLAlchemy будет считать, что класс `User` связан с таблицей `users`.

---

## 9.3 Тип `Mapped`

```python
id: Mapped[int]
```

В SQLAlchemy 2 современный стиль такой:

```python
field_name: Mapped[python_type] = mapped_column(...)
```

`Mapped[int]` говорит:

> это ORM-поле, которое будет связано с колонкой таблицы, и в Python это будет `int`.

---

## 9.4 `mapped_column`

```python
id: Mapped[int] = mapped_column(primary_key=True)
```

`mapped_column` описывает колонку таблицы.

Например:

```python
email: Mapped[str] = mapped_column(String(255), nullable=False)
```

Это значит:

```sql
email VARCHAR(255) NOT NULL
```

---

## 9.5 Primary Key

```python
id: Mapped[int] = mapped_column(primary_key=True)
```

Primary Key — это уникальный идентификатор строки.

PostgreSQL сам будет генерировать значения для `id`.

Обычно это:

```sql
id SERIAL PRIMARY KEY
```

или в новых версиях:

```sql
id INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY
```

SQLAlchemy сам создаст подходящую конструкцию.

---

## 9.6 `nullable`

```python
nullable=False
```

Означает, что колонка не может быть `NULL`.

Пример:

```python
email: Mapped[str] = mapped_column(nullable=False)
```

В SQL:

```sql
email VARCHAR NOT NULL
```

Если попытаться вставить строку без email, PostgreSQL выдаст ошибку.

---

## 9.7 `unique`

```python
unique=True
```

Означает, что значение должно быть уникальным.

Например:

```python
email: Mapped[str] = mapped_column(unique=True)
```

Нельзя создать двух пользователей с одинаковым email.

---

## 9.8 `index`

```python
index=True
```

Создаёт индекс.

Индекс ускоряет поиск.

Например, если часто ищем пользователя по email:

```python
select(User).where(User.email == "test@example.com")
```

то индекс на `email` полезен.

Но индексы не бесплатные:

- ускоряют чтение
- замедляют запись
- занимают место

---

## 9.9 `server_default`

```python
server_default=func.now()
```

Это значение по умолчанию на стороне базы данных.

То есть PostgreSQL сам подставит текущее время.

Важно различать:

```python
default=datetime.utcnow
```

и

```python
server_default=func.now()
```

### `default`

Работает на стороне Python.

```python
created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)
```

### `server_default`

Работает на стороне БД.

```python
created_at: Mapped[datetime] = mapped_column(server_default=func.now())
```

Для дат создания обычно лучше `server_default`.

---

# 10. Создание таблиц без Alembic

Для обучения можно создать таблицы напрямую:

```python
Base.metadata.create_all(engine)
```

Но в реальных проектах с Alembic так обычно не делают.

Почему?

Потому что `create_all`:

- создаёт таблицы
- но плохо управляет изменениями
- не хранит историю миграций
- не умеет нормально обновлять уже существующие таблицы

Поэтому дальше будем использовать Alembic.

---

# 11. Инициализация Alembic

В корне проекта выполним:

```bash
alembic init alembic
```

Появится:

```text
alembic/
alembic.ini
```

---

# 12. Настройка `alembic.ini`

Открой `alembic.ini`.

Найди строку:

```ini
sqlalchemy.url = driver://user:pass@localhost/dbname
```

Можно заменить напрямую:

```ini
sqlalchemy.url = postgresql+psycopg://app_user:app_password@localhost:5432/app_db
```

Но лучше не хранить пароль в `alembic.ini`.

Мы настроим Alembic так, чтобы он брал URL из `.env`.

---

# 13. Настройка `alembic/env.py`

Открой `alembic/env.py`.

Там будет много стандартного кода.

Нужно сделать так, чтобы Alembic знал про наши модели.

Пример полноценного `alembic/env.py`:

```python
from logging.config import fileConfig

from sqlalchemy import engine_from_config
from sqlalchemy import pool

from alembic import context

from dotenv import load_dotenv
import os

from app.database import Base
from app import models  # важно: импортируем модели, чтобы они попали в metadata


load_dotenv()

config = context.config

if config.config_file_name is not None:
    fileConfig(config.config_file_name)


DATABASE_URL = os.getenv("DATABASE_URL")

if DATABASE_URL is None:
    raise RuntimeError("DATABASE_URL is not set")


config.set_main_option("sqlalchemy.url", DATABASE_URL)

target_metadata = Base.metadata


def run_migrations_offline() -> None:
    url = config.get_main_option("sqlalchemy.url")

    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
        compare_type=True,
    )

    with context.begin_transaction():
        context.run_migrations()


def run_migrations_online() -> None:
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            compare_type=True,
        )

        with context.begin_transaction():
            context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

---

# 14. Очень важный момент про импорт моделей

В `env.py` есть строка:

```python
from app import models
```

Она нужна обязательно.

Почему?

Alembic смотрит на:

```python
Base.metadata
```

Но если модель `User` не была импортирована, Python о ней не знает.

Значит, она не попадёт в `Base.metadata`.

Поэтому Alembic не увидит таблицу.

---

# 15. Первая миграция

Создаём миграцию:

```bash
alembic revision --autogenerate -m "create users table"
```

Alembic сравнит:

- текущее состояние базы
- состояние моделей SQLAlchemy

И создаст файл в:

```text
alembic/versions/
```

Например:

```text
a1b2c3d4_create_users_table.py
```

Открой файл. Там будет примерно:

```python
"""create users table

Revision ID: a1b2c3d4
Revises: 
Create Date: ...

"""

from typing import Sequence, Union

from alembic import op
import sqlalchemy as sa


revision: str = "a1b2c3d4"
down_revision: Union[str, None] = None
branch_labels: Union[str, Sequence[str], None] = None
depends_on: Union[str, Sequence[str], None] = None


def upgrade() -> None:
    op.create_table(
        "users",
        sa.Column("id", sa.Integer(), nullable=False),
        sa.Column("email", sa.String(length=255), nullable=False),
        sa.Column("username", sa.String(length=100), nullable=False),
        sa.Column("hashed_password", sa.String(length=255), nullable=False),
        sa.Column("is_active", sa.Boolean(), nullable=False),
        sa.Column(
            "created_at",
            sa.DateTime(timezone=True),
            server_default=sa.text("now()"),
            nullable=False,
        ),
        sa.Column(
            "updated_at",
            sa.DateTime(timezone=True),
            server_default=sa.text("now()"),
            nullable=False,
        ),
        sa.PrimaryKeyConstraint("id"),
        sa.UniqueConstraint("email"),
        sa.UniqueConstraint("username"),
    )

    op.create_index(
        op.f("ix_users_email"),
        "users",
        ["email"],
        unique=False,
    )


def downgrade() -> None:
    op.drop_index(op.f("ix_users_email"), table_name="users")
    op.drop_table("users")
```

---

# 16. Применение миграции

```bash
alembic upgrade head
```

Это применит все миграции до последней версии.

Проверим в PostgreSQL:

```bash
docker exec -it sqlalchemy_postgres psql -U app_user -d app_db
```

Внутри:

```sql
\dt
```

Должны увидеть:

```text
alembic_version
users
```

Посмотреть структуру таблицы:

```sql
\d users
```

---

# 17. Что такое `alembic_version`

Alembic создаёт служебную таблицу:

```text
alembic_version
```

В ней хранится текущая версия миграции.

Например:

```sql
SELECT * FROM alembic_version;
```

Результат:

```text
version_num
------------
a1b2c3d4
```

Это значит:

> база находится на миграции `a1b2c3d4`.

---

# 18. Создание данных через SQLAlchemy

Создадим `app/main.py`.

```python
from sqlalchemy import select
from sqlalchemy.exc import IntegrityError

from app.database import SessionLocal
from app.models import User


def create_user(
    email: str,
    username: str,
    hashed_password: str,
) -> User:
    with SessionLocal() as session:
        user = User(
            email=email,
            username=username,
            hashed_password=hashed_password,
        )

        session.add(user)

        try:
            session.commit()
        except IntegrityError:
            session.rollback()
            raise ValueError("User with this email or username already exists")

        session.refresh(user)

        return user


if __name__ == "__main__":
    user = create_user(
        email="john@example.com",
        username="john",
        hashed_password="not_real_hash",
    )

    print(user.id)
    print(user.email)
```

Запуск:

```bash
python -m app.main
```

---

# 19. Разбор создания объекта

```python
user = User(...)
```

Пока это просто Python-объект.

Он ещё не находится в базе.

---

## 19.1 `session.add`

```python
session.add(user)
```

Теперь объект добавлен в сессию.

Но SQL `INSERT` может ещё не выполниться.

Объект находится в состоянии `pending`.

---

## 19.2 `commit`

```python
session.commit()
```

`commit` делает несколько вещей:

1. Выполняет `flush`
2. Отправляет SQL-запросы в БД
3. Фиксирует транзакцию

Пример SQL:

```sql
INSERT INTO users (email, username, hashed_password, is_active)
VALUES (...)
RETURNING users.id, users.created_at, users.updated_at;
```

---

## 19.3 `rollback`

```python
session.rollback()
```

Если произошла ошибка, транзакцию нужно откатить.

Например, если нарушен `unique=True`.

---

## 19.4 `refresh`

```python
session.refresh(user)
```

Обновляет объект из базы.

Это нужно, чтобы получить:

- `id`
- `created_at`
- `updated_at`
- значения, которые поставила БД

---

# 20. Состояния ORM-объекта

В SQLAlchemy ORM объект может быть в разных состояниях.

## 20.1 Transient

```python
user = User(email="a@b.com")
```

Объект создан, но сессия о нём не знает.

---

## 20.2 Pending

```python
session.add(user)
```

Объект добавлен в сессию, но ещё не записан в БД.

---

## 20.3 Persistent

```python
session.commit()
```

Объект связан с реальной строкой в БД.

---

## 20.4 Detached

```python
session.close()
```

Сессия закрыта, объект больше не связан с ней.

---

# 21. Чтение данных

## 21.1 Получить всех пользователей

```python
from sqlalchemy import select

from app.database import SessionLocal
from app.models import User


def get_users() -> list[User]:
    with SessionLocal() as session:
        stmt = select(User)

        users = session.scalars(stmt).all()

        return list(users)
```

Современный стиль SQLAlchemy 2:

```python
stmt = select(User)
users = session.scalars(stmt).all()
```

---

## 21.2 Получить пользователя по id

Вариант 1:

```python
def get_user_by_id(user_id: int) -> User | None:
    with SessionLocal() as session:
        return session.get(User, user_id)
```

`session.get()` используется для поиска по primary key.

Это самый простой и правильный способ получить объект по `id`.

---

## 21.3 Получить пользователя по email

```python
def get_user_by_email(email: str) -> User | None:
    with SessionLocal() as session:
        stmt = select(User).where(User.email == email)

        return session.scalar(stmt)
```

Обрати внимание:

```python
session.scalar(stmt)
```

Возвращает:

- первый объект
- или `None`

---

## 21.4 Разница `scalar` и `scalars`

### `scalar`

```python
result = session.scalar(select(User))
```

Возвращает одно значение или `None`.

### `scalars`

```python
result = session.scalars(select(User)).all()
```

Возвращает поток ORM-объектов.

Пример:

```python
users = session.scalars(select(User)).all()
```

---

# 22. Фильтрация

```python
stmt = select(User).where(User.is_active == True)
```

Лучше для boolean:

```python
stmt = select(User).where(User.is_active.is_(True))
```

Примеры:

```python
select(User).where(User.email == "john@example.com")

select(User).where(User.username != "admin")

select(User).where(User.id > 10)

select(User).where(User.email.like("%@gmail.com"))

select(User).where(User.email.ilike("%@gmail.com"))
```

`like` — чувствителен к регистру.

`ilike` — нечувствителен к регистру в PostgreSQL.

---

# 23. Несколько условий

## 23.1 Через несколько `.where`

```python
stmt = (
    select(User)
    .where(User.is_active.is_(True))
    .where(User.email.ilike("%@gmail.com"))
)
```

Это будет SQL:

```sql
WHERE users.is_active IS true
  AND users.email ILIKE '%@gmail.com'
```

---

## 23.2 Через `and_`

```python
from sqlalchemy import and_

stmt = select(User).where(
    and_(
        User.is_active.is_(True),
        User.email.ilike("%@gmail.com"),
    )
)
```

---

## 23.3 Через `or_`

```python
from sqlalchemy import or_

stmt = select(User).where(
    or_(
        User.email == "admin@example.com",
        User.username == "admin",
    )
)
```

---

# 24. Сортировка

```python
stmt = select(User).order_by(User.created_at.desc())
```

По возрастанию:

```python
stmt = select(User).order_by(User.created_at.asc())
```

Несколько сортировок:

```python
stmt = select(User).order_by(
    User.is_active.desc(),
    User.created_at.desc(),
)
```

---

# 25. Limit и offset

```python
stmt = (
    select(User)
    .order_by(User.id)
    .limit(10)
    .offset(20)
)
```

Это означает:

> пропустить первые 20 строк и взять следующие 10.

Используется для пагинации.

---

# 26. Обновление данных

ORM-вариант:

```python
def update_user_username(user_id: int, new_username: str) -> User | None:
    with SessionLocal() as session:
        user = session.get(User, user_id)

        if user is None:
            return None

        user.username = new_username

        session.commit()
        session.refresh(user)

        return user
```

SQLAlchemy сам поймёт, что поле изменилось, и выполнит:

```sql
UPDATE users SET username = ... WHERE users.id = ...
```

---

# 27. Удаление данных

```python
def delete_user(user_id: int) -> bool:
    with SessionLocal() as session:
        user = session.get(User, user_id)

        if user is None:
            return False

        session.delete(user)
        session.commit()

        return True
```

---

# 28. Bulk update и bulk delete

Иногда нужно обновить сразу много строк.

```python
from sqlalchemy import update


def deactivate_all_users() -> None:
    with SessionLocal() as session:
        stmt = (
            update(User)
            .where(User.is_active.is_(True))
            .values(is_active=False)
        )

        session.execute(stmt)
        session.commit()
```

Удаление:

```python
from sqlalchemy import delete


def delete_inactive_users() -> None:
    with SessionLocal() as session:
        stmt = delete(User).where(User.is_active.is_(False))

        session.execute(stmt)
        session.commit()
```

Важно:

> Bulk update/delete обходят обычную ORM-логику объектов. Используй аккуратно.

---

# 29. Транзакции

Транзакция — это набор операций, которые должны выполниться вместе.

Например:

1. Создать пользователя
2. Создать профиль пользователя

Если профиль не создался — пользователь тоже не должен сохраниться.

Пример:

```python
def create_two_users():
    with SessionLocal() as session:
        try:
            user1 = User(
                email="a@example.com",
                username="a",
                hashed_password="hash",
            )
            user2 = User(
                email="b@example.com",
                username="b",
                hashed_password="hash",
            )

            session.add_all([user1, user2])

            session.commit()
        except Exception:
            session.rollback()
            raise
```

Лучший стиль:

```python
def create_two_users():
    with SessionLocal() as session:
        with session.begin():
            user1 = User(
                email="a@example.com",
                username="a",
                hashed_password="hash",
            )
            user2 = User(
                email="b@example.com",
                username="b",
                hashed_password="hash",
            )

            session.add_all([user1, user2])
```

`with session.begin()`:

- начинает транзакцию
- если всё хорошо — делает commit
- если ошибка — делает rollback

---

# 30. Flush

`flush` отправляет SQL в базу, но не делает commit.

Пример:

```python
with SessionLocal() as session:
    user = User(
        email="flush@example.com",
        username="flush_user",
        hashed_password="hash",
    )

    session.add(user)
    session.flush()

    print(user.id)

    session.commit()
```

После `flush()` у пользователя уже может появиться `id`.

Но если потом сделать:

```python
session.rollback()
```

то запись исчезнет.

---

# 31. Commit vs flush

## `flush`

- отправляет SQL
- транзакция ещё не зафиксирована
- можно откатить

## `commit`

- вызывает flush
- фиксирует транзакцию
- изменения становятся постоянными

---

# 32. Связи между таблицами

Добавим задачи пользователя.

Один пользователь может иметь много задач.

Это связь:

```text
User 1 ---- N Task
```

---

# 33. Модель `Task`

Обновим `app/models.py`.

```python
from datetime import datetime

from sqlalchemy import (
    String,
    DateTime,
    Text,
    ForeignKey,
    func,
)
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.database import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    email: Mapped[str] = mapped_column(
        String(255),
        unique=True,
        nullable=False,
        index=True,
    )

    username: Mapped[str] = mapped_column(
        String(100),
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

    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        onupdate=func.now(),
        nullable=False,
    )

    tasks: Mapped[list["Task"]] = relationship(
        back_populates="user",
        cascade="all, delete-orphan",
    )


class Task(Base):
    __tablename__ = "tasks"

    id: Mapped[int] = mapped_column(primary_key=True)

    title: Mapped[str] = mapped_column(
        String(200),
        nullable=False,
    )

    description: Mapped[str | None] = mapped_column(
        Text,
        nullable=True,
    )

    is_done: Mapped[bool] = mapped_column(
        default=False,
        nullable=False,
    )

    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
        index=True,
    )

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False,
    )

    user: Mapped["User"] = relationship(
        back_populates="tasks",
    )
```

---

# 34. Разбор связи

## 34.1 ForeignKey

```python
user_id: Mapped[int] = mapped_column(
    ForeignKey("users.id", ondelete="CASCADE"),
    nullable=False,
)
```

Это внешний ключ.

Он говорит:

> поле `tasks.user_id` ссылается на `users.id`.

В SQL:

```sql
FOREIGN KEY(user_id) REFERENCES users(id)
```

---

## 34.2 Relationship

```python
tasks: Mapped[list["Task"]] = relationship(back_populates="user")
```

Это ORM-связь.

Она не всегда создаёт колонку в БД.

Колонку создаёт `ForeignKey`.

`relationship` нужен, чтобы писать удобно:

```python
user.tasks
```

и получать задачи пользователя.

---

## 34.3 Двусторонняя связь

В `User`:

```python
tasks: Mapped[list["Task"]] = relationship(back_populates="user")
```

В `Task`:

```python
user: Mapped["User"] = relationship(back_populates="tasks")
```

Это значит:

```python
task.user
```

даёт пользователя задачи.

```python
user.tasks
```

даёт список задач пользователя.

---

## 34.4 Cascade

```python
cascade="all, delete-orphan"
```

Это ORM-каскад.

Он означает:

- если удалить пользователя через ORM, его задачи тоже удалятся
- если убрать задачу из `user.tasks`, она будет удалена

Пример:

```python
user.tasks.remove(task)
session.commit()
```

`task` будет удалена.

---

## 34.5 `ondelete="CASCADE"`

```python
ForeignKey("users.id", ondelete="CASCADE")
```

Это каскад на уровне базы данных.

Если удалить пользователя SQL-запросом напрямую:

```sql
DELETE FROM users WHERE id = 1;
```

PostgreSQL сам удалит связанные задачи.

---

# 35. Миграция для Task

Создаём миграцию:

```bash
alembic revision --autogenerate -m "create tasks table"
```

Проверяем файл миграции.

Применяем:

```bash
alembic upgrade head
```

---

# 36. Создание пользователя с задачами

```python
def create_user_with_tasks():
    with SessionLocal() as session:
        user = User(
            email="alice@example.com",
            username="alice",
            hashed_password="hash",
        )

        task1 = Task(title="Learn SQLAlchemy")
        task2 = Task(title="Learn Alembic")

        user.tasks.append(task1)
        user.tasks.append(task2)

        session.add(user)
        session.commit()

        session.refresh(user)

        return user
```

SQLAlchemy сам:

1. Создаст пользователя
2. Получит его `id`
3. Создаст задачи с `user_id`

---

# 37. Создание задачи для существующего пользователя

```python
def create_task_for_user(user_id: int, title: str) -> Task | None:
    with SessionLocal() as session:
        user = session.get(User, user_id)

        if user is None:
            return None

        task = Task(title=title, user=user)

        session.add(task)
        session.commit()
        session.refresh(task)

        return task
```

Можно и так:

```python
task = Task(title=title, user_id=user_id)
```

Но если пользователя с таким id нет — база выдаст ошибку внешнего ключа.

---

# 38. JOIN-запросы

## 38.1 Получить задачи вместе с пользователями

```python
from sqlalchemy import select

stmt = (
    select(Task)
    .join(Task.user)
    .where(User.email == "alice@example.com")
)

tasks = session.scalars(stmt).all()
```

SQL примерно:

```sql
SELECT tasks.*
FROM tasks
JOIN users ON users.id = tasks.user_id
WHERE users.email = 'alice@example.com';
```

---

## 38.2 Выбрать конкретные колонки

```python
stmt = (
    select(User.email, Task.title)
    .join(Task, Task.user_id == User.id)
)

rows = session.execute(stmt).all()

for email, title in rows:
    print(email, title)
```

Здесь мы используем:

```python
session.execute(stmt)
```

потому что возвращаются не ORM-объекты `User`, а строки с колонками.

---

# 39. Lazy loading и проблема N+1

Допустим:

```python
users = session.scalars(select(User)).all()

for user in users:
    print(user.tasks)
```

Что произойдёт?

1 запрос:

```sql
SELECT * FROM users;
```

Потом для каждого пользователя отдельный запрос:

```sql
SELECT * FROM tasks WHERE user_id = 1;
SELECT * FROM tasks WHERE user_id = 2;
SELECT * FROM tasks WHERE user_id = 3;
...
```

Это называется проблема **N+1**.

---

# 40. `selectinload`

Чтобы загрузить связи эффективно:

```python
from sqlalchemy.orm import selectinload

stmt = select(User).options(selectinload(User.tasks))

users = session.scalars(stmt).all()

for user in users:
    print(user.tasks)
```

SQLAlchemy сделает примерно:

```sql
SELECT * FROM users;

SELECT * FROM tasks WHERE tasks.user_id IN (1, 2, 3, ...);
```

Это обычно лучший вариант для коллекций.

---

# 41. `joinedload`

```python
from sqlalchemy.orm import joinedload

stmt = select(Task).options(joinedload(Task.user))

tasks = session.scalars(stmt).all()
```

`joinedload` делает `JOIN`.

Хорошо подходит для связи многие-к-одному:

```text
Task -> User
```

Для коллекций `joinedload(User.tasks)` может размножать строки, поэтому часто нужен:

```python
users = session.scalars(stmt).unique().all()
```

---

# 42. Полная CRUD-логика для User и Task

Создадим `app/crud.py`.

```python
from sqlalchemy import select
from sqlalchemy.orm import selectinload

from app.database import SessionLocal
from app.models import User, Task


def create_user(
    email: str,
    username: str,
    hashed_password: str,
) -> User:
    with SessionLocal() as session:
        user = User(
            email=email,
            username=username,
            hashed_password=hashed_password,
        )

        session.add(user)
        session.commit()
        session.refresh(user)

        return user


def get_user(user_id: int) -> User | None:
    with SessionLocal() as session:
        return session.get(User, user_id)


def get_user_with_tasks(user_id: int) -> User | None:
    with SessionLocal() as session:
        stmt = (
            select(User)
            .options(selectinload(User.tasks))
            .where(User.id == user_id)
        )

        return session.scalar(stmt)


def list_users(limit: int = 100, offset: int = 0) -> list[User]:
    with SessionLocal() as session:
        stmt = (
            select(User)
            .order_by(User.id)
            .limit(limit)
            .offset(offset)
        )

        return list(session.scalars(stmt).all())


def update_user_email(user_id: int, email: str) -> User | None:
    with SessionLocal() as session:
        user = session.get(User, user_id)

        if user is None:
            return None

        user.email = email

        session.commit()
        session.refresh(user)

        return user


def delete_user(user_id: int) -> bool:
    with SessionLocal() as session:
        user = session.get(User, user_id)

        if user is None:
            return False

        session.delete(user)
        session.commit()

        return True


def create_task(user_id: int, title: str, description: str | None = None) -> Task:
    with SessionLocal() as session:
        task = Task(
            user_id=user_id,
            title=title,
            description=description,
        )

        session.add(task)
        session.commit()
        session.refresh(task)

        return task


def list_user_tasks(user_id: int) -> list[Task]:
    with SessionLocal() as session:
        stmt = (
            select(Task)
            .where(Task.user_id == user_id)
            .order_by(Task.created_at.desc())
        )

        return list(session.scalars(stmt).all())


def mark_task_done(task_id: int) -> Task | None:
    with SessionLocal() as session:
        task = session.get(Task, task_id)

        if task is None:
            return None

        task.is_done = True

        session.commit()
        session.refresh(task)

        return task
```

---

# 43. Индексы

Индекс ускоряет поиск.

Простой индекс:

```python
email: Mapped[str] = mapped_column(index=True)
```

Но для более сложных случаев лучше использовать `Index`.

Пример:

```python
from sqlalchemy import Index


class Task(Base):
    __tablename__ = "tasks"

    id: Mapped[int] = mapped_column(primary_key=True)
    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
    is_done: Mapped[bool] = mapped_column(default=False, nullable=False)

    __table_args__ = (
        Index("ix_tasks_user_id_is_done", "user_id", "is_done"),
    )
```

Это составной индекс:

```sql
CREATE INDEX ix_tasks_user_id_is_done
ON tasks (user_id, is_done);
```

Полезно для запросов:

```python
select(Task).where(
    Task.user_id == user_id,
    Task.is_done.is_(False),
)
```

---

# 44. Constraints

Constraints — ограничения базы данных.

Например:

- `UNIQUE`
- `CHECK`
- `FOREIGN KEY`
- `PRIMARY KEY`

Пример `CheckConstraint`:

```python
from sqlalchemy import CheckConstraint


class Product(Base):
    __tablename__ = "products"

    id: Mapped[int] = mapped_column(primary_key=True)

    price: Mapped[int] = mapped_column(nullable=False)

    __table_args__ = (
        CheckConstraint("price >= 0", name="ck_products_price_positive"),
    )
```

Теперь база не позволит сохранить отрицательную цену.

---

# 45. Naming convention

Очень хорошая практика — задать имена constraints автоматически.

Обновим `app/database.py`.

```python
from sqlalchemy import MetaData, create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker

from app.config import DATABASE_URL


convention = {
    "ix": "ix_%(column_0_label)s",
    "uq": "uq_%(table_name)s_%(column_0_name)s",
    "ck": "ck_%(table_name)s_%(constraint_name)s",
    "fk": "fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s",
    "pk": "pk_%(table_name)s",
}


class Base(DeclarativeBase):
    metadata = MetaData(naming_convention=convention)


engine = create_engine(
    DATABASE_URL,
    echo=True,
)

SessionLocal = sessionmaker(
    bind=engine,
    autoflush=False,
    expire_on_commit=False,
)
```

Зачем это нужно?

Alembic лучше работает с миграциями, когда constraints имеют предсказуемые имена.

Например:

```text
pk_users
uq_users_email
fk_tasks_user_id_users
```

---

# 46. Важное предупреждение про naming convention

Если ты уже создал миграции без naming convention, а потом добавил convention, Alembic может увидеть изменения имён constraints.

В учебном проекте можно сбросить базу и миграции.

В реальном проекте нужно аккуратно мигрировать.

---

# 47. UUID вместо integer id

В PostgreSQL часто используют UUID.

```python
import uuid
from sqlalchemy.dialects.postgresql import UUID


class User(Base):
    __tablename__ = "users"

    id: Mapped[uuid.UUID] = mapped_column(
        UUID(as_uuid=True),
        primary_key=True,
        default=uuid.uuid4,
    )
```

Тогда `id` будет UUID:

```text
550e8400-e29b-41d4-a716-446655440000
```

Важно:

```python
default=uuid.uuid4
```

Без скобок.

Правильно:

```python
default=uuid.uuid4
```

Неправильно:

```python
default=uuid.uuid4()
```

Почему?

- `uuid.uuid4` — функция, SQLAlchemy вызовет её при создании строки
- `uuid.uuid4()` — вызовется один раз при запуске приложения

---

# 48. PostgreSQL JSONB

PostgreSQL умеет хранить JSON.

```python
from sqlalchemy.dialects.postgresql import JSONB


class Event(Base):
    __tablename__ = "events"

    id: Mapped[int] = mapped_column(primary_key=True)

    payload: Mapped[dict] = mapped_column(
        JSONB,
        nullable=False,
    )
```

Создание:

```python
event = Event(
    payload={
        "type": "user_registered",
        "user_id": 1,
    }
)
```

Фильтрация:

```python
stmt = select(Event).where(
    Event.payload["type"].astext == "user_registered"
)
```

---

# 49. Enum

Можно использовать Python Enum.

```python
import enum
from sqlalchemy import Enum


class TaskStatus(enum.Enum):
    TODO = "todo"
    IN_PROGRESS = "in_progress"
    DONE = "done"


class Task(Base):
    __tablename__ = "tasks"

    id: Mapped[int] = mapped_column(primary_key=True)

    status: Mapped[TaskStatus] = mapped_column(
        Enum(TaskStatus, name="task_status"),
        default=TaskStatus.TODO,
        nullable=False,
    )
```

Alembic создаст PostgreSQL enum type.

С enum нужно быть осторожным: изменение enum в PostgreSQL требует специальных миграций.

---

# 50. Миграции Alembic подробно

## 50.1 Создать пустую миграцию

```bash
alembic revision -m "manual migration"
```

Она создаст файл без автогенерации.

---

## 50.2 Создать миграцию автоматически

```bash
alembic revision --autogenerate -m "add something"
```

Alembic сравнивает модели и базу.

---

## 50.3 Применить все миграции

```bash
alembic upgrade head
```

---

## 50.4 Откатить последнюю миграцию

```bash
alembic downgrade -1
```

---

## 50.5 Откатиться в самое начало

```bash
alembic downgrade base
```

---

## 50.6 Посмотреть историю

```bash
alembic history
```

---

## 50.7 Посмотреть текущую версию базы

```bash
alembic current
```

---

## 50.8 Посмотреть последнюю миграцию

```bash
alembic heads
```

---

# 51. Пример изменения модели

Допустим, хотим добавить пользователю `full_name`.

В модель `User` добавляем:

```python
full_name: Mapped[str | None] = mapped_column(
    String(200),
    nullable=True,
)
```

Создаём миграцию:

```bash
alembic revision --autogenerate -m "add full_name to users"
```

Alembic создаст:

```python
def upgrade() -> None:
    op.add_column(
        "users",
        sa.Column("full_name", sa.String(length=200), nullable=True),
    )


def downgrade() -> None:
    op.drop_column("users", "full_name")
```

Применяем:

```bash
alembic upgrade head
```

---

# 52. Добавление NOT NULL колонки в существующую таблицу

Это частая ошибка новичков.

Допустим, в таблице `users` уже есть данные.

Ты добавляешь:

```python
age: Mapped[int] = mapped_column(nullable=False)
```

Alembic сгенерирует:

```python
op.add_column("users", sa.Column("age", sa.Integer(), nullable=False))
```

PostgreSQL скажет:

```text
column "age" contains null values
```

Почему?

Потому что старые строки не имеют значения `age`.

Правильный вариант:

## Шаг 1. Добавить nullable колонку

```python
op.add_column("users", sa.Column("age", sa.Integer(), nullable=True))
```

## Шаг 2. Заполнить данные

```python
op.execute("UPDATE users SET age = 0 WHERE age IS NULL")
```

## Шаг 3. Сделать NOT NULL

```python
op.alter_column("users", "age", nullable=False)
```

Полная миграция:

```python
def upgrade() -> None:
    op.add_column(
        "users",
        sa.Column("age", sa.Integer(), nullable=True),
    )

    op.execute("UPDATE users SET age = 0 WHERE age IS NULL")

    op.alter_column(
        "users",
        "age",
        nullable=False,
    )


def downgrade() -> None:
    op.drop_column("users", "age")
```

---

# 53. Изменение типа колонки

Например, `username` был `String(50)`, стал `String(100)`.

Миграция:

```python
def upgrade() -> None:
    op.alter_column(
        "users",
        "username",
        existing_type=sa.String(length=50),
        type_=sa.String(length=100),
        existing_nullable=False,
    )


def downgrade() -> None:
    op.alter_column(
        "users",
        "username",
        existing_type=sa.String(length=100),
        type_=sa.String(length=50),
        existing_nullable=False,
    )
```

---

# 54. Переименование колонки

Alembic autogenerate часто не понимает переименование.

Если было:

```python
username
```

стало:

```python
login
```

Alembic может сгенерировать:

```python
drop_column("username")
add_column("login")
```

Это опасно: данные потеряются.

Правильно вручную:

```python
def upgrade() -> None:
    op.alter_column(
        "users",
        "username",
        new_column_name="login",
    )


def downgrade() -> None:
    op.alter_column(
        "users",
        "login",
        new_column_name="username",
    )
```

---

# 55. Переименование таблицы

```python
def upgrade() -> None:
    op.rename_table("users", "accounts")


def downgrade() -> None:
    op.rename_table("accounts", "users")
```

---

# 56. Индексы в миграциях

Создать индекс:

```python
def upgrade() -> None:
    op.create_index(
        "ix_users_email",
        "users",
        ["email"],
        unique=False,
    )


def downgrade() -> None:
    op.drop_index(
        "ix_users_email",
        table_name="users",
    )
```

Уникальный индекс:

```python
op.create_index(
    "uq_users_email_lower",
    "users",
    [sa.text("lower(email)")],
    unique=True,
)
```

---

# 57. Миграции данных

Иногда миграция меняет не только структуру, но и данные.

Например:

```python
def upgrade() -> None:
    op.execute("""
        UPDATE users
        SET is_active = true
        WHERE is_active IS NULL
    """)
```

Можно использовать SQLAlchemy table:

```python
users = sa.table(
    "users",
    sa.column("id", sa.Integer),
    sa.column("email", sa.String),
)

def upgrade() -> None:
    connection = op.get_bind()

    connection.execute(
        users.update()
        .where(users.c.email == "admin@example.com")
        .values(email="root@example.com")
    )
```

---

# 58. Alembic autogenerate не всесилен

Alembic хорошо видит:

- новые таблицы
- удалённые таблицы
- новые колонки
- удалённые колонки
- индексы
- foreign keys
- изменение nullable
- изменение типа, если `compare_type=True`

Alembic плохо или не всегда видит:

- переименование колонок
- переименование таблиц
- некоторые изменения constraints
- сложные PostgreSQL enum изменения
- сложные server defaults

Всегда проверяй миграцию перед применением.

---

# 59. Хорошая модельная база

Можно разделить модели по файлам.

```text
app/
├── models/
│   ├── __init__.py
│   ├── user.py
│   └── task.py
```

`app/models/user.py`:

```python
from datetime import datetime

from sqlalchemy import String, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.database import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    email: Mapped[str] = mapped_column(
        String(255),
        unique=True,
        nullable=False,
        index=True,
    )

    username: Mapped[str] = mapped_column(
        String(100),
        unique=True,
        nullable=False,
    )

    tasks: Mapped[list["Task"]] = relationship(
        back_populates="user",
        cascade="all, delete-orphan",
    )
```

`app/models/task.py`:

```python
from datetime import datetime

from sqlalchemy import String, Text, ForeignKey, DateTime, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.database import Base


class Task(Base):
    __tablename__ = "tasks"

    id: Mapped[int] = mapped_column(primary_key=True)

    title: Mapped[str] = mapped_column(String(200), nullable=False)

    description: Mapped[str | None] = mapped_column(Text)

    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
    )

    user: Mapped["User"] = relationship(back_populates="tasks")
```

Но нужно учитывать циклические импорты.

`app/models/__init__.py`:

```python
from app.models.user import User
from app.models.task import Task

__all__ = [
    "User",
    "Task",
]
```

В `alembic/env.py`:

```python
from app import models
```

или:

```python
from app.models import User, Task
```

Главное — импортировать все модели.

---

# 60. Типичные ошибки новичков

## 60.1 Не импортировали модели в Alembic

Симптом:

```bash
alembic revision --autogenerate
```

создаёт пустую миграцию.

Причина:

```python
target_metadata = Base.metadata
```

есть, но модели не импортированы.

Решение:

```python
from app import models
```

---

## 60.2 Используешь старый синтаксис

Плохо:

```python
session.query(User).filter(User.email == email).first()
```

Хорошо:

```python
stmt = select(User).where(User.email == email)
user = session.scalar(stmt)
```

---

## 60.3 Забыл `commit`

```python
session.add(user)
```

Этого недостаточно.

Нужно:

```python
session.commit()
```

---

## 60.4 После ошибки не сделал rollback

Если commit упал:

```python
try:
    session.commit()
except IntegrityError:
    session.rollback()
```

Без rollback сессия останется в ошибочном состоянии.

---

## 60.5 Вернул ORM-объект после закрытия session и полез в lazy relationship

```python
user = get_user()
print(user.tasks)
```

Если `tasks` не были загружены заранее, а session уже закрыта, будет ошибка.

Решение:

```python
select(User).options(selectinload(User.tasks))
```

---

# 61. Async SQLAlchemy 2

SQLAlchemy 2 поддерживает async.

Для PostgreSQL установим:

```bash
pip install asyncpg
```

DATABASE_URL:

```env
DATABASE_URL_ASYNC=postgresql+asyncpg://app_user:app_password@localhost:5432/app_db
```

`app/async_database.py`:

```python
import os
from dotenv import load_dotenv

from sqlalchemy.ext.asyncio import (
    create_async_engine,
    async_sessionmaker,
    AsyncSession,
)

load_dotenv()

DATABASE_URL_ASYNC = os.getenv("DATABASE_URL_ASYNC")

if DATABASE_URL_ASYNC is None:
    raise RuntimeError("DATABASE_URL_ASYNC is not set")


async_engine = create_async_engine(
    DATABASE_URL_ASYNC,
    echo=True,
)

AsyncSessionLocal = async_sessionmaker(
    bind=async_engine,
    expire_on_commit=False,
    class_=AsyncSession,
)
```

Пример запроса:

```python
from sqlalchemy import select

from app.async_database import AsyncSessionLocal
from app.models import User


async def get_user_by_email(email: str) -> User | None:
    async with AsyncSessionLocal() as session:
        stmt = select(User).where(User.email == email)

        result = await session.scalar(stmt)

        return result
```

Создание:

```python
async def create_user(email: str, username: str, hashed_password: str) -> User:
    async with AsyncSessionLocal() as session:
        user = User(
            email=email,
            username=username,
            hashed_password=hashed_password,
        )

        session.add(user)

        await session.commit()
        await session.refresh(user)

        return user
```

Транзакция:

```python
async def create_user_transaction():
    async with AsyncSessionLocal() as session:
        async with session.begin():
            user = User(
                email="async@example.com",
                username="async_user",
                hashed_password="hash",
            )
            session.add(user)
```

Важно:

> Alembic чаще настраивают через sync engine даже в async-проектах, либо используют async env.py. Для junior-уровня проще оставить Alembic sync.

---

# 62. Мини-шпаргалка SQLAlchemy 2

## Создать объект

```python
obj = User(email="a@example.com", username="a", hashed_password="hash")
session.add(obj)
session.commit()
session.refresh(obj)
```

## Получить по id

```python
user = session.get(User, user_id)
```

## Получить один по условию

```python
stmt = select(User).where(User.email == email)
user = session.scalar(stmt)
```

## Получить список

```python
stmt = select(User).order_by(User.id)
users = session.scalars(stmt).all()
```

## Обновить

```python
user = session.get(User, user_id)
user.email = "new@example.com"
session.commit()
```

## Удалить

```python
user = session.get(User, user_id)
session.delete(user)
session.commit()
```

## JOIN

```python
stmt = select(Task).join(Task.user).where(User.email == email)
tasks = session.scalars(stmt).all()
```

## Загрузить связи

```python
stmt = select(User).options(selectinload(User.tasks))
users = session.scalars(stmt).all()
```

---

# 63. Мини-шпаргалка Alembic

## Инициализация

```bash
alembic init alembic
```

## Новая автогенерируемая миграция

```bash
alembic revision --autogenerate -m "message"
```

## Применить миграции

```bash
alembic upgrade head
```

## Откатить одну

```bash
alembic downgrade -1
```

## Откатить всё

```bash
alembic downgrade base
```

## История

```bash
alembic history
```

## Текущая версия

```bash
alembic current
```

---

# 64. Что ты должен понимать на уровне junior+

После изучения этого материала ты должен уверенно понимать:

- что такое SQLAlchemy ORM
- что такое SQLAlchemy Core
- что такое Alembic
- зачем нужны миграции
- как подключиться к PostgreSQL
- как описывать модели в стиле SQLAlchemy 2
- как использовать `Mapped` и `mapped_column`
- как создавать таблицы через Alembic
- как создавать, читать, обновлять и удалять данные
- как работают `Session`, `commit`, `rollback`, `flush`
- как делать связи `one-to-many`
- как работают `ForeignKey` и `relationship`
- что такое lazy loading
- что такое проблема N+1
- когда использовать `selectinload`
- как создавать индексы
- как писать constraints
- как добавлять новые поля через миграции
- как не потерять данные при переименовании колонок
- почему нужно проверять autogenerated migrations
- чем sync SQLAlchemy отличается от async SQLAlchemy

---

# 65. Рекомендуемый финальный вариант базовых файлов

## `.env`

```env
DATABASE_URL=postgresql+psycopg://app_user:app_password@localhost:5432/app_db
```

## `app/config.py`

```python
import os
from dotenv import load_dotenv

load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL")

if DATABASE_URL is None:
    raise RuntimeError("DATABASE_URL is not set")
```

## `app/database.py`

```python
from sqlalchemy import MetaData, create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker

from app.config import DATABASE_URL


convention = {
    "ix": "ix_%(column_0_label)s",
    "uq": "uq_%(table_name)s_%(column_0_name)s",
    "ck": "ck_%(table_name)s_%(constraint_name)s",
    "fk": "fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s",
    "pk": "pk_%(table_name)s",
}


class Base(DeclarativeBase):
    metadata = MetaData(naming_convention=convention)


engine = create_engine(
    DATABASE_URL,
    echo=True,
)

SessionLocal = sessionmaker(
    bind=engine,
    autoflush=False,
    expire_on_commit=False,
)
```

## `app/models.py`

```python
from datetime import datetime

from sqlalchemy import (
    String,
    Text,
    DateTime,
    ForeignKey,
    func,
)
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.database import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    email: Mapped[str] = mapped_column(
        String(255),
        unique=True,
        nullable=False,
        index=True,
    )

    username: Mapped[str] = mapped_column(
        String(100),
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

    updated_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        onupdate=func.now(),
        nullable=False,
    )

    tasks: Mapped[list["Task"]] = relationship(
        back_populates="user",
        cascade="all, delete-orphan",
    )


class Task(Base):
    __tablename__ = "tasks"

    id: Mapped[int] = mapped_column(primary_key=True)

    title: Mapped[str] = mapped_column(
        String(200),
        nullable=False,
    )

    description: Mapped[str | None] = mapped_column(
        Text,
        nullable=True,
    )

    is_done: Mapped[bool] = mapped_column(
        default=False,
        nullable=False,
    )

    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
        index=True,
    )

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False,
    )

    user: Mapped["User"] = relationship(
        back_populates="tasks",
    )
```

## `alembic/env.py`

```python
from logging.config import fileConfig

from sqlalchemy import engine_from_config
from sqlalchemy import pool

from alembic import context

from dotenv import load_dotenv
import os

from app.database import Base
from app import models


load_dotenv()

config = context.config

if config.config_file_name is not None:
    fileConfig(config.config_file_name)


DATABASE_URL = os.getenv("DATABASE_URL")

if DATABASE_URL is None:
    raise RuntimeError("DATABASE_URL is not set")


config.set_main_option("sqlalchemy.url", DATABASE_URL)

target_metadata = Base.metadata


def run_migrations_offline() -> None:
    url = config.get_main_option("sqlalchemy.url")

    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
        compare_type=True,
    )

    with context.begin_transaction():
        context.run_migrations()


def run_migrations_online() -> None:
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
            compare_type=True,
        )

        with context.begin_transaction():
            context.run_migrations()


if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

---

# 66. Практическое задание для закрепления

Сделай сам:

1. Подними PostgreSQL через Docker.
2. Создай проект.
3. Настрой SQLAlchemy.
4. Опиши модели:
   - `User`
   - `Task`
5. Настрой Alembic.
6. Создай первую миграцию.
7. Примени миграцию.
8. Напиши функции:
   - создать пользователя
   - получить пользователя по email
   - создать задачу пользователю
   - получить пользователя со всеми задачами
   - отметить задачу выполненной
   - удалить пользователя
9. Добавь поле `full_name`.
10. Сделай новую миграцию.
11. Примени её.
12. Добавь индекс на `(user_id, is_done)`.
13. Сделай миграцию.
14. Проверь SQL-запросы через `echo=True`.

Если ты это сделаешь руками и поймёшь каждый шаг — у тебя уже будет крепкая база SQLAlchemy 2 + Alembic на уровне junior+.
