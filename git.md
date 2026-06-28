# Полная документация: Git + GitHub + `.gitignore` с нуля до уверенного Junior+

## 0. Что ты изучишь

После этой документации ты должен уверенно понимать:

- что такое Git;
- что такое GitHub;
- чем Git отличается от GitHub;
- как создать репозиторий;
- как сохранять изменения;
- как работать с ветками;
- как объединять ветки;
- как решать конфликты;
- как работать с удалённым репозиторием;
- как делать Pull Request;
- как работать в команде;
- как правильно писать `.gitignore`;
- какие команды Git нужны junior-разработчику;
- как исправлять типичные ошибки.

---

# 1. Что такое Git

## 1.1 Простое объяснение

**Git** — это система контроля версий.

Она нужна, чтобы:

- сохранять историю изменений проекта;
- возвращаться к старым версиям файлов;
- работать над разными задачами параллельно;
- работать в команде;
- понимать, кто, когда и зачем изменил код.

Представь, что ты пишешь проект:

```text
project/
  index.html
  style.css
  script.js
```

Ты что-то меняешь каждый день.

Без Git у тебя могут быть файлы:

```text
project-final.zip
project-final-2.zip
project-final-2-fixed.zip
project-final-real.zip
project-final-real-last.zip
```

Это неудобно.

Git решает эту проблему.

Он хранит историю примерно так:

```text
Коммит 1: создал index.html
Коммит 2: добавил style.css
Коммит 3: исправил кнопку
Коммит 4: добавил форму регистрации
Коммит 5: исправил ошибку в форме
```

Каждый такой сохранённый этап называется **commit**.

---

# 2. Что такое GitHub

## 2.1 Простое объяснение

**GitHub** — это сайт/платформа, где можно хранить Git-репозитории онлайн.

Git работает у тебя на компьютере.

GitHub находится в интернете.

Пример:

```text
Твой компьютер:
  Git-репозиторий

GitHub:
  копия этого репозитория онлайн
```

GitHub нужен, чтобы:

- хранить код в облаке;
- делиться кодом;
- работать в команде;
- делать Pull Request;
- проверять код;
- вести issues, задачи, обсуждения;
- подключать CI/CD;
- показывать портфолио.

---

# 3. Git и GitHub — это не одно и то же

Очень важно понять:

| Git | GitHub |
|---|---|
| Программа | Сайт/платформа |
| Работает локально | Работает онлайн |
| Сохраняет историю кода | Хранит репозитории в интернете |
| Используется через терминал | Используется через браузер и Git |
| Можно использовать без GitHub | Нужен Git-репозиторий |

Можно использовать Git без GitHub.

Например:

```bash
git init
git add .
git commit -m "Initial commit"
```

Это уже Git.

Но чтобы отправить проект на GitHub, нужен удалённый репозиторий.

---

# 4. Основные понятия Git

## 4.1 Repository / Репозиторий

**Репозиторий** — это папка проекта, за которой следит Git.

Например:

```text
my-project/
  index.html
  style.css
  script.js
  .git/
```

Папка `.git` — это служебная папка Git.

В ней хранится:

- история изменений;
- информация о ветках;
- настройки репозитория;
- коммиты.

Обычно руками папку `.git` не трогают.

---

## 4.2 Commit / Коммит

**Коммит** — это сохранённое состояние проекта.

Можно представить как контрольную точку в игре.

Например:

```text
Commit 1: создал проект
Commit 2: добавил header
Commit 3: добавил footer
Commit 4: исправил баг в меню
```

У каждого коммита есть:

- уникальный ID;
- автор;
- дата;
- сообщение;
- список изменений.

Пример ID коммита:

```text
a3f5c9e7b1d2...
```

Обычно используют короткую версию:

```text
a3f5c9e
```

---

## 4.3 Working Directory / Рабочая директория

**Рабочая директория** — это обычные файлы проекта, которые ты видишь в редакторе.

Например:

```text
index.html
style.css
script.js
```

Когда ты меняешь файл, Git видит, что файл изменился.

---

## 4.4 Staging Area / Индекс

**Staging Area** — это промежуточная зона перед коммитом.

Процесс в Git выглядит так:

```text
1. Ты изменил файл
2. Добавил файл в staging area
3. Создал commit
```

Схема:

```text
Working Directory -> Staging Area -> Commit
```

Команды:

```bash
git add index.html
git commit -m "Add index page"
```

---

## 4.5 Branch / Ветка

**Ветка** — это отдельная линия разработки.

Например, у тебя есть стабильная версия проекта:

```text
main
```

Ты хочешь добавить авторизацию.

Чтобы не ломать стабильный код, создаёшь ветку:

```text
feature/auth
```

Схема:

```text
main:          A---B---C
                    \
feature/auth:        D---E
```

