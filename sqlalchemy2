# SQLAlchemy 2 + PostgreSQL: подробная практическая документация с нуля до уровня junior+

Ниже — большой практический конспект по **современному SQLAlchemy 2.x** на примере **PostgreSQL**.

Мы будем использовать **только новый стиль SQLAlchemy 2**:

- `select(...)`, а не старый `session.query(...)`
- `Mapped[...]`, `mapped_column(...)`
- `DeclarativeBase`
- `Session.execute(...)`, `Session.scalars(...)`, `Session.scalar(...)`
- явные транзакции через `with session.begin():`
- PostgreSQL через современный драйвер `psycopg`

---

# 1. Что такое SQLAlchemy

**SQLAlchemy** — это библиотека для работы с базами данных из Python.

Она состоит из двух больших частей:

## 1.1. SQLAlchemy Core

Это низкоуровневый конструктор SQL-запросов.

Например:

```python
from sqlalchemy import select

stmt = select(users_table).where(users_table.c.id == 1)
```

Core помогает писать SQL на Python, но без привязки к ORM-моделям.

## 1.2. SQLAlchemy ORM

ORM расшифровывается как **Object-Relational Mapping**.

ORM позволяет работать с таблицами как с Python-классами.

Например, есть таблица `users`:

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email TEXT NOT NULL
);
```

В SQLAlchemy ORM ты описываешь её как класс:

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str]
```

И потом работаешь с обычными объектами:

```python
user = User(email="test@example.com")
session.add(user)
session.commit()
```

SQLAlchemy сам создаст SQL:

```sql
INSERT INTO users (email) VALUES ('test@example.com');
```

---

# 2. Что нужно знать перед SQLAlchemy

Желательно понимать базовые вещи:

## 2.1. Таблица

Таблица — это структура данных в БД.

Например, таблица `users`:

| id | email              | name  |
|----|--------------------|-------|
| 1  | ivan@example.com   | Ivan  |
| 2  | anna@example.com   | Anna  |

## 2.2. Строка

Одна запись в таблице.

Например:

```text
1 | ivan@example.com | Ivan
```

## 2.3. Колонка

Поле таблицы:

- `id`
- `email`
- `name`

## 2.4. Первичный ключ

**Primary key** — уникальный идентификатор строки.

Обычно это `id`.

```sql
id SERIAL PRIMARY KEY
```

## 2.5. Внешний ключ

**Foreign key** — ссылка на другую таблицу.

Например:

```sql
posts.user_id -> users.id
```

То есть пост принадлежит пользователю.

---

# 3. Установка PostgreSQL через Docker

Если PostgreSQL ещё нет, проще всего поднять его через Docker.

Создай файл `docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:16
    container_name: sqlalchemy_postgres
    environment:
      POSTGRES_USER: app
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

---

# 4. Создание Python-проекта

Создадим проект:

```bash
mkdir sqlalchemy2_tutorial
cd sqlalchemy2_tutorial
```

Создадим виртуальное окружение:

```bash
python -m venv .venv
```

Активируем:

## Linux/macOS

```bash
source .venv/bin/activate
```

## Windows PowerShell

```powershell
.venv\Scripts\Activate.ps1
```

Установим зависимости:

```bash
pip install "sqlalchemy>=2.0" psycopg[binary] python-dotenv alembic
```

Что мы установили:

```text
sqlalchemy      — сама библиотека SQLAlchemy
psycopg         — драйвер PostgreSQL
python-dotenv   — чтобы читать настройки из .env
alembic         — миграции базы данных
```

---

# 5. Структура проекта

Сделаем такую структуру:

```text
sqlalchemy2_tutorial/
│
├── app/
│   ├── __init__.py
│   ├── config.py
│   ├── database.py
│   ├── models.py
│   └── main.py
│
├── .env
└── docker-compose.yml
```

---

# 6. Настройки подключения

Создай файл `.env`:

```env
DATABASE_URL=postgresql+psycopg://app:app_password@localhost:5432/app_db
```

Разберём строку подключения:

```text
postgresql+psycopg://app:app_password@localhost:5432/app_db
```

Это значит:

```text
postgresql       — тип базы данных
psycopg          — драйвер
app              — пользователь
app_password     — пароль
localhost        — адрес сервера
5432             — порт PostgreSQL
app_db           — имя базы данных
```

---

# 7. Конфигурация приложения

Файл `app/config.py`:

```python
from dotenv import load_dotenv
import os

load_dotenv()


class Settings:
    DATABASE_URL: str = os.environ["DATABASE_URL"]


settings = Settings()
```

Здесь:

```python
load_dotenv()
```

загружает переменные из файла `.env`.

```python
os.environ["DATABASE_URL"]
```

читает строку подключения к базе.

---

# 8. Engine: подключение к базе

Файл `app/database.py`:

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker

from app.config import settings


engine = create_engine(
    settings.DATABASE_URL,
    echo=True,
)
```

Разберём:

```python
create_engine(...)
```

создаёт объект `Engine`.

## Что такое Engine?

`Engine` — это главный объект SQLAlchemy для подключения к базе.

Он:

- хранит настройки подключения;
- управляет пулом соединений;
- отправляет SQL-запросы в базу;
- получает результаты.

Важно: `create_engine()` **не открывает соединение сразу**. Соединение реально откроется, когда ты сделаешь запрос.

---

## 8.1. Параметр `echo=True`

```python
echo=True
```

означает: SQLAlchemy будет печатать SQL-запросы в консоль.

Это очень полезно при обучении.

В продакшене обычно ставят:

```python
echo=False
```

---

# 9. Базовый класс моделей

В `app/database.py` добавим:

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker

from app.config import settings


engine = create_engine(
    settings.DATABASE_URL,
    echo=True,
)


class Base(DeclarativeBase):
    pass
```

## Что такое `Base`

`Base` — это родительский класс для всех ORM-моделей.

Все модели будут наследоваться от него:

```python
class User(Base):
    ...
```

SQLAlchemy через `Base` узнаёт:

- какие есть модели;
- какие таблицы нужно создать;
- какие поля есть у таблиц;
- какие связи между таблицами.

---

# 10. Session: единица работы с базой

В `app/database.py` добавим фабрику сессий:

```python
SessionLocal = sessionmaker(
    bind=engine,
    autoflush=False,
    expire_on_commit=False,
)
```

Итоговый `app/database.py`:

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker

from app.config import settings


engine = create_engine(
    settings.DATABASE_URL,
    echo=True,
)


class Base(DeclarativeBase):
    pass


SessionLocal = sessionmaker(
    bind=engine,
    autoflush=False,
    expire_on_commit=False,
)
```

