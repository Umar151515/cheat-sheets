# SQLAlchemy 2 + Alembic + PostgreSQL с нуля до уровня junior+

Ниже — большая практическая документация в формате мини-курса. Мы будем изучать SQLAlchemy 2 и Alembic на реальном примере: небольшой блог с пользователями, постами, комментариями и тегами.

---

# 0. Что ты изучишь

После этого материала ты должен уверенно понимать:

1. Что такое SQLAlchemy.
2. Что такое ORM.
3. Что такое Alembic.
4. Как подключиться к PostgreSQL.
5. Как описывать таблицы Python-классами.
6. Как создавать связи:
   - one-to-many,
   - many-to-one,
   - many-to-many.
7. Как делать CRUD:
   - create,
   - read,
   - update,
   - delete.
8. Как писать запросы через SQLAlchemy 2.
9. Как работать с транзакциями.
10. Как использовать миграции Alembic.
11. Как обновлять структуру базы данных.
12. Как понимать ошибки.
13. Как писать код на уровне уверенного junior-разработчика.

---

# 1. Что такое SQLAlchemy

SQLAlchemy — это библиотека для работы с базами данных в Python.

Она состоит из двух больших частей:

## 1.1. SQLAlchemy Core

Это низкоуровневый способ писать SQL через Python-объекты.

Пример:

```python
from sqlalchemy import select

stmt = select(users_table).where(users_table.c.email == "test@example.com")
```

Core ближе к чистому SQL.

---

## 1.2. SQLAlchemy ORM

ORM означает Object Relational Mapper.

Идея простая:

В базе данных есть таблица:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255),
    username VARCHAR(100)
);
```

А в Python мы описываем её классом:

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str]
    username: Mapped[str]
```

После этого строка из таблицы `users` становится Python-объектом:

```python
user = User(email="test@example.com", username="john")
```

То есть ORM позволяет работать с базой почти как с обычными Python-объектами.

---

# 2. Что такое Alembic

Alembic — это инструмент миграций для SQLAlchemy.

Миграция — это изменение структуры базы данных.

Например:

- создать таблицу `users`;
- добавить колонку `email`;
- удалить колонку;
- добавить индекс;
- изменить тип данных;
- создать связь между таблицами.

Без Alembic тебе пришлось бы вручную писать SQL:

```sql
ALTER TABLE users ADD COLUMN age INTEGER;
```

С Alembic ты делаешь миграцию:

```bash
alembic revision --autogenerate -m "add age to users"
alembic upgrade head
```

И Alembic сам применяет изменения к базе.

---

# 3. Что такое PostgreSQL

PostgreSQL — это мощная реляционная база данных.

Реляционная база данных хранит данные в таблицах.

Пример таблицы `users`:

| id | email            | username |
|----|------------------|----------|
| 1  | john@example.com | john     |
| 2  | ann@example.com  | ann      |

У каждой таблицы есть:

- колонки;
- строки;
- типы данных;
- индексы;
- ограничения;
- связи с другими таблицами.

---

# 4. Подготовка проекта

Создадим проект:

```bash
mkdir sqlalchemy_alembic_project
cd sqlalchemy_alembic_project
```

Создадим виртуальное окружение:

```bash
python -m venv .venv
```

Активируем его.

Linux/macOS:

```bash
source .venv/bin/activate
```

Windows PowerShell:

```bash
.venv\Scripts\Activate.ps1
```

---

# 5. Установка зависимостей

Установим SQLAlchemy, Alembic и драйвер PostgreSQL:

```bash
pip install sqlalchemy alembic "psycopg[binary]" python-dotenv
```

Что это такое:

```text
sqlalchemy      — ORM и работа с базой
alembic         — миграции
psycopg         — драйвер для PostgreSQL
python-dotenv   — чтение .env-файла
```

Проверим:

```bash
pip freeze
```

---

# 6. Запуск PostgreSQL через Docker

Если PostgreSQL не установлен, удобно запустить через Docker.

Создай файл `docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:16
    container_name: sqlalchemy_postgres
    restart: always
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: app
      POSTGRES_DB: app
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

Запусти:

```bash
docker compose up -d
```

Проверить контейнеры:

```bash
docker ps
```

Теперь у нас есть база:

```text
host: localhost
port: 5432
database: app
user: app
password: app
```

---

# 7. Структура проекта

Сделаем нормальную структуру:

```text
sqlalchemy_alembic_project/
│
├── app/
│   ├── __init__.py
│   │
│   ├── db/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   └── session.py
│   │
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── post.py
│   │   ├── comment.py
│   │   └── tag.py
│   │
│   └── main.py
│
├── docker-compose.yml
├── .env
└── alembic.ini
```

Папки можно создать так:

```bash
mkdir -p app/db app/models
touch app/__init__.py app/db/__init__.py app/models/__init__.py
```

---

# 8. Файл `.env`

Создай `.env`:

```env
DATABASE_URL=postgresql+psycopg://app:app@localhost:5432/app
```

Разберём строку подключения:

```text
postgresql+psycopg://app:app@localhost:5432/app
```

Это значит:

```text
postgresql      — тип базы данных
psycopg         — драйвер
app             — пользователь
app             — пароль
localhost       — хост
5432            — порт
app             — имя базы данных
```

---

# 9. Базовый класс SQLAlchemy

Создай файл `app/db/base.py`:

```python
from sqlalchemy import MetaData
from sqlalchemy.orm import DeclarativeBase


convention = {
    "ix": "ix_%(column_0_label)s",
    "uq": "uq_%(table_name)s_%(column_0_name)s",
    "ck": "ck_%(table_name)s_%(constraint_name)s",
    "fk": "fk_%(table_name)s_%(column_0_name)s_%(referred_table_name)s",
    "pk": "pk_%(table_name)s",
}


class Base(DeclarativeBase):
    metadata = MetaData(naming_convention=convention)
```

Разберём.

## 9.1. Что такое `Base`

`Base` — это базовый класс для всех моделей.

Каждая модель будет наследоваться от него:

```python
class User(Base):
    ...
```

SQLAlchemy через `Base` понимает:

- какие есть таблицы;
- какие у них колонки;
- какие связи;
- какие индексы;
- какие ограничения.

---

## 9.2. Что такое `metadata`

`metadata` — это объект, который хранит описание всех таблиц.

Можно сказать:

```text
Base.metadata = карта всей структуры базы данных
```

Alembic будет смотреть на `Base.metadata` и понимать, какие таблицы нужно создать или изменить.

---

## 9.3. Зачем `naming_convention`

PostgreSQL создаёт имена ограничений автоматически, но эти имена могут быть неудобными.

Например:

```text
users_email_key
```

Или:

```text
fk_123abc_random
```

С `naming_convention` имена становятся предсказуемыми.

Например:

```text
pk_users
uq_users_email
fk_posts_author_id_users
```

Это очень полезно для миграций.

---

# 10. Подключение к базе данных

Создай файл `app/db/session.py`:

```python
import os