Пока ты работаешь в `feature/auth`, ветка `main` не изменяется.

---

## 4.6 Merge / Слияние

**Merge** — это объединение одной ветки с другой.

Например, ты закончил авторизацию в ветке `feature/auth`.

Теперь хочешь добавить её в `main`.

```bash
git checkout main
git merge feature/auth
```

---

## 4.7 Remote / Удалённый репозиторий

**Remote** — это удалённая копия репозитория, например на GitHub.

Обычно удалённый репозиторий называется:

```text
origin
```

Пример:

```bash
git remote -v
```

Результат:

```text
origin  https://github.com/user/project.git (fetch)
origin  https://github.com/user/project.git (push)
```

---

# 5. Установка Git

## 5.1 Windows

Скачай Git:

```text
https://git-scm.com/download/win
```

Установи с настройками по умолчанию.

После установки открой:

```text
Git Bash
```

Проверь версию:

```bash
git --version
```

Пример:

```bash
git version 2.45.0
```

---

## 5.2 macOS

Если установлен Homebrew:

```bash
brew install git
```

Проверка:

```bash
git --version
```

---

## 5.3 Linux Ubuntu/Debian

```bash
sudo apt update
sudo apt install git
```

Проверка:

```bash
git --version
```

---

# 6. Первичная настройка Git

После установки нужно указать имя и email.

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

Пример:

```bash
git config --global user.name "Ivan Petrov"
git config --global user.email "ivan@example.com"
```

Проверить настройки:

```bash
git config --list
```

Или отдельно:

```bash
git config user.name
git config user.email
```

---

## 6.1 Почему это важно

Каждый коммит хранит автора.

Пример:

```text
commit a3f5c9e
Author: Ivan Petrov <ivan@example.com>
Date: ...
```

Если email совпадает с email на GitHub, коммиты будут отображаться в твоём профиле.

---

# 7. Создание первого Git-репозитория

## 7.1 Создадим проект

```bash
mkdir my-first-project
cd my-first-project
```

Создадим файл:

```bash
echo "# My First Project" > README.md
```

Проверим файлы:

```bash
ls
```

---

## 7.2 Инициализируем Git

```bash
git init
```

Git ответит примерно так:

```text
Initialized empty Git repository in /path/my-first-project/.git/
```

Теперь папка стала Git-репозиторием.

---

## 7.3 Проверяем статус

```bash
git status
```

Пример:

```text
On branch main

No commits yet

Untracked files:
  README.md
```

Что значит `Untracked files`?

Это файлы, которые Git видит, но пока не отслеживает.

---

## 7.4 Добавляем файл в staging area

```bash
git add README.md
```

Проверяем:

```bash
git status
```

Пример:

```text
Changes to be committed:
  new file: README.md
```

Теперь файл готов к коммиту.

---

## 7.5 Создаём первый коммит

```bash
git commit -m "Initial commit"
```

Пример результата:

```text
[main a1b2c3d] Initial commit
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
```

Теперь история проекта содержит первый коммит.

---

# 8. Базовый цикл работы Git

Почти всегда ты работаешь так:

```text
1. Изменил файлы
2. Проверил статус
3. Добавил изменения
4. Сделал коммит
```

Команды:

```bash
git status
git add .
git commit -m "Your message"
```

Пример:

```bash
echo "Hello Git" >> README.md

git status
git add README.md
git commit -m "Update README"
```

---

# 9. Команда `git status`

## 9.1 Что делает

Показывает текущее состояние репозитория:

- какие файлы изменены;
- какие новые;
- какие удалены;
- что добавлено в staging area;
- на какой ты ветке.

Команда:

```bash
git status
```

Краткий вариант:

```bash
git status -s
```

Пример:

```text
 M README.md
?? index.html
```

Расшифровка:

```text
 M README.md   файл изменён, но не добавлен в staging
?? index.html  новый файл, Git его не отслеживает
```

---

# 10. Команда `git add`

## 10.1 Добавить один файл

```bash
git add index.html
```

## 10.2 Добавить несколько файлов

```bash
git add index.html style.css script.js
```

## 10.3 Добавить все изменения

```bash
git add .
```

Это добавляет:

- новые файлы;
- изменённые файлы;
- удалённые файлы.

## 10.4 Добавить все изменения во всём репозитории

```bash
git add -A
```

Обычно `git add .` достаточно, если ты находишься в корне проекта.

---

# 11. Команда `git commit`

## 11.1 Обычный коммит

```bash
git commit -m "Add header"
```

## 11.2 Хорошее сообщение коммита

Плохо:

```bash
git commit -m "fix"
git commit -m "changes"
git commit -m "asdf"
```

Хорошо:

```bash
git commit -m "Add login form"
git commit -m "Fix navbar layout on mobile"
git commit -m "Update README with installation steps"
```