---

## Что такое Session?

`Session` — это объект, через который ты работаешь с ORM.

Через сессию ты:

- добавляешь объекты;
- изменяешь объекты;
- удаляешь объекты;
- делаешь запросы;
- открываешь и завершаешь транзакции.

Пример:

```python
with SessionLocal() as session:
    user = User(email="ivan@example.com")
    session.add(user)
    session.commit()
```

---

## Важно: Session — это не соединение напрямую

`Session` — это более высокий уровень.

Она:

- берёт соединение из `Engine`, когда нужно;
- отслеживает изменения объектов;
- управляет транзакциями;
- синхронизирует Python-объекты с базой.

---

# 11. Первая модель User

Файл `app/models.py`:

```python
from sqlalchemy.orm import Mapped, mapped_column

from app.database import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(unique=True, nullable=False)
    name: Mapped[str | None] = mapped_column(nullable=True)
```

Разберём подробно.

---

## 11.1. `__tablename__`

```python
__tablename__ = "users"
```

Это имя таблицы в PostgreSQL.

То есть SQLAlchemy понимает:

```python
class User(Base)
```

соответствует таблице:

```sql
users
```

---

## 11.2. `Mapped[...]`

```python
id: Mapped[int]
```

`Mapped` — это специальный тип SQLAlchemy 2.

Он говорит:

> Это поле является колонкой ORM-модели.

Новый стиль SQLAlchemy 2 использует типизацию.

---

## 11.3. `mapped_column(...)`

```python
id: Mapped[int] = mapped_column(primary_key=True)
```

`mapped_column()` описывает колонку таблицы.

Например:

```python
email: Mapped[str] = mapped_column(unique=True, nullable=False)
```

означает:

- колонка `email`;
- тип Python — `str`;
- значение обязательно;
- значение должно быть уникальным.

---

## 11.4. `primary_key=True`

```python
id: Mapped[int] = mapped_column(primary_key=True)
```

Это первичный ключ.

В PostgreSQL для `int primary_key=True` SQLAlchemy обычно сделает автоинкремент.

---

## 11.5. `unique=True`

```python
email: Mapped[str] = mapped_column(unique=True)
```

Значит в таблице нельзя иметь двух пользователей с одинаковым email.

---

## 11.6. `nullable=False`

```python
email: Mapped[str] = mapped_column(nullable=False)
```

Значит колонка не может быть `NULL`.

---

## 11.7. `nullable=True`

```python
name: Mapped[str | None] = mapped_column(nullable=True)
```

Значит имя может отсутствовать.

В Python это отражается как:

```python
str | None
```

То есть значение может быть строкой или `None`.

---

# 12. Создание таблиц без Alembic

На начальном этапе можно создать таблицы напрямую.

Файл `app/main.py`:

```python
from app.database import Base, engine
from app import models


def main() -> None:
    Base.metadata.create_all(bind=engine)


if __name__ == "__main__":
    main()
```

Запусти:

```bash
python -m app.main
```

SQLAlchemy создаст таблицу `users`.

---

## Важно про `create_all`

```python
Base.metadata.create_all(bind=engine)
```

создаёт таблицы, если их ещё нет.

Но `create_all()`:

- не умеет нормально менять уже существующие таблицы;
- не заменяет миграции;
- подходит для обучения и простых тестов;
- в реальных проектах обычно используют Alembic.

---

# 13. Добавление пользователя

Изменим `app/main.py`:

```python
from app.database import SessionLocal
from app.models import User


def main() -> None:
    with SessionLocal() as session:
        user = User(
            email="ivan@example.com",
            name="Ivan",
        )

        session.add(user)
        session.commit()

        print(user.id)


if __name__ == "__main__":
    main()
```

Разберём.

---

## 13.1. Создание объекта

```python
user = User(email="ivan@example.com", name="Ivan")
```

Пока это просто Python-объект.

Он ещё не находится в базе.

---

## 13.2. `session.add(user)`

```python
session.add(user)
```

говорит сессии:

> Этот объект нужно сохранить в базе.

Но SQL-запрос ещё может не выполниться прямо в этот момент.

---

## 13.3. `session.commit()`

```python
session.commit()
```

фиксирует транзакцию.

SQLAlchemy отправляет в базу примерно такой SQL:

```sql
INSERT INTO users (email, name)
VALUES ('ivan@example.com', 'Ivan')
RETURNING users.id;
```

После `commit()` пользователь сохранён.

---

# 14. Транзакции

## 14.1. Что такое транзакция?

Транзакция — это группа операций, которая выполняется целиком или не выполняется вообще.

Например:

```text
1. Создать пользователя
2. Создать профиль пользователя
3. Создать настройки пользователя
```

Если шаг 2 упал с ошибкой, нельзя оставлять только шаг 1.

Тогда нужна транзакция:

- если всё хорошо — `COMMIT`;
- если ошибка — `ROLLBACK`.

---

## 14.2. Рекомендуемый стиль SQLAlchemy 2

Лучше писать так:

```python
from app.database import SessionLocal
from app.models import User


def main() -> None:
    with SessionLocal() as session:
        with session.begin():
            user = User(email="anna@example.com", name="Anna")
            session.add(user)

        print(user.id)


if __name__ == "__main__":
    main()
```

Здесь:

```python
with session.begin():
```

автоматически:

- делает `commit`, если ошибок нет;
- делает `rollback`, если произошла ошибка.

---

## 14.3. Почему это лучше

Вместо:

```python
try:
    session.add(user)
    session.commit()
except:
    session.rollback()
    raise
```

можно писать:

```python
with session.begin():
    session.add(user)
```

Это чище и безопаснее.

---

# 15. Получение объекта по primary key

```python
from app.database import SessionLocal
from app.models import User


def main() -> None:
    with SessionLocal() as session:
        user = session.get(User, 1)

        if user is None:
            print("Пользователь не найден")
        else:
            print(user.email, user.name)


if __name__ == "__main__":
    main()
```

## `session.get(Model, id)`

Это самый простой и правильный способ получить объект по первичному ключу.

```python
user = session.get(User, 1)
```

Он ищет:

```sql
SELECT * FROM users WHERE id = 1;
```

---

# 16. SELECT в SQLAlchemy 2

В новом SQLAlchemy 2 для запросов используется `select()`.

