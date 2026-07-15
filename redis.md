# Redis + асинхронный Python с нуля до уровня уверенного Junior+

Это подробное практическое руководство по Redis и работе с ним из асинхронного Python.

Мы будем разбирать всё с нуля:

- что такое Redis;
- зачем он нужен;
- как его установить;
- как устроены ключи, значения и TTL;
- какие есть типы данных;
- как работать через `redis-cli`;
- как работать из Python асинхронно;
- как делать кэширование;
- как делать очереди;
- как делать Pub/Sub;
- как делать Streams;
- как использовать транзакции;
- как писать безопасный и нормальный junior+ код;
- какие ошибки чаще всего делают новички.

---

# 1. Что такое Redis

**Redis** — это очень быстрая in-memory база данных формата `key-value`.

Проще:

> Redis хранит данные в оперативной памяти и позволяет очень быстро читать и записывать их по ключам.

Например:

```text
ключ: user:1:name
значение: "Алексей"
```

Redis часто используют как:

1. **Кэш**
2. **Брокер сообщений**
3. **Очередь задач**
4. **Хранилище сессий**
5. **Счётчики**
6. **Rate limiter**
7. **Pub/Sub систему**
8. **Временное хранилище токенов**
9. **Хранилище online-статусов**
10. **Быструю структуру данных**

---

# 2. Чем Redis отличается от обычной базы данных

Например, PostgreSQL хранит данные на диске и хорошо подходит для постоянного хранения.

Redis в основном хранит данные в памяти, поэтому он очень быстрый.

## PostgreSQL

Подходит для:

- пользователей;
- заказов;
- платежей;
- товаров;
- важных данных;
- сложных SQL-запросов.

## Redis

Подходит для:

- кэша;
- временных данных;
- токенов;
- очередей;
- счётчиков;
- блокировок;
- данных с TTL;
- быстрых операций.

## Важная мысль

Redis **не заменяет PostgreSQL** в большинстве проектов.

Часто архитектура такая:

```text
FastAPI / Django / Backend
        |
        |--- PostgreSQL — основные данные
        |
        |--- Redis — кэш, очереди, сессии, rate limit
```

---

# 3. Почему Redis такой быстрый

Главные причины:

1. Данные лежат в RAM.
2. Redis использует простые структуры данных.
3. Многие операции выполняются за `O(1)` или `O(log n)`.
4. Redis однопоточный для исполнения команд, поэтому внутри нет сложных блокировок.
5. Поддерживает неблокирующий I/O.

---

# 4. Установка Redis

## Вариант 1. Через Docker

Самый простой вариант.

```bash
docker run --name redis-dev -p 6379:6379 -d redis:7
```

Проверить:

```bash
docker ps
```

Подключиться:

```bash
docker exec -it redis-dev redis-cli
```

Если всё хорошо, увидишь:

```text
127.0.0.1:6379>
```

Проверка:

```redis
PING
```

Ответ:

```text
PONG
```

---

## Вариант 2. Через docker-compose

Создай файл `docker-compose.yml`:

```yaml
services:
  redis:
    image: redis:7
    container_name: redis-dev
    ports:
      - "6379:6379"
    restart: unless-stopped
```

Запуск:

```bash
docker compose up -d
```

Подключение:

```bash
docker exec -it redis-dev redis-cli
```

---

# 5. Базовые команды Redis

## SET

Сохраняет значение по ключу.

```redis
SET name "Alex"
```

## GET

Получает значение по ключу.

```redis
GET name
```

Ответ:

```text
"Alex"
```

## DEL

Удаляет ключ.

```redis
DEL name
```

## EXISTS

Проверяет, существует ли ключ.

```redis
EXISTS name
```

Ответ:

```text
1
```

или:

```text
0
```

## KEYS

Показывает ключи по шаблону.

```redis
KEYS *
```

Но важно:

> `KEYS *` нельзя использовать на production с большим количеством данных, потому что команда может заблокировать Redis.

Для production лучше использовать `SCAN`.

---

# 6. Что такое key-value

Redis хранит данные по принципу:

```text
ключ -> значение
```

Пример:

```redis
SET user:1:name "Ivan"
GET user:1:name
```

Ключ:

```text
user:1:name
```

Значение:

```text
Ivan
```

---

# 7. Как правильно называть ключи

В Redis нет строгой схемы, поэтому важно самому договориться о стиле.

Обычно используют такой формат:

```text
entity:id:field
```

Примеры:

```text
user:1:name
user:1:email
user:1:session
product:15:views
order:1001:status
```

Для кэша:

```text
cache:user:1
cache:product:15
cache:feed:main
```

Для rate limit:

```text
rate_limit:user:1:login
rate_limit:ip:127.0.0.1
```

Для блокировок:

```text
lock:order:1001
lock:payment:555
```

---

# 8. TTL — время жизни ключа

TTL — это время, через которое ключ автоматически удалится.

Это одна из важнейших возможностей Redis.

## Установить ключ на 10 секунд

```redis
SET token "abc123" EX 10
```

Через 10 секунд ключ исчезнет.

## Проверить TTL

```redis
TTL token
```

Ответ:

```text
7
```

Это значит, что ключ живёт ещё 7 секунд.

## Установить TTL отдельно

```redis
SET code "123456"
EXPIRE code 60
```

Ключ `code` удалится через 60 секунд.

---

# 9. Основные типы данных Redis

Redis — это не просто строка по ключу.

Он поддерживает разные структуры:

1. String
2. List
3. Hash
4. Set
5. Sorted Set
6. Bitmap
7. HyperLogLog
8. Geospatial
9. Stream

Junior+ должен хорошо знать первые 5 и понимать, зачем нужны Streams.

---

# 10. String

String — самый простой тип.

Это может быть:

- строка;
- число;
- JSON;
- бинарные данные.

## Пример

```redis
SET user:1:name "Ivan"
GET user:1:name
```

## Счётчик

```redis
SET views:post:1 0
INCR views:post:1
INCR views:post:1
GET views:post:1
```

Ответ:

```text
"2"
```

## Увеличение на конкретное число

```redis
INCRBY views:post:1 10
```

## Уменьшение

```redis
DECR views:post:1
DECRBY views:post:1 5
```

## Где использовать String

- кэш JSON;
- токены;
- счётчики;
- временные коды;
- простые значения.

---

# 11. Hash

Hash — это словарь внутри Redis.

Пример:

```text
user:1 -> {
    name: "Ivan",
    age: "25",
    email: "ivan@example.com"
}
```

## Создать hash

```redis
HSET user:1 name "Ivan" age "25" email "ivan@example.com"
```

## Получить одно поле

```redis
HGET user:1 name
```

## Получить весь hash

```redis
HGETALL user:1
```

## Проверить поле

```redis
HEXISTS user:1 email
```

## Удалить поле

```redis
HDEL user:1 age
```

## Увеличить числовое поле

```redis
HINCRBY user:1 login_count 1
```

## Где использовать Hash

- профиль пользователя;
- настройки;
- объект с полями;
- статистика.

---

# 12. List

List — это список строк.

Redis List похож на массив, но чаще используется как очередь или стек.

## Добавить в начало

```redis
LPUSH tasks "task1"
LPUSH tasks "task2"
```

Список:

```text
task2, task1
```

## Добавить в конец

```redis
RPUSH tasks "task3"
```

Список:

```text
task2, task1, task3
```

## Достать из начала

```redis
LPOP tasks
```

## Достать из конца

```redis
RPOP tasks
```

## Посмотреть список

```redis
LRANGE tasks 0 -1
```

## Очередь FIFO

FIFO — first in, first out.

Первый вошёл — первый вышел.

Добавляем справа:

```redis
RPUSH queue "task1"
RPUSH queue "task2"
```

Забираем слева:

```redis
LPOP queue
```

Получим:

```text
task1
```

## Блокирующее ожидание

```redis
BLPOP queue 0
```

Команда будет ждать, пока в списке появится элемент.

Где использовать List:

- простые очереди;
- логи;
- недавние действия;
- список последних элементов.

---

# 13. Set

Set — множество уникальных значений.

В Set не может быть дублей.

## Добавить элементы

```redis
SADD online_users 1
SADD online_users 2
SADD online_users 1
```

Пользователь `1` добавится только один раз.

## Получить все элементы

```redis
SMEMBERS online_users
```

## Проверить наличие

```redis
SISMEMBER online_users 1
```

## Удалить

```redis
SREM online_users 1
```

## Количество

```redis
SCARD online_users
```

## Пересечение множеств

```redis
SINTER set1 set2
```

## Объединение

```redis
SUNION set1 set2
```

Где использовать Set:

- уникальные пользователи;
- лайки;
- теги;
- online users;
- множество прав доступа.

---

# 14. Sorted Set

Sorted Set — это множество уникальных элементов, но у каждого элемента есть score.

Score — число, по которому Redis сортирует элементы.

Пример рейтинга:

```text
Ivan -> 100
Anna -> 250
Oleg -> 150
```

## Добавить элементы

```redis
ZADD leaderboard 100 Ivan
ZADD leaderboard 250 Anna
ZADD leaderboard 150 Oleg
```

## Получить по возрастанию

```redis
ZRANGE leaderboard 0 -1 WITHSCORES
```

## Получить по убыванию

```redis
ZREVRANGE leaderboard 0 -1 WITHSCORES
```

Ответ:

```text
Anna 250
Oleg 150
Ivan 100
```

## Увеличить score

```redis
ZINCRBY leaderboard 50 Ivan
```

## Получить позицию

```redis
ZREVRANK leaderboard Ivan
```

Где использовать Sorted Set:

- рейтинги;
- топ пользователей;
- отложенные задачи;
- сортировка по времени;
- priority queue.

---

# 15. Pub/Sub

Pub/Sub — механизм публикации сообщений.

Есть:

- publisher — отправляет сообщение;
- subscriber — слушает канал.

## Подписка

В одном терминале:

```redis
SUBSCRIBE news
```

## Публикация

В другом терминале:

```redis
PUBLISH news "Hello"
```

Подписчик получит:

```text
Hello
```

## Важно

Pub/Sub не хранит сообщения.

Если подписчика не было онлайн — он сообщение потеряет.

Где использовать Pub/Sub:

- уведомления онлайн-пользователям;
- websocket fanout;
- системные события;
- простой realtime.

Если нужно хранить сообщения — лучше использовать Streams.

---

# 16. Streams

Redis Streams — это лог событий.

В отличие от Pub/Sub, сообщения сохраняются.

Stream похож на очередь с историей.

## Добавить сообщение

```redis
XADD orders * user_id 1 amount 500
```

`*` значит, что Redis сам создаст ID.

## Прочитать сообщения

```redis
XREAD COUNT 10 STREAMS orders 0
```

## Consumer Groups

Consumer Group позволяет нескольким воркерам обрабатывать сообщения совместно.