Хороший коммит отвечает на вопрос:

```text
Что было сделано?
```

---

## 11.3 Стиль сообщений

Часто используют английский:

```bash
git commit -m "Add user profile page"
```

Можно использовать русский, но в командах чаще пишут на английском.

---

## 11.4 Conventional Commits

В реальных проектах часто используют стиль:

```text
type: description
```

Примеры:

```bash
git commit -m "feat: add user registration"
git commit -m "fix: correct email validation"
git commit -m "docs: update README"
git commit -m "refactor: simplify auth service"
git commit -m "test: add login tests"
```

Популярные типы:

| Тип | Значение |
|---|---|
| `feat` | новая функция |
| `fix` | исправление бага |
| `docs` | документация |
| `style` | форматирование |
| `refactor` | рефакторинг |
| `test` | тесты |
| `chore` | технические изменения |

---

# 12. История коммитов

## 12.1 Посмотреть историю

```bash
git log
```

Пример:

```text
commit a1b2c3d4...
Author: Ivan Petrov
Date: ...

    Initial commit
```

## 12.2 Короткий лог

```bash
git log --oneline
```

Пример:

```text
f3e2d1c Add login form
a1b2c3d Initial commit
```

## 12.3 Красивый граф

```bash
git log --oneline --graph --decorate --all
```

Пример:

```text
* f3e2d1c (HEAD -> main) Add login form
* a1b2c3d Initial commit
```

---

# 13. Просмотр изменений

## 13.1 Посмотреть изменения в файлах

```bash
git diff
```

Показывает изменения, которые ещё не добавлены в staging area.

## 13.2 Посмотреть staged изменения

```bash
git diff --staged
```

или:

```bash
git diff --cached
```

## 13.3 Посмотреть изменения конкретного файла

```bash
git diff README.md
```

---

# 14. Удаление и переименование файлов

## 14.1 Удалить файл через Git

```bash
git rm old-file.txt
git commit -m "Remove old file"
```

## 14.2 Переименовать файл через Git

```bash
git mv old-name.txt new-name.txt
git commit -m "Rename file"
```

Можно переименовать и обычным способом, но `git mv` удобнее.

---

# 15. Отмена изменений

Это очень важный раздел.

---

## 15.1 Отменить изменения в файле до `git add`

Ты изменил файл, но хочешь вернуть его к последнему коммиту.

```bash
git restore file.txt
```

Пример:

```bash
git restore README.md
```

Осторожно: изменения исчезнут.

---

## 15.2 Убрать файл из staging area

Ты сделал:

```bash
git add README.md
```

Но передумал добавлять файл.

```bash
git restore --staged README.md
```

Файл останется изменённым, но выйдет из staging area.

---

## 15.3 Отменить последний коммит, но оставить изменения

```bash
git reset --soft HEAD~1
```

Что произойдёт:

- последний коммит удалится;
- изменения останутся в staging area.

---

## 15.4 Отменить последний коммит и убрать изменения из staging

```bash
git reset --mixed HEAD~1
```

или просто:

```bash
git reset HEAD~1
```

Что произойдёт:

- последний коммит удалится;
- изменения останутся в файлах;
- staging очистится.

---

## 15.5 Полностью удалить последний коммит и изменения

```bash
git reset --hard HEAD~1
```

Очень осторожно.

Что произойдёт:

- последний коммит удалится;
- изменения исчезнут.

---

## 15.6 Безопасная отмена опубликованного коммита

Если коммит уже отправлен на GitHub, лучше использовать:

```bash
git revert <commit-id>
```

Пример:

```bash
git revert a1b2c3d
```

`revert` создаёт новый коммит, который отменяет изменения старого.

Это безопаснее для командной работы.

---

# 16. Ветки в Git

## 16.1 Зачем нужны ветки

Ветки нужны, чтобы работать над задачами отдельно.

Например:

```text
main              стабильная версия
feature/login     разработка логина
feature/profile   разработка профиля
fix/navbar         исправление navbar
```

Так ты не ломаешь основную ветку.

---

## 16.2 Посмотреть ветки

```bash
git branch
```

Пример:

```text
* main
  feature/login
```

Звёздочка показывает текущую ветку.

---

## 16.3 Создать ветку

```bash
git branch feature/login
```

---

## 16.4 Переключиться на ветку

```bash
git checkout feature/login
```

Современный вариант:

```bash
git switch feature/login
```

---

## 16.5 Создать и сразу перейти

Старый способ:

```bash
git checkout -b feature/login
```

Новый способ:

```bash
git switch -c feature/login
```

---

## 16.6 Удалить ветку

Если ветка уже слита:

```bash
git branch -d feature/login
```

Принудительно:

```bash
git branch -D feature/login
```

---

## 16.7 Пример работы с веткой

