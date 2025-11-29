# ШПАРГАЛКА PANDAS: ОТ НУЛЯ

## ОГЛАВЛЕНИЕ
1. [Установка и настройка](#установка-и-настройка)
2. [Создание структур данных](#создание-структур-данных)
3. [Загрузка и сохранение данных](#загрузка-и-сохранение-данных)
4. [Базовые операции и просмотр данных](#базовые-операции-и-просмотр-данных)
5. [Выбор и фильтрация данных](#выбор-и-фильтрация-данных)
6. [Работа с пропущенными значениями](#работа-с-пропущенными-значениями)
7. [Изменение и преобразование данных](#изменение-и-преобразование-данных)
8. [Сортировка и переиндексация](#сортировка-и-переиндексация)
9. [Группировка и агрегация](#группировка-и-агрегация)
10. [Объединение и соединение данных](#объединение-и-соединение-данных)
11. [Временные ряды и даты](#временные-ряды-и-даты)
12. [Текстовая обработка](#текстовая-обработка)
13. [Статистические функции](#статистические-функции)
14. [Визуализация данных](#визуализация-данных)
15. [Продвинутые техники](#продвинутые-техники)
16. [Оптимизация производительности](#оптимизация-производительности)
17. [Работа с большими данными](#работа-с-большими-данными)
18. [Пример полного анализа](#пример-полного-анализа)

---

## Установка и настройка

```python
# Установка pandas
pip install pandas
pip install pandas numpy matplotlib seaborn  # полный стек для анализа

# Импорт с алиасами
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns

# Настройка отображения
pd.set_option('display.max_columns', None)  # показывать все столбцы
pd.set_option('display.max_rows', 100)      # максимум 100 строк
pd.set_option('display.width', 1000)        # ширина вывода
pd.set_option('display.precision', 2)       # точность чисел

# Версия pandas
print(pd.__version__)
```

## Создание структур данных

### DataFrame из различных источников
```python
# Из словаря
data = {
    'Имя': ['Анна', 'Борис', 'Виктор', 'Дарья', 'Елена'],
    'Возраст': [25, 30, 35, 28, 32],
    'Город': ['Москва', 'СПб', 'Казань', 'Москва', 'Новосибирск'],
    'Зарплата': [50000, 75000, 60000, 55000, 80000],
    'Отдел': ['IT', 'Маркетинг', 'IT', 'HR', 'IT']
}
df = pd.DataFrame(data)

# Из списка списков
data_list = [
    [1, 'Анна', 25, 'Москва'],
    [2, 'Борис', 30, 'СПб'],
    [3, 'Виктор', 35, 'Казань']
]
df = pd.DataFrame(data_list, columns=['ID', 'Имя', 'Возраст', 'Город'])

# Из NumPy массива
arr = np.random.randn(5, 3)
df = pd.DataFrame(arr, columns=['A', 'B', 'C'], index=pd.date_range('20230101', periods=5))

# Пустой DataFrame
df_empty = pd.DataFrame(columns=['A', 'B', 'C'])

# С определенными типами данных
df_typed = pd.DataFrame({
    'int_col': pd.Series([1, 2, 3], dtype='int32'),
    'float_col': pd.Series([1.1, 2.2, 3.3], dtype='float64'),
    'str_col': pd.Series(['a', 'b', 'c'], dtype='object'),
    'bool_col': pd.Series([True, False, True], dtype='bool'),
    'category_col': pd.Series(['X', 'Y', 'X'], dtype='category')
})
```

### Series
```python
# Создание Series разными способами
s1 = pd.Series([1, 3, 5, np.nan, 6, 8])
s2 = pd.Series({'a': 1, 'b': 2, 'c': 3})
s3 = pd.Series([1, 2, 3], index=['x', 'y', 'z'], dtype='float64', name='Мой_ряд')

# Series с датами
dates = pd.date_range('20230101', periods=6)
s4 = pd.Series([10, 20, 30, 40, 50, 60], index=dates)
```

## Загрузка и сохранение данных

### Чтение данных
```python
# CSV файлы
df = pd.read_csv('file.csv')
df = pd.read_csv('file.csv', sep=';')                    # разделитель
df = pd.read_csv('file.csv', encoding='utf-8')          # кодировка
df = pd.read_csv('file.csv', header=0)                  # строка заголовков
df = pd.read_csv('file.csv', names=['A', 'B', 'C'])     # свои названия
df = pd.read_csv('file.csv', index_col=0)               # столбец как индекс
df = pd.read_csv('file.csv', usecols=['A', 'B'])        # только нужные столбцы
df = pd.read_csv('file.csv', nrows=1000)                # только первые n строк
df = pd.read_csv('file.csv', skiprows=[0, 2])           # пропустить строки
df = pd.read_csv('file.csv', na_values=['NULL', 'N/A']) # задать значения NaN

# Excel файлы
df = pd.read_excel('file.xlsx')
df = pd.read_excel('file.xlsx', sheet_name='Sheet1')    # конкретный лист
df = pd.read_excel('file.xlsx', header=1)               # вторая строка как заголовок

# JSON
df = pd.read_json('file.json')
df = pd.read_json('file.json', orient='records')        # ориентация данных

# SQL базы данных
import sqlite3
conn = sqlite3.connect('database.db')
df = pd.read_sql('SELECT * FROM table_name', conn)
df = pd.read_sql_table('table_name', conn)

# HTML таблицы
dfs = pd.read_html('http://example.com/table.html')     # список DataFrame

# Парсинг URL
df = pd.read_csv('https://example.com/data.csv')
```

### Сохранение данных
```python
# CSV
df.to_csv('output.csv', index=False)                    # без индекса
df.to_csv('output.csv', sep=';', encoding='utf-8')      # с разделителем
df.to_csv('output.csv', float_format='%.2f')            # формат чисел

# Excel
df.to_excel('output.xlsx', index=False)
df.to_excel('output.xlsx', sheet_name='Data')

# JSON
df.to_json('output.json')
df.to_json('output.json', orient='records')

# SQL
df.to_sql('table_name', conn, if_exists='replace')

# Pickle (бинарный формат Python)
df.to_pickle('data.pkl')
df = pd.read_pickle('data.pkl')
```

## Базовые операции и просмотр данных

### Основные методы просмотра
```python
# Базовая информация
df.head()                    # первые 5 строк
df.head(10)                  # первые 10 строк
df.tail()                    # последние 5 строк
df.tail(8)                   # последние 8 строк
df.sample(5)                 # случайные 5 строк
df.shape                     # кортеж (строки, столбцы)
df.size                      # общее количество элементов
df.ndim                      # количество измерений (2 для DataFrame)

# Информация о данных
df.info()                    # подробная информация о типах
df.describe()                # статистика для числовых столбцов
df.describe(include='all')   # статистика для всех столбцов
df.describe(include=['object', 'category'])  # для нечисловых

# Атрибуты DataFrame
df.columns                   # названия столбцов
df.index                     # индексы
df.dtypes                    # типы данных каждого столбца
df.values                    # данные как NumPy массив
```

### Проверка данных
```python
# Пропущенные значения
df.isnull()                  # булева матрица пропусков
df.isna()                    # аналогично isnull()
df.notnull()                 # обратная матрица
df.isnull().sum()            # количество пропусков по столбцам
df.isnull().sum().sum()      # общее количество пропусков
df.isnull().mean()           # доля пропусков по столбцам

# Дубликаты
df.duplicated()              # булев массив дубликатов
df.duplicated().sum()        # количество дубликатов
df.duplicated(subset=['Имя', 'Город'])  # проверка по подмножеству столбцов

# Уникальные значения
df['Город'].unique()         # массив уникальных значений
df['Город'].nunique()        # количество уникальных значений
df['Город'].value_counts()   # частотное распределение
```

## Выбор и фильтрация данных

### Выбор столбцов
```python
# Один столбец (возвращает Series)
df['Имя']
df.Имя                       # только если имя без пробелов и спецсимволов

# Несколько столбцов (возвращает DataFrame)
df[['Имя', 'Возраст']]
df[['Имя']]                  # один столбец как DataFrame

# Фильтр по типам данных
df.select_dtypes(include=['number'])      # только числовые
df.select_dtypes(include=['object'])      # только строковые
df.select_dtypes(exclude=['datetime64'])  # исключая даты
```

### Выбор строк
```python
# По позиции (iloc)
df.iloc[0]                   # первая строка
df.iloc[[0, 2, 4]]           # строки 0, 2, 4
df.iloc[0:5]                 # срез строк 0-4
df.iloc[0:5, 1:3]            # строки 0-4, столбцы 1-2

# По индексу (loc)
df.loc[0]                    # строка с индексом 0
df.loc[[0, 2, 4]]            # строки с индексами 0, 2, 4
df.loc[0:5]                  # строки с индексами от 0 до 5 (включительно!)
df.loc[0:5, 'Имя':'Город']   # срез по индексам и названиям столбцов

# Комбинированный доступ
df.at[0, 'Имя']              # одно значение по индексу и столбцу
df.iat[0, 1]                 # одно значение по позициям
```

### Булева индексация (фильтрация)
```python
# Простые условия
df[df['Возраст'] > 30]
df[df['Город'] == 'Москва']
df[df['Имя'].str.startswith('А')]

# Множественные условия
df[(df['Возраст'] > 25) & (df['Город'] == 'Москва')]   # И
df[(df['Возраст'] < 25) | (df['Возраст'] > 35)]        # ИЛИ
df[~(df['Город'] == 'Москва')]                         # НЕ

# Другие методы фильтрации
df[df['Возраст'].between(25, 35)]                      # диапазон
df[df['Имя'].isin(['Анна', 'Борис'])]                  # вхождение в список
df[df['Зарплата'].gt(50000)]                           # больше чем
df[df['Зарплата'].lt(70000)]                           # меньше чем
df[df['Имя'].str.contains('ан')]                       # содержит подстроку
df[df['Имя'].str.match('^А.*')]                        # регулярное выражение

# Фильтрация с query()
df.query('Возраст > 30')
df.query('Возраст > 30 and Город == "Москва"')
df.query('Имя in ["Анна", "Борис"]')
df.query('Зарплата > @threshold', local_dict={'threshold': 60000})
```

## Работа с пропущенными значениями

### Обнаружение пропусков
```python
# Основные методы
df.isnull().sum()                          # по столбцам
df.isnull().sum(axis=1)                    # по строкам
df.isnull().any()                          # есть ли хоть один пропуск
df.isnull().any(axis=1)                    # по строкам

# Визуализация пропусков
import matplotlib.pyplot as plt
import seaborn as sns

plt.figure(figsize=(10, 6))
sns.heatmap(df.isnull(), cbar=True, yticklabels=False, cmap='viridis')
plt.show()

# Более сложный анализ пропусков
def analyze_missingness(df):
    missing = df.isnull().sum()
    missing_percent = (missing / len(df)) * 100
    missing_table = pd.DataFrame({
        'Количество': missing,
        'Процент': missing_percent
    })
    return missing_table[missing_table['Количество'] > 0].sort_values('Количество', ascending=False)

analyze_missingness(df)
```

### Обработка пропусков
```python
# Удаление пропусков
df.dropna()                                # удалить строки с любыми пропусками
df.dropna(axis=1)                          # удалить столбцы с пропусками
df.dropna(how='all')                       # удалить если все значения NaN
df.dropna(how='any')                       # удалить если любое значение NaN
df.dropna(thresh=3)                        # оставить строки с минимум 3 не-NaN
df.dropna(subset=['Имя', 'Возраст'])       # проверять только указанные столбцы

# Заполнение пропусков
df.fillna(0)                               # заполнить нулями
df.fillna(method='ffill')                  # заполнить предыдущим значением
df.fillna(method='bfill')                  # заполнить следующим значением
df.fillna(df.mean())                       # заполнить средним по столбцу
df.fillna(df.median())                     # заполнить медианой
df.fillna(df.mode().iloc[0])               # заполнить модой

# Разные стратегии для разных столбцов
fill_values = {'Возраст': df['Возраст'].mean(), 'Город': 'Неизвестно'}
df.fillna(fill_values)

# Интерполяция
df.interpolate()                           # линейная интерполяция
df.interpolate(method='time')              # интерполяция по времени
df.interpolate(method='polynomial', order=2)  # полиномиальная

# Продвинутые методы
from sklearn.impute import SimpleImputer, KNNImputer

# Простой импьютер
imputer = SimpleImputer(strategy='mean')
df_imputed = pd.DataFrame(imputer.fit_transform(df), columns=df.columns)

# KNN импьютер
knn_imputer = KNNImputer(n_neighbors=2)
df_knn = pd.DataFrame(knn_imputer.fit_transform(df), columns=df.columns)
```

## Изменение и преобразование данных

### Добавление и удаление столбцов
```python
# Добавление новых столбцов
df['Зарплата_в_евро'] = df['Зарплата'] / 95
df['Возрастная_группа'] = np.where(df['Возраст'] > 30, 'Старше 30', 'Моложе 30')
df['Полное_имя'] = df['Имя'] + ' из ' + df['Город']

# Метод assign (создает копию)
df_new = df.assign(
    Зарплата_в_долларах = df['Зарплата'] / 90,
    Стаж = [2, 5, 8, 3, 6]
)

# Удаление столбцов
df.drop('Зарплата_в_евро', axis=1, inplace=True)
df.drop(['Столбец1', 'Столбец2'], axis=1)
del df['Ненужный_столбец']

# Переименование
df.rename(columns={'Имя': 'ФИО', 'Возраст': 'Возраст_лет'})
df.rename(columns=lambda x: x.lower().replace(' ', '_'))  # функция для всех названий
df.columns = ['col1', 'col2', 'col3']                    # полная замена
```

### Изменение строк
```python
# Добавление строк
new_row = {'Имя': 'Федор', 'Возраст': 40, 'Город': 'Москва'}
df = df.append(new_row, ignore_index=True)

# Изменение существующих строк
df.loc[0, 'Возраст'] = 26                    # прямое присваивание
df.at[0, 'Имя'] = 'Анна-Мария'

# Удаление строк
df.drop([0, 2], axis=0)                      # по индексам
df.drop(df[df['Возраст'] < 25].index)        # по условию
```

### Применение функций
```python
# apply для Series
df['Имя_верхний_регистр'] = df['Имя'].apply(lambda x: x.upper())
df['Длина_имени'] = df['Имя'].apply(len)

# apply для DataFrame
df[['Имя', 'Город']] = df[['Имя', 'Город']].apply(lambda x: x.str.upper())
df_numeric = df.select_dtypes(include=[np.number]).apply(lambda x: x * 2)

# applymap для поэлементного преобразования
df_str = df.applymap(str)                    # все значения в строки

# transform для групповых преобразований
df['Зарплата_средняя_по_городу'] = df.groupby('Город')['Зарплата'].transform('mean')

# pipe для цепочек операций
def add_bonus(df, bonus):
    df['Зарплата_с_бонусом'] = df['Зарплата'] + bonus
    return df

df = df.pipe(add_bonus, bonus=5000)
```

### Map и Replace
```python
# Map для замены значений
city_codes = {'Москва': 1, 'СПб': 2, 'Казань': 3, 'Новосибирск': 4}
df['Код_города'] = df['Город'].map(city_codes)

# Replace для замены
df['Город'] = df['Город'].replace({'СПб': 'Санкт-Петербург'})
df.replace({'Москва': 'MOW', 'СПб': 'LED'}, inplace=True)

# Категориальные данные
df['Город_cat'] = df['Город'].astype('category')
df['Город_cat'] = df['Город_cat'].cat.add_categories(['Екатеринбург'])
```

## Сортировка и переиндексация

### Сортировка
```python
# По значениям
df.sort_values('Возраст')                    # по возрастанию
df.sort_values('Возраст', ascending=False)   # по убыванию
df.sort_values(['Город', 'Возраст'])         # по нескольким столбцам
df.sort_values(['Город', 'Возраст'], ascending=[True, False])

# По индексу
df.sort_index()                              # сортировка индекса
df.sort_index(ascending=False)

# По значениям с игнорированием индекса
df.sort_values('Возраст', ignore_index=True)

# nlargest и nsmallest
df.nlargest(3, 'Зарплата')                   # 3 наибольшие зарплаты
df.nsmallest(2, 'Возраст')                   # 2 наименьших возраста
```

### Работа с индексами
```python
# Установка индекса
df.set_index('Имя')                          # столбец как индекс
df.set_index(['Город', 'Имя'])               # мультииндекс

# Сброс индекса
df.reset_index()                             # индекс становится столбцом
df.reset_index(drop=True)                    # удалить старый индекс

# Переиндексация
new_index = ['a', 'b', 'c', 'd', 'e']
df_reindexed = df.reindex(new_index)         # новый порядок индексов

# Мультииндекс
df_multi = df.set_index(['Город', 'Имя'])
df_multi.index.names                         # имена уровней индекса
df_multi.index.levels                        # уровни индекса
df_multi.xs('Москва', level='Город')         # выбор по уровню
```

## Группировка и агрегация

### Базовая группировка
```python
# Простая группировка
grouped = df.groupby('Город')
grouped.size()                               # количество в каждой группе
grouped.count()                              # количество не-NaN значений
grouped.mean()                               # среднее по группам

# Группировка по нескольким столбцам
df.groupby(['Город', 'Отдел']).size()

# Итерация по группам
for city, group in df.groupby('Город'):
    print(f"Город: {city}")
    print(f"Количество: {len(group)}")
    print(group)
    print()

# Получение конкретной группы
moscow_group = df.groupby('Город').get_group('Москва')
```

### Агрегация
```python
# Одна агрегирующая функция
df.groupby('Город')['Зарплата'].mean()

# Несколько функций для одного столбца
df.groupby('Город')['Зарплата'].agg(['mean', 'std', 'min', 'max'])

# Разные функции для разных столбцов
agg_dict = {
    'Возраст': ['mean', 'max'],
    'Зарплата': ['sum', 'mean', 'std'],
    'Имя': 'count'
}
df.groupby('Город').agg(agg_dict)

# Пользовательские функции агрегации
def salary_range(series):
    return series.max() - series.min()

df.groupby('Город')['Зарплата'].agg(salary_range)

# agg с лямбда-функциями
df.groupby('Город').agg({
    'Зарплата': lambda x: x.quantile(0.75) - x.quantile(0.25)
})
```

### Transform и Filter
```python
# Transform - применяет функцию и возвращает объект того же размера
df['Средняя_зарплата_по_городу'] = df.groupby('Город')['Зарплата'].transform('mean')
df['Зарплата_нормированная'] = df.groupby('Город')['Зарплата'].transform(lambda x: (x - x.mean()) / x.std())

# Filter - фильтрация групп
# Оставить только группы с более чем 1 сотрудником
df_filtered = df.groupby('Город').filter(lambda x: len(x) > 1)
```

### Сводные таблицы
```python
# Pivot table
pivot = pd.pivot_table(df, 
                      values='Зарплата',
                      index='Город',
                      columns='Отдел',
                      aggfunc='mean',
                      fill_value=0,
                      margins=True)          # итоги

# Pivot (изменение формы)
df_pivot = df.pivot(index='Имя', columns='Город', values='Зарплата')

# Melt (преобразование в длинный формат)
df_melted = pd.melt(df, 
                   id_vars=['Имя', 'Город'],
                   value_vars=['Возраст', 'Зарплата'],
                   var_name='Параметр',
                   value_name='Значение')
```

## Объединение и соединение данных

### Конкатенация
```python
# Объединение по строкам (дополнительные строки)
df1 = pd.DataFrame({'A': [1, 2], 'B': [3, 4]})
df2 = pd.DataFrame({'A': [5, 6], 'B': [7, 8]})
result = pd.concat([df1, df2], ignore_index=True)

# Объединение по столбцам
df3 = pd.DataFrame({'C': [9, 10], 'D': [11, 12]})
result = pd.concat([df1, df3], axis=1)

# Разные параметры
result = pd.concat([df1, df2], 
                  ignore_index=True,
                  keys=['df1', 'df2'])       # метки источников
```

### Слияние (JOIN)
```python
# Подготовка данных
df1 = pd.DataFrame({
    'key': ['A', 'B', 'C', 'D'],
    'value1': [1, 2, 3, 4]
})

df2 = pd.DataFrame({
    'key': ['B', 'D', 'E', 'F'],
    'value2': [5, 6, 7, 8]
})

# INNER JOIN (пересечение)
pd.merge(df1, df2, on='key')

# LEFT JOIN (все из левой таблицы)
pd.merge(df1, df2, on='key', how='left')

# RIGHT JOIN (все из правой таблицы)
pd.merge(df1, df2, on='key', how='right')

# OUTER JOIN (объединение)
pd.merge(df1, df2, on='key', how='outer')

# JOIN по разным названиям столбцов
df3 = pd.DataFrame({'key1': ['A', 'B'], 'value3': [10, 20]})
pd.merge(df1, df3, left_on='key', right_on='key1')

# JOIN по индексу
df1.join(df2, how='inner')

# Параметры слияния
pd.merge(df1, df2, on='key', 
         suffixes=('_left', '_right'),      # суффиксы для одинаковых названий
         indicator=True)                    # показывать источник данных
```

### Сравнение и поиск различий
```python
# Поиск различий
df1.compare(df2)                            # сравнение двух DataFrame

# Проверка равенства
df1.equals(df2)

# Объединение с сохранением информации о источнике
pd.merge(df1, df2, on='key', indicator=True).query('_merge == "left_only"')
```

## Временные ряды и даты

### Создание временных данных
```python
# Диапазоны дат
dates = pd.date_range('2023-01-01', periods=10, freq='D')  # ежедневно
dates = pd.date_range('2023-01-01', periods=12, freq='M')  # ежемесячно
dates = pd.date_range('2023-01-01', '2023-12-31', freq='W') # еженедельно

# Временные Series и DataFrame
ts = pd.Series(np.random.randn(10), index=dates)
df_time = pd.DataFrame({'A': np.random.randn(10), 
                       'B': np.random.randn(10)}, 
                      index=dates)
```

### Работа с датами
```python
# Преобразование в datetime
df['date_str'] = ['2023-01-01', '2023-01-02', '2023-01-03']
df['date'] = pd.to_datetime(df['date_str'])
df['date'] = pd.to_datetime(df['date_str'], format='%Y-%m-%d')

# Извлечение компонентов даты
df['год'] = df['date'].dt.year
df['месяц'] = df['date'].dt.month
df['день'] = df['date'].dt.day
df['день_недели'] = df['date'].dt.dayofweek
df['название_дня'] = df['date'].dt.day_name()
df['квартал'] = df['date'].dt.quarter
df['это_выходной'] = df['date'].dt.dayofweek.isin([5, 6])

# Операции с датами
df['date'] + pd.Timedelta(days=1)           # добавить 1 день
df['date'] - pd.Timedelta(hours=6)          # вычесть 6 часов
df['date'].max() - df['date'].min()         # разница между датами
```

### Ресемплинг и скользящие окна
```python
# Ресемплинг временных рядов
df_time.resample('M').mean()                # по месяцам
df_time.resample('W').sum()                 # по неделям
df_time.resample('Q').std()                 # по кварталам

# Несколько агрегаций при ресемплинге
df_time.resample('M').agg({'A': 'mean', 'B': 'sum'})

# Скользящие окна
df_time.rolling(window=3).mean()            # скользящее среднее за 3 периода
df_time.rolling(window=5, min_periods=1).mean()  # с минимальным количеством периодов
df_time.rolling(window=7, center=True).mean()    # центрированное окно

# Расширяющееся окно
df_time.expanding().mean()                  # среднее от начала до текущей точки

# Экспоненциальное сглаживание
df_time.ewm(span=3).mean()
```

## Текстовая обработка

### Строковые методы
```python
# Базовые операции
df['Имя'].str.upper()                       # верхний регистр
df['Имя'].str.lower()                       # нижний регистр
df['Имя'].str.title()                       # заглавные первые буквы
df['Имя'].str.len()                         # длина строк
df['Имя'].str.strip()                       # удаление пробелов
df['Имя'].str.lstrip()                      # удаление слева
df['Имя'].str.rstrip()                      # удаление справа

# Поиск и замена
df['Имя'].str.contains('ан')               # содержит подстроку
df['Имя'].str.startswith('А')              # начинается с
df['Имя'].str.endswith('а')                # заканчивается на
df['Имя'].str.replace('а', 'о')            # замена
df['Имя'].str.find('н')                    # позиция подстроки

# Разделение и соединение
df['Имя'].str.split(' ')                   # разделение по пробелу
df['Имя'].str.split(' ', expand=True)      # разделение в DataFrame
df['Имя'].str.split(' ', n=1)              # ограничение количества разделений
df['Имя'].str.cat(sep=', ')                # соединение всех строк
```

### Регулярные выражения
```python
# Поиск с регулярными выражениями
df['Имя'].str.contains('^А.*', regex=True)  # начинается с А
df['Имя'].str.extract('(\w+)')              # извлечение групп
df['Имя'].str.extractall('(\w+)')           # все совпадения
df['Имя'].str.findall('\w+')                # список всех совпадений

# Замена с регулярными выражениями
df['Текст'] = df['Текст'].str.replace(r'\d+', 'NUM', regex=True)
```

### Категориальные данные
```python
# Работа с категориями
df['Город_cat'] = df['Город'].astype('category')
df['Город_cat'].cat.categories              # категории
df['Город_cat'].cat.codes                   # коды категорий
df['Город_cat'].cat.rename_categories({'Москва': 'Moscow'})  # переименование

# Добавление/удаление категорий
df['Город_cat'] = df['Город_cat'].cat.add_categories(['Владивосток'])
df['Город_cat'] = df['Город_cat'].cat.remove_categories(['Казань'])

# Упорядоченные категории
df['Размер'] = pd.Categorical(df['Размер'], 
                             categories=['S', 'M', 'L', 'XL'], 
                             ordered=True)
```

## Статистические функции

### Описательная статистика
```python
# Основные статистики
df.mean()                                   # среднее
df.median()                                 # медиана
df.mode()                                   # мода
df.std()                                    # стандартное отклонение
df.var()                                    # дисперсия
df.sem()                                    # стандартная ошибка среднего
df.skew()                                   # асимметрия
df.kurtosis()                               # эксцесс

# Квантили и процентили
df.quantile(0.25)                           # 25-й процентиль
df.quantile([0.25, 0.5, 0.75])             # несколько квантилей

# Минимум/максимум
df.min()
df.max()
df.idxmin()                                 # индекс минимального значения
df.idxmax()                                 # индекс максимального значения

# Суммы и накопленные суммы
df.sum()
df.cumsum()                                 # накопленная сумма
df.cumprod()                                # накопленное произведение
df.cummax()                                 # накопленный максимум
df.cummin()                                 # накопленный минимум
```

### Корреляции и ковариации
```python
# Корреляции
df.corr()                                   # матрица корреляций Пирсона
df.corr(method='spearman')                  # корреляция Спирмена
df.corr(method='kendall')                   # корреляция Кендалла

# Ковариации
df.cov()

# Попарные корреляции
df['Зарплата'].corr(df['Возраст'])
```

### Статистические тесты
```python
from scipy import stats

# t-тест
stats.ttest_1samp(df['Зарплата'], 60000)    # одновыборочный
stats.ttest_ind(df[df['Город']=='Москва']['Зарплата'], 
                df[df['Город']!='Москва']['Зарплата'])  # двухвыборочный

# Другие тесты
stats.pearsonr(df['Зарплата'], df['Возраст'])  # корреляция Пирсона
stats.chi2_contingency(pd.crosstab(df['Город'], df['Отдел']))  # хи-квадрат
```

## Визуализация данных

### Встроенная визуализация pandas
```python
# Линейные графики
df.plot()                                   # все числовые столбцы
df['Зарплата'].plot()                       # один столбец
df[['Зарплата', 'Возраст']].plot()          # несколько столбцов

# Столбчатые диаграммы
df['Зарплата'].plot.bar()                   # вертикальные столбцы
df['Зарплата'].plot.barh()                  # горизонтальные столбцы
df.groupby('Город')['Зарплата'].mean().plot.bar()

# Гистограммы
df['Зарплата'].plot.hist(bins=20, alpha=0.7)
df[['Зарплата', 'Возраст']].plot.hist(alpha=0.5, bins=15)

# Ящики с усами
df['Зарплата'].plot.box()
df[['Зарплата', 'Возраст']].plot.box()

# Круговые диаграммы
df['Город'].value_counts().plot.pie(autopct='%1.1f%%')

# Диаграммы рассеяния
df.plot.scatter(x='Возраст', y='Зарплата', s=100, alpha=0.6)

# Настройка графиков
ax = df.plot(figsize=(10, 6), 
            title='Заголовок',
            grid=True,
            legend=True)
ax.set_xlabel('Ось X')
ax.set_ylabel('Ось Y')
plt.tight_layout()
plt.show()
```

### Интеграция с Matplotlib и Seaborn
```python
import matplotlib.pyplot as plt
import seaborn as sns

# Matplotlib
plt.figure(figsize=(12, 8))
plt.subplot(2, 2, 1)
df['Зарплата'].hist(bins=15)
plt.title('Распределение зарплат')

plt.subplot(2, 2, 2)
df.boxplot(column='Зарплата', by='Город')
plt.title('Зарплаты по городам')

plt.tight_layout()
plt.show()

# Seaborn
plt.figure(figsize=(10, 6))
sns.boxplot(x='Город', y='Зарплата', data=df)
plt.title('Распределение зарплат по городам')
plt.xticks(rotation=45)
plt.show()

# Heatmap корреляций
plt.figure(figsize=(8, 6))
sns.heatmap(df.corr(), annot=True, cmap='coolwarm', center=0)
plt.title('Матрица корреляций')
plt.show()
```

## Продвинутые техники

### Стилизация DataFrame
```python
# Условное форматирование
def highlight_high(s):
    return ['background-color: yellow' if v > s.quantile(0.8) else '' for v in s]

df.style.apply(highlight_high, subset=['Зарплата'])

# Градиентная заливка
df.style.background_gradient(cmap='Blues', subset=['Зарплата'])

# Форматирование чисел
df.style.format({
    'Зарплата': '{:.0f} руб.',
    'Возраст': '{:.0f} лет'
})

# Барчарты в ячейках
df.style.bar(subset=['Зарплата'], color='lightblue')
```

### Мультииндекс
```python
# Создание мультииндекса
arrays = [['Москва', 'Москва', 'СПб', 'СПб', 'Казань'],
          ['IT', 'Маркетинг', 'IT', 'HR', 'IT']]
index = pd.MultiIndex.from_arrays(arrays, names=['Город', 'Отдел'])
df_multi = pd.DataFrame({'Зарплата': [50000, 55000, 60000, 45000, 52000]}, index=index)

# Работа с мультииндексом
df_multi.xs('Москва', level='Город')        # выбор по первому уровню
df_multi.xs(('Москва', 'IT'), level=['Город', 'Отдел'])  # по нескольким уровням

# Срезы
df_multi.loc['Москва':'СПб']                # срез по первому уровню
df_multi.loc[(slice(None), 'IT'), :]        # все города, отдел IT
```

### Пользовательские функции
```python
# Векторизованные функции
@np.vectorize
def salary_category(salary):
    if salary < 50000:
        return 'Низкая'
    elif salary < 70000:
        return 'Средняя'
    else:
        return 'Высокая'

df['Категория_зарплаты'] = salary_category(df['Зарплата'])

# Функции с несколькими аргументами
def calculate_bonus(row, multiplier=0.1):
    return row['Зарплата'] * multiplier

df['Бонус'] = df.apply(calculate_bonus, axis=1, multiplier=0.15)
```

## Оптимизация производительности

### Эффективные типы данных
```python
# Оптимизация типов
def optimize_dtypes(df):
    # Числовые столбцы
    int_cols = df.select_dtypes(include=['int']).columns
    df[int_cols] = df[int_cols].apply(pd.to_numeric, downcast='integer')
    
    # Вещественные столбцы
    float_cols = df.select_dtypes(include=['float']).columns
    df[float_cols] = df[float_cols].apply(pd.to_numeric, downcast='float')
    
    # Категориальные столбцы
    obj_cols = df.select_dtypes(include=['object']).columns
    for col in obj_cols:
        num_unique = df[col].nunique()
        num_total = len(df[col])
        if num_unique / num_total < 0.5:  # если меньше 50% уникальных значений
            df[col] = df[col].astype('category')
    
    return df

df_optimized = optimize_dtypes(df)
```

### Векторизация операций
```python
# НЕВЕКТОРИЗОВАННЫЙ подход (медленный)
def slow_calculation(row):
    if row['Возраст'] > 30:
        return row['Зарплата'] * 1.1
    else:
        return row['Зарплата']

# ВЕКТОРИЗОВАННЫЙ подход (быстрый)
df['Новая_зарплата'] = np.where(df['Возраст'] > 30, 
                               df['Зарплата'] * 1.1, 
                               df['Зарплата'])

# Использование NumPy для сложных вычислений
df['Сложный_расчет'] = np.log1p(df['Зарплата']) * np.sqrt(df['Возраст'])
```

### Методы оптимизации
```python
# Использование query() для фильтрации
# Медленно
df[df['Возраст'] > 30][df['Зарплата'] > 50000]

# Быстро
df.query('Возраст > 30 and Зарплата > 50000')

# Использование inplace=True
df.reset_index(drop=True, inplace=True)     # избегаем создания копии

# Предварительная фильтрация
# Медленно
df.groupby('Город').mean().loc['Москва']

# Быстро
df[df['Город'] == 'Москва'].mean()
```

## Работа с большими данными

### Чтение по частям
```python
# Чтение файла по частям
chunk_size = 10000
chunks = []
for chunk in pd.read_csv('large_file.csv', chunksize=chunk_size):
    # Обработка каждой части
    processed_chunk = chunk[chunk['Возраст'] > 25]
    chunks.append(processed_chunk)

df_combined = pd.concat(chunks, ignore_index=True)

# Использование iterator
reader = pd.read_csv('large_file.csv', iterator=True, chunksize=10000)
first_chunk = reader.get_chunk(10000)
```

### Dask для распределенных вычислений
```python
import dask.dataframe as dd

# Чтение больших файлов
ddf = dd.read_csv('very_large_*.csv')       # можно использовать шаблоны
ddf = ddf[ddf['Возраст'] > 25]              # ленивые вычисления
result = ddf.compute()                      # выполнение вычислений

# Группировки в Dask
result = ddf.groupby('Город')['Зарплата'].mean().compute()
```

### Оптимизация памяти
```python
# Анализ использования памяти
df.info(memory_usage='deep')
df.memory_usage(deep=True)

# Сжатие данных
def compress_dataframe(df):
    result = df.copy()
    for col in result.columns:
        col_type = result[col].dtype
        
        if col_type == 'object':
            result[col] = result[col].astype('category')
        
        elif col_type in ['int64', 'int32']:
            c_min = result[col].min()
            c_max = result[col].max()
            
            if c_min > np.iinfo(np.int8).min and c_max < np.iinfo(np.int8).max:
                result[col] = result[col].astype(np.int8)
            elif c_min > np.iinfo(np.int16).min and c_max < np.iinfo(np.int16).max:
                result[col] = result[col].astype(np.int16)
            elif c_min > np.iinfo(np.int32).min and c_max < np.iinfo(np.int32).max:
                result[col] = result[col].astype(np.int32)
            else:
                result[col] = result[col].astype(np.int64)
        
        elif col_type in ['float64', 'float32']:
            c_min = result[col].min()
            c_max = result[col].max()
            
            if c_min > np.finfo(np.float32).min and c_max < np.finfo(np.float32).max:
                result[col] = result[col].astype(np.float32)
            else:
                result[col] = result[col].astype(np.float64)
    
    return result

df_compressed = compress_dataframe(df)
```

## Пример полного анализа

```python
def complete_data_analysis(file_path):
    """
    Полный цикл анализа данных от загрузки до визуализации
    """
    
    # 1. Загрузка данных
    print("1. Загрузка данных...")
    df = pd.read_csv(file_path)
    
    # 2. Первичный осмотр
    print("2. Первичный осмотр...")
    print(f"Размер данных: {df.shape}")
    print(f"\nТипы данных:\n{df.dtypes}")
    print(f"\nПервые 5 строк:\n{df.head()}")
    
    # 3. Анализ пропусков
    print("3. Анализ пропусков...")
    missing_info = pd.DataFrame({
        'Количество': df.isnull().sum(),
        'Процент': (df.isnull().sum() / len(df)) * 100
    }).sort_values('Количество', ascending=False)
    
    print(missing_info[missing_info['Количество'] > 0])
    
    # 4. Обработка пропусков
    print("4. Обработка пропусков...")
    # Для числовых столбцов - медиана
    numeric_cols = df.select_dtypes(include=[np.number]).columns
    df[numeric_cols] = df[numeric_cols].fillna(df[numeric_cols].median())
    
    # Для категориальных - мода
    categorical_cols = df.select_dtypes(include=['object']).columns
    for col in categorical_cols:
        df[col] = df[col].fillna(df[col].mode()[0] if not df[col].mode().empty else 'Неизвестно')
    
    # 5. Анализ выбросов
    print("5. Анализ выбросов...")
    numeric_df = df.select_dtypes(include=[np.number])
    
    Q1 = numeric_df.quantile(0.25)
    Q3 = numeric_df.quantile(0.75)
    IQR = Q3 - Q1
    
    outliers = ((numeric_df < (Q1 - 1.5 * IQR)) | (numeric_df > (Q3 + 1.5 * IQR))).sum()
    print(f"Выбросы по столбцам:\n{outliers}")
    
    # 6. Статистический анализ
    print("6. Статистический анализ...")
    print("Основные статистики:")
    print(df.describe())
    
    print("\nКорреляционная матрица:")
    print(df.corr())
    
    # 7. Группировки и агрегации
    print("7. Группировки и агрегации...")
    if 'Город' in df.columns and 'Зарплата' in df.columns:
        city_stats = df.groupby('Город').agg({
            'Зарплата': ['mean', 'median', 'std', 'count'],
            'Возраст': 'mean'
        }).round(2)
        print("Статистика по городам:")
        print(city_stats)
    
    # 8. Визуализация
    print("8. Визуализация...")
    import matplotlib.pyplot as plt
    import seaborn as sns
    
    plt.figure(figsize=(15, 10))
    
    # Распределение числовых переменных
    numeric_cols = df.select_dtypes(include=[np.number]).columns
    if len(numeric_cols) > 0:
        plt.subplot(2, 2, 1)
        df[numeric_cols.iloc[0]].hist(bins=20, alpha=0.7)
        plt.title(f'Распределение {numeric_cols[0]}')
    
    # Boxplot
    if len(numeric_cols) > 0:
        plt.subplot(2, 2, 2)
        df[numeric_cols.iloc[0]].plot.box()
        plt.title(f'Boxplot {numeric_cols[0]}')
    
    # Корреляционная матрица
    if len(numeric_cols) > 1:
        plt.subplot(2, 2, 3)
        sns.heatmap(df.corr(), annot=True, cmap='coolwarm', center=0)
        plt.title('Матрица корреляций')
    
    # Количества по категориям
    categorical_cols = df.select_dtypes(include=['object']).columns
    if len(categorical_cols) > 0:
        plt.subplot(2, 2, 4)
        df[categorical_cols.iloc[0]].value_counts().plot.pie(autopct='%1.1f%%')
        plt.title(f'Распределение {categorical_cols[0]}')
    
    plt.tight_layout()
    plt.show()
    
    # 9. Сохранение результатов
    print("9. Сохранение результатов...")
    df.to_csv('processed_data.csv', index=False)
    
    # 10. Генерация отчета
    print("10. Генерация отчета...")
    report = f"""
    ОТЧЕТ ПО АНАЛИЗУ ДАННЫХ
    =======================
    Исходный файл: {file_path}
    Размер данных: {df.shape}
    Период анализа: {pd.Timestamp.now()}
    
    ОСНОВНЫЕ МЕТРИКИ:
    - Обработано строк: {len(df)}
    - Количество столбцов: {len(df.columns)}
    - Пропущенные значения: {df.isnull().sum().sum()}
    - Дубликаты: {df.duplicated().sum()}
    
    СТАТИСТИКА ПО ЧИСЛОВЫМ СТОЛБЦАМ:
    {df.describe()}
    """
    
    with open('analysis_report.txt', 'w', encoding='utf-8') as f:
        f.write(report)
    
    print("Анализ завершен!")
    return df

# Запуск полного анализа
# df_analyzed = complete_data_analysis('your_data.csv')
```