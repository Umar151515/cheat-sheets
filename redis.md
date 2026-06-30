# Redis с нуля до уверенного Junior+ на Python

Redis — это очень быстая in-memory база данных типа **key-value**.  
То есть Redis хранит данные в формате:

```text
ключ -> значение
```

Например:

```text
"user:1:name" -> "Ivan"
"user:1:age"  -> "25"
```

Redis часто используют для:

1. **Кэша**
2. **Сессий пользователей**
3. **Очередей задач**
4. **Rate limiting**
5. **Хранения временных данных**
6. **Pub/Sub**
7. **Счётчиков**
8. **Лидеров рейтингов**
9. **Блокировок**
10. **Streams / событий**

---

# 1. Что такое Redis простыми словами

Обычная база данных, например PostgreSQL, хранит данные на диске.

Redis в первую очередь хранит данные в **оперативной памяти**.

Из-за этого Redis очень быстрый.

Примерно:

```text
PostgreSQL:
запрос -> диск -> данные -> ответ

Redis:
запрос -> RAM -> ответ
```

Поэтому Redis может обрабатывать десятки и сотни тысяч операций в секунду.

---

# 2. Где Redis НЕ заменяет PostgreSQL/MySQL

Redis — это не универсальная замена SQL-базе.

Redis хорошо подходит для:

```text
быстро достать
быстро сохранить
быстро посчитать
быстро удалить
хранить временные данные
```

Но Redis плохо подходит как единственное хранилище для:

```text
банковских транзакций
сложных SQL-запросов
связанных таблиц
долгосрочной бизнес-информации
сложной аналитики
```

Обычно Redis используют вместе с основной БД:

```text
Python-приложение
        |
        |---- PostgreSQL — постоянные данные
        |
        |---- Redis — кэш, очереди, сессии, счётчики
```

---

# 3. Установка Redis

## Вариант 1. Через Docker

Самый удобный способ для обучения.

```bash
docker run --name my-redis -p 6379:6379 -d redis:7
```

Проверить, что контейнер работает:

```bash
docker ps
```

Подключиться к Redis CLI:

```bash
docker exec -it my-redis redis-cli
```

Проверка:

```redis
PING
```

Ответ:

```redis
PONG
```

---

## Вариант 2. Docker Compose

Создай файл `docker-compose.yml`:

```yaml
services:
  redis:
    image: redis:7
    container_name: redis-learning
    ports:
      - "6379:6379"
```

Запуск:

```bash
docker compose up -d
```

Остановка:

```bash
docker compose down
```

---

# 4. Установка Python-клиента

Для работы с Redis из Python чаще всего используют библиотеку `redis`.

```bash
pip install redis
```

Проверка:

```python
import redis

r = redis.Redis(
    host="localhost",
    port=6379,
    db=0,
    decode_responses=True
)

print(r.ping())
```

Если всё хорошо, увидишь:

```text
True
```

---

# 5. Подключение к Redis из Python

```python
import redis

r = redis.Redis(
    host="localhost",
    port=6379,
    db=0,
    decode_responses=True
)
```

Разберём параметры:

```python
host="localhost"
```

Redis запущен на твоём компьютере.

```python
port=6379
```

Стандартный порт Redis.

```python
db=0
```

Redis поддерживает несколько логических баз данных: `0`, `1`, `2` и так далее.  
Обычно используют `0`.

```python
decode_responses=True
```

Очень важный параметр.

Без него Redis будет возвращать байты:

```python
b"hello"
```

С ним Redis возвращает обычные строки:

```python
"hello"
```

---

# 6. Redis CLI: базовые команды

Redis можно использовать через консоль:

```bash
redis-cli
```

Примеры:

```redis
SET name Ivan
GET name
DEL name
EXISTS name
```

То же самое из Python:

```python
r.set("name", "Ivan")
print(r.get("name"))
r.delete("name")
print(r.exists("name"))
```

---

# 7. Главная идея Redis: ключи

В Redis почти всё строится вокруг ключей.

Пример:

```redis
SET user:1:name Ivan
SET user:1:age 25
SET product:10:title "iPhone"
SET cart:5:item:10 2
```

Хороший стиль именования ключей:

```text
entity:id:field
```

Например:

```text
user:1:name
user:1:email
order:100:status
product:50:price
```

Для разделения используют двоеточие `:`.

---

# 8. Типы данных Redis

Redis — это не просто строки. У него есть разные структуры данных:

| Тип Redis | Для чего используется |
|---|---|
| String | строки, числа, JSON, счётчики |
| Hash | объект с полями |
| List | очередь, стек, список |
| Set | уникальные значения |
| Sorted Set | рейтинги, лидерборды |
| Stream | события, очереди сообщений |
| Bitmap | битовые флаги |
| HyperLogLog | приблизительный подсчёт уникальных |
| Geo | географические координаты |

---

# 9. String

## 9.1 Что такое String

String — самый простой тип Redis.

Можно хранить:

```text
строку
число
JSON
токен
флаг
```

Пример:

```python
r.set("name", "Ivan")
print(r.get("name"))
```

Результат:

```text
Ivan
```

---

## 9.2 SET и GET

```python
r.set("user:1:name", "Ivan")

name = r.get("user:1:name")

print(name)
```

---

## 9.3 Проверка существования ключа

```python
if r.exists("user:1:name"):
    print("Ключ существует")
else:
    print("Ключа нет")
```