Создать группу:

```redis
XGROUP CREATE orders order_workers $ MKSTREAM
```

Читать из группы:

```redis
XREADGROUP GROUP order_workers worker1 COUNT 1 BLOCK 5000 STREAMS orders >
```

Подтвердить обработку:

```redis
XACK orders order_workers message_id
```

Где использовать Streams:

- очереди событий;
- обработка заказов;
- аудит;
- фоновые задачи;
- микросервисные события.

---

# 17. Установка Python-клиента Redis

Раньше часто использовали библиотеку `aioredis`.

Сейчас `aioredis` объединён с официальной библиотекой `redis-py`.

Для асинхронной работы используется:

```python
import redis.asyncio as redis
```

Установка:

```bash
pip install redis
```

Проверка версии:

```bash
pip show redis
```

Желательно использовать современную версию:

```bash
pip install "redis>=5"
```

---

# 18. Первый асинхронный пример на Python

Создай файл `main.py`:

```python
import asyncio
import redis.asyncio as redis


async def main():
    client = redis.Redis(
        host="localhost",
        port=6379,
        db=0,
        decode_responses=True,
    )

    await client.set("name", "Ivan")
    value = await client.get("name")

    print(value)

    await client.aclose()


asyncio.run(main())
```

Запуск:

```bash
python main.py
```

Результат:

```text
Ivan
```

---

# 19. Что значит async/await

Обычный код выполняется последовательно.

Асинхронный код позволяет не блокировать программу, пока мы ждём ответ от Redis.

Например:

```python
value = await client.get("name")
```

Здесь программа говорит:

> Я отправила запрос в Redis. Пока Redis отвечает, event loop может выполнять другие задачи.

Это особенно важно в FastAPI, aiohttp, aiogram и других async-фреймворках.

---

# 20. Подключение к Redis через URL

Часто в проектах используют URL:

```python
import redis.asyncio as redis

client = redis.from_url(
    "redis://localhost:6379/0",
    decode_responses=True,
)
```

Если Redis с паролем:

```text
redis://:password@localhost:6379/0
```

Пример:

```python
client = redis.from_url(
    "redis://:my_password@localhost:6379/0",
    decode_responses=True,
)
```

---

# 21. decode_responses=True

По умолчанию Redis возвращает bytes.

Пример без `decode_responses=True`:

```python
b'Ivan'
```

С `decode_responses=True`:

```python
'Ivan'
```

Для большинства обычных проектов удобнее использовать:

```python
decode_responses=True
```

---

# 22. Работа со String из Python

```python
import asyncio
import redis.asyncio as redis


async def main():
    r = redis.Redis(decode_responses=True)

    await r.set("user:1:name", "Ivan")

    name = await r.get("user:1:name")
    print(name)

    await r.delete("user:1:name")

    exists = await r.exists("user:1:name")
    print(exists)

    await r.aclose()


asyncio.run(main())
```

---

# 23. TTL из Python

```python
import asyncio
import redis.asyncio as redis


async def main():
    r = redis.Redis(decode_responses=True)

    await r.set("auth:code:1", "123456", ex=60)

    ttl = await r.ttl("auth:code:1")
    print(ttl)

    await r.aclose()


asyncio.run(main())
```

`ex=60` означает TTL 60 секунд.

Также есть `px`, если нужно в миллисекундах:

```python
await r.set("key", "value", px=5000)
```

Это 5000 миллисекунд, то есть 5 секунд.

---

# 24. NX и XX

## NX

Установить ключ только если его ещё нет.

```python
result = await r.set("lock:1", "locked", nx=True, ex=10)
```

Если ключа не было — вернёт `True`.

Если ключ уже есть — вернёт `None`.

## XX

Установить ключ только если он уже существует.

```python
result = await r.set("some:key", "value", xx=True)
```

---

# 25. Hash из Python

```python
import asyncio
import redis.asyncio as redis


async def main():
    r = redis.Redis(decode_responses=True)

    await r.hset(
        "user:1",
        mapping={
            "name": "Ivan",
            "age": "25",
            "email": "ivan@example.com",
        },
    )

    name = await r.hget("user:1", "name")
    print(name)

    user = await r.hgetall("user:1")
    print(user)

    await r.hincrby("user:1", "login_count", 1)

    user = await r.hgetall("user:1")
    print(user)

    await r.aclose()


asyncio.run(main())
```

Результат примерно:

```python
Ivan
{'name': 'Ivan', 'age': '25', 'email': 'ivan@example.com'}
{'name': 'Ivan', 'age': '25', 'email': 'ivan@example.com', 'login_count': '1'}
```

Обрати внимание:

> Redis хранит значения как строки, поэтому `age` будет `'25'`, а не `25`.

---

# 26. List из Python

```python
import asyncio
import redis.asyncio as redis


async def main():
    r = redis.Redis(decode_responses=True)

    await r.delete("tasks")

    await r.rpush("tasks", "task1")
    await r.rpush("tasks", "task2")
    await r.rpush("tasks", "task3")

    tasks = await r.lrange("tasks", 0, -1)
    print(tasks)

    first_task = await r.lpop("tasks")
    print(first_task)

    tasks = await r.lrange("tasks", 0, -1)
    print(tasks)

    await r.aclose()


asyncio.run(main())
```

---

# 27. Простая очередь на List

Producer добавляет задачи.