from dotenv import load_dotenv
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL")

if DATABASE_URL is None:
    raise RuntimeError("DATABASE_URL is not set")


engine = create_engine(
    DATABASE_URL,
    echo=True,
    pool_pre_ping=True,
)

SessionLocal = sessionmaker(
    bind=engine,
    autoflush=False,
    expire_on_commit=False,
)
```

Разберём подробно.

---

## 10.1. `create_engine`

```python
engine = create_engine(DATABASE_URL)
```

`engine` — это объект, который умеет подключаться к базе данных.

Важно:

`engine` — это не одно подключение. Это менеджер подключений.

Он управляет пулом соединений.

---

## 10.2. `echo=True`

```python
echo=True
```

SQLAlchemy будет выводить SQL-запросы в консоль.

Это очень полезно для обучения.

Ты увидишь примерно такое:

```sql
SELECT users.id, users.email FROM users WHERE users.id = %(id_1)s
```

На продакшене обычно ставят:

```python
echo=False
```

---

## 10.3. `pool_pre_ping=True`

```python
pool_pre_ping=True
```

SQLAlchemy проверяет, живо ли соединение с базой, прежде чем использовать его.

Полезно, если база перезапускалась или соединение протухло.

---

## 10.4. `SessionLocal`

```python
SessionLocal = sessionmaker(...)
```

`SessionLocal` — это фабрика сессий.

То есть через неё мы создаём сессию:

```python
session = SessionLocal()
```

---

# 11. Что такое Session

`Session` — один из самых важных объектов SQLAlchemy ORM.

Простыми словами:

```text
Session — это рабочая область для общения с базой данных.
```

Через неё мы:

- добавляем объекты;
- удаляем объекты;
- ищем объекты;
- изменяем объекты;
- коммитим транзакции;
- откатываем изменения.

Пример:

```python
with SessionLocal() as session:
    user = User(email="john@example.com", username="john")
    session.add(user)
    session.commit()
```

---

## 11.1. Session — это не просто соединение

Session:

- хранит объекты в памяти;
- отслеживает изменения;
- управляет транзакцией;
- выполняет SQL-запросы;
- кеширует объекты в рамках одной сессии.

---

## 11.2. Важные методы Session

```python
session.add(obj)
```

Добавить объект.

```python
session.delete(obj)
```

Удалить объект.

```python
session.commit()
```

Сохранить изменения в базе.

```python
session.rollback()
```

Откатить изменения.

```python
session.flush()
```

Отправить SQL в базу, но не завершать транзакцию.

```python
session.refresh(obj)
```

Обновить объект данными из базы.

```python
session.get(Model, id)
```

Получить объект по primary key.

```python
session.execute(stmt)
```

Выполнить SQLAlchemy-запрос.

---

# 12. Модели

Мы сделаем мини-блог.

У нас будут таблицы:

```text
users       — пользователи
posts       — посты
comments    — комментарии
tags        — теги
post_tags   — связь постов и тегов
```

Связи:

```text
User 1 -> N Post
User 1 -> N Comment
Post 1 -> N Comment
Post N -> N Tag
```

То есть:

- один пользователь может написать много постов;
- один пользователь может написать много комментариев;
- один пост может иметь много комментариев;
- один пост может иметь много тегов;
- один тег может быть у многих постов.

---

# 13. Модель User

Создай файл `app/models/user.py`:

```python
from datetime import datetime
from typing import TYPE_CHECKING

from sqlalchemy import DateTime, String, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db.base import Base

if TYPE_CHECKING:
    from app.models.post import Post
    from app.models.comment import Comment


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
        String(100),
        unique=True,
        index=True,
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

    comments: Mapped[list["Comment"]] = relationship(
        back_populates="author",
        cascade="all, delete-orphan",
    )

    def __repr__(self) -> str:
        return f"User(id={self.id!r}, email={self.email!r}, username={self.username!r})"
```

---

## 13.1. `__tablename__`

```python
__tablename__ = "users"
```

Это имя таблицы в базе данных.

Класс называется `User`, а таблица будет называться `users`.

---

## 13.2. `Mapped`

```python
id: Mapped[int]
```

`Mapped` — это типизация SQLAlchemy 2.

Она говорит:

```text
это поле является ORM-колонкой или ORM-связью
```

---

## 13.3. `mapped_column`

```python
id: Mapped[int] = mapped_column(primary_key=True)
```

`mapped_column` описывает колонку таблицы.

---

## 13.4. Primary Key

```python
primary_key=True
```

Primary key — это уникальный идентификатор строки.

Например:

```text
users.id
```

Каждый пользователь имеет уникальный `id`.

---

## 13.5. `String(255)`

```python
email: Mapped[str] = mapped_column(String(255))
```

В PostgreSQL это примерно:

```sql
VARCHAR(255)
```

---

## 13.6. `unique=True`

```python
unique=True
```

Значит, в таблице нельзя иметь два одинаковых значения.

Например, нельзя создать двух пользователей с одинаковым email.

---

## 13.7. `index=True`

```python
index=True
```

Создаёт индекс.

Индекс ускоряет поиск:

```python
where(User.email == "john@example.com")
```

Но индексы занимают место и замедляют запись, поэтому не нужно индексировать всё подряд.

---

## 13.8. `nullable=False`

```python
nullable=False
```

Колонка не может быть `NULL`.

То есть значение обязательно.

---

## 13.9. `default=True`

```python
is_active: Mapped[bool] = mapped_column(default=True)
```

Python-дефолт.

Если создать объект:

```python
user = User(...)
```

и не указать `is_active`, SQLAlchemy поставит `True`.

---

## 13.10. `server_default=func.now()`

```python
created_at: Mapped[datetime] = mapped_column(server_default=func.now())
```

Значение создаётся на стороне базы данных.

PostgreSQL сам поставит текущее время.

---

## 13.11. `relationship`

```python
posts: Mapped[list["Post"]] = relationship(
    back_populates="author",
    cascade="all, delete-orphan",
)
```

Это не колонка.

Это ORM-связь.

Она означает:

```text
у пользователя есть список постов
```

---

## 13.12. `cascade="all, delete-orphan"`

Если удалить пользователя, его посты тоже будут удалены через ORM.

```python
session.delete(user)
session.commit()
```

Удалятся:

- пользователь;
- его посты;
- его комментарии.

Важно: cascade в ORM и `ON DELETE CASCADE` в базе — разные механизмы. Часто используют оба, но нужно понимать разницу.

---

# 14. Модель Post

Создай файл `app/models/post.py`:

```python
from datetime import datetime
from typing import TYPE_CHECKING