---

## 9.4 Удаление ключа

```python
r.delete("user:1:name")
```

---

## 9.5 Хранение чисел

Redis хранит значения как строки, но умеет увеличивать и уменьшать числа.

```python
r.set("counter", 0)

r.incr("counter")
r.incr("counter")
r.incr("counter")

print(r.get("counter"))
```

Результат:

```text
3
```

Уменьшить:

```python
r.decr("counter")
```

Увеличить на конкретное число:

```python
r.incrby("counter", 10)
```

---

## 9.6 Практический пример: счётчик просмотров

```python
def add_view(article_id: int):
    key = f"article:{article_id}:views"
    return r.incr(key)


def get_views(article_id: int):
    key = f"article:{article_id}:views"
    views = r.get(key)

    if views is None:
        return 0

    return int(views)


add_view(10)
add_view(10)

print(get_views(10))
```

---

# 10. TTL и expiration

Redis умеет автоматически удалять ключи через определённое время.

Это называется TTL — Time To Live.

---

## 10.1 SETEX

Сохранить ключ на 60 секунд:

```python
r.setex("temporary:key", 60, "hello")
```

Через 60 секунд ключ исчезнет.

---

## 10.2 EXPIRE

Можно сначала создать ключ, потом назначить время жизни:

```python
r.set("session:123", "user_id=1")
r.expire("session:123", 3600)
```

Ключ исчезнет через 1 час.

---

## 10.3 TTL

Узнать, сколько ключу осталось жить:

```python
ttl = r.ttl("session:123")
print(ttl)
```

Возможные значения:

```text
> 0   количество секунд до удаления
-1    ключ существует, но TTL нет
-2    ключа не существует
```

---

## 10.4 Практический пример: код подтверждения

```python
import random

def create_confirmation_code(user_id: int):
    code = random.randint(100000, 999999)
    key = f"confirmation:{user_id}"

    r.setex(key, 300, code)

    return code


def check_confirmation_code(user_id: int, code: str):
    key = f"confirmation:{user_id}"

    saved_code = r.get(key)

    if saved_code is None:
        return False

    return saved_code == str(code)


code = create_confirmation_code(1)
print("Код:", code)

print(check_confirmation_code(1, code))
```

Код живёт 5 минут.

---

# 11. SET с параметрами NX и XX

## 11.1 NX

`NX` означает: установить значение только если ключа ещё нет.

Python:

```python
result = r.set("lock:task", "1", nx=True)

print(result)
```

Если ключа не было:

```text
True
```

Если ключ уже есть:

```text
None
```

---

## 11.2 XX

`XX` означает: установить значение только если ключ уже существует.

```python
r.set("name", "Ivan")

result = r.set("name", "Petr", xx=True)

print(result)
```

---

## 11.3 SET с expiration

```python
r.set("token:abc", "user_id=1", ex=3600)
```

Ключ будет жить 3600 секунд.

---

## 11.4 SET NX EX — база для блокировок

```python
r.set("lock:payment:100", "locked", nx=True, ex=10)
```

Это значит:

```text
Создай ключ lock:payment:100 только если его ещё нет.
Если создал — удали через 10 секунд.
```

---

# 12. Hash

## 12.1 Что такое Hash

Hash — это объект внутри Redis.

Пример:

```text
user:1
    name  -> Ivan
    age   -> 25
    email -> ivan@example.com
```

Вместо:

```text
user:1:name
user:1:age
user:1:email
```

можно хранить всё в одном hash:

```text
user:1
```

---

## 12.2 HSET

```python
r.hset("user:1", "name", "Ivan")
r.hset("user:1", "age", 25)
r.hset("user:1", "email", "ivan@example.com")
```

---

## 12.3 HMSET через mapping

```python
r.hset(
    "user:2",
    mapping={
        "name": "Maria",
        "age": 30,
        "email": "maria@example.com"
    }
)
```

---

## 12.4 HGET

```python
name = r.hget("user:1", "name")
print(name)
```

---

## 12.5 HGETALL

```python
user = r.hgetall("user:1")
print(user)
```

Результат:

```python
{
    "name": "Ivan",
    "age": "25",
    "email": "ivan@example.com"
}
```

Обрати внимание: Redis вернул `"25"` как строку.

---

## 12.6 HEXISTS

```python
if r.hexists("user:1", "email"):
    print("Email есть")
```

---

## 12.7 HDEL

```python
r.hdel("user:1", "email")
```

---

## 12.8 Практический пример: профиль пользователя

```python
def save_user(user_id: int, name: str, age: int, email: str):
    key = f"user:{user_id}"

    r.hset(
        key,
        mapping={
            "name": name,
            "age": age,
            "email": email
        }
    )


def get_user(user_id: int):
    key = f"user:{user_id}"

    user = r.hgetall(key)

    if not user:
        return None

    return {
        "name": user["name"],
        "age": int(user["age"]),
        "email": user["email"]
    }


save_user(1, "Ivan", 25, "ivan@example.com")

print(get_user(1))
```

---

# 13. List

## 13.1 Что такое List

List — это список строк.

Можно добавлять элементы:

```text
слева
справа
```

И доставать:

```text
слева
справа
```

Redis List часто используют для простых очередей.

---

## 13.2 LPUSH и RPUSH

```python
r.lpush("tasks", "task1")
r.lpush("tasks", "task2")
r.rpush("tasks", "task3")
```