```python
import asyncio
import redis.asyncio as redis


async def producer():
    r = redis.Redis(decode_responses=True)

    for i in range(5):
        task = f"send_email:{i}"
        await r.rpush("queue:emails", task)
        print("Added:", task)

    await r.aclose()


asyncio.run(producer())
```

Consumer забирает задачи:

```python
import asyncio
import redis.asyncio as redis


async def consumer():
    r = redis.Redis(decode_responses=True)

    while True:
        result = await r.blpop("queue:emails", timeout=0)

        queue_name, task = result
        print("Processing:", task)

        await asyncio.sleep(1)
        print("Done:", task)


asyncio.run(consumer())
```

`BLPOP` ждёт, пока появится элемент.

---

# 28. Set из Python

```python
import asyncio
import redis.asyncio as redis


async def main():
    r = redis.Redis(decode_responses=True)

    await r.delete("online_users")

    await r.sadd("online_users", "1")
    await r.sadd("online_users", "2")
    await r.sadd("online_users", "1")

    users = await r.smembers("online_users")
    print(users)

    is_online = await r.sismember("online_users", "1")
    print(is_online)

    count = await r.scard("online_users")
    print(count)

    await r.srem("online_users", "1")

    await r.aclose()


asyncio.run(main())
```

---

# 29. Sorted Set из Python

```python
import asyncio
import redis.asyncio as redis


async def main():
    r = redis.Redis(decode_responses=True)

    await r.delete("leaderboard")

    await r.zadd(
        "leaderboard",
        {
            "Ivan": 100,
            "Anna": 250,
            "Oleg": 150,
        },
    )

    top = await r.zrevrange("leaderboard", 0, -1, withscores=True)
    print(top)

    await r.zincrby("leaderboard", 50, "Ivan")

    top = await r.zrevrange("leaderboard", 0, -1, withscores=True)
    print(top)

    rank = await r.zrevrank("leaderboard", "Ivan")
    print(rank)

    await r.aclose()


asyncio.run(main())
```

---

# 30. JSON в Redis

Redis сам по себе не хранит Python-словарь как объект.

Обычно делают так:

```python
import json
```

Сохраняют словарь как JSON-строку.

```python
import asyncio
import json
import redis.asyncio as redis


async def main():
    r = redis.Redis(decode_responses=True)

    user = {
        "id": 1,
        "name": "Ivan",
        "age": 25,
    }

    await r.set("cache:user:1", json.dumps(user), ex=300)

    raw = await r.get("cache:user:1")

    if raw is not None:
        user_from_cache = json.loads(raw)
        print(user_from_cache)

    await r.aclose()


asyncio.run(main())
```

Важно:

```python
json.dumps()
```

превращает Python-объект в строку.

```python
json.loads()
```

превращает строку обратно в Python-объект.

---

# 31. Кэширование: главная идея

Кэш нужен, чтобы не ходить каждый раз в медленную базу.

Обычная схема:

```text
1. Проверяем Redis
2. Если данные есть — возвращаем
3. Если данных нет — идём в PostgreSQL/API
4. Кладём данные в Redis
5. Возвращаем результат
```

Это называется **cache-aside**.

---

# 32. Пример cache-aside

Допустим, у нас есть медленная функция:

```python
async def get_user_from_database(user_id: int) -> dict:
    await asyncio.sleep(2)

    return {
        "id": user_id,
        "name": "Ivan",
        "age": 25,
    }
```

Теперь добавим Redis-кэш:

```python
import asyncio
import json
import redis.asyncio as redis


async def get_user_from_database(user_id: int) -> dict:
    print("Getting from database...")
    await asyncio.sleep(2)

    return {
        "id": user_id,
        "name": "Ivan",
        "age": 25,
    }


async def get_user(user_id: int, r: redis.Redis) -> dict:
    cache_key = f"cache:user:{user_id}"

    cached = await r.get(cache_key)

    if cached is not None:
        print("Getting from cache...")
        return json.loads(cached)

    user = await get_user_from_database(user_id)

    await r.set(
        cache_key,
        json.dumps(user),
        ex=300,
    )

    return user


async def main():
    r = redis.Redis(decode_responses=True)

    user = await get_user(1, r)
    print(user)

    user = await get_user(1, r)
    print(user)

    await r.aclose()


asyncio.run(main())
```

Первый вызов будет 2 секунды.

Второй вызов будет почти мгновенным.

---

# 33. Инвалидация кэша

Инвалидация — это удаление или обновление кэша, когда данные изменились.

Например, пользователь изменил имя.

Плохой вариант:

```text
В базе имя уже новое, а в Redis всё ещё старое.
```

Правильный вариант:

```python
async def update_user_name(user_id: int, new_name: str, r: redis.Redis):
    # 1. Обновили в базе
    # await update_user_in_database(user_id, new_name)

    # 2. Удалили кэш
    await r.delete(f"cache:user:{user_id}")
```

Обычно проще удалить кэш, чем пытаться обновлять его вручную.

---

# 34. Cache stampede

Проблема:

1. Кэш истёк.
2. Пришло 1000 запросов.
3. Все 1000 пошли в базу.
4. База перегрузилась.

Решения:

1. Блокировка на пересоздание кэша.
2. Разный TTL с небольшим random.
3. Фоновое обновление кэша.
4. Использовать stale cache.

Простой вариант с random TTL:

```python
import random

ttl = 300 + random.randint(0, 60)
await r.set(cache_key, value, ex=ttl)
```

---

# 35. Pipeline

Pipeline позволяет отправить несколько команд Redis одним пакетом.

Без pipeline:

```text
команда -> ответ
команда -> ответ
команда -> ответ
```