```python
from sqlalchemy import select

stmt = select(User)
```

`stmt` — это объект SQL-запроса.

Он ещё не выполнен.

---

## 16.1. Получить всех пользователей

```python
from sqlalchemy import select

from app.database import SessionLocal
from app.models import User


def main() -> None:
    with SessionLocal() as session:
        stmt = select(User)

        users = session.scalars(stmt).all()

        for user in users:
            print(user.id, user.email, user.name)


if __name__ == "__main__":
    main()
```

Разбор:

```python
stmt = select(User)
```

создаёт SQL:

```sql
SELECT users.id, users.email, users.name
FROM users;
```

```python
session.scalars(stmt)
```

выполняет запрос и возвращает ORM-объекты `User`.

```python
.all()
```

возвращает список.

---

## 16.2. `session.scalars(...)`

Используется, когда ты выбираешь одну ORM-сущность или одну колонку.

Например:

```python
session.scalars(select(User)).all()
```

вернёт:

```python
list[User]
```

А:

```python
session.scalars(select(User.email)).all()
```

вернёт:

```python
list[str]
```

---

## 16.3. `session.scalar(...)`

Используется, когда нужен один результат.

```python
user = session.scalar(
    select(User).where(User.email == "ivan@example.com")
)
```

Если найден — вернёт `User`.

Если не найден — вернёт `None`.

---

# 17. Фильтрация WHERE

```python
from sqlalchemy import select

stmt = select(User).where(User.email == "ivan@example.com")
```

Полный пример:

```python
from sqlalchemy import select

from app.database import SessionLocal
from app.models import User


def main() -> None:
    with SessionLocal() as session:
        user = session.scalar(
            select(User).where(User.email == "ivan@example.com")
        )

        if user:
            print(user.id, user.name)


if __name__ == "__main__":
    main()
```

---

## 17.1. Несколько условий

```python
stmt = select(User).where(
    User.email == "ivan@example.com",
    User.name == "Ivan",
)
```

Это означает `AND`.

SQL:

```sql
WHERE email = 'ivan@example.com'
  AND name = 'Ivan'
```

---

## 17.2. `and_`

```python
from sqlalchemy import and_

stmt = select(User).where(
    and_(
        User.email == "ivan@example.com",
        User.name == "Ivan",
    )
)
```

Но чаще в SQLAlchemy 2 пишут проще:

```python
stmt = select(User).where(
    User.email == "ivan@example.com",
    User.name == "Ivan",
)
```

---

## 17.3. `or_`

```python
from sqlalchemy import or_

stmt = select(User).where(
    or_(
        User.email == "ivan@example.com",
        User.email == "anna@example.com",
    )
)
```

SQL:

```sql
WHERE email = 'ivan@example.com'
   OR email = 'anna@example.com'
```

---

## 17.4. `in_`

```python
stmt = select(User).where(
    User.id.in_([1, 2, 3])
)
```

SQL:

```sql
WHERE id IN (1, 2, 3)
```

---

## 17.5. `like` и `ilike`

```python
stmt = select(User).where(
    User.email.like("%@example.com")
)
```

`LIKE` чувствителен к регистру не во всех БД одинаково.

В PostgreSQL часто используют `ILIKE`:

```python
stmt = select(User).where(
    User.email.ilike("%ivan%")
)
```

`ILIKE` — поиск без учёта регистра.

---

# 18. Сортировка ORDER BY

```python
stmt = select(User).order_by(User.id)
```

По убыванию:

```python
stmt = select(User).order_by(User.id.desc())
```

По возрастанию:

```python
stmt = select(User).order_by(User.id.asc())
```

Пример:

```python
users = session.scalars(
    select(User).order_by(User.id.desc())
).all()
```

---

# 19. LIMIT и OFFSET

```python
stmt = select(User).limit(10)
```

Пропустить первые 20:

```python
stmt = select(User).offset(20).limit(10)
```

Пример пагинации:

```python
page = 2
page_size = 10

stmt = (
    select(User)
    .order_by(User.id)
    .offset((page - 1) * page_size)
    .limit(page_size)
)

users = session.scalars(stmt).all()
```

---

# 20. Обновление объекта

Самый ORM-способ:

```python
from app.database import SessionLocal
from app.models import User


def main() -> None:
    with SessionLocal() as session:
        with session.begin():
            user = session.get(User, 1)

            if user is None:
                print("Не найден")
                return

            user.name = "Ivan Updated"


if __name__ == "__main__":
    main()
```

SQLAlchemy сам увидит, что поле изменилось, и выполнит:

```sql
UPDATE users SET name = 'Ivan Updated' WHERE id = 1;
```

---

## 20.1. Почему не нужен `session.add(user)`?

Если объект был получен через эту же сессию:

```python
user = session.get(User, 1)
```

он уже находится под управлением сессии.

SQLAlchemy отслеживает изменения.

---

# 21. Удаление объекта

```python
from app.database import SessionLocal
from app.models import User


def main() -> None:
    with SessionLocal() as session:
        with session.begin():
            user = session.get(User, 1)

            if user is None:
                print("Не найден")
                return

            session.delete(user)


if __name__ == "__main__":
    main()
```

SQL:

```sql
DELETE FROM users WHERE id = 1;
```

---

# 22. Добавим несколько моделей

Сделаем мини-блог:

- `User` — пользователь
- `Post` — пост
- `Comment` — комментарий
- `Tag` — тег

Связи:

```text
User 1 -> N Post
User 1 -> N Comment
Post 1 -> N Comment
Post N -> M Tag
```

---

# 23. Модель User и Post: связь один-ко-многим

Обновим `app/models.py`:

```python
from datetime import datetime
from typing import List

from sqlalchemy import ForeignKey, String, Text, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.database import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    email: Mapped[str] = mapped_column(
        String(255),
        unique=True,
        nullable=False,
    )

    name: Mapped[str | None] = mapped_column(
        String(100),
        nullable=True,
    )

    created_at: Mapped[datetime] = mapped_column(
        server_default=func.now(),
        nullable=False,
    )

    posts: Mapped[List["Post"]] = relationship(
        back_populates="author",
    )


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)

    title: Mapped[str] = mapped_column(
        String(200),
        nullable=False,
    )

    body: Mapped[str] = mapped_column(
        Text,
        nullable=False,
    )

    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id"),
        nullable=False,
    )

    created_at: Mapped[datetime] = mapped_column(
        server_default=func.now(),
        nullable=False,
    )

    author: Mapped["User"] = relationship(
        back_populates="posts",
    )
```