Если делаем:

```python
r.lrange("tasks", 0, -1)
```

можем получить:

```python
["task2", "task1", "task3"]
```

---

## 13.3 LRANGE

```python
items = r.lrange("tasks", 0, -1)
print(items)
```

`0` — первый элемент.  
`-1` — последний элемент.

---

## 13.4 LPOP и RPOP

```python
task = r.lpop("tasks")
print(task)
```

```python
task = r.rpop("tasks")
print(task)
```

---

## 13.5 Простая очередь

Если добавляем справа:

```python
r.rpush("queue:emails", "send email to user 1")
r.rpush("queue:emails", "send email to user 2")
```

А забираем слева:

```python
task = r.lpop("queue:emails")
```

Получается FIFO:

```text
первым пришёл -> первым ушёл
```

---

## 13.6 Блокирующее ожидание BLPOP

Обычный `lpop` сразу вернёт `None`, если очередь пустая.

`blpop` может ждать задачу.

```python
task = r.blpop("queue:emails", timeout=10)

print(task)
```

Если задача появилась, вернётся:

```python
("queue:emails", "send email to user 1")
```

---

## 13.7 Практический пример: worker

producer.py:

```python
import redis
import json

r = redis.Redis(decode_responses=True)

def add_email_task(email: str, text: str):
    task = {
        "email": email,
        "text": text
    }

    r.rpush("queue:emails", json.dumps(task))


add_email_task("user@example.com", "Hello!")
```

worker.py:

```python
import redis
import json
import time

r = redis.Redis(decode_responses=True)

while True:
    result = r.blpop("queue:emails", timeout=5)

    if result is None:
        print("Задач нет")
        continue

    queue_name, task_json = result
    task = json.loads(task_json)

    print("Отправляем письмо:", task)

    time.sleep(1)

    print("Готово")
```

---

# 14. Set

## 14.1 Что такое Set

Set — это множество уникальных значений.

То есть в Set не может быть дублей.

Пример:

```text
online_users = {1, 2, 3}
```

---

## 14.2 SADD

```python
r.sadd("online_users", 1)
r.sadd("online_users", 2)
r.sadd("online_users", 2)
```

Даже если добавить `2` два раза, он будет храниться один раз.

---

## 14.3 SMEMBERS

```python
users = r.smembers("online_users")
print(users)
```

---

## 14.4 SREM

```python
r.srem("online_users", 2)
```

---

## 14.5 SISMEMBER

```python
if r.sismember("online_users", 1):
    print("Пользователь онлайн")
```

---

## 14.6 Практический пример: лайки поста

```python
def like_post(post_id: int, user_id: int):
    key = f"post:{post_id}:likes"
    r.sadd(key, user_id)


def unlike_post(post_id: int, user_id: int):
    key = f"post:{post_id}:likes"
    r.srem(key, user_id)


def is_liked(post_id: int, user_id: int):
    key = f"post:{post_id}:likes"
    return r.sismember(key, user_id)


def likes_count(post_id: int):
    key = f"post:{post_id}:likes"
    return r.scard(key)


like_post(10, 1)
like_post(10, 2)
like_post(10, 1)

print(likes_count(10))
print(is_liked(10, 1))
```

---

# 15. Sorted Set

## 15.1 Что такое Sorted Set

Sorted Set похож на Set, но у каждого элемента есть score.

```text
player_id -> score
```

Например рейтинг игроков:

```text
Ivan  -> 100
Petr  -> 150
Maria -> 120
```

Redis хранит их отсортированными по score.

---

## 15.2 ZADD

```python
r.zadd(
    "game:rating",
    {
        "Ivan": 100,
        "Petr": 150,
        "Maria": 120
    }
)
```

---

## 15.3 ZRANGE

Получить от меньшего score к большему:

```python
players = r.zrange("game:rating", 0, -1, withscores=True)

print(players)
```

---

## 15.4 ZREVRANGE

Получить от большего score к меньшему:

```python
players = r.zrevrange("game:rating", 0, -1, withscores=True)

print(players)
```

---

## 15.5 ZINCRBY

Увеличить score:

```python
r.zincrby("game:rating", 50, "Ivan")
```

---

## 15.6 Практический пример: таблица лидеров

```python
def add_score(user_id: int, score: int):
    key = "leaderboard"
    r.zincrby(key, score, user_id)


def get_top_users(limit: int = 10):
    key = "leaderboard"

    users = r.zrevrange(
        key,
        0,
        limit - 1,
        withscores=True
    )

    return [
        {
            "user_id": int(user_id),
            "score": int(score)
        }
        for user_id, score in users
    ]


add_score(1, 100)
add_score(2, 250)
add_score(3, 150)

print(get_top_users())
```

---

# 16. JSON в Redis

Redis сам по себе хранит строки.

Если нужно сохранить сложный объект, можно использовать JSON.

```python
import json

user = {
    "id": 1,
    "name": "Ivan",
    "age": 25
}

r.set("user:1:json", json.dumps(user))
```

Получить:

```python
data = r.get("user:1:json")

user = json.loads(data)

print(user["name"])
```

---

## Когда использовать JSON, а когда Hash?

### Hash удобен, если:

```text
нужно часто менять отдельные поля
нужно читать отдельные поля
структура простая
```

Пример:

```text
user:1 -> name, age, email
```