С pipeline:

```text
команда
команда
команда
-> ответы
```

Это быстрее при большом количестве команд.

Пример:

```python
import asyncio
import redis.asyncio as redis


async def main():
    r = redis.Redis(decode_responses=True)

    pipe = r.pipeline()

    pipe.set("user:1:name", "Ivan")
    pipe.set("user:2:name", "Anna")
    pipe.get("user:1:name")
    pipe.get("user:2:name")

    results = await pipe.execute()

    print(results)

    await r.aclose()


asyncio.run(main())
```

Результат:

```python
[True, True, 'Ivan', 'Anna']
```

---

# 36. Транзакции Redis

Redis поддерживает транзакции через:

```redis
MULTI
EXEC
```

Команды в транзакции выполняются последовательно и атомарно.

В Python:

```python
pipe = r.pipeline(transaction=True)
```

Пример:

```python
import asyncio
import redis.asyncio as redis


async def main():
    r = redis.Redis(decode_responses=True)

    pipe = r.pipeline(transaction=True)

    pipe.incr("counter")
    pipe.incr("counter")
    pipe.get("counter")

    result = await pipe.execute()
    print(result)

    await r.aclose()


asyncio.run(main())
```

---

# 37. WATCH — оптимистическая блокировка

`WATCH` используется, когда нужно:

1. прочитать значение;
2. принять решение;
3. записать новое значение;
4. но только если за это время ключ никто не изменил.

Пример: списание денег.

```python
import asyncio
from redis.exceptions import WatchError
import redis.asyncio as redis


async def withdraw(r: redis.Redis, user_id: int, amount: int):
    key = f"user:{user_id}:balance"

    while True:
        try:
            async with r.pipeline() as pipe:
                await pipe.watch(key)

                balance_raw = await pipe.get(key)
                balance = int(balance_raw or 0)

                if balance < amount:
                    await pipe.unwatch()
                    return False

                pipe.multi()
                pipe.set(key, balance - amount)

                await pipe.execute()
                return True

        except WatchError:
            continue


async def main():
    r = redis.Redis(decode_responses=True)

    await r.set("user:1:balance", 100)

    result = await withdraw(r, 1, 30)
    print(result)

    balance = await r.get("user:1:balance")
    print(balance)

    await r.aclose()


asyncio.run(main())
```

Если кто-то изменит баланс между `WATCH` и `EXEC`, Redis отменит транзакцию.

---

# 38. Атомарность команд

Многие команды Redis атомарны сами по себе.

Например:

```redis
INCR counter
```

Если 100 клиентов одновременно сделают `INCR`, Redis корректно увеличит счётчик 100 раз.

То есть не нужно делать:

```python
value = await r.get("counter")
value += 1
await r.set("counter", value)
```

Это опасно при конкурентности.

Правильно:

```python
await r.incr("counter")
```

---

# 39. Rate limiter на Redis

Rate limiter ограничивает количество действий.

Например:

> Пользователь может сделать максимум 5 запросов за 60 секунд.

Простой вариант:

```python
import asyncio
import redis.asyncio as redis


async def is_allowed(r: redis.Redis, user_id: int) -> bool:
    key = f"rate_limit:user:{user_id}"

    current = await r.incr(key)

    if current == 1:
        await r.expire(key, 60)

    return current <= 5


async def main():
    r = redis.Redis(decode_responses=True)

    for i in range(10):
        allowed = await is_allowed(r, 1)
        print(i, allowed)

    await r.aclose()


asyncio.run(main())
```

Как работает:

1. Первый запрос создаёт ключ и ставит TTL 60 секунд.
2. Каждый запрос увеличивает счётчик.
3. Если счётчик больше 5 — запрещаем.

Минус:

В редких случаях между `INCR` и `EXPIRE` может произойти сбой.

Более надёжно использовать Lua-скрипт.

---

# 40. Lua-скрипты в Redis

Redis умеет выполнять Lua-скрипты атомарно.

Пример rate limiter:

```python
import asyncio
import redis.asyncio as redis


RATE_LIMIT_SCRIPT = """
local current
current = redis.call("INCR", KEYS[1])

if tonumber(current) == 1 then
    redis.call("EXPIRE", KEYS[1], ARGV[1])
end

return current
"""


async def is_allowed(r: redis.Redis, user_id: int) -> bool:
    key = f"rate_limit:user:{user_id}"

    current = await r.eval(
        RATE_LIMIT_SCRIPT,
        1,
        key,
        60,
    )

    return int(current) <= 5


async def main():
    r = redis.Redis(decode_responses=True)

    for i in range(10):
        allowed = await is_allowed(r, 1)
        print(i, allowed)

    await r.aclose()


asyncio.run(main())
```

`EVAL` параметры:

```python
await r.eval(script, number_of_keys, key1, arg1, arg2)
```

---

# 41. Распределённая блокировка

Иногда нужно сделать так, чтобы только один процесс выполнял действие.

Пример:

- только один воркер обрабатывает заказ;
- только один сервер обновляет кэш;
- только один процесс запускает cron-задачу.

Простая блокировка:

```python
import uuid
import redis.asyncio as redis


async def acquire_lock(r: redis.Redis, key: str, ttl: int = 10) -> str | None:
    token = str(uuid.uuid4())

    acquired = await r.set(
        key,
        token,
        nx=True,
        ex=ttl,
    )

    if acquired:
        return token

    return None
```

Почему нужен token?

Потому что удалять блокировку должен только тот, кто её создал.