---

## 23.1. Что такое `ForeignKey`

```python
user_id: Mapped[int] = mapped_column(
    ForeignKey("users.id"),
    nullable=False,
)
```

Это значит:

```text
posts.user_id ссылается на users.id
```

В SQL:

```sql
FOREIGN KEY (user_id) REFERENCES users(id)
```

То есть каждый пост принадлежит пользователю.

---

## 23.2. Что такое `relationship`

```python
posts: Mapped[List["Post"]] = relationship(
    back_populates="author",
)
```

Это не колонка в таблице.

Это ORM-связь.

Она позволяет писать:

```python
user.posts
```

и получать список постов пользователя.

---

## 23.3. `back_populates`

В `User`:

```python
posts = relationship(back_populates="author")
```

В `Post`:

```python
author = relationship(back_populates="posts")
```

Это две стороны одной связи.

```text
User.posts <-> Post.author
```

---

# 24. Создание пользователя с постами

```python
from app.database import SessionLocal
from app.models import User, Post


def main() -> None:
    with SessionLocal() as session:
        with session.begin():
            user = User(
                email="blogger@example.com",
                name="Blogger",
            )

            post1 = Post(
                title="Первый пост",
                body="Текст первого поста",
                author=user,
            )

            post2 = Post(
                title="Второй пост",
                body="Текст второго поста",
                author=user,
            )

            session.add(user)


if __name__ == "__main__":
    main()
```

Здесь мы создали пользователя и два поста.

Важно:

```python
author=user
```

автоматически связывает пост с пользователем.

Если связь настроена правильно, достаточно сделать:

```python
session.add(user)
```

SQLAlchemy добавит и пользователя, и посты.

---

# 25. Получение пользователя и его постов

```python
from app.database import SessionLocal
from app.models import User


def main() -> None:
    with SessionLocal() as session:
        user = session.get(User, 1)

        if user is None:
            return

        print(user.email)

        for post in user.posts:
            print(post.title)


if __name__ == "__main__":
    main()
```

---

## 25.1. Lazy loading

Когда ты пишешь:

```python
user = session.get(User, 1)
```

SQLAlchemy сначала загружает только пользователя.

Когда ты потом обращаешься:

```python
user.posts
```

SQLAlchemy делает дополнительный SQL-запрос за постами.

Это называется **lazy loading** — ленивая загрузка.

---

# 26. Проблема N+1

Представь:

```python
users = session.scalars(select(User)).all()

for user in users:
    print(user.posts)
```

Что произойдёт:

```text
1 запрос — получить всех пользователей
N запросов — получить посты каждого пользователя
```

Если пользователей 100, будет 101 запрос.

Это называется проблема **N+1**.

---

# 27. `selectinload`: правильная загрузка коллекций

Чтобы избежать N+1, используем `selectinload`.

```python
from sqlalchemy import select
from sqlalchemy.orm import selectinload

from app.database import SessionLocal
from app.models import User


def main() -> None:
    with SessionLocal() as session:
        stmt = (
            select(User)
            .options(selectinload(User.posts))
        )

        users = session.scalars(stmt).all()

        for user in users:
            print(user.email)

            for post in user.posts:
                print("  ", post.title)


if __name__ == "__main__":
    main()
```

SQLAlchemy сделает примерно:

```sql
SELECT * FROM users;
SELECT * FROM posts WHERE posts.user_id IN (...);
```

То есть вместо 101 запроса будет 2.

---

# 28. `joinedload`

`joinedload` загружает связь через `JOIN`.

```python
from sqlalchemy.orm import joinedload

stmt = select(User).options(joinedload(User.posts))
```

Но для коллекций нужно использовать `.unique()`:

```python
result = session.execute(
    select(User).options(joinedload(User.posts))
)

users = result.unique().scalars().all()
```

Почему?

Потому что `JOIN` может вернуть несколько строк на одного пользователя:

```text
user1 + post1
user1 + post2
user1 + post3
```

`.unique()` убирает дубли ORM-объектов.

---

## 28.1. Когда использовать `selectinload`, а когда `joinedload`

Обычно:

```text
selectinload — хороший выбор для one-to-many и many-to-many
joinedload   — хороший выбор для many-to-one / one-to-one
```

Пример:

```python
select(Post).options(joinedload(Post.author))
```

Постов много, но у каждого один автор — `joinedload` удобен.

---

# 29. Комментарии: несколько связей

Добавим модель `Comment`.

```python
from datetime import datetime
from typing import List

from sqlalchemy import ForeignKey, String, Text, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.database import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    email: Mapped[str] = mapped_column(String(255), unique=True, nullable=False)

    name: Mapped[str | None] = mapped_column(String(100), nullable=True)

    created_at: Mapped[datetime] = mapped_column(
        server_default=func.now(),
        nullable=False,
    )

    posts: Mapped[List["Post"]] = relationship(
        back_populates="author",
    )

    comments: Mapped[List["Comment"]] = relationship(
        back_populates="author",
    )


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)

    title: Mapped[str] = mapped_column(String(200), nullable=False)

    body: Mapped[str] = mapped_column(Text, nullable=False)

    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id"),
        nullable=False,
    )

    created_at: Mapped[datetime] = mapped_column(
        server_default=func.now(),
        nullable=False,
    )

    author: Mapped["User"] = relationship(
        back_populates="posts",
    )

    comments: Mapped[List["Comment"]] = relationship(
        back_populates="post",
        cascade="all, delete-orphan",
    )


class Comment(Base):
    __tablename__ = "comments"

    id: Mapped[int] = mapped_column(primary_key=True)

    text: Mapped[str] = mapped_column(Text, nullable=False)

    post_id: Mapped[int] = mapped_column(
        ForeignKey("posts.id"),
        nullable=False,
    )

    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id"),
        nullable=False,
    )

    created_at: Mapped[datetime] = mapped_column(
        server_default=func.now(),
        nullable=False,
    )

    post: Mapped["Post"] = relationship(
        back_populates="comments",
    )

    author: Mapped["User"] = relationship(
        back_populates="comments",
    )
```

---

# 30. Cascade

```python
comments: Mapped[List["Comment"]] = relationship(
    back_populates="post",
    cascade="all, delete-orphan",
)
```

Это значит:

- если удалить `Post`, SQLAlchemy удалит его комментарии;
- если комментарий убрать из `post.comments`, он будет удалён из базы.