### JSON удобен, если:

```text
объект сложный
вложенные структуры
читаешь и пишешь весь объект целиком
```

Пример:

```json
{
  "id": 1,
  "profile": {
    "name": "Ivan",
    "contacts": {
      "email": "ivan@example.com"
    }
  }
}
```

---

# 17. Кэширование

Кэш — одно из самых популярных применений Redis.

---

## 17.1 Зачем нужен кэш

Допустим, есть медленная функция:

```python
def get_user_from_database(user_id):
    # долгий запрос в PostgreSQL
    ...
```

Если много пользователей запрашивают одни и те же данные, можно:

1. Сначала проверить Redis.
2. Если данные есть — вернуть из Redis.
3. Если данных нет — сходить в БД.
4. Сохранить результат в Redis.
5. Вернуть данные.

Это называется **cache-aside pattern**.

---

## 17.2 Cache-aside пример

```python
import json
import time

def get_user_from_db(user_id: int):
    print("Идём в базу данных...")
    time.sleep(2)

    return {
        "id": user_id,
        "name": "Ivan",
        "age": 25
    }


def get_user(user_id: int):
    cache_key = f"cache:user:{user_id}"

    cached_user = r.get(cache_key)

    if cached_user is not None:
        print("Берём из Redis")
        return json.loads(cached_user)

    print("В Redis нет, берём из БД")

    user = get_user_from_db(user_id)

    r.setex(
        cache_key,
        60,
        json.dumps(user)
    )

    return user


print(get_user(1))
print(get_user(1))
```

Первый вызов:

```text
В Redis нет, берём из БД
Идём в базу данных...
```

Второй вызов:

```text
Берём из Redis
```

---

## 17.3 Инвалидация кэша

Главная сложность кэша — понять, когда его удалять.

Если пользователь изменился в базе, старый кэш нужно удалить.

```python
def update_user(user_id: int, new_name: str):
    # 1. Обновили в БД
    print("Обновляем пользователя в БД")

    # 2. Удалили кэш
    r.delete(f"cache:user:{user_id}")
```

Правило:

```text
Изменил данные в основной БД -> удали или обнови кэш.
```

---

## 17.4 Типичные ошибки кэширования

### Ошибка 1

Хранить кэш без TTL.

Плохо:

```python
r.set("cache:user:1", json.dumps(user))
```

Лучше:

```python
r.setex("cache:user:1", 300, json.dumps(user))
```

---

### Ошибка 2

Забыть инвалидировать кэш.

```text
В БД пользователь уже Petr,
а Redis всё ещё хранит Ivan.
```

---

### Ошибка 3

Кэшировать всё подряд.

Не надо кэшировать данные, которые:

```text
постоянно меняются
почти не читаются
слишком большие
не нужны часто
```

---

# 18. Pipeline

## 18.1 Проблема многих запросов

Если сделать 1000 команд Redis по одной:

```python
for i in range(1000):
    r.set(f"key:{i}", i)
```

Python 1000 раз отправит запрос в Redis и 1000 раз получит ответ.

Это может быть медленно из-за сетевых задержек.

---

## 18.2 Pipeline

Pipeline позволяет отправить много команд пачкой.

```python
pipe = r.pipeline()

for i in range(1000):
    pipe.set(f"key:{i}", i)

pipe.execute()
```

Команды отправятся вместе.

---

## 18.3 Пример чтения пачкой

```python
pipe = r.pipeline()

for i in range(10):
    pipe.get(f"key:{i}")

results = pipe.execute()

print(results)
```

---

## 18.4 Когда использовать pipeline

Используй pipeline, когда:

```text
нужно выполнить много независимых команд
нужно уменьшить количество round-trip запросов
```

Например:

```text
загрузить 1000 ключей
обновить 500 счётчиков
получить много значений
```

---

# 19. Атомарность

## 19.1 Что такое атомарность

Атомарная операция — это операция, которая выполняется целиком и не может быть прервана в середине.

Например:

```python
r.incr("counter")
```

Это атомарно.

Даже если 100 клиентов одновременно сделают `INCR`, Redis корректно увеличит счётчик.

---

## 19.2 Неатомарный пример

Плохо:

```python
value = r.get("counter")
value = int(value)
value += 1
r.set("counter", value)
```

Проблема:

```text
Клиент A прочитал 10
Клиент B прочитал 10
Клиент A записал 11
Клиент B записал 11
```

Хотя должно быть 12.

Правильно:

```python
r.incr("counter")
```

---

# 20. Transactions: MULTI / EXEC

Redis поддерживает транзакции.

В redis-py pipeline по умолчанию работает как транзакция.

```python
pipe = r.pipeline(transaction=True)

pipe.set("a", 1)
pipe.set("b", 2)
pipe.incr("counter")

results = pipe.execute()

print(results)
```

Команды будут выполнены последовательно одним блоком.

---

# 21. WATCH: оптимистическая блокировка

`WATCH` нужен, когда ты хочешь:

1. Прочитать значение.
2. Посчитать новое.
3. Записать.
4. Но только если за это время значение никто не изменил.

---

## Пример: безопасное списание баланса