```bash
git switch -c feature/about-page

echo "<h1>About</h1>" > about.html

git add about.html
git commit -m "Add about page"

git switch main
git merge feature/about-page
```

---

# 17. Merge — слияние веток

## 17.1 Что такое merge

`merge` объединяет изменения из одной ветки в другую.

Обычно:

```bash
git switch main
git merge feature/login
```

Это значит:

```text
Взять изменения из feature/login и добавить их в main
```

---

## 17.2 Fast-forward merge

Если в `main` не было новых коммитов, Git просто передвинет указатель.

Пример:

```text
main:          A---B
                    \
feature:             C---D
```

После merge:

```text
main:          A---B---C---D
```

---

## 17.3 Merge commit

Если обе ветки развивались отдельно:

```text
main:          A---B---C
                    \
feature:             D---E
```

После merge:

```text
main:          A---B---C-------M
                    \         /
feature:             D---E---
```

`M` — merge commit.

---

# 18. Конфликты

## 18.1 Что такое конфликт

Конфликт возникает, когда Git не может сам понять, какие изменения оставить.

Пример.

В ветке `main` файл:

```js
const title = "Home Page";
```

В ветке `feature` тот же файл изменили:

```js
const title = "Main Page";
```

Git не знает, что выбрать.

---

## 18.2 Как выглядит конфликт

Файл может стать таким:

```js
<<<<<<< HEAD
const title = "Home Page";
=======
const title = "Main Page";
>>>>>>> feature/title
```

Значение:

```text
<<<<<<< HEAD
изменения текущей ветки
=======
разделитель
>>>>>>> feature/title
изменения другой ветки
```

---

## 18.3 Как решить конфликт

Нужно вручную отредактировать файл.

Например, выбрать итоговый вариант:

```js
const title = "Main Page";
```

Потом:

```bash
git add file.js
git commit
```

Если конфликт был во время merge, Git сам предложит сообщение коммита.

---

## 18.4 Полный пример конфликта

```bash
git switch main
echo "Hello main" > text.txt
git add text.txt
git commit -m "Add main text"

git switch -c feature/text
echo "Hello feature" > text.txt
git add text.txt
git commit -m "Change text in feature"

git switch main
echo "Hello from main updated" > text.txt
git add text.txt
git commit -m "Change text in main"

git merge feature/text
```

Git покажет конфликт.

Открываешь `text.txt`, исправляешь, потом:

```bash
git add text.txt
git commit
```

---

# 19. Rebase

## 19.1 Что такое rebase

`rebase` переносит коммиты текущей ветки поверх другой ветки.

Пример до rebase:

```text
main:      A---B---C
                \
feature:         D---E
```

После:

```text
main:      A---B---C
                    \
feature:             D'---E'
```

Коммиты `D` и `E` как будто созданы заново поверх `C`.

---

## 19.2 Когда использовать rebase

Часто используют, чтобы обновить feature-ветку актуальным `main`.

```bash
git switch feature/login
git rebase main
```

---

## 19.3 Merge vs Rebase

| Merge | Rebase |
|---|---|
| Сохраняет историю как есть | Делает историю линейной |
| Создаёт merge commit | Переписывает коммиты |
| Безопаснее для новичков | Нужно быть аккуратнее |
| Хорош для командной истории | Хорош для чистой истории |

---

## 19.4 Важное правило

Не делай rebase публичных веток, которыми уже пользуются другие люди, если не понимаешь последствия.

Плохо:

```bash
git switch main
git rebase some-branch
```

Обычно `main` не ребейзят.

---

# 20. GitHub: создание аккаунта и репозитория

## 20.1 Создай аккаунт

Перейди:

```text
https://github.com
```

Создай аккаунт.

---

## 20.2 Создай новый репозиторий

На GitHub:

```text
New repository
```

Укажи:

```text
Repository name: my-first-project
Visibility: Public или Private
```

Опции:

- `Public` — виден всем;
- `Private` — виден только тебе и тем, кому дашь доступ.

---

# 21. Связь локального Git с GitHub

## 21.1 Если проект уже есть локально

Допустим, у тебя локально есть проект с Git.

На GitHub создай пустой репозиторий.

Потом в терминале:

```bash
git remote add origin https://github.com/username/my-first-project.git
```

Проверить:

```bash
git remote -v
```

Отправить код:

```bash
git push -u origin main
```

---

## 21.2 Что значит `origin`

`origin` — стандартное имя удалённого репозитория.

Команда:

```bash
git remote add origin URL
```

означает:

```text
Добавить удалённый репозиторий с именем origin
```

---

## 21.3 Что значит `git push -u origin main`

```bash
git push -u origin main
```

Расшифровка:

```text
git push      отправить коммиты
-u            связать локальную ветку с удалённой
origin        имя удалённого репозитория
main          имя ветки
```