Пример:

```python
post.comments.remove(comment)
```

С `delete-orphan` комментарий станет «сиротой» и будет удалён.

---

## Важно

Cascade на уровне ORM — это не то же самое, что `ON DELETE CASCADE` в базе.

Для уровня PostgreSQL можно писать:

```python
ForeignKey("posts.id", ondelete="CASCADE")
```

Но тогда ещё нужно правильно настроить отношения, например:

```python
passive_deletes=True
```

---

# 31. Many-to-many: посты и теги

Связь многие-ко-многим:

```text
Post N <-> M Tag
```

Один пост может иметь много тегов.

Один тег может быть у многих постов.

Для этого нужна промежуточная таблица:

```text
post_tags
```

---

## 31.1. Таблица связи

```python
from sqlalchemy import Table, Column, ForeignKey

post_tags = Table(
    "post_tags",
    Base.metadata,
    Column("post_id", ForeignKey("posts.id"), primary_key=True),
    Column("tag_id", ForeignKey("tags.id"), primary_key=True),
)
```

---

## 31.2. Полные модели с Tag

```python
from datetime import datetime
from typing import List

from sqlalchemy import (
    ForeignKey,
    String,
    Text,
    func,
    Table,
    Column,
)
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.database import Base


post_tags = Table(
    "post_tags",
    Base.metadata,
    Column("post_id", ForeignKey("posts.id"), primary_key=True),
    Column("tag_id", ForeignKey("tags.id"), primary_key=True),
)


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    email: Mapped[str] = mapped_column(String(255), unique=True, nullable=False)

    name: Mapped[str | None] = mapped_column(String(100), nullable=True)

    created_at: Mapped[datetime] = mapped_column(server_default=func.now(), nullable=False)

    posts: Mapped[List["Post"]] = relationship(back_populates="author")

    comments: Mapped[List["Comment"]] = relationship(back_populates="author")


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)

    title: Mapped[str] = mapped_column(String(200), nullable=False)

    body: Mapped[str] = mapped_column(Text, nullable=False)

    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), nullable=False)

    created_at: Mapped[datetime] = mapped_column(server_default=func.now(), nullable=False)

    author: Mapped["User"] = relationship(back_populates="posts")

    comments: Mapped[List["Comment"]] = relationship(
        back_populates="post",
        cascade="all, delete-orphan",
    )

    tags: Mapped[List["Tag"]] = relationship(
        secondary=post_tags,
        back_populates="posts",
    )


class Comment(Base):
    __tablename__ = "comments"

    id: Mapped[int] = mapped_column(primary_key=True)

    text: Mapped[str] = mapped_column(Text, nullable=False)

    post_id: Mapped[int] = mapped_column(ForeignKey("posts.id"), nullable=False)

    user_id: Mapped[int] = mapped_column(ForeignKey("users.id"), nullable=False)

    created_at: Mapped[datetime] = mapped_column(server_default=func.now(), nullable=False)

    post: Mapped["Post"] = relationship(back_populates="comments")

    author: Mapped["User"] = relationship(back_populates="comments")


class Tag(Base):
    __tablename__ = "tags"

    id: Mapped[int] = mapped_column(primary_key=True)

    name: Mapped[str] = mapped_column(String(50), unique=True, nullable=False)

    posts: Mapped[List["Post"]] = relationship(
        secondary=post_tags,
        back_populates="tags",
    )
```

---

# 32. Создание поста с тегами

```python
from app.database import SessionLocal
from app.models import User, Post, Tag


def main() -> None:
    with SessionLocal() as session:
        with session.begin():
            user = User(
                email="tag_user@example.com",
                name="Tag User",
            )

            python_tag = Tag(name="python")
            sqlalchemy_tag = Tag(name="sqlalchemy")

            post = Post(
                title="SQLAlchemy 2 Guide",
                body="Очень подробный текст",
                author=user,
                tags=[python_tag, sqlalchemy_tag],
            )

            session.add(post)


if __name__ == "__main__":
    main()
```

SQLAlchemy добавит:

- пользователя;
- теги;
- пост;
- записи в таблицу `post_tags`.

---

# 33. JOIN-запросы

## 33.1. Получить посты с авторами

```python
from sqlalchemy import select
from sqlalchemy.orm import joinedload

from app.database import SessionLocal
from app.models import Post


def main() -> None:
    with SessionLocal() as session:
        posts = session.scalars(
            select(Post).options(joinedload(Post.author))
        ).all()

        for post in posts:
            print(post.title, post.author.email)


if __name__ == "__main__":
    main()
```

Это ORM-способ.

---

## 33.2. Явный JOIN

```python
from sqlalchemy import select

from app.database import SessionLocal
from app.models import User, Post


def main() -> None:
    with SessionLocal() as session:
        stmt = (
            select(Post, User)
            .join(User, Post.user_id == User.id)
        )

        rows = session.execute(stmt).all()

        for post, user in rows:
            print(post.title, user.email)


if __name__ == "__main__":
    main()
```

Здесь:

```python
select(Post, User)
```

возвращает пары:

```python
(Post, User)
```

---

## 33.3. JOIN через relationship

Можно проще:

```python
stmt = select(Post, User).join(Post.author)
```

SQLAlchemy сам понимает условие связи.

---

# 34. Агрегации: COUNT, GROUP BY

## 34.1. Посчитать пользователей

```python
from sqlalchemy import select, func

from app.database import SessionLocal
from app.models import User


def main() -> None:
    with SessionLocal() as session:
        count = session.scalar(
            select(func.count(User.id))
        )

        print(count)


if __name__ == "__main__":
    main()
```

---

## 34.2. Количество постов у каждого пользователя

```python
from sqlalchemy import select, func

from app.database import SessionLocal
from app.models import User, Post


def main() -> None:
    with SessionLocal() as session:
        stmt = (
            select(User.email, func.count(Post.id))
            .join(Post, Post.user_id == User.id)
            .group_by(User.id)
        )

        rows = session.execute(stmt).all()

        for email, posts_count in rows:
            print(email, posts_count)


if __name__ == "__main__":
    main()
```

---

## 34.3. LEFT JOIN

Если нужно показать пользователей даже без постов:

```python
stmt = (
    select(User.email, func.count(Post.id))
    .outerjoin(Post, Post.user_id == User.id)
    .group_by(User.id)
)
```

---

# 35. Индексы и ограничения

## 35.1. Index

Индекс ускоряет поиск.