```python
import redis

def withdraw(user_id: int, amount: int):
    key = f"user:{user_id}:balance"

    with r.pipeline() as pipe:
        while True:
            try:
                pipe.watch(key)

                balance = pipe.get(key)

                if balance is None:
                    pipe.unwatch()
                    return False

                balance = int(balance)

                if balance < amount:
                    pipe.unwatch()
                    return False

                new_balance = balance - amount

                pipe.multi()
                pipe.set(key, new_balance)
                pipe.execute()

                return True

            except redis.WatchError:
                continue
```

Использование:

```python
r.set("user:1:balance", 100)

print(withdraw(1, 30))
print(r.get("user:1:balance"))
```

---

# 22. Lua-скрипты

Redis может выполнять Lua-скрипты.

Главный плюс: Lua-скрипт выполняется атомарно.

То есть никто не вмешается между командами внутри скрипта.

---

## 22.1 Пример: rate limiter через Lua

Ограничим пользователя: максимум 5 запросов в минуту.

```python
lua_script = """
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local ttl = tonumber(ARGV[2])

local current = redis.call("GET", key)

if current and tonumber(current) >= limit then
    return 0
end

current = redis.call("INCR", key)

if current == 1 then
    redis.call("EXPIRE", key, ttl)
end

return 1
"""

rate_limiter = r.register_script(lua_script)


def is_allowed(user_id: int):
    key = f"rate_limit:user:{user_id}"

    result = rate_limiter(
        keys=[key],
        args=[5, 60]
    )

    return result == 1


for i in range(10):
    print(i, is_allowed(1))
```

Первые 5 запросов будут `True`, остальные `False`.

---

# 23. Блокировки Redis

Иногда нужно сделать так, чтобы только один процесс выполнял конкретную задачу.

Пример:

```text
нельзя одновременно два раза обработать один платёж
```

---

## 23.1 Простая блокировка

```python
lock = r.lock("lock:payment:100", timeout=10)

with lock:
    print("Обрабатываем платёж")
```

Если другой процесс попробует взять такой же lock, он будет ждать или получит ошибку в зависимости от настроек.

---

## 23.2 Более явный пример

```python
def process_payment(payment_id: int):
    lock_key = f"lock:payment:{payment_id}"

    lock = r.lock(lock_key, timeout=30, blocking_timeout=5)

    acquired = lock.acquire()

    if not acquired:
        print("Не удалось взять блокировку")
        return False

    try:
        print("Обрабатываем платёж", payment_id)
        return True
    finally:
        lock.release()
```

---

## 23.3 Важные правила блокировок

1. Всегда ставь `timeout`.
2. Всегда освобождай lock в `finally`.
3. Не держи lock долго.
4. Не используй Redis-lock как замену полноценным транзакциям в БД для критичных денег.

---

# 24. Pub/Sub

## 24.1 Что такое Pub/Sub

Pub/Sub — это механизм:

```text
один публикует сообщение
другие получают
```

Пример:

```text
service A публикует "user_created"
service B слушает и отправляет email
service C слушает и пишет аналитику
```

Важно:

```text
Pub/Sub не хранит сообщения.
Если подписчик был выключен, он пропустит сообщение.
```

---

## 24.2 Publisher

publisher.py:

```python
import redis
import json

r = redis.Redis(decode_responses=True)

message = {
    "user_id": 1,
    "event": "user_created"
}

r.publish("events", json.dumps(message))
```

---

## 24.3 Subscriber

subscriber.py:

```python
import redis
import json

r = redis.Redis(decode_responses=True)

pubsub = r.pubsub()
pubsub.subscribe("events")

print("Ждём сообщения...")

for message in pubsub.listen():
    if message["type"] != "message":
        continue

    data = json.loads(message["data"])
    print("Получено:", data)
```

---

## 24.4 Когда использовать Pub/Sub

Подходит:

```text
уведомления между сервисами
онлайн-события
обновление локальных кэшей
простые real-time события
```

Не подходит:

```text
надёжные очереди
гарантированная доставка
важные бизнес-события
```

Для надёжных очередей лучше Redis Streams, RabbitMQ, Kafka, Celery.

---

# 25. Redis Streams

## 25.1 Что такое Stream

Stream — это журнал событий.

В отличие от Pub/Sub, сообщения сохраняются.

Пример:

```text
orders_stream:
    1690000000000-0 -> order_id=1 status=created
    1690000000001-0 -> order_id=2 status=created
```

---

## 25.2 XADD

Добавить событие:

```python
event_id = r.xadd(
    "orders",
    {
        "order_id": 1,
        "status": "created"
    }
)

print(event_id)
```

---

## 25.3 XREAD

Читать события:

```python
messages = r.xread(
    streams={
        "orders": "0-0"
    },
    count=10,
    block=1000
)

print(messages)
```

---

## 25.4 Consumer Groups

Consumer Group позволяет нескольким воркерам читать один stream и делить работу.

Создать группу:

```python
try:
    r.xgroup_create(
        name="orders",
        groupname="workers",
        id="0-0",
        mkstream=True
    )
except redis.ResponseError as e:
    if "BUSYGROUP" not in str(e):
        raise
```

Читать как consumer:

```python
messages = r.xreadgroup(
    groupname="workers",
    consumername="worker-1",
    streams={
        "orders": ">"
    },
    count=10,
    block=5000
)

print(messages)
```

Подтвердить обработку:

```python
r.xack("orders", "workers", event_id)
```

---

## 25.5 Полный пример Streams worker