После этого можно писать просто:

```bash
git push
```

---

# 22. Клонирование репозитория

Если проект уже есть на GitHub, его можно скачать:

```bash
git clone https://github.com/username/project.git
```

Пример:

```bash
git clone https://github.com/facebook/react.git
```

После этого появится папка:

```bash
cd react
```

---

# 23. Push, Pull, Fetch

## 23.1 `git push`

Отправляет твои локальные коммиты на GitHub.

```bash
git push
```

Или явно:

```bash
git push origin main
```

---

## 23.2 `git fetch`

Скачивает информацию с GitHub, но не меняет твои файлы.

```bash
git fetch
```

Полезно, чтобы узнать, что изменилось удалённо.

---

## 23.3 `git pull`

Скачивает изменения и сразу объединяет их с твоей веткой.

```bash
git pull
```

По сути часто похоже на:

```bash
git fetch
git merge origin/main
```

---

## 23.4 Когда делать pull

Перед началом работы:

```bash
git pull
```

Перед push:

```bash
git pull
git push
```

Особенно если работаешь в команде.

---

# 24. Pull Request на GitHub

## 24.1 Что такое Pull Request

**Pull Request**, или **PR**, — это запрос на добавление изменений из одной ветки в другую.

Обычно:

```text
feature/login -> main
```

Ты говоришь команде:

```text
Я сделал задачу. Проверьте мой код и добавьте его в main.
```

---

## 24.2 Типичный процесс работы через PR

```text
1. Создать ветку от main
2. Сделать изменения
3. Закоммитить
4. Запушить ветку на GitHub
5. Создать Pull Request
6. Получить review
7. Исправить замечания
8. Merge в main
```

---

## 24.3 Пример

```bash
git switch main
git pull

git switch -c feature/login-page

# изменяешь файлы

git add .
git commit -m "feat: add login page"

git push -u origin feature/login-page
```

После push GitHub обычно покажет кнопку:

```text
Compare & pull request
```

Нажимаешь, создаёшь PR.

---

## 24.4 Хорошее описание Pull Request

Пример:

```markdown
## Что сделано

- Добавлена страница логина
- Добавлена форма email/password
- Добавлена базовая валидация

## Как проверить

1. Открыть `/login`
2. Ввести email и password
3. Нажать Submit

## Скриншоты

...
```

---

# 25. Работа в команде: правильный workflow

## 25.1 Самый частый junior workflow

```bash
git switch main
git pull

git switch -c feature/task-name

# работаешь над задачей

git add .
git commit -m "feat: implement task"

git push -u origin feature/task-name
```

Потом создаёшь Pull Request.

После merge:

```bash
git switch main
git pull
git branch -d feature/task-name
```

---

## 25.2 Никогда не работай напрямую в `main`

В большинстве команд нельзя пушить напрямую в `main`.

Плохо:

```bash
git switch main
# пишешь код
git add .
git commit -m "some changes"
git push
```

Хорошо:

```bash
git switch -c feature/some-changes
```

---

## 25.3 Имена веток

Хорошие имена:

```text
feature/login-page
feature/user-profile
fix/navbar-mobile
fix/email-validation
docs/update-readme
refactor/auth-service
```

Плохие:

```text
test
new
ivan
branch1
fix
```

---

# 26. `.gitignore`

## 26.1 Что такое `.gitignore`

`.gitignore` — это файл, в котором указывается, какие файлы Git должен игнорировать.

Например, в проекте есть:

```text
node_modules/
.env
dist/
```

Эти файлы не нужно хранить в Git.

---

## 26.2 Зачем нужен `.gitignore`

Он нужен, чтобы не добавлять в репозиторий:

- зависимости;
- временные файлы;
- секретные данные;
- сборки;
- логи;
- кэш;
- файлы IDE;
- системные файлы ОС.

---

## 26.3 Пример `.gitignore`

```gitignore
# Dependencies
node_modules/

# Environment variables
.env

# Build
dist/
build/

# Logs
*.log

# OS files
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
```

---

# 27. Как работает `.gitignore`

## 27.1 Важно

`.gitignore` игнорирует только файлы, которые ещё не отслеживаются Git.

Если файл уже был добавлен в Git, добавление его в `.gitignore` не удалит его из истории автоматически.

Пример:

```bash
git add .env
git commit -m "Add env"
```

Потом ты добавил `.env` в `.gitignore`.

Git всё равно продолжит отслеживать `.env`.

Чтобы перестать отслеживать:

```bash
git rm --cached .env
git commit -m "Remove env from tracking"
```

Файл останется локально, но исчезнет из Git.

---

## 27.2 Синтаксис `.gitignore`

### Игнорировать файл

```gitignore
.env
```

### Игнорировать папку

```gitignore
node_modules/
```

### Игнорировать все файлы с расширением

