# ПОЛНАЯ ШПАРГАЛКА ПО HTML ТЭГАМ

## ОГЛАВЛЕНИЕ
1. [Историческая эволюция HTML](#историческая-эволюция-html)
2. [Современные семантические теги](#современные-семантические-теги)
3. [Медиа-контент и мультимедиа](#медиа-контент-и-мультимедиа)
4. [Формы и пользовательский ввод](#формы-и-пользовательский-ввод)
5. [Мета-теги и SEO оптимизация](#мета-теги-и-seo-оптимизация)
6. [Устаревшие теги](#устаревшие-теги)
7. [Глобальные атрибуты и данные](#глобальные-атрибуты-и-данные)
8. [Оптимизация производительности](#оптимизация-производительности)
9. [Будущее HTML и новые стандарты](#будущее-html-и-новые-стандарты)

---

## ИСТОРИЧЕСКАЯ ЭВОЛЮЦИЯ HTML

### Версии HTML и их особенности

**HTML 2.0 (1995)**
- Первая стандартизированная версия
- Базовые теги: `<html>`, `<head>`, `<body>`, `<p>`, `<a>`, `<img>`
- Ограниченные возможности оформления

**HTML 3.2 (1997)**
- Добавлены таблицы (`<table>`, `<tr>`, `<td>`)
- Апплеты (`<applet>`)
- Теги визуального форматирования

**HTML 4.01 (1999)**
- Разделение структуры и представления
- Введение CSS
- Устаревание визуальных тегов
- Поддержка скриптов

**XHTML (2000)**
- Синтаксис XML
- Строгие правила закрытия тегов
- Четкая структура документа

**HTML5 (2014)**
- Семантические теги
- Мультимедийная поддержка
- API для сложных приложений
- Окончательное устаревание неправильных практик

---

## СОВРЕМЕННЫЕ СЕМАНТИЧЕСКИЕ ТЕГИ

### Детальная семантика документов

#### `<article>` - расширенное использование
```html
<article itemscope itemtype="https://schema.org/Article">
    <header>
        <h1 itemprop="headline">Заголовок статьи</h1>
        <div class="meta">
            <time datetime="2024-01-15T10:00:00Z" itemprop="datePublished">
                15 января 2024, 10:00
            </time>
            <address itemprop="author" itemscope itemtype="https://schema.org/Person">
                Автор: <span itemprop="name">Иван Иванов</span>
            </address>
        </div>
    </header>
    
    <div itemprop="articleBody">
        <p>Введение статьи...</p>
        
        <section aria-labelledby="section1">
            <h2 id="section1">Первый раздел</h2>
            <p>Содержимое раздела...</p>
            <figure>
                <img src="image.jpg" alt="Описание" itemprop="image">
                <figcaption>Подпись к изображению</figcaption>
            </figure>
        </section>
        
        <aside role="note">
            <p>Дополнительная информация по теме</p>
        </aside>
    </div>
    
    <footer>
        <div class="tags" itemprop="keywords">
            <span>HTML</span>, <span>Семантика</span>, <span>Доступность</span>
        </div>
        <div class="comments" itemprop="comment">
            <!-- Комментарии -->
        </div>
    </footer>
</article>
```

#### `<section>` vs `<div>` - тонкие различия
```html
<!-- Правильное использование section -->
<section aria-labelledby="features-heading">
    <h2 id="features-heading">Особенности продукта</h2>
    <div class="feature-grid">
        <!-- Сетка особенностей -->
    </div>
</section>

<!-- Неправильное использование section -->
<section class="sidebar">
    <!-- Боковая панель - должен быть aside -->
</section>

<!-- div для стилизации без семантики -->
<div class="button-group" role="group" aria-label="Действия с документом">
    <button>Сохранить</button>
    <button>Отменить</button>
</div>
```

### Сложная навигационная структура

#### `<nav>` - многоуровневая навигация
```html
<nav aria-label="Основная навигация по сайту">
    <ul class="nav-main">
        <li>
            <a href="/" aria-current="page">Главная</a>
        </li>
        <li aria-haspopup="true">
            <a href="/products" aria-expanded="false">Продукты</a>
            <ul class="nav-sub">
                <li><a href="/products/web">Веб-разработка</a></li>
                <li><a href="/products/mobile">Мобильные приложения</a></li>
            </ul>
        </li>
        <li><a href="/about">О компании</a></li>
    </ul>
</nav>

<nav aria-label="Вторичная навигация">
    <ul class="nav-secondary">
        <li><a href="/support">Поддержка</a></li>
        <li><a href="/blog">Блог</a></li>
        <li><a href="/contact">Контакты</a></li>
    </ul>
</nav>
```

---

## МЕДИА-КОНТЕНТ И МУЛЬТИМЕДИА

### Расширенные возможности изображений

#### `<picture>` - сложные сценарии
```html
<picture>
    <!-- Адаптивность + формат -->
    <source media="(min-width: 1200px)" 
            type="image/avif"
            srcset="hero-1920.avif 1920w,
                    hero-1280.avif 1280w">
    
    <source media="(min-width: 1200px)" 
            type="image/webp"
            srcset="hero-1920.webp 1920w,
                    hero-1280.webp 1280w">
    
    <source media="(min-width: 768px)" 
            type="image/avif"
            srcset="hero-1024.avif 1024w,
                    hero-768.avif 768w">
    
    <source media="(min-width: 768px)" 
            type="image/webp"
            srcset="hero-1024.webp 1024w,
                    hero-768.webp 768w">
    
    <!-- Мобильные устройства -->
    <source type="image/avif"
            srcset="hero-480.avif 480w,
                    hero-320.avif 320w">
    
    <source type="image/webp"
            srcset="hero-480.webp 480w,
                    hero-320.webp 320w">
    
    <!-- Fallback -->
    <img src="hero-1024.jpg" 
         alt="Главное изображение продукта"
         srcset="hero-1920.jpg 1920w,
                 hero-1280.jpg 1280w,
                 hero-1024.jpg 1024w,
                 hero-768.jpg 768w,
                 hero-480.jpg 480w,
                 hero-320.jpg 320w"
         sizes="(min-width: 1200px) 80vw,
                (min-width: 768px) 90vw,
                100vw"
         loading="eager"
         width="1920" 
         height="1080"
         decoding="async">
</picture>
```

### Видео с расширенными функциями

```html
<video controls 
       width="1280" 
       height="720"
       poster="video-preview.jpg"
       preload="metadata"
       crossorigin="anonymous"
       playsinline
       muted
       aria-labelledby="video-title"
       aria-describedby="video-desc">
    
    <!-- Multiple codecs and formats -->
    <source src="video.hvc1.mp4" type="video/mp4; codecs=hvc1">
    <source src="video.avc1.mp4" type="video/mp4; codecs=avc1">
    <source src="video.vp9.webm" type="video/webm; codecs=vp9">
    <source src="video.vp8.webm" type="video/webm; codecs=vp8">
    <source src="video.ogv" type="video/ogg">
    
    <!-- Multiple audio tracks -->
    <track kind="subtitles" src="subs_en.vtt" srclang="en" label="English" default>
    <track kind="subtitles" src="subs_ru.vtt" srclang="ru" label="Русский">
    <track kind="captions" src="caps_en.vtt" srclang="en" label="English Captions">
    <track kind="descriptions" src="desc_en.vtt" srclang="en" label="Audio Descriptions">
    <track kind="chapters" src="chapters.vtt" srclang="en" label="Chapters">
    <track kind="metadata" src="metadata.vtt" srclang="en">
    
    <!-- Interactive transcript -->
    <div class="transcript" role="region" aria-label="Transcript">
        <button onclick="showTranscript()">Показать транскрипт</button>
        <div id="transcript-content" hidden>
            <!-- Транскрипт видео -->
        </div>
    </div>
    
    <!-- Fallback content -->
    <div class="video-fallback">
        <p>Ваш браузер не поддерживает HTML5 видео.</p>
        <a href="video.mp4" download>Скачать видео (MP4, 45MB)</a>
        <a href="transcript.txt">Читать транскрипт</a>
    </div>
</video>
```

---

## ФОРМЫ И ПОЛЬЗОВАТЕЛЬСКИЙ ВВОД

### Расширенные типы полей ввода

```html
<form id="advanced-form" 
      novalidate 
      autocomplete="on"
      enctype="multipart/form-data"
      aria-labelledby="form-title">
    
    <h2 id="form-title">Расширенная форма</h2>
    
    <!-- Группа полей с валидацией -->
    <fieldset>
        <legend>Личная информация</legend>
        
        <div class="field-group">
            <label for="fullname">Полное имя:</label>
            <input type="text" 
                   id="fullname" 
                   name="fullname"
                   required
                   minlength="2"
                   maxlength="100"
                   pattern="[A-Za-zА-Яа-я\s]+"
                   title="Только буквы и пробелы"
                   autocomplete="name">
            <span class="hint">Минимум 2 символа, только буквы</span>
        </div>
        
        <div class="field-group">
            <label for="email">Email:</label>
            <input type="email" 
                   id="email" 
                   name="email"
                   required
                   multiple
                   autocomplete="email"
                   placeholder="user@example.com">
        </div>
        
        <div class="field-group">
            <label for="phone">Телефон:</label>
            <input type="tel" 
                   id="phone" 
                   name="phone"
                   pattern="[\+]\d{1,3}\s?\(\d{3}\)\s?\d{3}-\d{2}-\d{2}"
                   autocomplete="tel"
                   placeholder="+7 (999) 999-99-99">
        </div>
    </fieldset>
    
    <!-- Динамические поля -->
    <fieldset>
        <legend>Настройки</legend>
        
        <div class="field-group">
            <label for="theme">Тема интерфейса:</label>
            <select id="theme" name="theme" onchange="updateTheme(this.value)">
                <option value="auto">Системная</option>
                <option value="light">Светлая</option>
                <option value="dark">Темная</option>
            </select>
        </div>
        
        <div class="field-group">
            <label for="volume">Громкость уведомлений:</label>
            <input type="range" 
                   id="volume"
                   name="volume"
                   min="0" max="100" value="50"
                   step="10"
                   list="volume-markers"
                   oninput="updateVolume(this.value)">
            <output id="volume-output">50%</output>
            <datalist id="volume-markers">
                <option value="0" label="Выкл"></option>
                <option value="25"></option>
                <option value="50" label="Средняя"></option>
                <option value="75"></option>
                <option value="100" label="Макс"></option>
            </datalist>
        </div>
        
        <div class="field-group">
            <label for="schedule">Расписание:</label>
            <input type="time" 
                   id="schedule"
                   name="schedule"
                   min="09:00" max="18:00"
                   step="900">
        </div>
    </fieldset>
    
    <!-- Файлы и медиа -->
    <fieldset>
        <legend>Медиафайлы</legend>
        
        <div class="field-group">
            <label for="avatar">Аватар:</label>
            <input type="file"
                   id="avatar"
                   name="avatar"
                   accept="image/jpeg, image/png, image/webp"
                   capture="user"
                   multiple>
        </div>
        
        <div class="field-group">
            <label for="document">Документ:</label>
            <input type="file"
                   id="document"
                   name="document"
                   accept=".pdf,.doc,.docx,.txt"
                   multiple>
        </div>
    </fieldset>
    
    <!-- Кастомная валидация -->
    <div class="form-actions">
        <button type="submit" 
                onclick="return validateForm()">
            Отправить
        </button>
        <button type="reset">Сбросить</button>
        <button type="button" onclick="saveDraft()">
            Сохранить черновик
        </button>
    </div>
</form>
```

### Динамические формы с JavaScript

```html
<form id="dynamic-form">
    <!-- Динамически добавляемые поля -->
    <template id="field-template">
        <div class="dynamic-field">
            <label>
                <input type="text" name="dynamic_field[]" placeholder="Новое поле">
            </label>
            <button type="button" onclick="removeField(this)">Удалить</button>
        </div>
    </template>
    
    <div id="dynamic-fields-container"></div>
    <button type="button" onclick="addField()">Добавить поле</button>
</form>

<script>
function addField() {
    const template = document.getElementById('field-template');
    const clone = template.content.cloneNode(true);
    document.getElementById('dynamic-fields-container').appendChild(clone);
}

function removeField(button) {
    button.closest('.dynamic-field').remove();
}
</script>
```

---

## МЕТА-ТЕГИ И SEO ОПТИМИЗАЦИЯ

### Полный набор мета-тегов

```html
<head>
    <!-- Базовые мета-теги -->
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    
    <!-- SEO мета-теги -->
    <title>Полное название страницы | Название сайта</title>
    <meta name="description" content="Подробное описание страницы длиной 150-160 символов">
    <meta name="keywords" content="ключевое слово 1, ключевое слово 2, ключевое слово 3">
    <meta name="author" content="Автор контента">
    <meta name="robots" content="index, follow, max-snippet:-1, max-image-preview:large, max-video-preview:-1">
    <meta name="googlebot" content="index, follow">
    
    <!-- Open Graph для социальных сетей -->
    <meta property="og:type" content="website">
    <meta property="og:url" content="https://example.com/page">
    <meta property="og:title" content="Заголовок для социальных сетей">
    <meta property="og:description" content="Описание для социальных сетей">
    <meta property="og:image" content="https://example.com/image.jpg">
    <meta property="og:image:width" content="1200">
    <meta property="og:image:height" content="630">
    <meta property="og:site_name" content="Название сайта">
    <meta property="og:locale" content="ru_RU">
    
    <!-- Twitter Card -->
    <meta name="twitter:card" content="summary_large_image">
    <meta name="twitter:site" content="@username">
    <meta name="twitter:creator" content="@author">
    <meta name="twitter:title" content="Заголовок для Twitter">
    <meta name="twitter:description" content="Описание для Twitter">
    <meta name="twitter:image" content="https://example.com/twitter-image.jpg">
    
    <!-- Дополнительные мета-теги -->
    <meta name="theme-color" content="#ffffff">
    <meta name="msapplication-TileColor" content="#ffffff">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="default">
    <meta name="format-detection" content="telephone=no">
    
    <!-- Канонические ссылки -->
    <link rel="canonical" href="https://example.com/canonical-url">
    <link rel="alternate" hreflang="ru" href="https://example.com/ru">
    <link rel="alternate" hreflang="en" href="https://example.com/en">
    
    <!-- Структурированные данные (JSON-LD) -->
    <script type="application/ld+json">
    {
        "@context": "https://schema.org",
        "@type": "Article",
        "headline": "Заголовок статьи",
        "description": "Описание статьи",
        "image": "https://example.com/image.jpg",
        "author": {
            "@type": "Person",
            "name": "Автор статьи"
        },
        "publisher": {
            "@type": "Organization",
            "name": "Название организации",
            "logo": {
                "@type": "ImageObject",
                "url": "https://example.com/logo.jpg"
            }
        },
        "datePublished": "2024-01-15T10:00:00Z",
        "dateModified": "2024-01-15T12:00:00Z"
    }
    </script>
</head>
```

---

## УСТАРЕВШИЕ ТЕГИ

### Категории устаревших тегов

#### 1. Теги визуального форматирования

**`<font>` - самый проблемный тег**
```html
<!-- ❌ АБСОЛЮТНО НЕПРАВИЛЬНО -->
<font face="Arial, Helvetica, sans-serif" 
      size="4" 
      color="#ff0000">
    Красный текст Arial размера 4
</font>

<!-- ✅ СОВРЕМЕННАЯ ЗАМЕНА -->
<span style="font-family: Arial, Helvetica, sans-serif; 
             font-size: 16px; 
             color: #ff0000;">
    Красный текст Arial размера 16px
</span>

<!-- ✅ СЕМАНТИЧЕСКИ ПРАВИЛЬНО -->
<strong class="important-text">Важный текст</strong>
<em class="emphasized-text">Акцентированный текст</em>

<style>
.important-text {
    font-family: Arial, Helvetica, sans-serif;
    font-size: 1.125rem; /* 18px */
    color: #d32f2f;
    font-weight: 600;
}

.emphasized-text {
    font-style: italic;
    color: #1976d2;
}
</style>
```

**Проблемы `<font>`:**
- Фиксированные размеры (1-7) вместо относительных единиц
- Ограниченный набор шрифтов
- Нет контроля над межстрочным интервалом
- Не адаптируется к пользовательским настройкам
- Нарушает принцип разделения содержания и оформления

#### 2. Теги позиционирования и выравнивания

**`<center>` - устаревшее центрирование**
```html
<!-- ❌ УСТАРЕЛО В HTML4 -->
<center>
    <h1>Центрированный заголовок</h1>
    <p>Центрированный текст</p>
</center>

<!-- ✅ СОВРЕМЕННЫЕ МЕТОДЫ ЦЕНТРИРОВАНИЯ -->

<!-- 1. Text-align для текста -->
<div style="text-align: center;">
    <h1>Центрированный заголовок</h1>
    <p>Центрированный текст</p>
</div>

<!-- 2. Margin auto для блоков -->
<div style="width: 80%; margin: 0 auto;">
    Центрированный блок
</div>

<!-- 3. Flexbox -->
<div style="display: flex; justify-content: center; align-items: center;">
    <div>Центрированный контент</div>
</div>

<!-- 4. Grid -->
<div style="display: grid; place-items: center;">
    <div>Центрированный контент</div>
</div>

<!-- 5. Абсолютное позиционирование -->
<div style="position: relative;">
    <div style="position: absolute; left: 50%; transform: translateX(-50%);">
        Центрированный контент
    </div>
</div>
```

**`<align>` атрибут в изображениях и таблицах**
```html
<!-- ❌ УСТАРЕЛО -->
<img src="image.jpg" align="left" hspace="10" vspace="5">
<table align="center" border="1">

<!-- ✅ СОВРЕМЕННЫЕ МЕТОДЫ -->
<img src="image.jpg" style="float: left; margin: 5px 10px;">
<table style="margin: 0 auto; border: 1px solid #ccc;">

<!-- Семантическое выравнивание -->
<figure style="float: left; margin: 0 1rem 1rem 0;">
    <img src="image.jpg" alt="Описание">
    <figcaption>Подпись к изображению</figcaption>
</figure>
```

#### 3. Фреймы и встроенные документы

**`<frameset>`, `<frame>`, `<noframes>` - архитектурная ошибка**
```html
<!-- ❌ КАТАСТРОФИЧЕСКИ УСТАРЕЛО -->
<frameset cols="200,*">
    <frame src="menu.html" name="menu" scrolling="auto">
    <frame src="content.html" name="content">
    <noframes>
        <body>
            <p>Ваш браузер не поддерживает фреймы</p>
        </body>
    </noframes>
</frameset>

<!-- ✅ СОВРЕМЕННЫЕ АЛЬТЕРНАТИВЫ -->

<!-- 1. Server-Side Includes (SSI) -->
<!--#include virtual="menu.html" -->
<div class="content">
    <!--#include virtual="content.html" -->
</div>

<!-- 2. Компонентный подход -->
<header-component></header-component>
<nav-component></nav-component>
<main-component></main-component>

<!-- 3. Iframe для внешнего контента (ограниченно) -->
<iframe src="external-widget.html" 
        title="Внешний виджет"
        loading="lazy"
        sandbox="allow-scripts allow-same-origin"
        referrerpolicy="no-referrer">
    <p>Альтернативный контент для браузеров без поддержки iframe</p>
</iframe>

<!-- 4. Web Components -->
<script>
class NavigationComponent extends HTMLElement {
    connectedCallback() {
        this.innerHTML = `
            <nav>
                <ul>
                    <li><a href="/">Главная</a></li>
                    <li><a href="/about">О нас</a></li>
                </ul>
            </nav>
        `;
    }
}
customElements.define('nav-component', NavigationComponent);
</script>
```

**Критические проблемы фреймов:**
- URL не отражает состояние приложения
- Поисковые системы не могут индексировать контент
- Проблемы с кнопкой "Назад"
- Недоступность для скринридеров
- Проблемы с печатью документов
- Уязвимости безопасности (clickjacking)
- Плохая мобильная адаптация

#### 4. Теги презентации и стилизации

**`<big>`, `<small>`, `<strike>`, `<s>`, `<u>`**
```html
<!-- ❌ УСТАРЕВШАЯ ПРЕЗЕНТАЦИЯ -->
<big>Большой текст</big>
<small>Маленький текст</small>
<strike>Зачеркнутый текст</strike>
<s>Тоже зачеркнутый</s>
<u>Подчеркнутый текст</u>
<tt>Моноширинный текст</tt>
<blink>Мигающий текст</blink>
<marquee>Бегущая строка</marquee>

<!-- ✅ СЕМАНТИЧЕСКИ ПРАВИЛЬНО -->

<!-- Для изменения размера с семантикой -->
<span class="lead-text">Крупный вводный текст</span>
<small class="legal-text">Юридическая информация</small>

<!-- Для зачеркивания с семантикой -->
<del datetime="2024-01-15">Удаленная информация</del>
<ins datetime="2024-01-16">Добавленная информация</ins>

<!-- Для выделения -->
<mark>Важная часть текста</mark>
<strong>Очень важный текст</strong>
<em>Акцентированный текст</em>

<!-- Для технических терминов -->
<code>printf("Hello World");</code>
<kbd>Ctrl+C</kbd>
<samp>Результат выполнения</samp>
<var>variable_name</var>

<!-- Стилизация через CSS -->
<style>
.lead-text {
    font-size: 1.25rem;
    font-weight: 300;
    line-height: 1.4;
}

.legal-text {
    font-size: 0.875rem;
    color: #666;
}

.highlighted {
    background-color: #fff3cd;
    padding: 0.2em 0.4em;
    border-radius: 0.25rem;
}

/* Анимации вместо blink/marquee */
.pulsing {
    animation: pulse 2s infinite;
}

.scrolling {
    animation: scroll 10s linear infinite;
}

@keyframes pulse {
    0%, 100% { opacity: 1; }
    50% { opacity: 0.5; }
}

@keyframes scroll {
    from { transform: translateX(100%); }
    to { transform: translateX(-100%); }
}
</style>
```

#### 5. Устаревшие табличные атрибуты

```html
<!-- ❌ УСТАРЕВШАЯ ТАБЛИЦА -->
<table border="1" 
       cellpadding="10" 
       cellspacing="5" 
       bgcolor="#f0f0f0" 
       align="center" 
       width="80%">
    <tr>
        <td width="30%" bgcolor="yellow" valign="top">
            Ячейка с устаревшими атрибутами
        </td>
        <td width="70%" valign="middle">
            Другая ячейка
        </td>
    </tr>
</table>

<!-- ✅ СОВРЕМЕННАЯ ТАБЛИЦА -->
<table class="modern-table" 
       role="grid"
       aria-labelledby="table-caption">
    <caption id="table-caption">Описание таблицы</caption>
    <thead>
        <tr>
            <th scope="col" style="width: 30%;">Заголовок 1</th>
            <th scope="col" style="width: 70%;">Заголовок 2</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td style="vertical-align: top;">Данные 1</td>
            <td style="vertical-align: middle;">Данные 2</td>
        </tr>
    </tbody>
</table>

<style>
.modern-table {
    width: 80%;
    margin: 0 auto;
    border: 1px solid #ddd;
    border-collapse: collapse;
    background-color: #f8f9fa;
}

.modern-table th,
.modern-table td {
    padding: 0.75rem;
    border: 1px solid #dee2e6;
}

.modern-table thead th {
    background-color: #e9ecef;
    vertical-align: bottom;
}
</style>
```

#### 6. Устаревшие формы и элементы ввода

**`<isindex>` - примитивный поиск**
```html
<!-- ❌ УСТАРЕЛО В HTML4 -->
<isindex prompt="Введите поисковый запрос:">

<!-- ✅ СОВРЕМЕННЫЙ ПОИСК -->
<form role="search" aria-label="Поиск по сайту">
    <label for="search-input" class="visually-hidden">
        Поиск по сайту
    </label>
    <input type="search" 
           id="search-input"
           name="q"
           placeholder="Введите поисковый запрос..."
           aria-describedby="search-help">
    <button type="submit">
        <span class="visually-hidden">Найти</span>
        🔍
    </button>
    <div id="search-help" class="help-text">
        Введите ключевые слова для поиска
    </div>
</form>
```

**Устаревшие атрибуты форм**
```html
<!-- ❌ УСТАРЕВШИЕ АТРИБУТЫ -->
<body bgcolor="white" text="black" link="blue" vlink="purple">
<form accept="text/html">
<textarea wrap="physical">

<!-- ✅ СОВРЕМЕННЫЕ АНАЛОГИ -->
<body style="background-color: white; color: black;">
<form enctype="text/html">
<textarea style="white-space: pre-wrap;">
```

### Полный список устаревших тегов и атрибутов

#### Теги, удаленные из HTML5:
- `<acronym>` → используйте `<abbr>`
- `<applet>` → используйте `<object>`
- `<basefont>`
- `<big>`
- `<blink>`
- `<center>`
- `<dir>`
- `<font>`
- `<frame>`
- `<frameset>`
- `<isindex>`
- `<keygen>`
- `<marquee>`
- `<menuitem>`
- `<nobr>`
- `<noembed>`
- `<noframes>`
- `<plaintext>`
- `<rb>`
- `<rtc>`
- `<shadow>`
- `<spacer>`
- `<strike>`
- `<tt>`
- `<xmp>`

#### Устаревшие атрибуты:
- `align` (во всех элементах)
- `alink`, `vlink`, `link`, `text`, `bgcolor` (body)
- `border` (img, object)
- `cellpadding`, `cellspacing`, `frame`, `rules` (table)
- `char`, `charoff`, `valign` (табличные элементы)
- `clear` (br)
- `compact` (dl, menu, ol, ul)
- `frame` (iframe)
- `hspace`, `vspace` (img, object)
- `marginheight`, `marginwidth` (iframe)
- `noshade` (hr)
- `nowrap` (td, th)
- `scrolling` (iframe)
- `size` (hr)
- `type` (li, ol, ul)
- `width` (hr, table, td, th, col, colgroup, pre)

---

## ГЛОБАЛЬНЫЕ АТРИБУТЫ И ДАННЫЕ

### Полный набор глобальных атрибутов

```html
<div id="unique-element"
     class="main-content featured"
     style="color: blue;"
     title="Всплывающая подсказка"
     
     <!-- Доступность -->
     role="main"
     aria-label="Основной контент"
     aria-hidden="false"
     aria-describedby="description1"
     aria-labelledby="title1"
     
     <!-- Данные -->
     data-user-id="12345"
     data-category="premium"
     data-config='{"theme": "dark"}'
     
     <!-- События -->
     onclick="handleClick()"
     onkeypress="handleKeypress()"
     
     <!-- Другие глобальные атрибуты -->
     contenteditable="true"
     draggable="true"
     dropzone="copy"
     hidden
     spellcheck="true"
     tabindex="0"
     translate="no"
     
     <!-- XML пространства имен -->
     xml:lang="ru"
     xmlns:svg="http://www.w3.org/2000/svg">
     
     Содержимое элемента
</div>

<p id="description1">Описание элемента</p>
<h1 id="title1">Заголовок элемента</h1>
```

### data-* атрибуты для хранения информации

```html
<div class="product-card"
     data-product-id="P12345"
     data-category="electronics"
     data-price="299.99"
     data-currency="USD"
     data-in-stock="true"
     data-rating="4.5"
     data-features='["wireless", "bluetooth", "noise-cancelling"]'
     data-metadata='{"created": "2024-01-15", "vendor": "TechCorp"}'>
     
    <h3>Беспроводные наушники</h3>
    <button onclick="addToCart(this.closest('.product-card'))">
        В корзину
    </button>
</div>

<script>
function addToCart(productCard) {
    const productData = {
        id: productCard.dataset.productId,
        category: productCard.dataset.category,
        price: parseFloat(productCard.dataset.price),
        currency: productCard.dataset.currency,
        inStock: productCard.dataset.inStock === 'true',
        rating: parseFloat(productCard.dataset.rating),
        features: JSON.parse(productCard.dataset.features),
        metadata: JSON.parse(productCard.dataset.metadata)
    };
    
    console.log('Добавлен товар:', productData);
    // Добавление в корзину...
}
</script>
```

---

## ОПТИМИЗАЦИЯ ПРОИЗВОДИТЕЛЬНОСТИ

### Современные практики загрузки ресурсов

```html
<!-- Оптимизированная загрузка CSS -->
<link rel="preload" href="critical.css" as="style" onload="this.rel='stylesheet'">
<link rel="preload" href="fonts.woff2" as="font" type="font/woff2" crossorigin>
<link rel="stylesheet" href="non-critical.css" media="print" onload="this.media='all'">

<!-- Оптимизированная загрузка JavaScript -->
<script src="critical.js" defer></script>
<script src="analytics.js" async></script>
<script type="module" src="modern-features.js"></script>
<script nomodule src="legacy-fallback.js"></script>

<!-- Resource Hints -->
<link rel="dns-prefetch" href="https://cdn.example.com">
<link rel="preconnect" href="https://api.example.com">
<link rel="prerender" href="/next-page">
<link rel="prefetch" href="/next-page" as="document">

<!-- Ленивая загрузка изображений -->
<img src="placeholder.jpg" 
     data-src="real-image.jpg" 
     alt="Описание"
     loading="lazy"
     decoding="async"
     width="800" height="600"
     onload="this.src=this.dataset.src">

<!-- Современные форматы -->
<picture>
    <source type="image/avif" srcset="image.avif">
    <source type="image/webp" srcset="image.webp">
    <img src="image.jpg" alt="Описание" loading="lazy">
</picture>
```

---

## БУДУЩЕЕ HTML И НОВЫЕ СТАНДАРТЫ

### Новые API и возможности

**Dialog Element**
```html
<dialog id="advanced-dialog">
    <form method="dialog">
        <h2>Диалоговое окно</h2>
        <p>Содержимое диалога...</p>
        <menu>
            <button value="cancel">Отмена</button>
            <button value="confirm">Подтвердить</button>
        </menu>
    </form>
</dialog>

<script>
const dialog = document.getElementById('advanced-dialog');

// Программное открытие
dialog.showModal();

// Программное закрытие
dialog.close('confirm');

// События
dialog.addEventListener('close', () => {
    console.log('Диалог закрыт с результатом:', dialog.returnValue);
});

dialog.addEventListener('cancel', (event) => {
    event.preventDefault(); // Предотвратить закрытие по ESC
});
</script>
```

**Details/Summary с расширенными возможностями**
```html
<details class="accordion" name="faq-group">
    <summary role="button" aria-expanded="false">
        <h3>Часто задаваемый вопрос 1</h3>
        <span class="icon" aria-hidden="true">+</span>
    </summary>
    <div class="content">
        <p>Подробный ответ на вопрос...</p>
    </div>
</details>

<details class="accordion" name="faq-group">
    <summary role="button" aria-expanded="false">
        <h3>Часто задаваемый вопрос 2</h3>
        <span class="icon" aria-hidden="true">+</span>
    </summary>
    <div class="content">
        <p>Подробный ответ на вопрос...</p>
    </div>
</details>
```

### Веб-компоненты и кастомные элементы

```html
<!-- Использование кастомных элементов -->
<user-card 
    name="Иван Иванов"
    avatar="ivan.jpg"
    role="Разработчик"
    data-user-id="123">
</user-card>

<product-gallery 
    images='["img1.jpg", "img2.jpg", "img3.jpg"]'
    zoomable
    autoplay>
</product-gallery>

<script>
// Определение кастомного элемента
class UserCard extends HTMLElement {
    static observedAttributes = ['name', 'avatar', 'role'];
    
    constructor() {
        super();
        this.attachShadow({ mode: 'open' });
    }
    
    connectedCallback() {
        this.render();
    }
    
    attributeChangedCallback() {
        this.render();
    }
    
    render() {
        this.shadowRoot.innerHTML = `
            <style>
                :host {
                    display: block;
                    border: 1px solid #ddd;
                    padding: 1rem;
                    border-radius: 0.5rem;
                }
                .avatar {
                    width: 64px;
                    height: 64px;
                    border-radius: 50%;
                }
            </style>
            <div class="user-card">
                <img src="${this.getAttribute('avatar')}" 
                     alt="${this.getAttribute('name')}" 
                     class="avatar">
                <h3>${this.getAttribute('name')}</h3>
                <p>${this.getAttribute('role')}</p>
                <slot name="actions"></slot>
            </div>
        `;
    }
}

customElements.define('user-card', UserCard);
</script>
```