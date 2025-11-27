# 🚀 ШПАРГАЛКА ПО КОМАНДАМ GIT

## 📋 ОГЛАВЛЕНИЕ
1. [Настройка и конфигурация](#настройка-и-конфигурация)
2. [Создание репозиториев](#создание-репозиториев)
3. [Внесение изменений](#внесение-изменений)
4. [Просмотр истории и сравнений](#просмотр-истории-и-сравнений)
5. [Ветвление и слияние](#ветвление-и-слияние)
6. [Совместная работа](#совместная-работа)
7. [Отмена изменений](#отмена-изменений)
8. [Временное сохранение](#временное-сохранение)
9. [Игнорирование файлов](#игнорирование-файлов)
10. [Продвинутые техники](#продвинутые-техники)
11. [🗂️ Продвинутая работа с файлами](#продвинутая-работа-с-файлами)
12. [🏷️ Теги (расширенный раздел)](#теги-расширенный-раздел)
13. [🎭 Git Worktrees](#git-worktrees)
14. [📝 ПОЛНЫЙ .gitignore](#полный-gitignore)
15. [🏷️ ПОЛНОЕ руководство по именованию](#полное-руководство-по-именованию)

---

## НАСТРОЙКА И КОНФИГУРАЦИЯ

### Базовая настройка пользователя
```bash
# Установка имени (обязательно для коммитов)
git config --global user.name "Иван Иванов"

# Установка email (обязательно для коммитов)
git config --global user.email "ivan@example.com"

# Проверка настроек
git config --list
git config user.name          # Показать только имя
git config user.email         # Показать только email
```

### Настройка редактора
```bash
# Для разных операционных систем и редакторов:
git config --global core.editor "code --wait"              # VS Code
git config --global core.editor "nano"                     # Nano
git config --global core.editor "vim"                      # Vim
git config --global core.editor "notepad++ -multiInst -notabbar -nosession -noPlugin"  # Notepad++
```

### Полезные настройки
```bash
# Включение цветного вывода
git config --global color.ui auto

# Настройка окончаний строк
git config --global core.autocrlf true        # Windows
git config --global core.autocrlf input       # Linux/Mac

# Создание алиасов (сокращений)
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
```

### Уровни конфигурации
```bash
# Системные настройки (для всех пользователей)
git config --system core.editor "vim"

# Глобальные настройки (для текущего пользователя)
git config --global user.name "Иван"

# Локальные настройки (для конкретного репозитория)
git config --local user.email "ivan@company.com"

# Приоритет: локальные > глобальные > системные
```

---

## СОЗДАНИЕ РЕПОЗИТОРИЕВ

### Инициализация нового репозитория
```bash
# Создать репозиторий в текущей папке
git init

# Создать репозиторий в указанной папке
git init my-project

# Просмотр скрытой папки .git
ls -la .git/
```

### Клонирование существующих репозиториев
```bash
# Базовое клонирование
git clone https://github.com/username/repository.git

# Клонирование в конкретную папку
git clone https://github.com/username/repository.git my-folder

# Клонирование по SSH
git clone git@github.com:username/repository.git

# Клонирование определенной ветки
git clone -b develop https://github.com/username/repository.git

# Клонирование с ограничением глубины истории
git clone --depth 1 https://github.com/username/repository.git
```

### Проверка состояния репозитория
```bash
# Проверить, является ли папка репозиторием Git
git rev-parse --git-dir

# Показать корневую папку репозитория
git rev-parse --show-toplevel
```

---

## ВНЕСЕНИЕ ИЗМЕНЕНИЙ

### Команда `git status` - просмотр состояния
```bash
# Полная информация о состоянии
git status

# Краткий вывод
git status -s
git status --short

# Интерпретация краткого вывода:
# ??  - неотслеживаемый файл
# A   - файл добавлен в индекс
# M   - файл изменен
# M   (в левой колонке) - изменения не в индексе
# M  (в правой колонке) - изменения в индексе
# D   - файл удален
# R   - файл переименован
```

### Команда `git add` - добавление в индекс
```bash
# Добавить конкретный файл
git add index.html

# Добавить все файлы в текущей папке
git add .

# Добавить все файлы в проекте
git add --all
git add -A

# Добавить все файлы с определенным расширением
git add *.js
git add *.css

# Добавить всю папку
git add src/

# Интерактивное добавление
git add -i
git add -p                      # Пошаговое добавление изменений

# Добавить только измененные/удаленные файлы (без новых)
git add -u
```

### Команда `git commit` - создание коммитов
```bash
# Простой коммит с сообщением
git commit -m "Добавлена главная страница"

# Коммит с подробным описанием
git commit -m "Заголовок" -m "Подробное описание изменений"

# Добавить изменения в отслеживаемых файлах и сделать коммит
git commit -am "Сообщение"

# Открыть редактор для написания сообщения
git commit

# Изменить последний коммит
git commit --amend
git commit --amend -m "Новое сообщение"

# Изменить последний коммит без изменения сообщения
git commit --amend --no-edit

# Создать пустой коммит (без изменений)
git commit --allow-empty -m "Пустой коммит"
```

### Формат сообщений коммитов
```markdown
Тип: Краткое описание (до 50 символов)

Подробное описание (если нужно)
- Что было изменено
- Почему это было изменено
- Какие проблемы решены

Типы коммитов:
feat:     Новая функциональность
fix:      Исправление ошибки
docs:     Изменения в документации
style:    Форматирование, отсутствующие точки с запятой и т.д.
refactor: Рефакторинг кода
test:     Добавление тестов
chore:    Обновление сборщика, задач и т.д.
```

---

## ПРОСМОТР ИСТОРИИ И СРАВНЕНИЙ

### Команда `git log` - просмотр истории
```bash
# Базовая история
git log

# Компактный вывод
git log --oneline

# Графическое представление ветвления
git log --graph --oneline

# Ограничение количества коммитов
git log -5
git log -n 10

# Показать историю за период
git log --since="2023-01-01"
git log --until="2023-12-31"
git log --since="1 week ago"
git log --since="2 months 1 day 3 hours ago"

# Фильтрация по автору
git log --author="Иван"
git log --author="ivan@example.com"

# Поиск по сообщению коммита
git log --grep="bugfix"
git log --grep="feature" -i        # Без учета регистра

# Показать изменения в файлах
git log -p
git log -p index.html

# Статистика по файлам
git log --stat
git log --numstat

# Показать только коммиты, затрагивающие определенный файл
git log --follow index.html

# Показать историю в формате raw
git log --pretty=raw
```

### Команда `git show` - детали коммита
```bash
# Показать последний коммит
git show

# Показать конкретный коммит
git show abc123
git show HEAD~2

# Показать только изменения файлов
git show --stat abc123

# Показать коммит в определенном формате
git show --pretty=fuller abc123
```

### Команда `git diff` - сравнение изменений
```bash
# Сравнение рабочей директории и индекса
git diff

# Сравнение индекса и последнего коммита
git diff --staged
git diff --cached

# Сравнение рабочей директории и последнего коммита
git diff HEAD

# Сравнение двух коммитов
git diff abc123 def456

# Сравнение двух веток
git diff main develop
git diff main..develop

# Сравнение с общим предком
git diff main...develop

# Показать только имена измененных файлов
git diff --name-only

# Показать статистику изменений
git diff --stat

# Игнорировать пробельные изменения
git diff -w
git diff --ignore-all-space

# Сравнение конкретного файла между коммитами
git diff abc123:file.txt def456:file.txt
```

### Другие команды просмотра
```bash
# Показать кто менял файл
git blame index.html
git blame -L 10,20 index.html    # Конкретные строки

# Поиск изменений, добавляющих/удаляющих строку
git log -S "function_name"
git log -G "regex_pattern"

# Показать историю ссылок
git reflog
```

---

## ВЕТВЛЕНИЕ И СЛИЯНИЕ

### Команда `git branch` - управление ветками
```bash
# Список локальных веток
git branch

# Список всех веток (локальных и удаленных)
git branch -a

# Список веток с последним коммитом
git branch -v

# Список веток, содержащих определенный коммит
git branch --contains abc123

# Создание новой ветки
git branch new-feature

# Удаление ветки (безопасное)
git branch -d completed-feature

# Принудительное удаление ветки
git branch -D abandoned-feature

# Переименование текущей ветки
git branch -m new-name

# Переименование другой ветки
git branch -m old-name new-name

# Настройка связи с удаленной веткой
git branch --set-upstream-to=origin/develop develop
```

### Команда `git checkout` - переключение между ветками
```bash
# Переключиться на существующую ветку
git checkout develop

# Создать и переключиться на новую ветку
git checkout -b new-feature

# Переключиться на предыдущую ветку
git checkout -

# Переключиться на коммит (detached HEAD)
git checkout abc123

# Восстановить файл из другого коммита/ветки
git checkout develop -- file.txt

# Создать ветку от определенного коммита
git checkout -b new-branch abc123
```

### Команда `git switch` - новое переключение (Git 2.23+)
```bash
# Переключиться на ветку
git switch develop

# Создать и переключиться на новую ветку
git switch -c new-feature

# Вернуться к предыдущей ветке
git switch -

# Переключиться на коммит
git switch --detach abc123
```

### Команда `git merge` - слияние веток
```bash
# Слияние ветки в текущую
git merge feature-branch

# Слияние без fast-forward (всегда создает коммит слияния)
git merge --no-ff feature-branch

# Слияние с отключением коммита
git merge --no-commit feature-branch

# Прервать слияние при конфликте
git merge --abort

# Слияние с стратегией
git merge -s ort feature-branch    # Новая стратегия по умолчанию
git merge -s recursive feature-branch
```

### Команда `git rebase` - перебазирование
```bash
# Перебазирование текущей ветки на другую
git rebase main

# Интерактивное перебазирование
git rebase -i HEAD~5
git rebase -i abc123

# Продолжить перебазирование после разрешения конфликтов
git rebase --continue

# Пропустить текущий коммит при конфликте
git rebase --skip

# Прервать перебазирование
git rebase --abort

# Перебазирование с сохранением слияний
git rebase -p feature-branch
```

### Разрешение конфликтов слияния
```bash
# После конфликта Git показывает:
# both modified:   file.txt

# 1. Открыть файл с конфликтами:
<<<<<<< HEAD
Ваша версия кода
=======
Версия из ветки
>>>>>>> branch-name

# 2. Редактировать файл, оставляя нужный код
# 3. Добавить разрешенный файл:
git add file.txt

# 4. Завершить слияние:
git commit
# ИЛИ для rebase:
git rebase --continue
```

---

## СОВМЕСТНАЯ РАБОТА

### Команда `git remote` - управление удаленными репозиториями
```bash
# Показать список удаленных репозиториев
git remote
git remote -v                  # С URL

# Добавить удаленный репозиторий
git remote add origin https://github.com/user/repo.git

# Показать информацию об удаленном репозитории
git remote show origin

# Переименовать удаленный репозиторий
git remote rename origin upstream

# Удалить удаленный репозиторий
git remote remove origin
```

### Команда `git fetch` - получение изменений
```bash
# Получить все изменения из всех удаленных репозиториев
git fetch

# Получить изменения из конкретного удаленного репозитория
git fetch origin

# Получить конкретную ветку
git fetch origin main

# Получить и очистить устаревшие ветки
git fetch --prune
git fetch -p

# Получить все ветки и теги
git fetch --all
```

### Команда `git pull` - получение и объединение изменений
```bash
# Стандартный pull (fetch + merge)
git pull
git pull origin main

# Pull с перебазированием
git pull --rebase
git pull -r

# Pull только определенной ветки
git pull origin feature-branch

# Принудительный pull (перезапись локальных изменений)
git pull --force
```

### Команда `git push` - отправка изменений
```bash
# Первая отправка ветки
git push -u origin main
git push --set-upstream origin main

# Отправка текущей ветки
git push

# Отправка конкретной ветки
git push origin feature-branch

# Отправка всех веток
git push --all

# Принудительная отправка (перезапись истории)
git push --force
git push --force-with-lease      # Безопасный force

# Удаление удаленной ветки
git push origin --delete old-branch

# Отправка тегов
git push --tags
```

### Рабочие процессы для командной работы
```bash
# Стандартный процесс:
git checkout main
git pull origin main
git checkout -b my-feature
# ... работа ...
git add .
git commit -m "Мои изменения"
git push -u origin my-feature
# Создать Pull Request через веб-интерфейс

# Обновление feature-ветки с main:
git checkout my-feature
git fetch origin
git merge origin/main
# ИЛИ
git rebase origin/main

# После принятия PR:
git branch -d my-feature
git push origin --delete my-feature
```

---

## ОТМЕНА ИЗМЕНЕНИЙ

### Отмена в рабочей директории
```bash
# Отменить изменения в конкретном файле
git checkout -- file.txt

# Новая команда (Git 2.23+)
git restore file.txt

# Отменить изменения во всех файлах
git checkout -- .

# Восстановить файл из индекса
git restore --staged file.txt
```

### Отмена в индексе (staging area)
```bash
# Убрать файл из индекса
git reset HEAD file.txt

# Убрать все файлы из индекса
git reset

# Новая команда (Git 2.23+)
git restore --staged file.txt
git restore --staged .
```

### Отмена коммитов
```bash
# Мягкий reset - отменяет коммит, оставляет изменения в индексе
git reset --soft HEAD~1

# Смешанный reset - отменяет коммит, оставляет изменения в рабочей директории
git reset --mixed HEAD~1
git reset HEAD~1                    # --mixed по умолчанию

# Жесткий reset - полностью удаляет коммит и изменения
git reset --hard HEAD~1

# Reset до определенного коммита
git reset --hard abc123

# Безопасный reset (сохраняет изменения в stash)
git reset --hard HEAD~1 && git stash pop
```

### Команда `git revert` - отмена через новый коммит
```bash
# Создать коммит, отменяющий изменения
git revert HEAD
git revert abc123

# Отменить несколько коммитов
git revert HEAD~3..HEAD

# Отменить без автоматического коммита
git revert --no-commit abc123
```

### Восстановление удаленных коммитов
```bash
# Показать историю всех действий (включая удаленные коммиты)
git reflog

# Найти потерянный коммит
git reflog | grep "потерянное сообщение"

# Восстановить коммит
git checkout -b recovered-branch abc123
```

---

## ВРЕМЕННОЕ СОХРАНЕНИЕ

### Команда `git stash` - временное сохранение изменений
```bash
# Сохранить текущие изменения
git stash
git stash push

# Сохранить с сообщением
git stash push -m "Сообщение о сохранении"

# Сохранить включая неотслеживаемые файлы
git stash -u
git stash --include-untracked

# Сохранить все (включая игнорируемые файлы)
git stash -a
git stash --all

# Показать список сохранений
git stash list

# Показать изменения в сохранении
git stash show
git stash show stash@{1}
git stash show -p stash@{0}      # С просмотром diff

# Применить сохранение (не удаляя из списка)
git stash apply
git stash apply stash@{1}

# Применить и удалить из списка
git stash pop
git stash pop stash@{1}

# Удалить сохранение
git stash drop
git stash drop stash@{1}

# Очистить все сохранения
git stash clear

# Создать ветку из сохранения
git stash branch new-branch stash@{1}
```

---

## ИГНОРИРОВАНИЕ ФАЙЛОВ

### Файл .gitignore
```bash
# Создать файл .gitignore в корне репозитория
touch .gitignore

# Проверить игнорируемые файлы
git check-ignore -v file.txt
```

### Синтаксис .gitignore
```gitignore
# Комментарии

# Игнорировать конкретный файл
secret.txt

# Игнорировать все файлы с расширением
*.log
*.tmp

# Игнорировать папку
node_modules/
build/

# Но включить конкретный файл в игнорируемой папке
!build/important.txt

# Игнорировать во всех папках
**/temp

# Игнорировать в конкретной папке
src/temp/

# Игнорировать файлы в любой папке с именем
temp/

# Шаблоны
*.?          # Файлы с одним символом расширения
!*.c         # Не игнорировать .c файлы
/TODO        # Только в корне
foo/         # Игнорировать папку foo
foo/**/bar   # Игнорировать bar в любой подпапке foo
```

### Глобальный .gitignore
```bash
# Создать глобальный файл игнорирования
git config --global core.excludesfile ~/.gitignore_global

# Добавить в него общие шаблоны:
.DS_Store
Thumbs.db
*.swp
*.swo
```

---

## ПРОДВИНУТЫЕ ТЕХНИКИ

### Интерактивное добавление
```bash
git add -i
# Команды в интерактивном режиме:
#  1: status   2: update   3: revert   4: add untracked
#  5: patch    6: diff     7: quit     8: help

# Пошаговое добавление
git add -p
# Команды для патчей:
# y - принять это изменение
# n - не принимать это изменение
# q - выйти
# a - принять это и все последующие изменения в файле
# d - не принимать это и все последующие изменения в файле
# / - поиск регулярного выражения
```

### Интерактивное перебазирование
```bash
git rebase -i HEAD~5
# Доступные команды:
# pick    - использовать коммит
# reword  - использовать коммит, но изменить сообщение
# edit    - использовать коммит, но остановиться для правки
# squash  - объединить с предыдущим коммитом
# fixup   - как squash, но отбросить сообщение коммита
# drop    - удалить коммит
# exec    - выполнить команду оболочки

# Пример последовательности:
pick abc123 Первый коммит
reword def456 Второй коммит
edit ghi789 Третий коммит
squash jkl012 Четвертый коммит
fixup mno345 Пятый коммит
```

### Работа с тегами
```bash
# Создать легковесный тег
git tag v1.0.0

# Создать аннотированный тег
git tag -a v1.0.0 -m "Версия 1.0.0"

# Создать тег на определенном коммите
git tag -a v1.0.0 abc123 -m "Версия 1.0.0"

# Показать все теги
git tag
git tag -l "v1.*"

# Показать информацию о теге
git show v1.0.0

# Удалить тег
git tag -d v1.0.0

# Отправить теги на удаленный репозиторий
git push origin v1.0.0
git push origin --tags

# Удалить удаленный тег
git push origin --delete v1.0.0
```

### Поиск проблем с git bisect
```bash
# Начать бисекцию
git bisect start

# Отметить плохой коммит
git bisect bad HEAD

# Отметить хороший коммит
git bisect good v1.0.0

# После проверки текущего коммита:
git bisect good    # Если проблема не воспроизводится
git bisect bad     # Если проблема есть

# Завершить бисекцию
git bisect reset

# Автоматическая бисекция с скриптом
git bisect run npm test
```

### Субмодули
```bash
# Добавить субмодуль
git submodule add https://github.com/user/repo.git external/lib

# Инициализировать субмодули
git submodule init

# Обновить субмодули
git submodule update

# Обновить до последней версии
git submodule update --remote

# Рекурсивная работа с субмодулями
git clone --recursive https://github.com/user/repo.git
```

### Очистка репозитория
```bash
# Показать что будет удалено
git clean -n
git clean --dry-run

# Удалить неотслеживаемые файлы
git clean -f

# Удалить неотслеживаемые файлы и папки
git clean -fd

# Интерактивная очистка
git clean -i
```

### Архивация
```bash
# Создать архив текущего состояния
git archive --format=zip HEAD > project.zip

# Архив определенной ветки
git archive --format=tar --prefix=project/ develop | gzip > project.tar.gz

# Архив без .git
git archive --output=project.zip HEAD
```

---

## ПОЛЕЗНЫЕ СЦЕНАРИИ И РЕШЕНИЯ ПРОБЛЕМ

### Частые проблемы и решения
```bash
# "Please commit your changes or stash them before switching branches"
git stash
git checkout other-branch
git stash pop

# "Your local changes to the following files would be overwritten by merge"
git stash
git pull
git stash pop

# "Failed to push some refs" (расхождение истории)
git pull --rebase
git push

# Удаление большого файла из истории
git filter-branch --tree-filter 'rm -f large-file.zip' HEAD
# ИЛИ с помощью BFG Repo-Cleaner (быстрее)

# Восстановление удаленного файла
git checkout HEAD -- deleted-file.txt

# Поиск коммита, который сломал сборку
git bisect start
git bisect bad HEAD
git bisect good v1.0.0
# ... тестируем ...
git bisect reset
```

### Полезные алиасы для .gitconfig
```ini
[alias]
    co = checkout
    br = branch
    ci = commit
    st = status
    unstage = reset HEAD --
    last = log -1 HEAD
    lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit
    lol = log --graph --decorate --pretty=oneline --abbrev-commit
    lola = log --graph --decorate --pretty=oneline --abbrev-commit --all
    find = "!f() { git log --all --grep=\"$1\"; }; f"
    changes = log --pretty=format:'* %s' --since='1 week ago'
```

---

## ПРОДВИНУТАЯ РАБОТА С ФАЙЛАМИ

### Детальное отслеживание файлов
```bash
# Показать информацию об отслеживании файла
git ls-files --stage file.txt
git ls-files --full-name file.txt    # Относительный путь от корня

# Найти все отслеживаемые файлы
git ls-files

# Найти игнорируемые файлы
git ls-files --others --ignored --exclude-standard

# Найти неотслеживаемые файлы
git ls-files --others

# Показать только удаленные файлы
git ls-files --deleted
```

### Продвинутое перемещение и переименование
```bash
# Перемещение/переименование файла с отслеживанием в Git
git mv old-name.txt new-name.txt

# Принудительное перемещение (если файл существует)
git mv -f old-file.txt new-file.txt

# Перемещение в папку
git mv file.txt folder/file.txt

# Восстановление переименованного файла
git log --follow --name-only --oneline file.txt
```

### Работа с бинарными файлами
```bash
# Показать различия в бинарных файлах
git diff --binary

# Скачать файл из другого коммита без переключения
git show commit-hash:file.txt > restored-file.txt

# Экспорт файла из другой ветки
git show branch-name:file.txt > file-from-branch.txt
```

### Поиск файлов в истории
```bash
# Найти коммиты, где изменялся файл
git log --follow file.txt

# Найти коммиты, где файл был добавлен
git log --diff-filter=A -- file.txt

# Найти коммиты, где файл был удален
git log --diff-filter=D -- file.txt

# Найти коммиты, где файл был переименован
git log --diff-filter=R -- file.txt

# Статистика изменений файла
git log --oneline --numstat -- file.txt
```

### Восстановление удаленных файлов
```bash
# Найти коммит, где файл был удален
git log --diff-filter=D --summary | grep delete

# Восстановить файл из коммита перед удалением
git checkout commit-hash^ -- deleted-file.txt

# Или найти все удаленные файлы
git log --all --full-history -- **/deleted-file.*
```

---

## ТЕГИ (РАСШИРЕННЫЙ РАЗДЕЛ)

### Типы тегов
```bash
# Легковесный тег (просто указатель на коммит)
git tag v1.0.0

# Аннотированный тег (полноценный объект Git)
git tag -a v1.0.0 -m "Релиз версии 1.0.0"
# Содержит: автора, дату, сообщение, подпись

# Подписанный тег (с GPG подписью)
git tag -s v1.0.0 -m "Подписанный релиз v1.0.0"
```

### Семантическое версионирование (SemVer)
```markdown
ВЕРСИЯ: MAJOR.MINOR.PATCH

MAJOR - несовместимые изменения API
MINOR - новая функциональность (обратно совместима)
PATCH - исправления ошибок (обратно совместимы)

Дополнительно:
- v1.0.0-alpha.1    # пре-релиз (alpha)
- v1.0.0-beta.1     # пре-релиз (beta)
- v1.0.0-rc.1       # release candidate
- v1.0.0+build.123  # метаданные сборки
```

### Полное управление тегами
```bash
# Создание тегов
git tag v1.0.0                    # Легковесный
git tag -a v1.0.1 -m "Описание"   # Аннотированный
git tag -s v1.0.2 -m "Подписанный" # Подписанный

# Просмотр тегов
git tag                           # Список тегов
git tag -l "v1.*"                 # Фильтрация по шаблону
git tag -n                        # С сообщениями
git tag -n9                       # Показать первые 9 строк сообщения
git show v1.0.0                   # Детали тега

# Удаление тегов
git tag -d v1.0.0                 # Локальное удаление
git push --delete origin v1.0.0   # Удаление на удаленном репозитории

# Перемещение тега
git tag -f v1.0.0 COMMIT_HASH     # Переместить тег
git push --force origin v1.0.0    # Принудительно отправить

# Работа с удаленными тегами
git push origin v1.0.0            # Отправить один тег
git push origin --tags            # Отправить все теги
git fetch --tags                  # Получить все теги
git fetch origin tag v1.0.0       # Получить конкретный тег

# Проверка подписей
git tag -v v1.0.0                 # Проверить подпись тега
```

### Автоматическое версионирование
```bash
# Получить последний тег
git describe --tags --abbrev=0

# Описать текущий коммит относительно тегов
git describe --tags

# Создать тег с текущей датой
git tag "build-$(date +%Y%m%d-%H%M%S)"

# Автоматическое увеличение версии (пример скрипта)
#!/bin/bash
LAST_TAG=$(git describe --tags --abbrev=0 2>/dev/null)
if [ -z "$LAST_TAG" ]; then
    NEW_TAG="v1.0.0"
else
    NEW_TAG=$(echo $LAST_TAG | awk -F. '{$NF+=1; OFS="."; print $0}')
fi
git tag -a $NEW_TAG -m "Релиз $NEW_TAG"
```

### Аннотированные теги с подробностями
```bash
# Создание тега с подробным сообщением
git tag -a v1.2.3 -m "Релиз версии 1.2.3

Основные изменения:
- Добавлена новая система аутентификации
- Исправлены критические ошибки безопасности
- Улучшена производительность на 30%

Дата релиза: $(date +%Y-%m-%d)
Ответственный: $(git config user.name)"

# Просмотр полной информации о теге
git cat-file tag v1.2.3
```

---

## GIT WORKTREES

### Что такое Worktrees?
**Рабочие деревья** позволяют работать с несколькими ветками одновременно в разных папках без переключения в одном репозитории.

### Базовые команды Worktrees
```bash
# Создать новое рабочее дерево
git worktree add ../feature-branch feature/branch-name

# Создать из определенного коммита
git worktree add ../hotfix abc123

# Создать новую ветку в рабочем дереве
git worktree add -b new-feature ../new-feature main

# Показать список рабочих деревьев
git worktree list

# Переместить рабочее дерево (Git 2.17+)
git worktree move ../old-path ../new-path

# Удалить рабочее дерево
git worktree remove ../feature-branch

# Принудительное удаление (если есть изменения)
git worktree remove -f ../feature-branch

# Очистка (удаление метаданных несуществующих рабочих деревьев)
git worktree prune
```

### Практические примеры использования
```bash
# Одновременная работа над фичей и хотфиксом
git worktree add ../hotfix hotfix/urgent
git worktree add ../feature feature/new-auth

# Теперь можно работать параллельно:
cd ../hotfix
# ... исправляем срочную ошибку ...
git add . && git commit -m "Fix critical bug"
git push

cd ../feature  
# ... продолжаем разработку фичи ...
git add . && git commit -m "Add new feature"
git push

# Просмотр всех рабочих деревьев
git worktree list
# /main-repo          abc123 [main]
# /main-repo/hotfix   def456 [hotfix/urgent] 
# /main-repo/feature  ghi789 [feature/new-auth]
```

### Автоматизация Worktrees
```bash
# Создать скрипт для быстрого создания рабочих деревьев
#!/bin/bash
BRANCH_NAME=$1
WORKTREE_PATH="../$BRANCH_NAME"

git worktree add $WORKTREE_PATH $BRANCH_NAME
cd $WORKTREE_PATH

# Использование: ./create-worktree.sh feature/new-feature
```

### Ограничения и лучшие практики
```bash
# Проверить можно ли добавить рабочее дерево
git worktree add --check ../new-path branch-name

# Worktrees не могут быть вложенными
# Одно рабочее дерево не может быть внутри другого

# Удаление основного рабочего дерева невозможно
# Основное рабочее дерево удаляется только при удалении всего репозитория
```

---

## ПОЛНЫЙ .gitignore

### Универсальный .gitignore шаблон
```gitignore
# ============================================
# ОСОБЕННОСТИ ОПЕРАЦИОННЫХ СИСТЕМ
# ============================================

# macOS
.DS_Store
.AppleDouble
.LSOverride
Icon?
._*
.Spotlight-V100
.Trashes
ehthumbs.db
Thumbs.db

# Windows
Thumbs.db
ehthumbs.db
Desktop.ini
$RECYCLE.BIN/
*.lnk

# Linux
*~

# ============================================  
# РЕДАКТОРЫ КОДА И IDE
# ============================================

# VS Code
.vscode/
!.vscode/settings.json
!.vscode/tasks.json
!.vscode/launch.json
!.vscode/extensions.json
*.code-workspace

# IntelliJ IDEA
.idea/
*.iws
*.iml
*.ipr

# Eclipse
.settings/
bin/
tmp/
*.tmp
*.bak
*.swp
*~.nib
local.properties
.loadpath
.factorypath

# NetBeans
nbproject/private/
build/
nbbuild/
dist/
nbdist/
.nb-gradle/

# ============================================
# ЯЗЫКИ ПРОГРАММИРОВАНИЯ И ФРЕЙМВОРКИ
# ============================================

# Node.js
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.lock-wscript
.env
.env.local
.env.development.local
.env.test.local
.env.production.local
.nyc_output
coverage/
.nyc_output
.grunt

# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
env.bak/
venv.bak/
pip-log.txt
pip-delete-this-directory.txt
.tox
.coverage
.coverage.*
.cache
nosetests.xml
coverage.xml
*.cover
.hypothesis/
.pytest_cache/

# Java
*.class
*.jar
*.war
*.nar
*.ear
*.zip
*.tar.gz
*.rar
hs_err_pid*

# C/C++
*.o
*.ko
*.obj
*.elf
*.ilk
*.map
*.exp
*.pdb

# Go
/bin/
/pkg/

# Rust
/target/
**/*.rs.bk

# ============================================
# СИСТЕМЫ СБОРКИ И ПАКЕТНЫЕ МЕНЕДЖЕРЫ
# ============================================

# Maven
target/
pom.xml.tag
pom.xml.releaseBackup
pom.xml.versionsBackup
pom.xml.next
release.properties
dependency-reduced-pom.xml
buildNumber.properties
.mvn/timing.properties

# Gradle
.gradle/
build/
!gradle/wrapper/gradle-wrapper.jar

# Android
*.apk
*.ap_
*.aab
*.aar

# ============================================
# БАЗЫ ДАННЫХ
# ============================================

*.sqlite
*.db
*.mdb
*.dump

# ============================================
# ДОКУМЕНТАЦИЯ И ЛОГИ
# ============================================

*.log
logs/
*.notes
*.patch

# ============================================
# ВРЕМЕННЫЕ ФАЙЛЫ
# ============================================

*.tmp
*.temp
*.swp
*.swo
*~

# ============================================
# ПРОЧЕЕ
# ============================================

# Dependency directories
jspm_packages/
web_modules/

# Optional npm cache directory
.npm

# Optional eslint cache
.eslintcache

# Microbundle cache
.rpt2_cache/
.rts2_cache_cjs/
.rts2_cache_es/
.rts2_cache_umd/

# Optional REPL history
.node_repl_history

# Output of 'npm pack'
*.tgz

# Yarn Integrity file
.yarn-integrity

# dotenv environment variables file
.env
.env.test

# parcel-bundler cache (https://parceljs.org/)
.cache
.parcel-cache

# next.js build output
.next

# nuxt.js build output
.nuxt

# gatsby files
.cache/
public

# vuepress build output
.vuepress/dist

# Serverless directories
.serverless/

# FuseBox cache
.fusebox/

# DynamoDB Local files
.dynamodb/

# TernJS port file
.tern-port

# Stores VSCode versions used for testing VSCode extensions
.vscode-test
```

### Специализированные .gitignore шаблоны

#### Для веб-разработки
```gitignore
# Веб-разработка
.sass-cache/
*.css.map
*.sass.map
*.scss.map
*.less.css
.stylelintcache

# Фреймворки
.next/
.nuxt/
dist/
build/
.out/

# CMS
wp-config.php
configuration.php
```

#### Для мобильной разработки
```gitignore
# Android
*.keystore
*.jks
local.properties
google-services.json

# iOS
*.xcuserstate
project.xcworkspace/
xcuserdata/
Pods/
```

#### Для игровой разработки
```gitignore
# Unity
/[Ll]ibrary/
/[Tt]emp/
/[Oo]bj/
/[Bb]uild/
/[Bb]uilds/
/Assets/AssetStoreTools*
*.csproj
*.unityproj
*.sln
*.suo
*.tmp
*.user
*.userprefs
*.pidb
*.booproj
*.svd
```

### Глобальный .gitignore
```bash
# Создать глобальный файл игнорирования
git config --global core.excludesfile ~/.gitignore_global

# Содержимое ~/.gitignore_global:
.DS_Store
Thumbs.db
*.swp
*.swo
*~
.idea/
.vscode/
```

---

## ПОЛНОЕ РУКОВОДСТВО ПО ИМЕНОВАНИЮ

### КОММИТЫ

#### Конвенция Conventional Commits
```markdown
<тип>[область]: <описание>

[тело]

[нижний колонтитул]
```

#### Типы коммитов:
```markdown
feat:        Новая функциональность
fix:         Исправление ошибки
docs:        Изменения в документации
style:       Изменения, не влияющие на смысл кода
refactor:    Рефакторинг кода производства
perf:        Изменения, улучшающие производительность
test:        Добавление или исправление тестов
build:       Изменения, влияющие на систему сборки
ci:          Изменения в CI-конфигурации
chore:       Другие изменения, не модифицирующие исходный код
revert:      Отмена предыдущего коммита
```

#### Примеры отличных коммитов:
```bash
# ✅ ПРАВИЛЬНО
feat(auth): добавить OAuth2 аутентификацию через Google
fix(api): исправить утечку памяти в обработчике запросов
docs(readme): обновить инструкции по установке
style(button): привести к единому кодстайлу
refactor(database): оптимизировать запросы пользователей
perf(images): сжать изображения на 40%
test(users): добавить тесты для CRUD операций
build(docker): обновить до Node.js 18
ci(github): добавить автоматическое тестирование
chore(deps): обновить зависимости до последних версий

# ❌ НЕПРАВИЛЬНО
update code
fix bug
changes
tmp commit
asdf
```

#### Структура идеального коммита:
```markdown
feat(api): добавить endpoint для восстановления пароля

- Реализован POST /api/auth/reset-password
- Добавлена валидация email
- Настроена отправка писем через SendGrid
- Добавлены unit-тесты для нового функционала

Closes #123, #124
BREAKING CHANGE: изменена структура ответа auth endpoints
```

### ВЕТКИ

#### Стратегия именования веток
```markdown
<тип>/<краткое-описание>

ИЛИ

<тип>/<номер-задачи>-<краткое-описание>
```

#### Типы веток:
```markdown
feature/    - Новая функциональность
bugfix/     - Исправление ошибки
hotfix/     - Срочное исправление в production
release/    - Подготовка релиза
docs/       - Работа над документацией
refactor/   - Рефакторинг кода
test/       - Добавление тестов
chore/      - Обслуживающие задачи
```

#### Примеры именования веток:
```bash
# ✅ ПРАВИЛЬНО
feature/user-profile
bugfix/login-validation
hotfix/critical-security-issue
release/v1.2.0
docs/api-reference
refactor/auth-module
test/user-service
chore/update-dependencies

# С Jira/GitHub Issues
feature/PROJ-123-add-search
bugfix/PROJ-456-fix-memory-leak

# ❌ НЕПРАВИЛЬНО
new-feature
fix
test-branch
my-branch
123
asdf
```

### ТЕГИ

#### Семантическое версионирование
```markdown
v<MAJOR>.<MINOR>.<PATCH>[-<PRE-RELEASE>][+<BUILD>]

Примеры:
v1.0.0          - Первый стабильный релиз
v1.2.3          - Обычный релиз
v2.0.0-rc.1     - Release candidate
v1.0.0-beta.2   - Beta версия
v1.0.0-alpha.1  - Alpha версия
v1.0.0+build.123- Версия с метаданными сборки
```

### РЕПОЗИТОРИИ

#### Именование репозиториев
```markdown
# Форматы:
project-name
project-name-api
project-name-web
project-name-mobile
organization-project
service-name

# Примеры:
auth-service
payment-gateway
user-profile-api
company-website
mobile-app-ios
```

#### Правила именования репозиториев:
```markdown
✅ ДОПУСТИМО:
- Только строчные буквы
- Дефисы вместо пробелов
- Короткие и описательные названия
- Соответствие содержанию

❌ НЕДОПУСТИМО:
- Пробелы
- Заглавные буквы
- Специальные символы (кроме дефиса)
- Слишком длинные названия
- Непонятные аббревиатуры
```

### ДОПОЛНИТЕЛЬНЫЕ СОВЕТЫ

#### Для командной работы:
```markdown
1. Используйте префиксы команд:
   - frontend/feature/search
   - backend/feature/api-auth
   - mobile/bugfix/crash-fix

2. Согласуйте стандарты в команде
3. Используйте одинаковые шаблоны
4. Документируйте соглашения
```

#### Автоматизация:
```bash
# Git hooks для проверки коммитов
#!/bin/sh
# .git/hooks/commit-msg

# Проверка формата коммита
if ! grep -qE "^(feat|fix|docs|style|refactor|perf|test|build|ci|chore|revert)(\(\w+\))?: .{1,50}" "$1"; then
    echo "Invalid commit message format!" >&2
    echo "Use: <type>[scope]: <description>" >&2
    exit 1
fi

# Pre-commit hook для проверки веток
#!/bin/sh
# .git/hooks/pre-commit

CURRENT_BRANCH=$(git symbolic-ref --short HEAD)
if ! echo "$CURRENT_BRANCH" | grep -qE "^(feature|bugfix|hotfix|release|docs|refactor|test|chore)/"; then
    echo "Warning: Branch name doesn't follow convention!" >&2
    echo "Expected: <type>/<description>" >&2
fi
```