```gitignore
*.log
```

### Игнорировать файлы в любой папке

```gitignore
*.tmp
```

### Игнорировать конкретный файл в конкретной папке

```gitignore
config/local.json
```

### Игнорировать всё внутри папки

```gitignore
cache/*
```

### Но оставить один файл

```gitignore
cache/*
!cache/.gitkeep
```

`!` означает исключение из игнорирования.

---

## 27.3 Комментарии

```gitignore
# Это комментарий
node_modules/
```

---

## 27.4 Примеры `.gitignore` для разных проектов

### Node.js / JavaScript / React / Vue

```gitignore
# Dependencies
node_modules/

# Build
dist/
build/

# Environment variables
.env
.env.local
.env.development.local
.env.test.local
.env.production.local

# Logs
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*

# Cache
.cache/
.parcel-cache/

# OS
.DS_Store
Thumbs.db

# IDE
.vscode/
.idea/
```

---

### Python

```gitignore
# Python cache
__pycache__/
*.py[cod]

# Virtual environments
venv/
env/
.venv/

# Environment variables
.env

# Build
build/
dist/
*.egg-info/

# Tests
.pytest_cache/
.coverage
htmlcov/

# IDE
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db
```

---

### Java / Maven / Gradle

```gitignore
# Build
target/
build/
out/

# Gradle
.gradle/

# IDE
.idea/
*.iml
.vscode/

# Logs
*.log

# OS
.DS_Store
Thumbs.db
```

---

### C# / .NET

```gitignore
bin/
obj/
.vs/
*.user
*.suo

# Rider
.idea/

# OS
.DS_Store
Thumbs.db
```

---

# 28. Что нельзя коммитить

Очень важно.

Не коммить:

```text
.env
passwords.txt
private_key.pem
id_rsa
node_modules/
venv/
dist/ если это генерируемая сборка
логи
кэш
личные настройки IDE
```

Особенно нельзя коммитить:

```text
API keys
пароли
токены
секретные ключи
данные пользователей
```

Если случайно закоммитил секрет, просто удалить коммит недостаточно. Секрет мог попасть в историю. Нужно:

1. удалить секрет из истории;
2. заменить токен/пароль в сервисе;
3. считать старый секрет скомпрометированным.

---

# 29. GitHub authentication: HTTPS и SSH

## 29.1 HTTPS

Пример URL:

```text
https://github.com/username/project.git
```

GitHub больше не принимает обычный пароль для Git-операций.

Нужно использовать:

- Personal Access Token;
- или GitHub CLI;
- или SSH.

---

## 29.2 SSH

SSH удобнее для постоянной работы.

URL выглядит так:

```text
git@github.com:username/project.git
```

---

## 29.3 Создание SSH-ключа

```bash
ssh-keygen -t ed25519 -C "your@email.com"
```

Нажимай Enter на вопросы, если не знаешь что выбирать.

Публичный ключ обычно здесь:

```bash
~/.ssh/id_ed25519.pub
```

Показать ключ:

```bash
cat ~/.ssh/id_ed25519.pub
```

Скопируй его и добавь на GitHub:

```text
GitHub -> Settings -> SSH and GPG keys -> New SSH key
```

Проверить подключение:

```bash
ssh -T git@github.com
```

Если всё хорошо:

```text
Hi username! You've successfully authenticated...
```

---

# 30. Изменение remote URL

Посмотреть remote:

```bash
git remote -v
```

Изменить HTTPS на SSH:

```bash
git remote set-url origin git@github.com:username/project.git
```

Проверить:

```bash
git remote -v
```

---

# 31. Stash

## 31.1 Что такое stash

`stash` временно сохраняет незакоммиченные изменения.

Ситуация:

Ты работаешь над задачей, но срочно нужно переключиться на другую ветку.

Git может не дать переключиться, если есть изменения.

Можно сделать:

```bash
git stash
```

Изменения временно спрячутся.

---

## 31.2 Вернуть изменения

```bash
git stash pop
```

---

## 31.3 Посмотреть список stash

```bash
git stash list
```

---

## 31.4 Сохранить с описанием

```bash
git stash push -m "WIP login form"
```

---

## 31.5 Применить конкретный stash

```bash
git stash apply stash@{0}
```

`apply` применяет, но не удаляет stash.

`pop` применяет и удаляет stash.

---

# 32. Tags / теги

## 32.1 Что такое tag

Тег — это метка на конкретном коммите.

Часто используется для версий:

```text
v1.0.0
v1.1.0
v2.0.0
```

---

## 32.2 Создать тег

```bash
git tag v1.0.0
```

---