```python
import redis
import time

r = redis.Redis(decode_responses=True)

STREAM = "orders"
GROUP = "order_workers"
CONSUMER = "worker_1"


def create_group():
    try:
        r.xgroup_create(
            name=STREAM,
            groupname=GROUP,
            id="0-0",
            mkstream=True
        )
    except redis.ResponseError as e:
        if "BUSYGROUP" not in str(e):
            raise


def add_order(order_id: int):
    r.xadd(
        STREAM,
        {
            "order_id": order_id,
            "status": "created"
        }
    )


def worker():
    while True:
        messages = r.xreadgroup(
            groupname=GROUP,
            consumername=CONSUMER,
            streams={STREAM: ">"},
            count=1,
            block=5000
        )

        if not messages:
            print("Нет новых сообщений")
            continue

        for stream_name, stream_messages in messages:
            for message_id, data in stream_messages:
                try:
                    print("Обрабатываем:", message_id, data)

                    time.sleep(1)

                    r.xack(STREAM, GROUP, message_id)

                    print("Подтвердили:", message_id)

                except Exception as e:
                    print("Ошибка:", e)


create_group()
add_order(1)
add_order(2)
worker()
```

---

# 26. SCAN вместо KEYS

## 26.1 KEYS

Команда:

```redis
KEYS *
```

показывает все ключи.

В Python:

```python
keys = r.keys("*")
```

Но на продакшене `KEYS *` опасна.

Почему?

Потому что если ключей миллионы, Redis может зависнуть на время выполнения команды.

---

## 26.2 SCAN

Лучше использовать `SCAN`.

```python
cursor = 0

while True:
    cursor, keys = r.scan(cursor=cursor, match="user:*", count=100)

    for key in keys:
        print(key)

    if cursor == 0:
        break
```

---

## 26.3 scan_iter

В redis-py есть удобный вариант:

```python
for key in r.scan_iter("user:*", count=100):
    print(key)
```

---

# 27. Удаление ключей: DEL и UNLINK

## DEL

```python
r.delete("key")
```

Удаляет ключ синхронно.

---

## UNLINK

```python
r.unlink("key")
```

Удаляет ключ асинхронно в фоне.

Для больших ключей `UNLINK` лучше, чем `DEL`.

---

# 28. Persistence: хранение данных на диске

Redis хранит данные в RAM, но может сохранять их на диск.

Есть два основных механизма:

1. RDB
2. AOF

---

## 28.1 RDB

RDB — это snapshot.

Redis периодически сохраняет снимок данных на диск.

Плюсы:

```text
компактный файл
быстро восстанавливается
хорошо для бэкапов
```

Минусы:

```text
можно потерять данные между snapshot-ами
```

Например Redis сохраняет snapshot раз в 5 минут.  
Если сервер упал, можно потерять последние 5 минут данных.

---

## 28.2 AOF

AOF — Append Only File.

Redis записывает каждую команду изменения в лог.

Пример:

```text
SET name Ivan
INCR counter
HSET user:1 name Ivan
```

При перезапуске Redis проигрывает эти команды заново.

Плюсы:

```text
меньше риск потери данных
```

Минусы:

```text
файл может быть больше
восстановление может быть дольше
```

---

## 28.3 appendfsync

Настройка AOF:

```text
appendfsync always
appendfsync everysec
appendfsync no
```

Самый популярный вариант:

```text
appendfsync everysec
```

Это означает:

```text
сохранять на диск примерно раз в секунду
```

---

## 28.4 Redis через Docker с AOF

```bash
docker run --name redis-aof \
  -p 6379:6379 \
  -d redis:7 redis-server --appendonly yes
```

---

# 29. Eviction: что делать, если память закончилась

Redis работает в памяти. Память не бесконечная.

Можно настроить максимум памяти:

```text
maxmemory 512mb
```

И политику удаления:

```text
maxmemory-policy allkeys-lru
```

---

## Основные политики

| Политика | Что делает |
|---|---|
| noeviction | не удаляет, новые записи получают ошибку |
| allkeys-lru | удаляет наименее недавно использованные ключи |
| volatile-lru | удаляет LRU только среди ключей с TTL |
| allkeys-random | удаляет случайные ключи |
| volatile-ttl | удаляет ключи с ближайшим истечением TTL |
| allkeys-lfu | удаляет наименее часто используемые |

Для кэша часто используют:

```text
allkeys-lru
```

или:

```text
allkeys-lfu
```

---

# 30. Безопасность Redis

## 30.1 Redis не должен быть открыт в интернет

Очень важно:

```text
Никогда не открывай Redis наружу без защиты.
```

Плохо:

```text
0.0.0.0:6379 доступен всем из интернета
```

Redis должен быть доступен только приложению.

---

## 30.2 Пароль

Запуск с паролем:

```bash
docker run --name redis-secure \
  -p 6379:6379 \
  -d redis:7 redis-server --requirepass mypassword
```

Python:

```python
r = redis.Redis(
    host="localhost",
    port=6379,
    password="mypassword",
    decode_responses=True
)
```

---

## 30.3 ACL

В Redis 6+ есть ACL — пользователи и права.

Пример в redis-cli:

```redis
ACL SETUSER appuser on >strongpassword ~app:* +get +set +del
```

Это означает:

```text
пользователь appuser включён
пароль strongpassword
может работать только с ключами app:*
разрешены команды get/set/del
```

---

# 31. Мониторинг Redis

## 31.1 INFO