Например, если часто ищешь по email:

```python
email: Mapped[str] = mapped_column(
    String(255),
    unique=True,
    index=True,
    nullable=False,
)
```

Но `unique=True` в PostgreSQL и так создаёт уникальный индекс.

---

## 35.2. Явный индекс

```python
from sqlalchemy import Index
```

```python
class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)
    title: Mapped[str] = mapped_column(String(200), nullable=False)
    body: Mapped[str] = mapped_column(Text, nullable=False)

    __table_args__ = (
        Index("ix_posts_title", "title"),
    )
```

---

## 35.3. UniqueConstraint

Если уникальность должна быть по нескольким полям:

```python
from sqlalchemy import UniqueConstraint
```

```python
class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(String(255), nullable=False)

    __table_args__ = (
        UniqueConstraint("email", name="uq_users_email"),
    )
```

---

## 35.4. CheckConstraint

Например, рейтинг должен быть от 1 до 5:

```python
from sqlalchemy import CheckConstraint
```

```python
class Review(Base):
    __tablename__ = "reviews"

    id: Mapped[int] = mapped_column(primary_key=True)
    rating: Mapped[int] = mapped_column(nullable=False)

    __table_args__ = (
        CheckConstraint("rating >= 1 AND rating <= 5", name="ck_rating_range"),
    )
```

---

# 36. PostgreSQL-специфичные типы

SQLAlchemy умеет использовать специальные типы PostgreSQL.

Импорт:

```python
from sqlalchemy.dialects.postgresql import UUID, JSONB, ARRAY
```

---

## 36.1. UUID primary key

Установим модель с UUID:

```python
import uuid

from sqlalchemy.dialects.postgresql import UUID
from sqlalchemy.orm import Mapped, mapped_column

from app.database import Base


class ApiKey(Base):
    __tablename__ = "api_keys"

    id: Mapped[uuid.UUID] = mapped_column(
        UUID(as_uuid=True),
        primary_key=True,
        default=uuid.uuid4,
    )

    name: Mapped[str]
```

Разбор:

```python
UUID(as_uuid=True)
```

значит SQLAlchemy будет работать с Python-объектом `uuid.UUID`, а не со строкой.

```python
default=uuid.uuid4
```

генерирует UUID на стороне Python.

Важно писать именно:

```python
default=uuid.uuid4
```

а не:

```python
default=uuid.uuid4()
```

Потому что `uuid.uuid4()` вызовется один раз при загрузке файла.

---

## 36.2. JSONB

PostgreSQL имеет мощный тип `JSONB`.

```python
from sqlalchemy.dialects.postgresql import JSONB
```

```python
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
        "user_id": 123,
        "source": "web",
    }
)
```

Поиск:

```python
stmt = select(Event).where(
    Event.payload["type"].astext == "user_registered"
)
```

---

## 36.3. ARRAY

```python
from sqlalchemy.dialects.postgresql import ARRAY
from sqlalchemy import String
```

```python
class Article(Base):
    __tablename__ = "articles"

    id: Mapped[int] = mapped_column(primary_key=True)

    tags: Mapped[list[str]] = mapped_column(
        ARRAY(String),
        nullable=False,
        default=list,
    )
```

Важно:

```python
default=list
```

а не:

```python
default=[]
```

Потому что список — изменяемый объект.

---

# 37. Enum

## 37.1. Python Enum

```python
import enum


class PostStatus(enum.Enum):
    DRAFT = "draft"
    PUBLISHED = "published"
    ARCHIVED = "archived"
```

## 37.2. SQLAlchemy Enum

```python
from sqlalchemy import Enum
```

```python
class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)

    status: Mapped[PostStatus] = mapped_column(
        Enum(PostStatus, name="post_status"),
        default=PostStatus.DRAFT,
        nullable=False,
    )
```

Теперь:

```python
post.status = PostStatus.PUBLISHED
```

---

# 38. Даты и время

Для PostgreSQL обычно используют timezone-aware `DateTime`.

```python
from datetime import datetime
from sqlalchemy import DateTime, func
```

```python
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

## Разница `default` и `server_default`

```python
default=...
```

значение создаётся Python-кодом.

```python
server_default=...
```

значение создаётся базой данных.

Для `created_at` часто лучше:

```python
server_default=func.now()
```

---

# 39. Alembic: миграции

`create_all()` подходит для старта, но в реальном проекте используют миграции.

## 39.1. Что такое миграция?

Миграция — это файл с изменениями схемы БД.

Например:

```text
добавить таблицу users
добавить колонку users.name
создать индекс
удалить колонку
```

---

## 39.2. Инициализация Alembic

В корне проекта:

```bash
alembic init alembic
```

Появится:

```text
alembic/
    env.py
    versions/
alembic.ini
```

---

## 39.3. Настроить alembic.ini

Найди строку:

```ini
sqlalchemy.url = driver://user:pass@localhost/dbname
```

Можно оставить пустой, если берём URL из `.env`.

---

## 39.4. Настроить `alembic/env.py`

Открой `alembic/env.py`.

Добавь импорт настроек и моделей:

```python
from app.config import settings
from app.database import Base
from app import models
```

Найди:

```python
target_metadata = None
```

Замени на:

```python
target_metadata = Base.metadata
```

Найди функцию `run_migrations_online()` и настрой URL:

```python
def run_migrations_online() -> None:
    configuration = config.get_section(config.config_ini_section)
    configuration["sqlalchemy.url"] = settings.DATABASE_URL

    connectable = engine_from_config(
        configuration,
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata,
        )

        with context.begin_transaction():
            context.run_migrations()
```

---

## 39.5. Создать миграцию

```bash
alembic revision --autogenerate -m "create initial tables"
```

Alembic сравнит модели SQLAlchemy с базой и создаст файл миграции.

---

## 39.6. Применить миграции

```bash
alembic upgrade head
```

---

## 39.7. Откатить последнюю миграцию

```bash
alembic downgrade -1
```

---

## 39.8. Посмотреть текущую версию

```bash
alembic current
```

---

## 39.9. Важно про autogenerate

`--autogenerate` удобен, но его надо проверять.

Alembic не всегда идеально понимает:

- переименование колонок;
- сложные изменения enum;
- кастомные SQL;
- некоторые изменения constraints.

Всегда открывай файл миграции и проверяй.

---

# 40. Bulk UPDATE и DELETE

Иногда нужно обновить много строк без загрузки объектов.

## 40.1. Bulk update

```python
from sqlalchemy import update