## 32.3 Создать annotated tag

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
```

---

## 32.4 Отправить тег на GitHub

```bash
git push origin v1.0.0
```

Отправить все теги:

```bash
git push --tags
```

---

# 33. GitHub Issues

## 33.1 Что такое Issues

Issues — это задачи, баги, предложения внутри GitHub-репозитория.

Примеры:

```text
Bug: Login form does not validate email
Feature: Add dark theme
Docs: Update installation guide
```

---

## 33.2 Хорошая issue

```markdown
## Описание

При вводе неправильного email форма всё равно отправляется.

## Шаги воспроизведения

1. Открыть /login
2. Ввести `abc`
3. Нажать Submit

## Ожидаемый результат

Появляется ошибка "Invalid email"

## Фактический результат

Форма отправляется

## Окружение

Chrome 125, Windows 11
```

---

# 34. GitHub README

## 34.1 Что такое README.md

`README.md` — главный файл описания проекта.

GitHub показывает его на странице репозитория.

---

## 34.2 Хороший README

Пример структуры:

```markdown
# Project Name

Краткое описание проекта.

## Features

- Авторизация
- Профиль пользователя
- Тёмная тема

## Tech Stack

- React
- TypeScript
- Vite

## Installation

```bash
npm install
```

## Development

```bash
npm run dev
```

## Build

```bash
npm run build
```

## Environment Variables

Создайте `.env`:

```env
VITE_API_URL=http://localhost:3000
```

## Screenshots

...

## License

MIT
```

---

# 35. GitHub Fork

## 35.1 Что такое fork

Fork — это копия чужого репозитория в твоём GitHub-аккаунте.

Используется, когда ты не имеешь прав пушить в оригинальный репозиторий.

---

## 35.2 Типичный процесс

```text
1. Нажать Fork на GitHub
2. Клонировать свой fork
3. Создать ветку
4. Сделать изменения
5. Запушить
6. Создать Pull Request в оригинальный репозиторий
```

---

## 35.3 Upstream

Оригинальный репозиторий часто называют `upstream`.

Добавить upstream:

```bash
git remote add upstream https://github.com/original/project.git
```

Проверить:

```bash
git remote -v
```

Получить изменения из оригинала:

```bash
git fetch upstream
git switch main
git merge upstream/main
```

---

# 36. Частые команды Git

## 36.1 Базовые

```bash
git init
git status
git add .
git commit -m "message"
git log --oneline
```

## 36.2 Ветки

```bash
git branch
git switch -c feature/name
git switch main
git merge feature/name
git branch -d feature/name
```

## 36.3 Remote

```bash
git remote -v
git remote add origin URL
git push -u origin main
git pull
git fetch
```

## 36.4 Отмена

```bash
git restore file.txt
git restore --staged file.txt
git reset --soft HEAD~1
git reset --hard HEAD~1
git revert commit_id
```

## 36.5 Stash

```bash
git stash
git stash list
git stash pop
```

---

# 37. Типичный рабочий день junior-разработчика

Допустим, тебе дали задачу:

```text
Добавить страницу профиля пользователя
```

Ты делаешь:

```bash
git switch main
git pull
```

Создаёшь ветку:

```bash
git switch -c feature/user-profile-page
```

Работаешь с файлами.

Проверяешь:

```bash
git status
git diff
```

Добавляешь:

```bash
git add .
```

Коммитишь:

```bash
git commit -m "feat: add user profile page"
```

Пушишь:

```bash
git push -u origin feature/user-profile-page
```

Создаёшь Pull Request на GitHub.

После замечаний исправляешь код:

```bash
git add .
git commit -m "fix: update profile page layout"
git push
```

После merge:

```bash
git switch main
git pull
git branch -d feature/user-profile-page
```

---

# 38. Типичные ошибки и решения

## 38.1 `fatal: not a git repository`

Ошибка:

```text
fatal: not a git repository
```

Значит ты не внутри Git-репозитория.

Решение:

```bash
cd path/to/project
```

или инициализировать:

```bash
git init
```

---

## 38.2 `nothing to commit`

Сообщение:

```text
nothing to commit, working tree clean
```

Это не ошибка.

Значит нет изменений для коммита.

---

## 38.3 `rejected because remote contains work`

Ошибка при push:

```text
rejected because the remote contains work that you do not have locally
```

Значит на GitHub есть изменения, которых нет у тебя.

Решение:

```bash
git pull
git push
```

Если есть конфликты — решить их.

---

## 38.4 Неправильный remote

Проверить:

```bash
git remote -v
```

Изменить:

```bash
git remote set-url origin URL
```

---

## 38.5 Забыл добавить файл в коммит

Если коммит ещё не отправлен:

```bash
git add forgotten-file.txt
git commit --amend
```

`--amend` изменяет последний коммит.

Если коммит уже отправлен в общую ветку, лучше сделать новый коммит.

---

# 39. `git commit --amend`

## 39.1 Изменить последний коммит

Например, ты сделал:

```bash
git commit -m "Add login"
```

Но забыл файл:

```bash
git add login.css
git commit --amend
```

Откроется редактор сообщения.

Можно изменить сообщение сразу:

```bash
git commit --amend -m "feat: add login page"
```

Важно:

Если коммит уже был запушен, `amend` переписывает историю.

---

# 40. Force push

## 40.1 Что такое force push

```bash
git push --force
```

Принудительно перезаписывает удалённую ветку.

Это опасно.

Лучше использовать:

```bash
git push --force-with-lease
```

Он безопаснее, потому что проверяет, не появились ли чужие изменения.

---

## 40.2 Когда используется

Например:

- ты сделал `rebase`;
- ты сделал `commit --amend`;
- нужно обновить свою feature-ветку.

Не делай force push в `main`.

---

# 41. GitHub Actions кратко

## 41.1 Что это

GitHub Actions — автоматизация на GitHub.

Например:

- запуск тестов при Pull Request;
- сборка проекта;
- деплой;
- линтер.

---

## 41.2 Пример workflow

Файл:

```text
.github/workflows/ci.yml
```

Пример для Node.js:

```yaml
name: CI