```python
info = r.info()

print(info["used_memory_human"])
print(info["connected_clients"])
```

В CLI:

```redis
INFO
```

---

## 31.2 SLOWLOG

Показать медленные команды:

```redis
SLOWLOG GET 10
```

Python:

```python
logs = r.slowlog_get(10)

for log in logs:
    print(log)
```

---

## 31.3 MEMORY USAGE

```python
memory = r.memory_usage("user:1")
print(memory)
```

---

## 31.4 MONITOR

```redis
MONITOR
```

Показывает все команды в реальном времени.

На продакшене использовать осторожно.

---

# 32. Connection Pool

redis-py по умолчанию использует пул соединений.

Но можно создать явно:

```python
pool = redis.ConnectionPool(
    host="localhost",
    port=6379,
    db=0,
    decode_responses=True,
    max_connections=20
)

r = redis.Redis(connection_pool=pool)
```

Это полезно в web-приложениях.

---

# 33. Таймауты и устойчивость

Лучше не создавать Redis-клиент совсем без таймаутов.

```python
r = redis.Redis(
    host="localhost",
    port=6379,
    db=0,
    decode_responses=True,
    socket_timeout=3,
    socket_connect_timeout=3,
    health_check_interval=30
)
```

Параметры:

```python
socket_timeout=3
```

Максимальное время ожидания ответа.

```python
socket_connect_timeout=3
```

Максимальное время подключения.

```python
health_check_interval=30
```

Периодическая проверка соединения.

---

# 34. Async Redis в Python

Если используешь async-приложение, например FastAPI, можно использовать async Redis.

```python
import redis.asyncio as redis

async def main():
    r = redis.Redis(
        host="localhost",
        port=6379,
        decode_responses=True
    )

    await r.set("name", "Ivan")
    name = await r.get("name")

    print(name)

    await r.aclose()
```

---

## Пример с FastAPI

```python
from fastapi import FastAPI
import redis.asyncio as redis

app = FastAPI()

redis_client: redis.Redis | None = None


@app.on_event("startup")
async def startup():
    global redis_client
    redis_client = redis.Redis(
        host="localhost",
        port=6379,
        decode_responses=True
    )


@app.on_event("shutdown")
async def shutdown():
    await redis_client.aclose()


@app.get("/counter")
async def counter():
    value = await redis_client.incr("api:counter")
    return {"counter": value}
```

---

# 35. Реальный пример: сервис кэша пользователей

Допустим, у нас есть приложение, где пользователи хранятся в БД.

Для примера БД заменим словарём.

```python
import redis
import json
import time

r = redis.Redis(decode_responses=True)

DATABASE = {
    1: {
        "id": 1,
        "name": "Ivan",
        "age": 25
    },
    2: {
        "id": 2,
        "name": "Maria",
        "age": 30
    }
}


def get_user_from_db(user_id: int):
    print("Запрос в БД...")
    time.sleep(1)
    return DATABASE.get(user_id)


def get_user(user_id: int):
    cache_key = f"cache:user:{user_id}"

    cached = r.get(cache_key)

    if cached is not None:
        print("Из Redis")
        return json.loads(cached)

    print("Из БД")
    user = get_user_from_db(user_id)

    if user is None:
        return None

    r.setex(
        cache_key,
        60,
        json.dumps(user)
    )

    return user


def update_user(user_id: int, name: str, age: int):
    if user_id not in DATABASE:
        return None

    DATABASE[user_id]["name"] = name
    DATABASE[user_id]["age"] = age

    r.delete(f"cache:user:{user_id}")

    return DATABASE[user_id]


print(get_user(1))
print(get_user(1))

update_user(1, "Petr", 26)

print(get_user(1))
```

---

# 36. Rate Limiting

Rate limiting — ограничение частоты запросов.

Например:

```text
один пользователь может сделать максимум 100 запросов в минуту
```

---

## 36.1 Простой fixed window limiter

```python
def is_request_allowed(user_id: int, limit: int = 100, window: int = 60):
    key = f"rate:user:{user_id}"

    current = r.incr(key)

    if current == 1:
        r.expire(key, window)

    if current > limit:
        return False

    return True
```

Использование:

```python
for i in range(105):
    print(i, is_request_allowed(1, limit=5, window=60))
```

---

## 36.2 Недостаток fixed window

Если окно 60 секунд:

```text
12:00:59 пользователь сделал 100 запросов
12:01:00 пользователь сделал ещё 100 запросов
```

Фактически за 1 секунду он сделал 200 запросов.

Для junior-уровня важно знать этот недостаток.

Для более точных лимитов используют:

```text
sliding window
token bucket
leaky bucket
```

---

# 37. Сессии пользователей

Redis часто используют для хранения сессий.

```python
import uuid
import json

def create_session(user_id: int):
    session_id = str(uuid.uuid4())

    key = f"session:{session_id}"

    data = {
        "user_id": user_id
    }

    r.setex(
        key,
        3600,
        json.dumps(data)
    )

    return session_id


def get_session(session_id: str):
    key = f"session:{session_id}"

    data = r.get(key)

    if data is None:
        return None

    return json.loads(data)


def delete_session(session_id: str):
    r.delete(f"session:{session_id}")


session_id = create_session(1)

print(session_id)
print(get_session(session_id))
```

---

# 38. Идемпотентность

Идемпотентность — это когда повторный запрос не ломает систему.

Пример:

```text
Клиент отправил оплату.
Сеть зависла.
Клиент повторил запрос.
Нельзя списать деньги два раза.
```

Redis можно использовать для хранения idempotency key.

```python
def process_request(idempotency_key: str):
    key = f"idempotency:{idempotency_key}"

    was_set = r.set(key, "processing", nx=True, ex=3600)

    if not was_set:
        return "Запрос уже обрабатывался"

    try:
        print("Выполняем действие")
        return "OK"
    except Exception:
        r.delete(key)
        raise
```

---

# 39. Типичные ошибки новичков

## Ошибка 1. Использовать KEYS на продакшене

Плохо:

```python
r.keys("*")
```

Лучше:

```python
r.scan_iter("*")
```

---

## Ошибка 2. Не ставить TTL кэшу

Плохо:

```python
r.set("cache:user:1", data)
```

Лучше:

```python
r.setex("cache:user:1", 300, data)
```

---

## Ошибка 3. Хранить огромные значения

Плохо:

```text
один ключ -> 500 MB JSON
```

Лучше разбивать данные.

---

## Ошибка 4. Использовать Redis как единственную базу для важных данных

Redis может быть persistent, но чаще его используют как дополнительное хранилище.

---

## Ошибка 5. Делать read-modify-write без атомарности

Плохо:

```python
value = int(r.get("counter"))
r.set("counter", value + 1)
```

Лучше:

```python
r.incr("counter")
```

---

## Ошибка 6. Забывать сериализацию

Redis хранит строки/байты.

Если сохраняешь dict напрямую, будет ошибка.

Правильно:

```python
r.set("user", json.dumps(user))
```

и потом:

```python
user = json.loads(r.get("user"))
```

---

# 40. Что должен уверенно знать Junior+ по Redis

Ты должен понимать:

## Базовое

```text
что такое Redis
зачем он нужен
чем Redis отличается от PostgreSQL
что Redis хранит данные в памяти
что такое key-value
```

## Команды

```text
SET / GET / DEL / EXISTS
EXPIRE / TTL / SETEX
INCR / DECR
HSET / HGET / HGETALL
LPUSH / RPUSH / LPOP / BLPOP
SADD / SMEMBERS / SISMEMBER
ZADD / ZRANGE / ZREVRANGE
SCAN
```

## Python

```text
как подключиться через redis-py
как сохранять строки
как сохранять JSON
как использовать pipeline
как работать с TTL
как обрабатывать None
```

## Практика

```text
кэширование
сессии
счётчики
rate limiting
простые очереди
pub/sub
streams на базовом уровне
distributed lock на базовом уровне
```

## Продакшен-понимание

```text
не использовать KEYS *
ставить TTL кэшу
понимать persistence RDB/AOF
понимать eviction policies
защищать Redis паролем/сетью
следить за памятью
использовать таймауты
```

---

# 41. Мини-шпаргалка Redis + Python

```python
r.set("name", "Ivan")
r.get("name")
r.delete("name")
r.exists("name")

r.setex("token", 3600, "abc")
r.ttl("token")
r.expire("token", 60)

r.incr("counter")
r.decr("counter")

r.hset("user:1", mapping={"name": "Ivan", "age": 25})
r.hget("user:1", "name")
r.hgetall("user:1")

r.rpush("queue", "task1")
r.blpop("queue", timeout=10)

r.sadd("likes:1", "user:1")
r.smembers("likes:1")
r.sismember("likes:1", "user:1")

r.zadd("rating", {"user:1": 100})
r.zrevrange("rating", 0, 9, withscores=True)

for key in r.scan_iter("user:*"):
    print(key)
```

---

# 42. Практические задания

Чтобы закрепить Redis до Junior+, сделай самостоятельно:

## Задание 1

Сделай счётчик просмотров статьи:

```text
article:1:views
article:2:views
```

Функции:

```python
add_view(article_id)
get_views(article_id)
```

---

## Задание 2

Сделай кэш пользователей:

```python
get_user(user_id)
update_user(user_id, data)
```

Требования:

```text
данные брать из fake DB
кэшировать на 60 секунд
при update удалять кэш
```

---

## Задание 3

Сделай систему лайков:

```python
like(post_id, user_id)
unlike(post_id, user_id)
is_liked(post_id, user_id)
likes_count(post_id)
```

Используй Set.

---

## Задание 4

Сделай leaderboard:

```python
add_score(user_id, score)
get_top_10()
get_user_rank(user_id)
```

Используй Sorted Set.

---

## Задание 5

Сделай rate limiter:

```text
максимум 5 запросов за 60 секунд
```

---

## Задание 6

Сделай простую очередь задач:

```python
add_task(data)
worker()
```

Используй List и `BLPOP`.

---

# 43. Итоговая картина

Redis — это очень быстрый инструмент для задач, где нужны:

```text
скорость
простые структуры данных
временные данные
атомарные счётчики
кэш
очереди
сессии
лимиты
```

Самая важная мысль:

```text
Redis — не просто "быстрый словарь".
Redis — набор готовых структур данных и атомарных операций,
которые помогают решать реальные backend-задачи.
```

Если ты уверенно понимаешь:

```text
String
Hash
List
Set
Sorted Set
TTL
кэширование
pipeline
атомарность
lock
pub/sub
streams basics
persistence
eviction
security
```

то для уровня Junior+ по Redis у тебя уже очень хорошая база.