Плохой вариант:

```python
await r.delete("lock:order:1")
```

Может удалить чужую блокировку.

Правильное удаление через Lua:

```python
RELEASE_LOCK_SCRIPT = """
if redis.call("GET", KEYS[1]) == ARGV[1] then
    return redis.call("DEL", KEYS[1])
else
    return 0
end
"""
```

Полный пример:

```python
import asyncio
import uuid
import redis.asyncio as redis


RELEASE_LOCK_SCRIPT = """
if redis.call("GET", KEYS[1]) == ARGV[1] then
    return redis.call("DEL", KEYS[1])
else
    return 0
end
"""


async def acquire_lock(r: redis.Redis, key: str, ttl: int = 10) -> str | None:
    token = str(uuid.uuid4())

    acquired = await r.set(key, token, nx=True, ex=ttl)

    if acquired:
        return token

    return None


async def release_lock(r: redis.Redis, key: str, token: str):
    await r.eval(RELEASE_LOCK_SCRIPT, 1, key, token)


async def process_order(r: redis.Redis, order_id: int):
    lock_key = f"lock:order:{order_id}"

    token = await acquire_lock(r, lock_key, ttl=30)

    if token is None:
        print("Order is already processing")
        return

    try:
        print("Processing order...")
        await asyncio.sleep(5)
        print("Done")
    finally:
        await release_lock(r, lock_key, token)


async def main():
    r = redis.Redis(decode_responses=True)

    await asyncio.gather(
        process_order(r, 1),
        process_order(r, 1),
    )

    await r.aclose()


asyncio.run(main())
```

---

# 42. Pub/Sub из Python

## Subscriber

```python
import asyncio
import redis.asyncio as redis


async def subscriber():
    r = redis.Redis(decode_responses=True)

    pubsub = r.pubsub()

    await pubsub.subscribe("news")

    print("Subscribed to news")

    async for message in pubsub.listen():
        if message["type"] == "message":
            print("Received:", message["data"])


asyncio.run(subscriber())
```

## Publisher

```python
import asyncio
import redis.asyncio as redis


async def publisher():
    r = redis.Redis(decode_responses=True)

    await r.publish("news", "Hello from Python")

    await r.aclose()


asyncio.run(publisher())
```

Важно:

Pub/Sub подходит только для online-доставки.

Если subscriber выключен — сообщения потеряются.

---

# 43. Streams из Python

## Добавить сообщение

```python
import asyncio
import redis.asyncio as redis


async def main():
    r = redis.Redis(decode_responses=True)

    message_id = await r.xadd(
        "orders",
        {
            "user_id": "1",
            "amount": "500",
        },
    )

    print(message_id)

    await r.aclose()


asyncio.run(main())
```

---

## Читать Stream

```python
import asyncio
import redis.asyncio as redis


async def main():
    r = redis.Redis(decode_responses=True)

    messages = await r.xread(
        streams={"orders": "0"},
        count=10,
        block=5000,
    )

    print(messages)

    await r.aclose()


asyncio.run(main())
```

---

## Consumer Group

Создание группы:

```python
import asyncio
import redis.asyncio as redis
from redis.exceptions import ResponseError


async def create_group(r: redis.Redis):
    try:
        await r.xgroup_create(
            name="orders",
            groupname="order_workers",
            id="$",
            mkstream=True,
        )
    except ResponseError as e:
        if "BUSYGROUP" in str(e):
            pass
        else:
            raise
```

Consumer:

```python
import asyncio
import redis.asyncio as redis
from redis.exceptions import ResponseError


async def create_group(r: redis.Redis):
    try:
        await r.xgroup_create(
            name="orders",
            groupname="order_workers",
            id="$",
            mkstream=True,
        )
    except ResponseError as e:
        if "BUSYGROUP" not in str(e):
            raise


async def worker(worker_name: str):
    r = redis.Redis(decode_responses=True)

    await create_group(r)

    while True:
        response = await r.xreadgroup(
            groupname="order_workers",
            consumername=worker_name,
            streams={"orders": ">"},
            count=1,
            block=5000,
        )

        if not response:
            continue

        for stream_name, messages in response:
            for message_id, data in messages:
                print(worker_name, "processing", message_id, data)

                try:
                    await asyncio.sleep(1)

                    await r.xack("orders", "order_workers", message_id)
                    print(worker_name, "done", message_id)

                except Exception as e:
                    print("Error:", e)


asyncio.run(worker("worker-1"))
```

---

# 44. SCAN вместо KEYS

Плохо:

```python
keys = await r.keys("*")
```

Почему плохо?

Потому что Redis может зависнуть, если ключей много.

Правильно:

```python
cursor = 0

while True:
    cursor, keys = await r.scan(
        cursor=cursor,
        match="cache:user:*",
        count=100,
    )

    for key in keys:
        print(key)

    if cursor == 0:
        break
```

---

# 45. Удаление большого количества ключей

Если ключей много, не надо делать:

```redis
KEYS cache:* 
DEL ...
```

Лучше:

```python
async def delete_by_pattern(r: redis.Redis, pattern: str):
    cursor = 0

    while True:
        cursor, keys = await r.scan(
            cursor=cursor,
            match=pattern,
            count=500,
        )

        if keys:
            await r.delete(*keys)

        if cursor == 0:
            break
```

---

# 46. Connection Pool

Redis-клиент использует пул соединений.

Не нужно создавать новый клиент на каждый запрос.

Плохо:

```python
async def handler():
    r = redis.Redis()
    await r.get("key")
    await r.aclose()
```

Так делать плохо в веб-приложении.

Лучше создать один клиент на всё приложение.

Например, в FastAPI:

```python
from contextlib import asynccontextmanager

import redis.asyncio as redis
from fastapi import FastAPI


@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.redis = redis.from_url(
        "redis://localhost:6379/0",
        decode_responses=True,
    )

    yield

    await app.state.redis.aclose()


app = FastAPI(lifespan=lifespan)


@app.get("/ping")
async def ping():
    result = await app.state.redis.ping()
    return {"redis": result}
```

---

# 47. Пример FastAPI + Redis cache

Установка:

```bash
pip install fastapi uvicorn redis
```

`main.py`:

```python
import asyncio
import json
from contextlib import asynccontextmanager

import redis.asyncio as redis
from fastapi import FastAPI, Request


async def fake_db_get_user(user_id: int):
    await asyncio.sleep(2)

    return {
        "id": user_id,
        "name": "Ivan",
        "age": 25,
    }


@asynccontextmanager
async def lifespan(app: FastAPI):
    app.state.redis = redis.from_url(
        "redis://localhost:6379/0",
        decode_responses=True,
    )

    yield

    await app.state.redis.aclose()


app = FastAPI(lifespan=lifespan)


@app.get("/users/{user_id}")
async def get_user(user_id: int, request: Request):
    r: redis.Redis = request.app.state.redis

    cache_key = f"cache:user:{user_id}"

    cached = await r.get(cache_key)

    if cached:
        return {
            "source": "cache",
            "data": json.loads(cached),
        }

    user = await fake_db_get_user(user_id)

    await r.set(
        cache_key,
        json.dumps(user),
        ex=60,
    )

    return {
        "source": "database",
        "data": user,
    }


@app.delete("/users/{user_id}/cache")
async def delete_user_cache(user_id: int, request: Request):
    r: redis.Redis = request.app.state.redis

    await r.delete(f"cache:user:{user_id}")

    return {"status": "deleted"}
```

Запуск:

```bash
uvicorn main:app --reload
```

Первый запрос:

```text
GET http://127.0.0.1:8000/users/1
```

Будет медленный.

Второй запрос будет быстрый.

---

# 48. Persistence: как Redis сохраняет данные

Redis хранит данные в памяти, но умеет сохранять на диск.

Есть два основных механизма:

1. RDB
2. AOF

---

## RDB

RDB — это snapshot.

Redis периодически делает снимок данных и сохраняет на диск.

Плюсы:

- компактный файл;
- быстро восстанавливается;
- хорошо для бэкапов.

Минусы:

- можно потерять последние изменения после snapshot.

---

## AOF

AOF — Append Only File.

Redis записывает каждую изменяющую команду в лог.

Плюсы:

- меньше риск потери данных;
- можно восстановить почти все операции.

Минусы:

- файл может быть больше;
- может быть медленнее RDB.

---

## Что выбрать

Для кэша часто persistence вообще не критична.

Для важных данных можно включать AOF.

Но Redis обычно не должен быть единственным хранилищем критически важных данных, если ты не понимаешь хорошо persistence, replication и backup.

---

# 49. Eviction policy

Redis работает в памяти.

Если память закончится, Redis должен решить, что делать.

Настройка:

```text
maxmemory
maxmemory-policy
```

Политики:

## noeviction

Не удалять ключи, а возвращать ошибку при записи.

## allkeys-lru

Удалять наименее недавно используемые ключи среди всех ключей.

Хорошо для кэша.

## volatile-lru

Удалять наименее недавно используемые ключи только среди ключей с TTL.

## allkeys-random

Удалять случайные ключи.

## volatile-ttl

Удалять ключи с ближайшим истечением TTL.

Для кэша часто используют:

```text
allkeys-lru
```

или:

```text
volatile-lru
```

---

# 50. Безопасность Redis

Redis нельзя просто так открывать в интернет.

Плохо:

```text
0.0.0.0:6379 без пароля
```

Правильно:

1. Не открывать Redis наружу.
2. Использовать private network.
3. Включить пароль.
4. Использовать ACL.
5. Ограничить доступ firewall.
6. Использовать TLS при необходимости.

Пример запуска Docker с паролем:

```bash
docker run --name redis-secure -p 6379:6379 -d redis:7 redis-server --requirepass my_password
```

Подключение:

```bash
redis-cli -a my_password
```

Python:

```python
r = redis.from_url(
    "redis://:my_password@localhost:6379/0",
    decode_responses=True,
)
```

---

# 51. Мониторинг Redis

Полезные команды:

## INFO

```redis
INFO
```

Показывает много информации.

Например:

```redis
INFO memory
INFO clients
INFO stats
INFO persistence
```

## DBSIZE

Количество ключей в текущей базе:

```redis
DBSIZE
```

## MEMORY USAGE

Сколько памяти занимает ключ:

```redis
MEMORY USAGE cache:user:1
```

## SLOWLOG

Показать медленные команды:

```redis
SLOWLOG GET 10
```

## CLIENT LIST

Показать подключенных клиентов:

```redis
CLIENT LIST
```

---

# 52. Частые ошибки новичков

## Ошибка 1. Использовать KEYS на production

Плохо:

```redis
KEYS *
```

Лучше:

```redis
SCAN
```

---

## Ошибка 2. Не ставить TTL для кэша

Плохо:

```python
await r.set("cache:user:1", value)
```

Лучше:

```python
await r.set("cache:user:1", value, ex=300)
```

---

## Ошибка 3. Создавать Redis-клиент на каждый запрос

Плохо:

```python
async def handler():
    r = redis.Redis()
```

Лучше:

Создать один клиент на приложение.

---

## Ошибка 4. Хранить критически важные данные только в Redis

Redis быстрый, но это не всегда основная база.

Для денег, заказов, пользователей чаще нужен PostgreSQL/MySQL.

---

## Ошибка 5. Делать read-modify-write без атомарности

Плохо:

```python
value = int(await r.get("counter"))
await r.set("counter", value + 1)
```

Лучше:

```python
await r.incr("counter")
```

---

## Ошибка 6. Не сериализовать сложные данные

Плохо:

```python
await r.set("user", {"id": 1})
```

Лучше:

```python
await r.set("user", json.dumps({"id": 1}))
```

---

# 53. Что должен знать Junior+ по Redis

Уверенный Junior+ должен понимать:

## База

- что такое Redis;
- зачем он нужен;
- чем отличается от PostgreSQL;
- что такое key-value;
- что такое TTL;
- как работают базовые команды.

## Типы данных

- String;
- Hash;
- List;
- Set;
- Sorted Set;
- базово Streams;
- базово Pub/Sub.

## Python

- как подключиться через `redis.asyncio`;
- как использовать `await`;
- как делать get/set;
- как работать с JSON;
- как использовать connection pool;
- как подключить Redis к FastAPI.

## Практика

- cache-aside;
- rate limiter;
- очередь на List;
- Pub/Sub;
- Streams consumer;
- distributed lock;
- pipeline;
- transactions;
- scan вместо keys.

## Production basics

- TTL;
- eviction policy;
- password/ACL;
- persistence;
- monitoring;
- memory usage;
- slowlog.

---

# 54. Мини-шпаргалка команд Redis

```redis
PING
SET key value
GET key
DEL key
EXISTS key
EXPIRE key seconds
TTL key
INCR key
DECR key

HSET key field value
HGET key field
HGETALL key
HDEL key field

LPUSH key value
RPUSH key value
LPOP key
RPOP key
LRANGE key 0 -1
BLPOP key 0

SADD key value
SMEMBERS key
SISMEMBER key value
SREM key value
SCARD key

ZADD key score value
ZRANGE key 0 -1 WITHSCORES
ZREVRANGE key 0 -1 WITHSCORES
ZINCRBY key increment value
ZREVRANK key value

PUBLISH channel message
SUBSCRIBE channel

XADD stream * field value
XREAD STREAMS stream 0
XGROUP CREATE stream group $ MKSTREAM
XREADGROUP GROUP group consumer STREAMS stream >
XACK stream group id

SCAN cursor MATCH pattern COUNT count
INFO
DBSIZE
SLOWLOG GET 10
```

---

# 55. Мини-шпаргалка Python async Redis

```python
import redis.asyncio as redis

r = redis.from_url(
    "redis://localhost:6379/0",
    decode_responses=True,
)

await r.set("key", "value", ex=60)
value = await r.get("key")

await r.delete("key")
await r.exists("key")

await r.hset("user:1", mapping={"name": "Ivan"})
user = await r.hgetall("user:1")

await r.rpush("queue", "task")
task = await r.blpop("queue", timeout=0)

await r.sadd("online", "1")
users = await r.smembers("online")

await r.zadd("rating", {"Ivan": 100})
top = await r.zrevrange("rating", 0, 10, withscores=True)

await r.aclose()
```

---

# 56. Практические задания для закрепления

## Задание 1

Сделай счётчик просмотров статьи.

Ключ:

```text
post:{post_id}:views
```

Команды:

```python
await r.incr(f"post:{post_id}:views")
```

---

## Задание 2

Сделай кэш пользователя на 5 минут.

Ключ:

```text
cache:user:{user_id}
```

Используй:

```python
json.dumps()
json.loads()
```

---

## Задание 3

Сделай очередь email-задач.

Producer:

```python
await r.rpush("queue:emails", json.dumps(task))
```

Consumer:

```python
await r.blpop("queue:emails", timeout=0)
```

---

## Задание 4

Сделай rate limiter:

```text
не больше 10 запросов в минуту
```

Ключ:

```text
rate_limit:user:{user_id}
```

---

## Задание 5

Сделай leaderboard.

Используй Sorted Set:

```python
await r.zadd("leaderboard", {"user:1": 100})
await r.zincrby("leaderboard", 50, "user:1")
```

---

# 57. Итог

Redis — это быстрый инструмент для работы с данными в памяти.

Главные вещи, которые нужно запомнить:

1. Redis хранит данные по ключам.
2. Почти всегда нужно продумывать структуру ключей.
3. Для временных данных используй TTL.
4. Для кэша обязательно ставь TTL.
5. Для счётчиков используй атомарные команды `INCR`.
6. Для очередей можно использовать List или Streams.
7. Для рейтингов используй Sorted Set.
8. Для уникальных значений используй Set.
9. Для объектов можно использовать Hash или JSON.
10. В Python используй `redis.asyncio`.
11. Не создавай новый Redis-клиент на каждый запрос.
12. Не используй `KEYS *` на production.
13. Redis не всегда замена PostgreSQL.
14. Для production нужны настройки памяти, безопасности и мониторинга.

Если ты уверенно понимаешь и умеешь применять всё из этого гайда, то по Redis ты уже находишься примерно на уровне уверенного Junior / Junior+.