stmt = (
    update(User)
    .where(User.email.ilike("%@old-domain.com"))
    .values(name="Old domain user")
)

with SessionLocal() as session:
    with session.begin():
        session.execute(stmt)
```

---

## 40.2. Bulk delete

```python
from sqlalchemy import delete

stmt = delete(User).where(User.name.is_(None))

with SessionLocal() as session:
    with session.begin():
        session.execute(stmt)
```

---

## Важно

Bulk-операции обходят часть ORM-механизмов.

Например, они не всегда синхронизируют уже загруженные объекты в сессии.

Для junior-уровня правило простое:

```text
Если работаешь с конкретным объектом — загружай объект и меняй его.
Если массовая операция — используй update/delete.
```

---

# 41. Обработка ошибок

## 41.1. IntegrityError

Если нарушена уникальность email:

```python
from sqlalchemy.exc import IntegrityError

try:
    with SessionLocal() as session:
        with session.begin():
            user = User(email="ivan@example.com")
            session.add(user)

except IntegrityError:
    print("Такой email уже существует")
```

---

## 41.2. Почему ошибка возникает на commit/flush?

Когда ты делаешь:

```python
session.add(user)
```

SQL ещё может не выполниться.

Ошибка может возникнуть на:

```python
session.flush()
```

или:

```python
session.commit()
```

---

# 42. Flush

## 42.1. Что такое `flush`

`flush` отправляет изменения в базу, но не делает `commit`.

Пример:

```python
with SessionLocal() as session:
    with session.begin():
        user = User(email="flush@example.com")
        session.add(user)

        session.flush()

        print(user.id)
```

После `flush()` у пользователя уже есть `id`.

Но если потом будет ошибка, вся транзакция откатится.

---

## 42.2. Когда нужен flush

Например, нужно создать пользователя, получить его `id`, и использовать для другой записи.

Хотя чаще можно использовать relationship:

```python
post = Post(author=user, title="...", body="...")
```

Но иногда `flush()` полезен.

---

# 43. Refresh

`refresh()` перезагружает объект из базы.

```python
session.refresh(user)
```

Например, если база сама заполнила поля:

```python
created_at
```

и ты хочешь сразу увидеть актуальное значение.

---

# 44. Expire on commit

Мы в `sessionmaker` поставили:

```python
expire_on_commit=False
```

По умолчанию SQLAlchemy после `commit()` «протухляет» объекты.

То есть при обращении к полю после коммита может быть дополнительный запрос.

Для веб-приложений часто удобно ставить:

```python
expire_on_commit=False
```

Чтобы после коммита спокойно читать:

```python
user.id
user.email
```

---

# 45. Репозиторий: простой слой доступа к данным

В junior-проектах часто делают функции для работы с БД.

Например `app/repositories/users.py`:

```python
from sqlalchemy import select
from sqlalchemy.orm import Session

from app.models import User


def create_user(
    session: Session,
    *,
    email: str,
    name: str | None = None,
) -> User:
    user = User(email=email, name=name)
    session.add(user)
    session.flush()
    return user


def get_user_by_id(session: Session, user_id: int) -> User | None:
    return session.get(User, user_id)


def get_user_by_email(session: Session, email: str) -> User | None:
    return session.scalar(
        select(User).where(User.email == email)
    )


def list_users(
    session: Session,
    *,
    limit: int = 100,
    offset: int = 0,
) -> list[User]:
    return list(
        session.scalars(
            select(User)
            .order_by(User.id)
            .limit(limit)
            .offset(offset)
        )
    )
```

Использование:

```python
from app.database import SessionLocal
from app.repositories.users import create_user, list_users


def main() -> None:
    with SessionLocal() as session:
        with session.begin():
            user = create_user(
                session,
                email="repo@example.com",
                name="Repo User",
            )

        users = list_users(session)

        for user in users:
            print(user.email)


if __name__ == "__main__":
    main()
```

---

# 46. Async SQLAlchemy 2

SQLAlchemy 2 поддерживает async.

Для PostgreSQL установи:

```bash
pip install "psycopg[binary]" sqlalchemy
```

Строка подключения:

```env
ASYNC_DATABASE_URL=postgresql+psycopg://app:app_password@localhost:5432/app_db
```

Для psycopg v3 async используется тот же драйвер `psycopg`, но через async engine.

---

## 46.1. Async database.py

```python
from sqlalchemy.ext.asyncio import (
    AsyncAttrs,
    AsyncSession,
    async_sessionmaker,
    create_async_engine,
)
from sqlalchemy.orm import DeclarativeBase

from app.config import settings


async_engine = create_async_engine(
    settings.ASYNC_DATABASE_URL,
    echo=True,
)


class AsyncBase(AsyncAttrs, DeclarativeBase):
    pass


AsyncSessionLocal = async_sessionmaker(
    bind=async_engine,
    expire_on_commit=False,
    autoflush=False,
)
```

---

## 46.2. Async select

```python
from sqlalchemy import select

from app.async_database import AsyncSessionLocal
from app.models import User


async def main() -> None:
    async with AsyncSessionLocal() as session:
        result = await session.scalars(
            select(User).where(User.email == "ivan@example.com")
        )

        user = result.first()

        if user:
            print(user.email)
```

---

## 46.3. Async transaction

```python
async with AsyncSessionLocal() as session:
    async with session.begin():
        user = User(email="async@example.com")
        session.add(user)
```

---

## 46.4. Важно про lazy loading в async

В async SQLAlchemy ленивые загрузки могут быть неудобными.

Лучше заранее использовать:

```python
selectinload(...)
```

Например:

```python
stmt = select(User).options(selectinload(User.posts))
users = (await session.scalars(stmt)).all()
```

---

# 47. Частые ошибки новичков

## 47.1. Использовать старый `session.query`

Старый стиль:

```python
session.query(User).filter(User.id == 1).first()
```

В SQLAlchemy 2 лучше так:

```python
session.scalar(
    select(User).where(User.id == 1)
)
```

---

## 47.2. Забыть commit

```python
session.add(user)
```

не означает, что данные уже навсегда записаны.

Нужно:

```python
session.commit()
```

или:

```python
with session.begin():
    session.add(user)
```

---

## 47.3. Использовать один Session глобально

Плохо:

```python
session = SessionLocal()
```

и потом использовать её везде.

Хорошо:

```python
with SessionLocal() as session:
    ...