on:
  pull_request:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm install

      - name: Run tests
        run: npm test
```

Junior не обязан глубоко знать Actions, но должен понимать идею.

---

# 42. Полезные настройки Git

## 42.1 Сделать `main` веткой по умолчанию

```bash
git config --global init.defaultBranch main
```

---

## 42.2 Включить цветной вывод

```bash
git config --global color.ui auto
```

---

## 42.3 Настроить редактор

VS Code:

```bash
git config --global core.editor "code --wait"
```

---

# 43. Полезные алиасы

Можно сделать сокращения:

```bash
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.cm "commit -m"
git config --global alias.lg "log --oneline --graph --decorate --all"
```

Теперь:

```bash
git st
git lg
```

---

# 44. Что junior должен знать уверенно

Junior+ должен уверенно уметь:

- создать Git-репозиторий;
- сделать commit;
- посмотреть status/diff/log;
- создать ветку;
- переключиться между ветками;
- сделать merge;
- решить простой конфликт;
- подключить GitHub remote;
- сделать push/pull/fetch;
- создать Pull Request;
- понимать `.gitignore`;
- не коммитить секреты;
- пользоваться stash;
- понимать разницу merge/rebase;
- уметь откатить изменения;
- читать историю коммитов;
- работать по workflow через feature branches.

---

# 45. Мини-практика

## Задание 1

Создай проект:

```bash
mkdir git-practice
cd git-practice
git init
```

Создай README:

```bash
echo "# Git Practice" > README.md
```

Закоммить:

```bash
git add README.md
git commit -m "Initial commit"
```

---

## Задание 2

Создай страницу:

```bash
echo "<h1>Hello</h1>" > index.html
git add index.html
git commit -m "Add index page"
```

---

## Задание 3

Создай ветку:

```bash
git switch -c feature/styles
```

Добавь CSS:

```bash
echo "h1 { color: red; }" > style.css
git add style.css
git commit -m "Add styles"
```

Вернись в main и слей:

```bash
git switch main
git merge feature/styles
```

---

## Задание 4

Создай `.gitignore`:

```bash
echo "node_modules/" > .gitignore
echo ".env" >> .gitignore
git add .gitignore
git commit -m "Add gitignore"
```

---

## Задание 5

Создай GitHub-репозиторий и отправь проект:

```bash
git remote add origin https://github.com/username/git-practice.git
git push -u origin main
```

---

# 46. Главная шпаргалка

```bash
# Проверка
git status
git log --oneline
git diff

# Добавление и коммит
git add .
git commit -m "message"

# Ветки
git branch
git switch -c feature/name
git switch main
git merge feature/name

# GitHub
git remote -v
git push
git pull
git fetch

# Отмена
git restore file.txt
git restore --staged file.txt
git reset --soft HEAD~1
git revert commit_id

# Stash
git stash
git stash pop
```

---

# 47. Самое важное правило Git

Перед коммитом всегда делай:

```bash
git status
git diff
```

Перед push в командном проекте часто делай:

```bash
git pull
```

Для новой задачи:

```bash
git switch main
git pull
git switch -c feature/task-name
```

Не коммить:

```text
.env
пароли
токены
node_modules
кэш
```

---

# 48. Итоговая картина

Git — это инструмент для истории проекта.

GitHub — место, где эта история хранится онлайн и где команда обсуждает изменения.

Обычный процесс:

```text
Создал ветку
Изменил код
Сделал commit
Отправил на GitHub
Создал Pull Request
Получил review
Исправил
Слил в main
```

Если ты уверенно понимаешь и практикуешь всё из этой документации, ты уже владеешь Git и GitHub на уровне уверенного junior / junior+.