from sqlalchemy import DateTime, ForeignKey, String, Text, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db.base import Base

if TYPE_CHECKING:
    from app.models.user import User
    from app.models.comment import Comment
    from app.models.tag import Tag


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)

    title: Mapped[str] = mapped_column(
        String(200),
        nullable=False,
    )

    slug: Mapped[str] = mapped_column(
        String(220),
        unique=True,
        index=True,
        nullable=False,
    )

    content: Mapped[str] = mapped_column(
        Text,
        nullable=False,
    )

    is_published: Mapped[bool] = mapped_column(
        default=False,
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

    updated_at: Mapped[datetime | None] = mapped_column(
        DateTime(timezone=True),
        onupdate=func.now(),
        nullable=True,
    )

    author: Mapped["User"] = relationship(
        back_populates="posts",
    )

    comments: Mapped[list["Comment"]] = relationship(
        back_populates="post",
        cascade="all, delete-orphan",
    )

    tags: Mapped[list["Tag"]] = relationship(
        secondary="post_tags",
        back_populates="posts",
    )

    def __repr__(self) -> str:
        return f"Post(id={self.id!r}, title={self.title!r}, author_id={self.author_id!r})"
```

---

## 14.1. ForeignKey

```python
author_id: Mapped[int] = mapped_column(
    ForeignKey("users.id", ondelete="CASCADE")
)
```

Это внешний ключ.

Он означает:

```text
posts.author_id ссылается на users.id
```

То есть каждый пост принадлежит какому-то пользователю.

---

## 14.2. `ondelete="CASCADE"`

Это поведение на уровне базы.

Если удалить пользователя из таблицы `users`, PostgreSQL может автоматически удалить его посты.

Но чтобы это работало корректно через ORM, часто добавляют ещё `passive_deletes=True` в relationship. В этом учебном примере мы оставим без усложнения.

---

## 14.3. `Text`

```python
content: Mapped[str] = mapped_column(Text)
```

`Text` — для длинного текста.

Например, содержимое статьи.

---

## 14.4. `onupdate=func.now()`

```python
updated_at: Mapped[datetime | None] = mapped_column(
    DateTime(timezone=True),
    onupdate=func.now(),
)
```

При обновлении объекта SQLAlchemy будет обновлять это поле.

---

# 15. Модель Comment

Создай файл `app/models/comment.py`:

```python
from datetime import datetime
from typing import TYPE_CHECKING

from sqlalchemy import DateTime, ForeignKey, Text, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db.base import Base

if TYPE_CHECKING:
    from app.models.user import User
    from app.models.post import Post


class Comment(Base):
    __tablename__ = "comments"

    id: Mapped[int] = mapped_column(primary_key=True)

    text: Mapped[str] = mapped_column(
        Text,
        nullable=False,
    )

    author_id: Mapped[int] = mapped_column(
        ForeignKey("users.id", ondelete="CASCADE"),
        nullable=False,
        index=True,
    )

    post_id: Mapped[int] = mapped_column(
        ForeignKey("posts.id", ondelete="CASCADE"),
        nullable=False,
        index=True,
    )

    created_at: Mapped[datetime] = mapped_column(
        DateTime(timezone=True),
        server_default=func.now(),
        nullable=False,
    )

    author: Mapped["User"] = relationship(
        back_populates="comments",
    )

    post: Mapped["Post"] = relationship(
        back_populates="comments",
    )

    def __repr__(self) -> str:
        return f"Comment(id={self.id!r}, author_id={self.author_id!r}, post_id={self.post_id!r})"
```

---

# 16. Модель Tag и many-to-many

Связь many-to-many:

```text
один пост может иметь много тегов
один тег может быть у многих постов
```

Для этого нужна промежуточная таблица:

```text
post_tags
```

Создай файл `app/models/tag.py`:

```python
from typing import TYPE_CHECKING

from sqlalchemy import ForeignKey, String
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.db.base import Base

if TYPE_CHECKING:
    from app.models.post import Post


class PostTag(Base):
    __tablename__ = "post_tags"

    post_id: Mapped[int] = mapped_column(
        ForeignKey("posts.id", ondelete="CASCADE"),
        primary_key=True,
    )

    tag_id: Mapped[int] = mapped_column(
        ForeignKey("tags.id", ondelete="CASCADE"),
        primary_key=True,
    )


class Tag(Base):
    __tablename__ = "tags"

    id: Mapped[int] = mapped_column(primary_key=True)

    name: Mapped[str] = mapped_column(
        String(50),
        unique=True,
        index=True,
        nullable=False,
    )

    posts: Mapped[list["Post"]] = relationship(
        secondary="post_tags",
        back_populates="tags",
    )

    def __repr__(self) -> str:
        return f"Tag(id={self.id!r}, name={self.name!r})"
```

---

## 16.1. Почему `PostTag` имеет два primary key

```python
post_id: Mapped[int] = mapped_column(..., primary_key=True)
tag_id: Mapped[int] = mapped_column(..., primary_key=True)
```

Вместе они образуют составной primary key.

Это запрещает дубли:

```text
post_id=1, tag_id=2
post_id=1, tag_id=2
```

Такую пару нельзя будет вставить два раза.

---

# 17. Импорт моделей

Очень важно, чтобы SQLAlchemy и Alembic знали обо всех моделях.

Создай `app/models/__init__.py`:

```python
from app.models.user import User
from app.models.post import Post
from app.models.comment import Comment
from app.models.tag import Tag, PostTag

__all__ = [
    "User",
    "Post",
    "Comment",
    "Tag",
    "PostTag",
]
```

---

# 18. Первый запуск без Alembic

В реальных проектах таблицы создают через Alembic.

Но для понимания можно временно создать таблицы через SQLAlchemy.

Создай `app/main.py`:

```python
from app.db.base import Base
from app.db.session import engine
from app import models


def main() -> None:
    Base.metadata.create_all(bind=engine)
    print("Tables created")


if __name__ == "__main__":
    main()
```

Запусти:

```bash
python -m app.main
```

SQLAlchemy создаст таблицы.

Но важно:

```text
Base.metadata.create_all() — удобно для обучения.
В реальном проекте лучше использовать Alembic.
```

Если ты уже создал таблицы таким способом, для Alembic лучше удалить базу или таблицы, чтобы начать чисто.

---

# 19. Alembic: инициализация

В корне проекта выполни:

```bash
alembic init migrations
```

Появится структура:

```text
migrations/
│
├── versions/
├── env.py
├── README
└── script.py.mako

alembic.ini
```

---

# 20. Настройка Alembic

Открой `alembic.ini`.

Найди строку:

```ini
sqlalchemy.url = driver://user:pass@localhost/dbname
```

Можно заменить на:

```ini
sqlalchemy.url =
```

Мы будем брать URL из `.env`.

---

# 21. Настройка `migrations/env.py`

Открой `migrations/env.py` и сделай примерно так:

```python
import os
from logging.config import fileConfig

from dotenv import load_dotenv
from sqlalchemy import engine_from_config, pool
from alembic import context

from app.db.base import Base
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
    configuration = config.get_section(config.config_ini_section)

    connectable = engine_from_config(
        configuration,
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

## 21.1. Самая важная строка

```python
target_metadata = Base.metadata
```

Alembic сравнивает:

```text
текущее состояние базы
с
Base.metadata
```

И генерирует миграцию.

---

## 21.2. Важно импортировать модели

```python
from app import models
```

Если не импортировать модели, SQLAlchemy не узнает о них, и Alembic может создать пустую миграцию.

---

# 22. Первая миграция

Если база чистая, создай миграцию:

```bash
alembic revision --autogenerate -m "create initial tables"
```

Появится файл в:

```text
migrations/versions/
```

Например:

```text
202401010101_create_initial_tables.py
```

Открой его.

Там будет примерно:

```python
def upgrade() -> None:
    op.create_table(...)
    op.create_index(...)
```

И:

```python
def downgrade() -> None:
    op.drop_index(...)
    op.drop_table(...)
```

---

# 23. Что такое `upgrade` и `downgrade`

## 23.1. `upgrade`

```python
def upgrade() -> None:
    ...
```

Это действия при применении миграции.

Например:

```text
создать таблицы
добавить колонку
создать индекс
```

---

## 23.2. `downgrade`

```python
def downgrade() -> None:
    ...
```

Это откат миграции.

Например:

```text
удалить таблицу
удалить колонку
удалить индекс
```

---

# 24. Применение миграции

```bash
alembic upgrade head
```

Что значит `head`?

```text
head — самая свежая миграция
```

Проверить текущую миграцию:

```bash
alembic current
```

История миграций:

```bash
alembic history
```

Откатить на одну миграцию назад:

```bash
alembic downgrade -1
```

Откатить полностью:

```bash
alembic downgrade base
```

---

# 25. CRUD: создание данных

Создай файл `app/main.py`:

```python
from app.db.session import SessionLocal
from app.models import User, Post, Comment, Tag


def create_data() -> None:
    with SessionLocal() as session:
        user = User(
            email="john@example.com",
            username="john",
            hashed_password="not-real-hash",
        )

        post = Post(
            title="Первый пост",
            slug="first-post",
            content="Это содержимое первого поста.",
            is_published=True,
            author=user,
        )

        comment = Comment(
            text="Отличный пост!",
            author=user,
            post=post,
        )

        tag_python = Tag(name="python")
        tag_sqlalchemy = Tag(name="sqlalchemy")

        post.tags.append(tag_python)
        post.tags.append(tag_sqlalchemy)

        session.add(user)
        session.add(post)
        session.add(comment)

        session.commit()

        print("Created user:", user)
        print("Created post:", post)


if __name__ == "__main__":
    create_data()
```

Запусти:

```bash
python -m app.main
```

---

## 25.1. Почему достаточно `session.add(user)`

У нас есть связи:

```python
user.posts
post.comments
post.tags
```

Когда объекты связаны между собой, SQLAlchemy может сохранить связанные объекты каскадно.

Но для понятности в примере мы добавили несколько объектов явно:

```python
session.add(user)
session.add(post)
session.add(comment)
```

---

## 25.2. Что делает `commit`

```python
session.commit()
```

Он:

1. Отправляет SQL-запросы в базу.
2. Завершает транзакцию.
3. Фиксирует изменения.

---

# 26. Более правильная работа с транзакцией

Можно писать так:

```python
with SessionLocal.begin() as session:
    user = User(
        email="ann@example.com",
        username="ann",
        hashed_password="not-real-hash",
    )

    session.add(user)
```

`SessionLocal.begin()` автоматически:

- делает commit, если ошибок нет;
- делает rollback, если была ошибка;
- закрывает session.

Это хороший стиль.

---

# 27. Получение данных по id

```python
from app.db.session import SessionLocal
from app.models import User


def get_user_by_id(user_id: int) -> None:
    with SessionLocal() as session:
        user = session.get(User, user_id)

        if user is None:
            print("User not found")
            return

        print(user)


if __name__ == "__main__":
    get_user_by_id(1)
```

---

## 27.1. `session.get`

```python
session.get(User, 1)
```

Это получить объект по primary key.

То есть примерно:

```sql
SELECT * FROM users WHERE id = 1;
```

---

# 28. SELECT через SQLAlchemy 2

В SQLAlchemy 2 основной способ запросов — через `select`.

```python
from sqlalchemy import select

from app.db.session import SessionLocal
from app.models import User


def find_user_by_email(email: str) -> None:
    with SessionLocal() as session:
        stmt = select(User).where(User.email == email)

        result = session.execute(stmt)

        user = result.scalar_one_or_none()

        print(user)


if __name__ == "__main__":
    find_user_by_email("john@example.com")
```

---

## 28.1. Что такое `stmt`

```python
stmt = select(User).where(User.email == email)
```

Это SQL-запрос, но ещё не выполненный.

Примерно:

```sql
SELECT users.id, users.email, users.username
FROM users
WHERE users.email = 'john@example.com';
```

---

## 28.2. `session.execute`

```python
result = session.execute(stmt)
```

Выполняет запрос.

---

## 28.3. `scalar_one_or_none`

```python
user = result.scalar_one_or_none()
```

Ожидает:

- либо один объект;
- либо `None`.

Если объектов больше одного — будет ошибка.

---

# 29. Варианты получения результата

## 29.1. `scalars().all()`

Получить список ORM-объектов:

```python
stmt = select(User)
users = session.execute(stmt).scalars().all()
```

Или короче:

```python
users = session.scalars(stmt).all()
```

---

## 29.2. `scalar_one()`

Ожидает ровно один результат:

```python
user = session.execute(stmt).scalar_one()
```

Если 0 результатов — ошибка.

Если больше 1 — ошибка.

---

## 29.3. `scalar_one_or_none()`

Ожидает 0 или 1 результат:

```python
user = session.execute(stmt).scalar_one_or_none()
```

---

## 29.4. `first()`

```python
user = session.execute(stmt).scalars().first()
```

Вернёт первый объект или `None`.

Важно: `first()` сам не добавляет `LIMIT 1`. Лучше явно писать:

```python
stmt = select(User).limit(1)
```

---

# 30. Фильтрация

```python
stmt = select(User).where(User.is_active == True)
```

Лучше для boolean:

```python
stmt = select(User).where(User.is_active.is_(True))
```

Несколько условий:

```python
stmt = select(User).where(
    User.is_active.is_(True),
    User.email.ilike("%example.com"),
)
```

Это означает `AND`.

---

## 30.1. `and_`

```python
from sqlalchemy import and_

stmt = select(User).where(
    and_(
        User.is_active.is_(True),
        User.email.ilike("%example.com"),
    )
)
```

---

## 30.2. `or_`

```python
from sqlalchemy import or_

stmt = select(User).where(
    or_(
        User.email == "john@example.com",
        User.username == "john",
    )
)
```

---

## 30.3. `in_`

```python
stmt = select(User).where(User.id.in_([1, 2, 3]))
```

SQL:

```sql
WHERE id IN (1, 2, 3)
```

---

## 30.4. `like` и `ilike`

```python
stmt = select(User).where(User.email.like("%@example.com"))
```

`like` — чувствителен к регистру.

```python
stmt = select(User).where(User.email.ilike("%@example.com"))
```

`ilike` — без учёта регистра в PostgreSQL.

---

# 31. Сортировка, limit, offset

```python
stmt = (
    select(Post)
    .where(Post.is_published.is_(True))
    .order_by(Post.created_at.desc())
    .limit(10)
    .offset(0)
)
```

---

## 31.1. Пагинация

```python
def get_posts(page: int, page_size: int):
    offset = (page - 1) * page_size

    stmt = (
        select(Post)
        .where(Post.is_published.is_(True))
        .order_by(Post.created_at.desc())
        .limit(page_size)
        .offset(offset)
    )

    with SessionLocal() as session:
        return session.scalars(stmt).all()
```

---

# 32. Подсчёт строк

```python
from sqlalchemy import func, select

from app.models import User


stmt = select(func.count()).select_from(User)
count = session.execute(stmt).scalar_one()
```

С условием:

```python
stmt = select(func.count()).select_from(User).where(User.is_active.is_(True))
count = session.execute(stmt).scalar_one()
```

---

# 33. Обновление объекта

Способ 1: через ORM-объект.

```python
from app.db.session import SessionLocal
from app.models import User


def update_user(user_id: int) -> None:
    with SessionLocal.begin() as session:
        user = session.get(User, user_id)

        if user is None:
            print("User not found")
            return

        user.username = "new_username"


if __name__ == "__main__":
    update_user(1)
```

SQLAlchemy сам поймёт, что поле изменилось, и сделает `UPDATE`.

---

# 34. Обновление через `update`

```python
from sqlalchemy import update

from app.db.session import SessionLocal
from app.models import User


def deactivate_user(user_id: int) -> None:
    with SessionLocal.begin() as session:
        stmt = (
            update(User)
            .where(User.id == user_id)
            .values(is_active=False)
        )

        session.execute(stmt)
```

Это полезно, если не нужно загружать объект в память.

---

# 35. Удаление объекта

```python
from app.db.session import SessionLocal
from app.models import User


def delete_user(user_id: int) -> None:
    with SessionLocal.begin() as session:
        user = session.get(User, user_id)

        if user is None:
            print("User not found")
            return

        session.delete(user)
```

---

# 36. Удаление через `delete`

```python
from sqlalchemy import delete

from app.db.session import SessionLocal
from app.models import Comment


def delete_comments_for_post(post_id: int) -> None:
    with SessionLocal.begin() as session:
        stmt = delete(Comment).where(Comment.post_id == post_id)
        session.execute(stmt)
```

---

# 37. Связи между таблицами

## 37.1. One-to-many

Один пользователь имеет много постов.

В `User`:

```python
posts: Mapped[list["Post"]] = relationship(back_populates="author")
```

В `Post`:

```python
author_id: Mapped[int] = mapped_column(ForeignKey("users.id"))
author: Mapped["User"] = relationship(back_populates="posts")
```

Использование:

```python
user = session.get(User, 1)

for post in user.posts:
    print(post.title)
```

---

## 37.2. Many-to-one

Много постов принадлежат одному пользователю.

```python
post = session.get(Post, 1)
print(post.author.email)
```

---

## 37.3. Many-to-many

Посты и теги.

```python
post = session.get(Post, 1)

tag = Tag(name="postgresql")
post.tags.append(tag)

session.commit()
```

---

# 38. Lazy loading и проблема N+1

По умолчанию связи часто загружаются лениво.

Например:

```python
users = session.scalars(select(User)).all()

for user in users:
    print(user.posts)
```

Что может произойти:

```text
1 запрос на пользователей
N запросов на посты каждого пользователя
```

Если пользователей 100, будет 101 запрос.

Это называется проблема N+1.

---

# 39. Eager loading: `selectinload`

Хороший способ загрузить связи заранее:

```python
from sqlalchemy.orm import selectinload

stmt = select(User).options(selectinload(User.posts))

users = session.scalars(stmt).all()

for user in users:
    print(user.posts)
```

Будет примерно:

```text
1 запрос на users
1 запрос на posts WHERE author_id IN (...)
```

`selectinload` часто лучший выбор для one-to-many.

---

# 40. Eager loading: `joinedload`

```python
from sqlalchemy.orm import joinedload

stmt = select(Post).options(joinedload(Post.author))

posts = session.scalars(stmt).all()
```

`joinedload` делает JOIN.

Хорошо для many-to-one:

```text
post -> author
comment -> author
```

Для коллекций `joinedload` может дублировать строки, поэтому там чаще используют `selectinload`.

---

# 41. JOIN-запросы

Найти все посты пользователя по email:

```python
from sqlalchemy import select

stmt = (
    select(Post)
    .join(Post.author)
    .where(User.email == "john@example.com")
)

posts = session.scalars(stmt).all()
```

SQL примерно:

```sql
SELECT posts.*
FROM posts
JOIN users ON users.id = posts.author_id
WHERE users.email = 'john@example.com';
```

---

# 42. JOIN many-to-many

Найти посты с тегом `python`:

```python
stmt = (
    select(Post)
    .join(Post.tags)
    .where(Tag.name == "python")
)

posts = session.scalars(stmt).all()
```

---

# 43. Выбор отдельных колонок

Не всегда нужно получать ORM-объекты.

Можно выбрать только поля:

```python
stmt = select(User.id, User.email)

rows = session.execute(stmt).all()

for row in rows:
    print(row.id, row.email)
```

Или:

```python
for user_id, email in rows:
    print(user_id, email)
```

---

# 44. `flush`, `commit`, `refresh`

Очень важная тема.

## 44.1. `flush`

```python
session.flush()
```

Отправляет изменения в базу, но не завершает транзакцию.

Пример:

```python
with SessionLocal.begin() as session:
    user = User(
        email="new@example.com",
        username="new",
        hashed_password="hash",
    )

    session.add(user)

    print(user.id)  # None

    session.flush()

    print(user.id)  # уже есть id
```

`flush` нужен, когда тебе нужен `id` до commit.

---

## 44.2. `commit`

```python
session.commit()
```

Фиксирует транзакцию.

После commit данные реально сохранены.

---

## 44.3. `refresh`

```python
session.refresh(user)
```

Обновляет объект из базы.

Полезно, если база сама проставила значения:

- `created_at`;
- server defaults;
- generated columns.

---

# 45. Транзакции

Транзакция — это группа операций, которые должны выполниться целиком.

Пример:

```text
создать пользователя
создать профиль
создать настройки
```

Если ошибка случилась на середине — нужно откатить всё.

---

## 45.1. Пример ручной транзакции

```python
session = SessionLocal()

try:
    user = User(
        email="transaction@example.com",
        username="transaction",
        hashed_password="hash",
    )

    session.add(user)
    session.commit()

except Exception:
    session.rollback()
    raise

finally:
    session.close()
```

---

## 45.2. Лучше через context manager

```python
with SessionLocal.begin() as session:
    user = User(
        email="transaction2@example.com",
        username="transaction2",
        hashed_password="hash",
    )

    session.add(user)
```

Если ошибок нет — commit.

Если ошибка есть — rollback.

---

# 46. Ошибки IntegrityError

Если попытаться создать пользователя с уже существующим email:

```python
user = User(
    email="john@example.com",
    username="another",
    hashed_password="hash",
)
session.add(user)
session.commit()
```

PostgreSQL вернёт ошибку уникальности.

В Python это будет:

```python
from sqlalchemy.exc import IntegrityError
```

Пример обработки:

```python
from sqlalchemy.exc import IntegrityError

try:
    session.commit()
except IntegrityError:
    session.rollback()
    print("Email or username already exists")
```

Важно:

После ошибки обязательно делать:

```python
session.rollback()
```

Иначе сессия будет в сломанном состоянии.

---

# 47. Сырые SQL-запросы

Иногда нужно выполнить чистый SQL:

```python
from sqlalchemy import text

with SessionLocal() as session:
    result = session.execute(text("SELECT now()"))
    print(result.scalar_one())
```

С параметрами:

```python
stmt = text("SELECT * FROM users WHERE email = :email")

result = session.execute(
    stmt,
    {"email": "john@example.com"},
)

rows = result.all()
```

Никогда не делай так:

```python
text(f"SELECT * FROM users WHERE email = '{email}'")
```

Это риск SQL-инъекции.

---

# 48. Добавление новой колонки через Alembic

Допустим, хотим добавить пользователю поле `bio`.

В `User` добавляем:

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

Alembic создаст что-то вроде:

```python
def upgrade() -> None:
    op.add_column("users", sa.Column("bio", sa.String(length=500), nullable=True))


def downgrade() -> None:
    op.drop_column("users", "bio")
```

Применяем:

```bash
alembic upgrade head
```

---

# 49. Важный случай: добавление NOT NULL колонки

Плохой вариант:

```python
age: Mapped[int] = mapped_column(nullable=False)
```

Если в таблице уже есть пользователи, PostgreSQL не сможет добавить обязательную колонку без значения.

Правильные варианты.

---

## 49.1. Вариант 1: добавить default

```python
age: Mapped[int] = mapped_column(
    nullable=False,
    server_default="0",
)
```

Но если default нужен только для миграции, лучше потом убрать.

---

## 49.2. Вариант 2: безопасная миграция в несколько шагов

1. Добавить колонку nullable:

```python
op.add_column("users", sa.Column("age", sa.Integer(), nullable=True))
```

2. Заполнить значения:

```python
op.execute("UPDATE users SET age = 0 WHERE age IS NULL")
```

3. Сделать NOT NULL:

```python
op.alter_column("users", "age", nullable=False)
```

Итоговая миграция:

```python
def upgrade() -> None:
    op.add_column("users", sa.Column("age", sa.Integer(), nullable=True))
    op.execute("UPDATE users SET age = 0 WHERE age IS NULL")
    op.alter_column("users", "age", nullable=False)


def downgrade() -> None:
    op.drop_column("users", "age")
```

Это очень частый junior-level кейс.

---

# 50. Переименование колонки

Alembic autogenerate часто не понимает rename.

Если ты переименовал:

```python
username -> nickname
```

Alembic может сгенерировать:

```text
drop column username
add column nickname
```

Это потеряет данные.

Правильно вручную написать:

```python
op.alter_column("users", "username", new_column_name="nickname")
```

---

# 51. Индексы

Индекс ускоряет поиск.

В модели:

```python
email: Mapped[str] = mapped_column(
    String(255),
    index=True,
)
```

Alembic создаст индекс.

Можно создать составной индекс:

```python
from sqlalchemy import Index


class Post(Base):
    __tablename__ = "posts"

    # columns...

    __table_args__ = (
        Index("ix_posts_author_published", "author_id", "is_published"),
    )
```

Это полезно для запросов:

```python
select(Post).where(
    Post.author_id == 1,
    Post.is_published.is_(True),
)
```

---

# 52. Constraints

Ограничения помогают базе защищать данные.

## 52.1. UniqueConstraint

```python
from sqlalchemy import UniqueConstraint


class Tag(Base):
    __tablename__ = "tags"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50), nullable=False)

    __table_args__ = (
        UniqueConstraint("name", name="uq_tags_name"),
    )
```

---

## 52.2. CheckConstraint

Например, рейтинг должен быть от 1 до 5:

```python
from sqlalchemy import CheckConstraint


class Review(Base):
    __tablename__ = "reviews"

    id: Mapped[int] = mapped_column(primary_key=True)
    rating: Mapped[int] = mapped_column(nullable=False)

    __table_args__ = (
        CheckConstraint("rating >= 1 AND rating <= 5", name="rating_between_1_and_5"),
    )
```

---

# 53. PostgreSQL-специфичные типы

SQLAlchemy поддерживает типы PostgreSQL.

```python
from sqlalchemy.dialects.postgresql import JSONB, UUID, ARRAY
```

---

## 53.1. JSONB

```python
from sqlalchemy.dialects.postgresql import JSONB


metadata_json: Mapped[dict] = mapped_column(
    JSONB,
    default=dict,
    nullable=False,
)
```

Пример:

```python
post.metadata_json = {
    "views": 100,
    "source": "telegram",
}
```

---

## 53.2. UUID

```python
import uuid

from sqlalchemy.dialects.postgresql import UUID


id: Mapped[uuid.UUID] = mapped_column(
    UUID(as_uuid=True),
    primary_key=True,
    default=uuid.uuid4,
)
```

Это генерирует UUID на стороне Python.

---

## 53.3. PostgreSQL server-side UUID

Если хочешь генерировать UUID на стороне PostgreSQL:

```python
from sqlalchemy import text
from sqlalchemy.dialects.postgresql import UUID


id: Mapped[uuid.UUID] = mapped_column(
    UUID(as_uuid=True),
    primary_key=True,
    server_default=text("gen_random_uuid()"),
)
```

Но нужна extension:

```python
op.execute("CREATE EXTENSION IF NOT EXISTS pgcrypto")
```

---

# 54. Enum

Можно использовать enum.

```python
import enum

from sqlalchemy import Enum


class PostStatus(enum.StrEnum):
    DRAFT = "draft"
    PUBLISHED = "published"
    ARCHIVED = "archived"


status: Mapped[PostStatus] = mapped_column(
    Enum(PostStatus, name="post_status"),
    default=PostStatus.DRAFT,
    nullable=False,
)
```

Миграции с enum в PostgreSQL требуют аккуратности, особенно при изменении значений.

---

# 55. Alembic: полезные команды

Создать пустую миграцию:

```bash
alembic revision -m "manual migration"
```

Создать автоматическую миграцию:

```bash
alembic revision --autogenerate -m "message"
```

Применить все миграции:

```bash
alembic upgrade head
```

Откатить одну:

```bash
alembic downgrade -1
```

Показать текущую:

```bash
alembic current
```

Показать историю:

```bash
alembic history
```

Показать SQL без применения:

```bash
alembic upgrade head --sql
```

---

# 56. Что Alembic не всегда умеет определить

Alembic autogenerate хорошо видит:

- новые таблицы;
- удалённые таблицы;
- новые колонки;
- удалённые колонки;
- изменение nullable;
- индексы;
- foreign keys;
- иногда изменение типов.

Но плохо или не всегда видит:

- переименование таблицы;
- переименование колонки;
- сложные изменения enum;
- изменение данных;
- сложные constraints;
- некоторые server_default.

Поэтому правило:

```text
Всегда открывай миграцию и проверяй её глазами.
```

---

# 57. Типичный repository layer

В проектах часто выносят работу с базой в отдельные функции.

Создай `app/repositories.py`:

```python
from sqlalchemy import select
from sqlalchemy.orm import Session, selectinload

from app.models import User, Post


def create_user(
    session: Session,
    *,
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
    session.flush()

    return user


def get_user_by_id(session: Session, user_id: int) -> User | None:
    return session.get(User, user_id)


def get_user_by_email(session: Session, email: str) -> User | None:
    stmt = select(User).where(User.email == email)
    return session.scalar(stmt)


def get_published_posts(session: Session, limit: int = 20) -> list[Post]:
    stmt = (
        select(Post)
        .where(Post.is_published.is_(True))
        .options(selectinload(Post.tags))
        .order_by(Post.created_at.desc())
        .limit(limit)
    )

    return list(session.scalars(stmt).all())


def create_post(
    session: Session,
    *,
    author: User,
    title: str,
    slug: str,
    content: str,
) -> Post:
    post = Post(
        author=author,
        title=title,
        slug=slug,
        content=content,
    )

    session.add(post)
    session.flush()

    return post
```

Использование:

```python
from app.db.session import SessionLocal
from app.repositories import create_user, create_post


with SessionLocal.begin() as session:
    user = create_user(
        session,
        email="repo@example.com",
        username="repo",
        hashed_password="hash",
    )

    post = create_post(
        session,
        author=user,
        title="Repository pattern",
        slug="repository-pattern",
        content="Example content",
    )
```

---

# 58. Почему лучше передавать session внутрь функций

Хорошо:

```python
def create_user(session: Session, ...):
    ...
```

Плохо:

```python
def create_user(...):
    with SessionLocal() as session:
        ...
```

Почему?

Потому что одна бизнес-операция может включать несколько действий в одной транзакции:

```python
with SessionLocal.begin() as session:
    user = create_user(session, ...)
    profile = create_profile(session, user=user)
    settings = create_settings(session, user=user)
```

Если что-то упадёт — откатится всё.

---

# 59. Async SQLAlchemy

SQLAlchemy 2 поддерживает async.

Для async нужен другой драйвер:

```bash
pip install asyncpg
```

URL:

```env
ASYNC_DATABASE_URL=postgresql+asyncpg://app:app@localhost:5432/app
```

Пример `app/db/async_session.py`:

```python
import os

from dotenv import load_dotenv
from sqlalchemy.ext.asyncio import (
    create_async_engine,
    async_sessionmaker,
    AsyncSession,
)

load_dotenv()

ASYNC_DATABASE_URL = os.getenv("ASYNC_DATABASE_URL")

if ASYNC_DATABASE_URL is None:
    raise RuntimeError("ASYNC_DATABASE_URL is not set")


async_engine = create_async_engine(
    ASYNC_DATABASE_URL,
    echo=True,
    pool_pre_ping=True,
)

AsyncSessionLocal = async_sessionmaker(
    bind=async_engine,
    expire_on_commit=False,
    autoflush=False,
    class_=AsyncSession,
)
```

Использование:

```python
from sqlalchemy import select

from app.db.async_session import AsyncSessionLocal
from app.models import User


async def get_user(email: str) -> User | None:
    async with AsyncSessionLocal() as session:
        stmt = select(User).where(User.email == email)
        result = await session.execute(stmt)
        return result.scalar_one_or_none()
```

Транзакция:

```python
async with AsyncSessionLocal.begin() as session:
    user = User(
        email="async@example.com",
        username="async",
        hashed_password="hash",
    )

    session.add(user)
```

Важно:

```text
Async SQLAlchemy нужен в async-фреймворках, например FastAPI.
Для обычных скриптов можно использовать sync-версию.
```

---

# 60. Частые ошибки новичков

## 60.1. Alembic создаёт пустую миграцию

Причины:

1. Не импортированы модели в `env.py`.
2. `target_metadata = None`.
3. Модели не наследуются от правильного `Base`.
4. Запускаешь Alembic не из корня проекта.

Проверь:

```python
from app.db.base import Base
from app import models

target_metadata = Base.metadata
```

---

## 60.2. `ModuleNotFoundError: No module named app`

Запускай команды из корня проекта:

```bash
alembic revision --autogenerate -m "..."
```

Если не помогает, можно временно:

```bash
PYTHONPATH=. alembic upgrade head
```

Windows PowerShell:

```powershell
$env:PYTHONPATH="."
alembic upgrade head
```

---

## 60.3. `DetachedInstanceError`

Обычно значит:

```text
объект был загружен в одной session,
session закрылась,
а ты пытаешься обратиться к ленивой связи
```

Пример:

```python
with SessionLocal() as session:
    user = session.get(User, 1)

print(user.posts)  # ошибка, session уже закрыта
```

Решения:

1. Работать со связями внутри session.
2. Использовать `selectinload`.
3. Не возвращать ORM-объекты наружу без подготовки.

---

## 60.4. `IntegrityError`

Нарушено ограничение базы.

Например:

- duplicate email;
- foreign key не существует;
- nullable=False, но передан None.

После ошибки:

```python
session.rollback()
```

---

## 60.5. Забыли `commit`

```python
session.add(user)
```

Но нет:

```python
session.commit()
```

Данные не сохранятся.

---

## 60.6. Забыли применить миграцию

Ты изменил модель, но база не изменилась.

Нужно:

```bash
alembic revision --autogenerate -m "..."
alembic upgrade head
```

---

# 61. Хорошие практики junior+

## 61.1. Не используй `Base.metadata.create_all()` в продакшене

Для реального проекта:

```text
Alembic only
```

---

## 61.2. Всегда проверяй миграции глазами

Не доверяй autogenerate на 100%.

---

## 61.3. Одна бизнес-операция — одна транзакция

Хорошо:

```python
with SessionLocal.begin() as session:
    ...
```

---

## 61.4. Не создавай session внутри каждой маленькой функции

Лучше передавай session аргументом.

---

## 61.5. Используй индексы осознанно

Индексируй поля, по которым часто:

- ищешь;
- сортируешь;
- соединяешь таблицы.

Например:

```text
email
username
slug
author_id
post_id
created_at
```

---

## 61.6. Не загружай лишнее

Если нужны только email:

```python
select(User.email)
```

А не:

```python
select(User)
```

---

## 61.7. Помни про N+1

Если работаешь со связями в цикле — подумай про:

```python
selectinload
joinedload
```

---

# 62. Мини-шпаргалка SQLAlchemy 2

## Создать

```python
with SessionLocal.begin() as session:
    user = User(email="a@a.com", username="a", hashed_password="hash")
    session.add(user)
```

## Получить по id

```python
user = session.get(User, 1)
```

## Найти по условию

```python
stmt = select(User).where(User.email == "a@a.com")
user = session.scalar(stmt)
```

## Получить список

```python
stmt = select(User).order_by(User.id.desc())
users = session.scalars(stmt).all()
```

## Обновить

```python
user = session.get(User, 1)
user.username = "new"
session.commit()
```

## Удалить

```python
user = session.get(User, 1)
session.delete(user)
session.commit()
```

## Подсчитать

```python
count = session.scalar(select(func.count()).select_from(User))
```

## JOIN

```python
stmt = select(Post).join(Post.author).where(User.email == "a@a.com")
posts = session.scalars(stmt).all()
```

## Eager loading

```python
stmt = select(User).options(selectinload(User.posts))
users = session.scalars(stmt).all()
```

---

# 63. Мини-шпаргалка Alembic

## Инициализация

```bash
alembic init migrations
```

## Создать миграцию

```bash
alembic revision --autogenerate -m "message"
```

## Применить

```bash
alembic upgrade head
```

## Откатить одну

```bash
alembic downgrade -1
```

## Текущая версия

```bash
alembic current
```

## История

```bash
alembic history
```

---

# 64. Что нужно уметь для уровня уверенный junior+

Ты должен уметь:

1. Объяснить, что такое ORM.
2. Подключить SQLAlchemy к PostgreSQL.
3. Написать модели с `Mapped` и `mapped_column`.
4. Создать связи:
   - one-to-many;
   - many-to-one;
   - many-to-many.
5. Написать CRUD.
6. Использовать `select`, `where`, `join`, `order_by`, `limit`.
7. Понимать `Session`, `commit`, `rollback`, `flush`.
8. Понимать транзакции.
9. Настроить Alembic.
10. Создать и применить миграции.
11. Проверять autogenerate-миграции.
12. Понимать `IntegrityError`.
13. Понимать проблему N+1.
14. Использовать `selectinload` и `joinedload`.
15. Аккуратно добавлять `NOT NULL` колонки.
16. Не терять данные при rename колонок.
17. Писать repository-функции.
18. Отличать sync и async SQLAlchemy.

---

# 65. Итоговая ментальная модель

Запомни так:

```text
PostgreSQL хранит данные.
SQLAlchemy описывает таблицы Python-классами.
Session выполняет операции с базой.
Alembic изменяет структуру базы через миграции.
```

Ещё проще:

```text
Model  -> описание таблицы
Session -> работа с данными
Alembic -> изменение схемы базы
```

Типичный цикл разработки:

```text
1. Изменил модель
2. alembic revision --autogenerate -m "..."
3. Проверил файл миграции
4. alembic upgrade head
5. Пишешь код работы с данными
```

Типичный цикл работы приложения:

```text
1. Открыть session
2. Выполнить запросы
3. commit или rollback
4. Закрыть session
```

Если ты хорошо понял этот материал и сам руками повторил проект, ты уже находишься примерно на уровне junior/junior+ по SQLAlchemy + Alembic для обычных backend-задач.