```

Сессия должна жить недолго.

Обычно:

```text
один request в веб-приложении = одна session
```

---

## 47.4. Путать `scalars` и `execute`

Если:

```python
select(User)
```

то удобно:

```python
users = session.scalars(select(User)).all()
```

Если:

```python
select(User.email, User.name)
```

то нужно:

```python
rows = session.execute(select(User.email, User.name)).all()
```

---

## 47.5. Не понимать `None`

Если поле nullable:

```python
name: Mapped[str | None] = mapped_column(nullable=True)
```

Если поле обязательное:

```python
email: Mapped[str] = mapped_column(nullable=False)
```

---

# 48. Мини-шпаргалка SQLAlchemy 2

## Создать объект

```python
with SessionLocal() as session:
    with session.begin():
        user = User(email="test@example.com")
        session.add(user)
```

## Получить по id

```python
user = session.get(User, 1)
```

## Получить один по условию

```python
user = session.scalar(
    select(User).where(User.email == "test@example.com")
)
```

## Получить список

```python
users = session.scalars(
    select(User).order_by(User.id)
).all()
```

## Обновить

```python
with session.begin():
    user = session.get(User, 1)
    user.name = "New name"
```

## Удалить

```python
with session.begin():
    user = session.get(User, 1)
    session.delete(user)
```

## JOIN

```python
rows = session.execute(
    select(Post, User).join(Post.author)
).all()
```

## Загрузить связь без N+1

```python
users = session.scalars(
    select(User).options(selectinload(User.posts))
).all()
```

---

# 49. Что junior должен уверенно понимать

После изучения этого материала ты должен понимать:

## База

- что такое таблица;
- что такое колонка;
- что такое primary key;
- что такое foreign key;
- что такое index;
- что такое unique constraint;
- что такое transaction.

## SQLAlchemy ORM

- как создать `engine`;
- что такое `Session`;
- что такое `Base`;
- как описывать модели через `Mapped` и `mapped_column`;
- как создавать записи;
- как получать записи;
- как обновлять записи;
- как удалять записи;
- как делать фильтрацию;
- как делать сортировку;
- как делать пагинацию.

## Связи

- one-to-many;
- many-to-one;
- many-to-many;
- `relationship`;
- `back_populates`;
- `ForeignKey`;
- `selectinload`;
- `joinedload`;
- проблема N+1.

## PostgreSQL

- UUID;
- JSONB;
- ARRAY;
- Enum;
- DateTime;
- server_default;
- индексы.

## Alembic

- зачем нужны миграции;
- как создать миграцию;
- как применить миграцию;
- почему надо проверять autogenerate.

---

# 50. Рекомендуемый стиль для SQLAlchemy 2

Используй так:

```python
stmt = select(User).where(User.email == email)
user = session.scalar(stmt)
```

А не так:

```python
user = session.query(User).filter(User.email == email).first()
```

Используй так:

```python
class User(Base):
    id: Mapped[int] = mapped_column(primary_key=True)
```

А не старый стиль:

```python
id = Column(Integer, primary_key=True)
```

Хотя старый стиль ещё может работать, для SQLAlchemy 2 лучше использовать современный типизированный ORM.

---

# 51. Полный минимальный пример

## `app/database.py`

```python
from sqlalchemy import create_engine
from sqlalchemy.orm import DeclarativeBase, sessionmaker

from app.config import settings


engine = create_engine(
    settings.DATABASE_URL,
    echo=True,
)


class Base(DeclarativeBase):
    pass


SessionLocal = sessionmaker(
    bind=engine,
    autoflush=False,
    expire_on_commit=False,
)
```

## `app/models.py`

```python
from datetime import datetime
from typing import List

from sqlalchemy import ForeignKey, String, Text, func
from sqlalchemy.orm import Mapped, mapped_column, relationship

from app.database import Base


class User(Base):
    __tablename__ = "users"

    id: Mapped[int] = mapped_column(primary_key=True)

    email: Mapped[str] = mapped_column(
        String(255),
        unique=True,
        nullable=False,
    )

    name: Mapped[str | None] = mapped_column(
        String(100),
        nullable=True,
    )

    created_at: Mapped[datetime] = mapped_column(
        server_default=func.now(),
        nullable=False,
    )

    posts: Mapped[List["Post"]] = relationship(
        back_populates="author",
    )


class Post(Base):
    __tablename__ = "posts"

    id: Mapped[int] = mapped_column(primary_key=True)

    title: Mapped[str] = mapped_column(
        String(200),
        nullable=False,
    )

    body: Mapped[str] = mapped_column(
        Text,
        nullable=False,
    )

    user_id: Mapped[int] = mapped_column(
        ForeignKey("users.id"),
        nullable=False,
    )

    created_at: Mapped[datetime] = mapped_column(
        server_default=func.now(),
        nullable=False,
    )

    author: Mapped["User"] = relationship(
        back_populates="posts",
    )
```

## `app/main.py`

```python
from sqlalchemy import select
from sqlalchemy.orm import selectinload

from app.database import Base, engine, SessionLocal
from app.models import User, Post


def create_tables() -> None:
    Base.metadata.create_all(bind=engine)


def create_data() -> None:
    with SessionLocal() as session:
        with session.begin():
            user = User(
                email="junior@example.com",
                name="Junior",
            )

            post = Post(
                title="Мой первый пост",
                body="Я изучаю SQLAlchemy 2",
                author=user,
            )

            session.add(user)


def read_data() -> None:
    with SessionLocal() as session:
        users = session.scalars(
            select(User).options(selectinload(User.posts))
        ).all()

        for user in users:
            print(user.email)

            for post in user.posts:
                print("  ", post.title)


def main() -> None:
    create_tables()
    create_data()
    read_data()


if __name__ == "__main__":
    main()
```

---

# 52. Самые важные правила

1. **Не используй `session.query` в новом коде.**
2. **Используй `select()` из SQLAlchemy 2.**
3. **Описывай модели через `Mapped` и `mapped_column`.**
4. **Сессия должна жить недолго.**
5. **Для транзакций используй `with session.begin()`.**
6. **Для связей используй `relationship` + `ForeignKey`.**
7. **Для one-to-many часто используй `selectinload`.**
8. **Для many-to-one часто используй `joinedload`.**
9. **В реальных проектах используй Alembic.**
10. **Всегда смотри SQL через `echo=True`, пока учишься.**

---

Если ты уверенно понимаешь всё выше, ты уже можешь работать с SQLAlchemy 2 на уровне уверенного junior/junior+.
