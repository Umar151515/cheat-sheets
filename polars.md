# Iпаргалка по Polars + Plotly Express Python

## ОГЛАВЛЕНИЕ

1. [Введение и установка](#введение-и-установка)
2. [Основы Polars - создание DataFrame](#основы-polars---создание-dataframe)
3. [Методы работы с DataFrame](#методы-работы-с-dataframe)
4. [Чтение и запись данных](#чтение-и-запись-данных)
5. [Селекция и фильтрация данных](#селекция-и-фильтрация-данных)
6. [Трансформации данных](#трансформации-данных)
7. [Агрегации и группировки](#агрегации-и-группировки)
8. [Оконные функции](#оконные-функции)
9. [Работа с временными рядами](#работа-с-временными-рядами)
10. [Работа со строками](#работа-со-строками)
11. [Работа с категориальными данными](#работа-с-категориальными-данными)
12. [Основы Plotly Express](#основы-plotly-express)
13. [Типы графиков Plotly Express](#типы-графиков-plotly-express)
14. [Настройка визуализаций](#настройка-визуализаций)
15. [Интеграция Polars + Plotly Express](#интеграция-polars--plotly-express)
16. [Практические примеры анализа](#практические-примеры-анализа)
17. [Создание дашбордов](#создание-дашбордов)
18. [Оптимизация производительности](#оптимизация-производительности)
19. [Полезные функции и утилиты](#полезные-функции-и-утилиты)

---

## Введение и установка

### Установка пакетов

```bash
# Базовые пакеты
pip install polars plotly

# Дополнительные пакеты для расширенной функциональности
pip install pandas numpy scipy matplotlib
pip install openpyxl  # для работы с Excel
pip install pyarrow   # для формата Parquet

# Установка последних версий
pip install polars -U
pip install plotly -U
```

### Импорт библиотек

```python
import polars as pl
import plotly.express as px
import plotly.graph_objects as go
from plotly.subplots import make_subplots
import plotly.io as pio

# Дополнительные импорты
import numpy as np
import datetime
from datetime import date, timedelta
import random
from typing import List, Dict, Any

# Настройка отображения Plotly
pio.templates.default = "plotly_white"
```

## Основы Polars - создание DataFrame

### Создание DataFrame из различных источников

```python
# 1. Из словаря
df_dict = pl.DataFrame({
    "id": [1, 2, 3, 4, 5],
    "name": ["Анна", "Борис", "Виктор", "Галина", "Дмитрий"],
    "age": [25, 32, 28, 41, 36],
    "salary": [50000, 75000, 60000, 90000, 80000],
    "department": ["IT", "HR", "IT", "Finance", "IT"],
    "join_date": [
        date(2020, 1, 15), date(2019, 3, 20), 
        date(2021, 6, 10), date(2018, 11, 5), 
        date(2019, 8, 25)
    ]
})

# 2. Из списка списков с указанием схемы
data = [
    [1, "A", 100.5, True],
    [2, "B", 200.3, False],
    [3, "A", 150.7, True]
]

schema = ["id", "category", "value", "flag"]
df_list = pl.DataFrame(data, schema=schema)

# 3. С явным указанием типов данных
df_typed = pl.DataFrame(
    {
        "int_col": [1, 2, 3],
        "float_col": [1.1, 2.2, 3.3],
        "str_col": ["a", "b", "c"],
        "bool_col": [True, False, True],
        "date_col": [date(2023, 1, 1), date(2023, 1, 2), date(2023, 1, 3)]
    },
    schema={
        "int_col": pl.Int32,
        "float_col": pl.Float64,
        "str_col": pl.Utf8,
        "bool_col": pl.Boolean,
        "date_col": pl.Date
    }
)

# 4. Создание с помощью генераторов
df_large = pl.DataFrame({
    "id": range(1, 1001),
    "value": [random.gauss(100, 15) for _ in range(1000)],
    "group": [f"group_{i%5}" for i in range(1000)]
})

# 5. Специальные конструкторы
# Пустой DataFrame с определенной схемой
df_empty = pl.DataFrame(schema={"name": pl.Utf8, "age": pl.Int32})

# DataFrame из одной колонки
df_single = pl.DataFrame({"numbers": range(10)})
```

## Методы работы с DataFrame

### Основные методы просмотра данных

```python
# Создадим тестовый DataFrame для демонстрации
df = pl.DataFrame({
    "name": ["Анна", "Борис", "Виктор", "Галина", "Дмитрий", "Елена"],
    "age": [25, 32, 28, 41, 36, 29],
    "salary": [50000, 75000, 60000, 90000, 80000, 55000],
    "department": ["IT", "HR", "IT", "Finance", "IT", "HR"],
    "experience": [2, 5, 3, 10, 7, 4]
})

# Методы просмотра
print("Первые 3 строки:")
print(df.head(3))

print("\nПоследние 3 строки:")
print(df.tail(3))

print("\nСлучайные 3 строки:")
print(df.sample(3))

print("\nСтатистическое описание:")
print(df.describe())

print("\nИнформация о типах данных:")
print("Типы данных:", df.dtypes)
print("Названия колонок:", df.columns)
print("Форма DataFrame:", df.shape)
print("Размерность:", f"{df.height} строк, {df.width} колонок")

# Дополнительные методы
print("\nУникальные значения в колонке department:")
print(df["department"].unique())

print("\nКоличество уникальных значений:")
print(df.n_unique())
```

### Проверка данных и метаинформации

```python
# Проверка структуры данных
print("Схема DataFrame:")
print(df.schema)

print("\nИнформация о наличии null значений:")
print(df.null_count())

print("\nПроверка на наличие null значений:")
print(df.is_null().any())

print("\nПроверка на уникальность колонок:")
print("Уникальность name:", df["name"].is_unique())
print("Уникальность age:", df["age"].is_unique())

# Быстрая статистика
print("\nБыстрая статистика по числовым колонкам:")
print("Среднее age:", df["age"].mean())
print("Медиана salary:", df["salary"].median())
print("Стандартное отклонение salary:", df["salary"].std())
print("Минимальный age:", df["age"].min())
print("Максимальный salary:", df["salary"].max())
```

## Чтение и запись данных

### Чтение из различных форматов

```python
# CSV файлы
df_csv = pl.read_csv("data.csv")
df_csv_custom = pl.read_csv(
    "data.csv",
    separator=";",           # разделитель
    has_header=True,         # наличие заголовка
    skip_rows=0,             # пропустить строк
    dtypes={"age": pl.Int32, "salary": pl.Float64},  # типы данных
    null_values=["NA", "N/A", ""]  # значения, считающиеся как null
)

# Parquet файлы
df_parquet = pl.read_parquet("data.parquet")
df_parquet_specific = pl.read_parquet(
    "data.parquet",
    columns=["name", "age", "salary"],  # только определенные колонки
    use_pyarrow=True  # использовать pyarrow для чтения
)

# JSON файлы
df_json = pl.read_json("data.json")
df_ndjson = pl.read_ndjson("data.ndjson")  # newline delimited JSON

# Excel файлы
df_excel = pl.read_excel(
    "data.xlsx",
    sheet_name="Sheet1",
    read_options={"header_row": 0}
)

# Чтение из pandas DataFrame
import pandas as pd
pandas_df = pd.DataFrame({"A": [1, 2, 3], "B": [4, 5, 6]})
df_from_pandas = pl.from_pandas(pandas_df)

# Чтение из словаря
data_dict = {"col1": [1, 2, 3], "col2": ["a", "b", "c"]}
df_from_dict = pl.from_dict(data_dict)

# Чтение из списка словарей
data_list_dict = [
    {"name": "Alice", "age": 25},
    {"name": "Bob", "age": 30}
]
df_from_list_dict = pl.from_dicts(data_list_dict)
```

### Запись данных

```python
# Запись в CSV
df.write_csv("output.csv")
df.write_csv(
    "output.csv",
    separator=";",
    include_header=True
)

# Запись в Parquet
df.write_parquet("output.parquet")
df.write_parquet(
    "output.parquet",
    compression="snappy"  # или "gzip", "brotli"
)

# Запись в JSON
df.write_json("output.json")
df.write_ndjson("output.ndjson")

# Запись в Excel
df.write_excel("output.xlsx")

# Запись в базу данных (пример)
# df.write_database("table_name", connection_uri)

# Параметры записи
df.write_csv(
    "output.csv",
    include_bom=True,  # добавить BOM для UTF-8
    datetime_format="%Y-%m-%d",  # формат даты
)
```

## Селекция и фильтрация данных

### Выбор колонок

```python
# Различные способы выбора колонок

# 1. Список колонок
selected = df.select(["name", "age", "salary"])

# 2. Использование pl.col
selected = df.select([
    pl.col("name"),
    pl.col("age"),
    pl.col("salary")
])

# 3. Выбор с переименованием
selected = df.select([
    pl.col("name").alias("employee_name"),
    pl.col("age").alias("employee_age")
])

# 4. Выбор с помощью регулярных выражений
selected = df.select([
    pl.col("^name|age$")  # колонки, начинающиеся на 'name' или заканчивающиеся на 'age'
])

# 5. Исключение колонок
selected = df.select([
    pl.exclude("department")  # все колонки кроме department
])

# 6. Выбор по типу данных
selected = df.select([
    pl.col(pl.Utf8),  # все строковые колонки
    pl.col(pl.Int64)   # все целочисленные колонки
])

# 7. Комбинированный выбор
selected = df.select([
    pl.col("name", "age"),  # конкретные колонки
    pl.col("salary").alias("annual_salary")  # с переименованием
])
```

### Фильтрация строк

```python
# Базовые фильтры
filtered = df.filter(pl.col("age") > 30)
filtered = df.filter(pl.col("department") == "IT")

# Множественные условия
filtered = df.filter(
    (pl.col("age") > 30) & 
    (pl.col("department") == "IT")
)

filtered = df.filter(
    (pl.col("age") > 30) | 
    (pl.col("salary") > 70000)
)

# Фильтрация с отрицанием
filtered = df.filter(~pl.col("department").is_in(["HR", "Finance"]))

# Фильтрация по null значениям
df_with_nulls = pl.DataFrame({
    "name": ["A", "B", "C", None],
    "age": [25, None, 30, 35]
})
filtered = df_with_nulls.filter(pl.col("name").is_not_null())

# Фильтрация по строковым условиям
filtered = df.filter(pl.col("name").str.contains("ан"))  # содержит подстроку
filtered = df.filter(pl.col("name").str.starts_with("А"))  # начинается с
filtered = df.filter(pl.col("name").str.ends_with("а"))  # заканчивается на

# Фильтрация по датам (если есть даты)
df_dates = pl.DataFrame({
    "date": [date(2023, 1, 1), date(2023, 1, 2), date(2023, 1, 3)],
    "value": [100, 200, 300]
})
filtered = df_dates.filter(pl.col("date") > date(2023, 1, 1))

# Комплексные фильтры
filtered = df.filter(
    ((pl.col("age") >= 25) & (pl.col("age") <= 35)) |
    (pl.col("salary") > 80000)
)
```

### Сортировка данных

```python
# Простая сортировка
sorted_df = df.sort("age")  # по возрастанию
sorted_df = df.sort("age", descending=True)  # по убыванию

# Сортировка по нескольким колонкам
sorted_df = df.sort(["department", "salary"])  # сначала по department, потом по salary
sorted_df = df.sort(["department", "salary"], descending=[False, True])  # department по возрастанию, salary по убыванию

# Сортировка с null значениями
df_with_nulls = pl.DataFrame({
    "name": ["A", "B", None, "C"],
    "value": [10, None, 30, 20]
})
sorted_df = df_with_nulls.sort("value", nulls_last=True)  # null значения в конце

# Сортировка по выражению
sorted_df = df.sort(pl.col("salary") / pl.col("age"))  # по вычисляемому полю
```

## Трансформации данных

### Добавление и модификация колонок

```python
# Базовые трансформации
df_transformed = df.with_columns([
    # Арифметические операции
    (pl.col("salary") * 1.1).alias("salary_with_bonus"),
    
    # Математические функции
    pl.col("age").sqrt().alias("sqrt_age"),
    
    # Логические преобразования
    pl.col("age").cast(pl.Float64).alias("age_float"),
    
    # Константные значения
    pl.lit("active").alias("status")
])

# Условные преобразования
df_transformed = df.with_columns([
    pl.when(pl.col("age") > 35)
      .then("Senior")
      .when(pl.col("age") > 25)
      .then("Middle")
      .otherwise("Junior")
      .alias("level"),
    
    pl.when(pl.col("salary") > 70000)
      .then(pl.col("salary") * 1.05)
      .otherwise(pl.col("salary") * 1.1)
      .alias("adjusted_salary")
])

# Работа с датами
df_dates = pl.DataFrame({
    "date": [date(2023, 1, 15), date(2023, 2, 20), date(2023, 3, 25)],
    "value": [100, 200, 300]
})

df_dates_transformed = df_dates.with_columns([
    pl.col("date").dt.year().alias("year"),
    pl.col("date").dt.month().alias("month"),
    pl.col("date").dt.day().alias("day"),
    pl.col("date").dt.weekday().alias("weekday"),
    pl.col("date").dt.ordinal_day().alias("day_of_year")
])

# Цепочка преобразований
result = (df
    .with_columns([
        (pl.col("salary") / 12).alias("monthly_salary"),
        pl.col("name").str.to_uppercase().alias("name_upper")
    ])
    .filter(pl.col("monthly_salary") > 5000)
    .sort("monthly_salary", descending=True)
)
```

### Агрегирующие трансформации

```python
# Трансформации с агрегациями
df_agg = df.with_columns([
    pl.col("salary").mean().alias("avg_salary"),  # среднее по всему DataFrame
    pl.col("age").max().alias("max_age")
])

# Оконные функции (подробнее в соответствующем разделе)
df_window = df.with_columns([
    pl.col("salary").rank("dense").over("department").alias("rank_in_dept"),
    pl.col("salary").mean().over("department").alias("avg_salary_in_dept")
])
```

## Агрегации и группировки

### Базовые агрегации

```python
# Простые агрегации без группировки
aggregations = df.select([
    pl.col("age").mean().alias("average_age"),
    pl.col("salary").mean().alias("average_salary"),
    pl.col("salary").median().alias("median_salary"),
    pl.col("salary").std().alias("std_salary"),
    pl.col("salary").min().alias("min_salary"),
    pl.col("salary").max().alias("max_salary"),
    pl.col("name").count().alias("total_employees"),
    pl.col("name").n_unique().alias("unique_names")
])

# Несколько агрегаций для одной колонки
salary_stats = df.select([
    pl.col("salary").mean().alias("mean"),
    pl.col("salary").median().alias("median"),
    pl.col("salary").std().alias("std_dev"),
    pl.col("salary").quantile(0.25).alias("q25"),
    pl.col("salary").quantile(0.75).alias("q75")
])
```

### Группировки

```python
# Простая группировка
grouped = df.group_by("department").agg([
    pl.col("salary").mean().alias("avg_salary"),
    pl.col("age").mean().alias("avg_age"),
    pl.col("name").count().alias("employee_count")
])

# Группировка по нескольким колонкам
multi_grouped = df.group_by(["department", "age"]).agg([
    pl.col("salary").mean().alias("avg_salary"),
    pl.col("name").count().alias("count")
])

# Различные агрегационные функции
detailed_agg = df.group_by("department").agg([
    # Статистики
    pl.col("salary").count().alias("count"),
    pl.col("salary").mean().alias("mean_salary"),
    pl.col("salary").median().alias("median_salary"),
    pl.col("salary").std().alias("std_salary"),
    pl.col("salary").min().alias("min_salary"),
    pl.col("salary").max().alias("max_salary"),
    
    # Другие агрегации
    pl.col("salary").sum().alias("total_salary"),
    pl.col("salary").first().alias("first_salary"),
    pl.col("salary").last().alias("last_salary"),
    
    # Агрегации для строк
    pl.col("name").first().alias("first_name"),
    pl.col("name").last().alias("last_name"),
    pl.col("name").n_unique().alias("unique_names")
])

# Группировка с фильтрацией
filtered_group = df.filter(pl.col("age") > 25).group_by("department").agg([
    pl.col("salary").mean().alias("avg_salary_over_25")
])
```

### Продвинутые агрегации

```python
# Агрегации с условиями
conditional_agg = df.group_by("department").agg([
    pl.col("salary").filter(pl.col("age") > 30).mean().alias("avg_salary_over_30"),
    pl.col("salary").filter(pl.col("age") <= 30).mean().alias("avg_salary_under_30")
])

# Множественные агрегации для разных колонок
multi_col_agg = df.group_by("department").agg([
    # Для salary
    pl.col("salary").mean().alias("avg_salary"),
    pl.col("salary").max().alias("max_salary"),
    
    # Для age
    pl.col("age").mean().alias("avg_age"),
    pl.col("age").max().alias("max_age"),
    
    # Комбинированные
    (pl.col("salary").sum() / pl.col("age").sum()).alias("salary_per_age_ratio")
])

# Агрегации с сортировкой
sorted_agg = (df
    .sort("salary", descending=True)
    .group_by("department")
    .agg([
        pl.col("name").first().alias("highest_paid"),
        pl.col("salary").first().alias("highest_salary")
    ])
)
```

## Оконные функции

### Базовые оконные функции

```python
# Ранжирование
df_ranked = df.with_columns([
    pl.col("salary").rank("dense").over("department").alias("rank_in_dept"),
    pl.col("salary").rank("ordinal").over("department").alias("row_number_in_dept"),
    pl.col("salary").rank("dense", descending=True).over("department").alias("rank_desc")
])

# Агрегации в окне
df_window_agg = df.with_columns([
    pl.col("salary").mean().over("department").alias("avg_salary_in_dept"),
    pl.col("salary").max().over("department").alias("max_salary_in_dept"),
    pl.col("salary").min().over("department").alias("min_salary_in_dept"),
    pl.col("salary").std().over("department").alias("std_salary_in_dept")
])

# Накопительные суммы
df_cumulative = df.sort("salary").with_columns([
    pl.col("salary").cum_sum().alias("cumulative_salary"),
    pl.col("salary").cum_sum().over("department").alias("cumulative_salary_by_dept")
])
```

### Продвинутые оконные функции

```python
# Разности и изменения
df_changes = df.sort(["department", "salary"]).with_columns([
    pl.col("salary").diff().alias("salary_diff"),
    pl.col("salary").diff().over("department").alias("salary_diff_in_dept"),
    pl.col("salary").pct_change().alias("salary_pct_change"),
    pl.col("salary").pct_change().over("department").alias("salary_pct_change_in_dept")
])

# Скользящие средние
df_rolling = df.sort("name").with_columns([
    pl.col("salary").rolling_mean(window_size=3).alias("rolling_avg_3"),
    pl.col("salary").rolling_mean(window_size=5).alias("rolling_avg_5"),
    pl.col("salary").rolling_std(window_size=3).alias("rolling_std_3")
])

# Оконные функции с несколькими колонками
df_complex_window = df.with_columns([
    # Нормализация внутри групп
    ((pl.col("salary") - pl.col("salary").mean().over("department")) / 
     pl.col("salary").std().over("department")).alias("salary_zscore"),
    
    # Процент от группы
    (pl.col("salary") / pl.col("salary").sum().over("department")).alias("salary_pct_of_dept")
])

# Окна по партициям
df_partitioned = df.with_columns([
    pl.col("salary").sum().over(["department", "age"]).alias("sum_by_dept_age")
])
```

## Работа с временными рядами

### Создание временных рядов

```python
# Создание датафрейма с временным рядом
date_ranges = pl.date_range(
    start=date(2023, 1, 1),
    end=date(2023, 12, 31),
    interval="1d",
    name="date"
)

ts_data = pl.DataFrame({
    "date": date_ranges,
    "value": np.random.randn(len(date_ranges)).cumsum() + 100,
    "category": np.random.choice(["A", "B", "C"], len(date_ranges))
})

# Альтернативное создание
ts_data_2 = pl.DataFrame({
    "date": [date(2023, 1, 1) + timedelta(days=i) for i in range(365)],
    "temperature": [20 + 10 * np.sin(i/365 * 2 * np.pi) + random.gauss(0, 2) for i in range(365)],
    "humidity": [50 + 20 * np.cos(i/365 * 2 * np.pi) + random.gauss(0, 5) for i in range(365)]
})
```

### Операции с временными рядами

```python
# Извлечение компонентов даты
ts_enhanced = ts_data.with_columns([
    pl.col("date").dt.year().alias("year"),
    pl.col("date").dt.month().alias("month"),
    pl.col("date").dt.day().alias("day"),
    pl.col("date").dt.weekday().alias("weekday"),
    pl.col("date").dt.quarter().alias("quarter"),
    pl.col("date").dt.ordinal_day().alias("day_of_year")
])

# Ресемплинг (агрегация по другому временному интервалу)
# Ежедневные данные к ежемесячным
monthly_agg = ts_data.group_by_dynamic(
    "date",
    every="1mo",
    period="1mo",
    include_boundaries=True
).agg([
    pl.col("value").mean().alias("avg_value"),
    pl.col("value").std().alias("std_value"),
    pl.col("value").min().alias("min_value"),
    pl.col("value").max().alias("max_value")
])

# Скользящие окна для временных рядов
ts_smoothed = ts_data.sort("date").with_columns([
    pl.col("value").rolling_mean(window_size=7).alias("7d_ma"),
    pl.col("value").rolling_mean(window_size=30).alias("30d_ma"),
    pl.col("value").rolling_std(window_size=7).alias("7d_std")
])

# Разности и темпы роста
ts_growth = ts_data.sort("date").with_columns([
    pl.col("value").diff().alias("daily_change"),
    pl.col("value").pct_change().alias("daily_pct_change"),
    pl.col("value").pct_change(7).alias("weekly_pct_change")
])
```

### Анализ временных рядов

```python
# Сезонная декомпозиция
def seasonal_decomposition(df: pl.DataFrame, date_col: str, value_col: str, period: int = 365):
    """Простая сезонная декомпозиция"""
    return df.sort(date_col).with_columns([
        pl.col(value_col).rolling_mean(window_size=period).alias("trend"),
        (pl.col(value_col) - pl.col(value_col).rolling_mean(window_size=period)).alias("seasonal"),
        (pl.col(value_col) - pl.col(value_col).rolling_mean(window_size=7)).alias("residual")
    ])

# Применение декомпозиции
ts_decomposed = seasonal_decomposition(ts_data, "date", "value")

# Анализ лагов
ts_lags = ts_data.sort("date").with_columns([
    pl.col("value").shift(1).alias("lag_1"),
    pl.col("value").shift(7).alias("lag_7"),
    pl.col("value").shift(30).alias("lag_30")
])

# Автокорреляция
def autocorrelation(df: pl.DataFrame, value_col: str, max_lag: int = 30):
    """Вычисление автокорреляции"""
    correlations = []
    for lag in range(1, max_lag + 1):
        corr = df[value_col].corr(df[value_col].shift(lag))
        correlations.append({"lag": lag, "autocorrelation": corr})
    return pl.DataFrame(correlations)

autocorr_df = autocorrelation(ts_data, "value")
```

## Работа со строками

### Базовые строковые операции

```python
# Создадим DataFrame для демонстрации строковых операций
string_df = pl.DataFrame({
    "name": ["  John Doe  ", "Alice Smith", "bob brown", "CAROL WHITE"],
    "email": ["john@example.com", "alice.smith@test.org", "bob@company.com", "carol@white.net"],
    "text": ["Quick brown fox", "Jumps over", "The lazy dog", "Sample text here"]
})

# Базовые преобразования
string_processed = string_df.with_columns([
    # Регистр
    pl.col("name").str.to_uppercase().alias("upper_name"),
    pl.col("name").str.to_lowercase().alias("lower_name"),
    pl.col("name").str.to_titlecase().alias("title_name"),
    
    # Обрезка пробелов
    pl.col("name").str.strip().alias("trimmed_name"),
    pl.col("name").str.lstrip().alias("ltrimmed_name"),
    pl.col("name").str.rstrip().alias("rtrimmed_name"),
    
    # Длина строк
    pl.col("name").str.len_bytes().alias("name_length_bytes"),
    pl.col("name").str.len_chars().alias("name_length_chars")
])
```

### Поиск и извлечение из строк

```python
# Поиск в строках
string_search = string_df.with_columns([
    # Содержит подстроку
    pl.col("name").str.contains("Alice").alias("contains_alice"),
    pl.col("email").str.contains(r"\.com$").alias("ends_with_com"),
    
    # Начинается/заканчивается с
    pl.col("name").str.starts_with("A").alias("starts_with_a"),
    pl.col("email").str.ends_with(".com").alias("ends_with_dot_com"),
    
    # Поиск по регулярным выражениям
    pl.col("email").str.extract(r"@(.+)\.", 1).alias("email_domain"),
    pl.col("name").str.extract(r"^(\w+)", 1).alias("first_name")
])

# Разделение строк
string_split = string_df.with_columns([
    pl.col("name").str.split(" ").alias("name_parts"),
    pl.col("name").str.split(" ").list.first().alias("first_name"),
    pl.col("name").str.split(" ").list.last().alias("last_name"),
    pl.col("email").str.split("@").list.last().alias("email_domain_full")
])

# Замена в строках
string_replace = string_df.with_columns([
    pl.col("name").str.replace(" ", "_").alias("name_underscore"),
    pl.col("email").str.replace(r"\.com$", ".org").alias("email_org"),
    pl.col("text").str.replace_all(r"\s", "-").alias("text_dashed")
])
```

### Продвинутые строковые операции

```python
# JSON операции (если есть JSON данные)
json_df = pl.DataFrame({
    "json_col": [
        '{"name": "John", "age": 30}',
        '{"name": "Alice", "age": 25}',
        'invalid json'
    ]
})

json_parsed = json_df.with_columns([
    pl.col("json_col").str.json_decode(pl.Struct({"name": pl.Utf8, "age": pl.Int64})).alias("parsed_json")
])

# Кодирование и хеширование
string_encoded = string_df.with_columns([
    pl.col("email").hash().alias("email_hash"),
    pl.col("name").str.encode("hex").alias("name_hex")
])

# Фильтрация по строковым условиям
filtered_strings = string_df.filter(
    pl.col("name").str.contains("John|Alice") &
    pl.col("email").str.ends_with(".com")
)
```

## Работа с категориальными данными

### Создание и работа с категориями

```python
# Создание категориальных данных
category_df = pl.DataFrame({
    "product": ["Laptop", "Phone", "Tablet", "Laptop", "Phone", "Tablet", "Laptop"],
    "category": ["Electronics", "Electronics", "Electronics", "Electronics", "Electronics", "Electronics", "Electronics"],
    "sales": [1000, 800, 600, 1200, 900, 700, 1100]
})

# Преобразование в категориальный тип
category_enhanced = category_df.with_columns([
    pl.col("product").cast(pl.Categorical).alias("product_cat"),
    pl.col("category").cast(pl.Categorical).alias("category_cat")
])

# Работа с категориями
category_ops = category_enhanced.with_columns([
    # Получение кодов категорий
    pl.col("product_cat").to_physical().alias("product_code"),
    
    # Получение категорий по кодам
    pl.col("product_code").cast(pl.Categorical).alias("product_from_code")
])

# One-hot encoding
one_hot = category_enhanced.to_dummies(columns=["product_cat"])

# Frequency encoding
frequency_encoding = (category_enhanced
    .group_by("product_cat")
    .agg(pl.count().alias("frequency"))
    .with_columns([
        (pl.col("frequency") / pl.col("frequency").sum()).alias("frequency_ratio")
    ])
)
```

## Основы Plotly Express

### Базовые настройки и создание графиков

```python
# Создание тестовых данных для визуализации
np.random.seed(42)
plot_data = pl.DataFrame({
    "x": np.random.randn(100),
    "y": np.random.randn(100),
    "category": np.random.choice(["A", "B", "C"], 100),
    "size": np.random.uniform(10, 100, 100),
    "value": np.random.uniform(0, 1, 100)
})

# Базовый точечный график
fig = px.scatter(plot_data.to_pandas(), x="x", y="y")
fig.show()

# График с настройками
fig = px.scatter(
    plot_data.to_pandas(),
    x="x",
    y="y",
    color="category",
    size="size",
    hover_data=["value"],
    title="Пример точечного графика",
    labels={"x": "X ось", "y": "Y ось"},
    template="plotly_white"
)
fig.show()
```

### Настройка макета и стилей

```python
# Расширенная настройка макета
fig.update_layout(
    title={
        'text': "Детальный заголовок",
        'x': 0.5,
        'xanchor': 'center',
        'font': {'size': 20}
    },
    xaxis_title="Название X оси",
    yaxis_title="Название Y оси",
    font=dict(family="Arial", size=12, color="black"),
    legend=dict(
        title="Легенда",
        x=1,
        y=1,
        bgcolor="rgba(255,255,255,0.8)"
    ),
    margin=dict(l=50, r=50, t=50, b=50),
    plot_bgcolor="white",
    paper_bgcolor="white"
)

# Настройка осей
fig.update_xaxes(
    showgrid=True,
    gridwidth=1,
    gridcolor="lightgray",
    zeroline=True,
    zerolinewidth=2,
    zerolinecolor="black"
)

fig.update_yaxes(
    showgrid=True,
    gridwidth=1,
    gridcolor="lightgray"
)

# Настройка маркеров
fig.update_traces(
    marker=dict(
        line=dict(width=1, color='black')
    ),
    selector=dict(mode='markers')
)
```

## Типы графиков Plotly Express

### Основные типы графиков

```python
# 1. Линейные графики
line_data = pl.DataFrame({
    "date": pl.date_range(date(2023, 1, 1), date(2023, 12, 31), "1d"),
    "value_a": np.random.randn(365).cumsum() + 100,
    "value_b": np.random.randn(365).cumsum() + 150
})

fig_line = px.line(
    line_data.to_pandas(),
    x="date",
    y=["value_a", "value_b"],
    title="Линейные графики"
)

# 2. Столбчатые диаграммы
bar_data = pl.DataFrame({
    "category": ["A", "B", "C", "D", "E"],
    "value1": [10, 20, 15, 25, 30],
    "value2": [5, 15, 10, 20, 25]
})

fig_bar = px.bar(
    bar_data.to_pandas(),
    x="category",
    y="value1",
    title="Столбчатая диаграмма"
)

# 3. Гистограммы
hist_data = pl.DataFrame({
    "values": np.random.randn(1000)
})

fig_hist = px.histogram(
    hist_data.to_pandas(),
    x="values",
    nbins=30,
    title="Гистограмма"
)

# 4. Круговые диаграммы
pie_data = pl.DataFrame({
    "category": ["A", "B", "C", "D"],
    "values": [25, 35, 20, 20]
})

fig_pie = px.pie(
    pie_data.to_pandas(),
    names="category",
    values="values",
    title="Круговая диаграмма"
)
```

### Специализированные графики

```python
# 5. Box plots
box_data = pl.DataFrame({
    "category": np.random.choice(["Group1", "Group2", "Group3"], 300),
    "value": np.concatenate([
        np.random.normal(100, 15, 100),
        np.random.normal(120, 20, 100),
        np.random.normal(90, 10, 100)
    ])
})

fig_box = px.box(
    box_data.to_pandas(),
    x="category",
    y="value",
    title="Box Plot"
)

# 6. Violin plots
fig_violin = px.violin(
    box_data.to_pandas(),
    x="category",
    y="value",
    title="Violin Plot"
)

# 7. Heatmaps
heatmap_data = pl.DataFrame({
    "x": np.repeat(["A", "B", "C", "D"], 4),
    "y": np.tile(["W", "X", "Y", "Z"], 4),
    "value": np.random.rand(16)
})

fig_heatmap = px.density_heatmap(
    heatmap_data.to_pandas(),
    x="x",
    y="y",
    z="value",
    title="Heatmap"
)

# 8. 3D графики
scatter3d_data = pl.DataFrame({
    "x": np.random.randn(100),
    "y": np.random.randn(100),
    "z": np.random.randn(100),
    "color": np.random.choice(["A", "B", "C"], 100),
    "size": np.random.uniform(5, 20, 100)
})

fig_3d = px.scatter_3d(
    scatter3d_data.to_pandas(),
    x="x",
    y="y",
    z="z",
    color="color",
    size="size",
    title="3D Scatter Plot"
)
```

## Настройка визуализаций

### Кастомизация цветов и стилей

```python
# Кастомные цветовые схемы
color_discrete_map = {
    'A': '#FF6B6B',
    'B': '#4ECDC4', 
    'C': '#45B7D1',
    'D': '#96CEB4',
    'E': '#FFEAA7'
}

color_continuous_scale = [
    [0, '#FF6B6B'],
    [0.5, '#4ECDC4'],
    [1, '#45B7D1']
]

# График с кастомными цветами
fig_custom = px.scatter(
    plot_data.to_pandas(),
    x="x",
    y="y",
    color="category",
    color_discrete_map=color_discrete_map,
    title="График с кастомными цветами"
)

# Настройка шаблонов
templates = [
    "plotly", 
    "plotly_white", 
    "plotly_dark", 
    "ggplot2", 
    "seaborn", 
    "simple_white",
    "none"
]

for template in templates:
    fig = px.scatter(plot_data.to_pandas(), x="x", y="y", template=template)
    fig.update_layout(title=f"Шаблон: {template}")
    fig.show()
```

### Интерактивные элементы

```python
# Добавление кнопок и элементов управления
fig_interactive = px.scatter(
    plot_data.to_pandas(),
    x="x",
    y="y",
    color="category",
    size="size",
    hover_data=["value"],
    title="Интерактивный график"
)

# Добавление кнопок
fig_interactive.update_layout(
    updatemenus=[
        dict(
            type="dropdown",
            direction="down",
            x=0.1,
            y=1.15,
            buttons=list([
                dict(
                    args=["type", "scatter"],
                    label="Scatter Plot",
                    method="restyle"
                ),
                dict(
                    args=["type", "histogram2d"],
                    label="2D Histogram",
                    method="restyle"
                )
            ])
        )
    ]
)

# Настройка hover информации
fig_interactive.update_traces(
    hovertemplate="<br>".join([
        "X: %{x}",
        "Y: %{y}",
        "Категория: %{marker.color}",
        "Размер: %{marker.size}",
        "<extra></extra>"
    ])
)
```

## Интеграция Polars + Plotly Express

### Утилиты для конвертации

```python
def polars_to_plotly(df: pl.DataFrame, sample_size: int = None, strategy: str = 'all') -> pl.DataFrame:
    """
    Конвертация Polars DataFrame для использования с Plotly
    
    Parameters:
    -----------
    df : pl.DataFrame
        Исходный DataFrame
    sample_size : int, optional
        Размер выборки (для больших данных)
    strategy : str
        Стратегия обработки больших данных:
        - 'all': все данные
        - 'sample': случайная выборка
        - 'aggregate': агрегация
        
    Returns:
    --------
    pl.DataFrame
        Подготовленный DataFrame
    """
    
    if sample_size and len(df) > sample_size:
        if strategy == 'sample':
            df = df.sample(sample_size)
        elif strategy == 'aggregate':
            # Пример агрегации - нужно адаптировать под конкретный случай
            numeric_cols = [col for col in df.columns if df[col].dtype in [pl.Int64, pl.Float64]]
            categorical_cols = [col for col in df.columns if df[col].dtype == pl.Utf8]
            
            if categorical_cols:
                df = df.group_by(categorical_cols[0]).agg([
                    pl.col(col).mean() for col in numeric_cols
                ])
    
    return df

def create_visualization(df: pl.DataFrame, viz_type: str, **kwargs):
    """
    Создание визуализации из Polars DataFrame
    
    Parameters:
    -----------
    df : pl.DataFrame
        Исходные данные
    viz_type : str
        Тип визуализации
    **kwargs : dict
        Дополнительные параметры для Plotly Express
        
    Returns:
    --------
    plotly.graph_objects.Figure
        Созданная фигура
    """
    
    # Конвертация в pandas
    plot_df = df.to_pandas()
    
    # Создание графика в зависимости от типа
    if viz_type == 'scatter':
        return px.scatter(plot_df, **kwargs)
    elif viz_type == 'line':
        return px.line(plot_df, **kwargs)
    elif viz_type == 'bar':
        return px.bar(plot_df, **kwargs)
    elif viz_type == 'histogram':
        return px.histogram(plot_df, **kwargs)
    elif viz_type == 'box':
        return px.box(plot_df, **kwargs)
    else:
        raise ValueError(f"Неизвестный тип визуализации: {viz_type}")
```

### Автоматизация анализа

```python
class PolarsPlotlyAnalyzer:
    """Класс для автоматизации анализа данных с Polars и Plotly"""
    
    def __init__(self, df: pl.DataFrame):
        self.df = df
        self.numeric_cols = self._get_numeric_cols()
        self.categorical_cols = self._get_categorical_cols()
    
    def _get_numeric_cols(self) -> List[str]:
        """Получить список числовых колонок"""
        return [col for col in self.df.columns 
                if self.df[col].dtype in [pl.Int8, pl.Int16, pl.Int32, pl.Int64, 
                                         pl.Float32, pl.Float64]]
    
    def _get_categorical_cols(self) -> List[str]:
        """Получить список категориальных колонок"""
        return [col for col in self.df.columns 
                if self.df[col].dtype in [pl.Utf8, pl.Categorical]]
    
    def create_correlation_heatmap(self) -> go.Figure:
        """Создать heatmap корреляций"""
        if len(self.numeric_cols) < 2:
            raise ValueError("Нужно как минимум 2 числовые колонки")
        
        corr_df = self.df.select(self.numeric_cols).to_pandas().corr()
        
        fig = px.imshow(
            corr_df,
            text_auto=True,
            aspect="auto",
            title="Матрица корреляций",
            color_continuous_scale="RdBu_r"
        )
        return fig
    
    def create_distribution_plots(self, col: str) -> go.Figure:
        """Создать графики распределения для колонки"""
        if col not in self.numeric_cols:
            raise ValueError(f"Колонка {col} не числовая")
        
        fig = make_subplots(
            rows=1, cols=2,
            subplot_titles=(f'Гистограмма {col}', f'Box Plot {col}')
        )
        
        # Гистограмма
        hist_data = self.df[col].to_pandas()
        fig.add_trace(go.Histogram(x=hist_data, name="Гистограмма"), row=1, col=1)
        
        # Box plot
        fig.add_trace(go.Box(y=hist_data, name="Box Plot"), row=1, col=2)
        
        fig.update_layout(title_text=f"Распределение {col}")
        return fig
    
    def create_summary_dashboard(self) -> Dict[str, go.Figure]:
        """Создать набор графиков для общего анализа"""
        dashboard = {}
        
        # Heatmap корреляций
        if len(self.numeric_cols) >= 2:
            dashboard['correlation'] = self.create_correlation_heatmap()
        
        # Распределения для числовых колонок
        for col in self.numeric_cols[:3]:  # Ограничим первыми тремя
            dashboard[f'distribution_{col}'] = self.create_distribution_plots(col)
        
        return dashboard

# Использование класса
analyzer = PolarsPlotlyAnalyzer(df)
dashboard = analyzer.create_summary_dashboard()
for name, fig in dashboard.items():
    fig.show()
```

## Практические примеры анализа

### Анализ продаж

```python
def create_sales_analysis():
    """Создание комплексного анализа продаж"""
    
    # Генерация данных о продажах
    np.random.seed(42)
    dates = pl.date_range(date(2023, 1, 1), date(2023, 12, 31), "1d")
    
    sales_data = pl.DataFrame({
        "date": dates,
        "product": np.random.choice(["Laptop", "Phone", "Tablet", "Monitor"], len(dates)),
        "category": np.random.choice(["Electronics", "Computers", "Mobile"], len(dates)),
        "region": np.random.choice(["North", "South", "East", "West"], len(dates)),
        "sales": np.random.randint(100, 5000, len(dates)),
        "price": np.random.uniform(100, 2000, len(dates)),
        "quantity": np.random.randint(1, 10, len(dates))
    })
    
    # Добавление дополнительных колонок
    sales_enhanced = sales_data.with_columns([
        (pl.col("sales") * pl.col("quantity")).alias("revenue"),
        pl.col("date").dt.month().alias("month"),
        pl.col("date").dt.quarter().alias("quarter"),
        pl.col("date").dt.weekday().alias("weekday")
    ])
    
    return sales_enhanced

# Создание данных
sales_df = create_sales_analysis()

# 1. Агрегация по месяцам
monthly_sales = sales_df.group_by("month").agg([
    pl.col("revenue").sum().alias("total_revenue"),
    pl.col("sales").mean().alias("avg_sales"),
    pl.col("quantity").sum().alias("total_quantity")
]).sort("month")

fig_monthly = px.line(
    monthly_sales.to_pandas(),
    x="month",
    y="total_revenue",
    title="Выручка по месяцам",
    markers=True
)

# 2. Продажи по регионам
regional_sales = sales_df.group_by("region").agg([
    pl.col("revenue").sum().alias("total_revenue"),
    pl.col("quantity").sum().alias("total_quantity")
]).sort("total_revenue", descending=True)

fig_regional = px.bar(
    regional_sales.to_pandas(),
    x="region",
    y="total_revenue",
    title="Выручка по регионам",
    color="total_revenue"
)

# 3. Топ продукты
product_sales = sales_df.group_by("product").agg([
    pl.col("revenue").sum().alias("total_revenue"),
    pl.col("quantity").sum().alias("total_quantity")
]).sort("total_revenue", descending=True)

fig_products = px.pie(
    product_sales.to_pandas(),
    names="product",
    values="total_revenue",
    title="Распределение выручки по продуктам"
)
```

### Анализ временных рядов

```python
def analyze_time_series(df: pl.DataFrame, date_col: str, value_col: str):
    """Комплексный анализ временного ряда"""
    
    results = {}
    
    # Базовые статистики
    basic_stats = df.select([
        pl.col(value_col).mean().alias("mean"),
        pl.col(value_col).std().alias("std"),
        pl.col(value_col).min().alias("min"),
        pl.col(value_col).max().alias("max")
    ])
    
    # Тренд и сезонность
    df_sorted = df.sort(date_col)
    trend = df_sorted.with_columns([
        pl.col(value_col).rolling_mean(window_size=30).alias("trend")
    ])
    
    # Сезонная декомпозиция
    seasonal = trend.with_columns([
        (pl.col(value_col) - pl.col("trend")).alias("seasonal")
    ])
    
    # Визуализация
    fig_trend = px.line(
        trend.to_pandas(),
        x=date_col,
        y=[value_col, "trend"],
        title="Временной ряд с трендом"
    )
    
    fig_seasonal = px.line(
        seasonal.to_pandas(),
        x=date_col,
        y="seasonal",
        title="Сезонная компонента"
    )
    
    results['basic_stats'] = basic_stats
    results['trend_plot'] = fig_trend
    results['seasonal_plot'] = fig_seasonal
    
    return results

# Применение анализа
ts_analysis = analyze_time_series(sales_df, "date", "revenue")
```

## Создание дашбордов

### Простой дашборд

```python
def create_simple_dashboard(df: pl.DataFrame):
    """Создание простого дашборда с основными метриками"""
    
    # Вычисление ключевых метрик
    total_revenue = df["revenue"].sum()
    avg_sales = df["sales"].mean()
    total_products = df["product"].n_unique()
    
    # Создание подграфиков
    fig = make_subplots(
        rows=2, cols=2,
        subplot_titles=(
            'Выручка по месяцам', 
            'Продажи по регионам',
            'Распределение по продуктам',
            'Тренд продаж'
        ),
        specs=[
            [{"type": "scatter"}, {"type": "bar"}],
            [{"type": "pie"}, {"type": "scatter"}]
        ]
    )
    
    # 1. Выручка по месяцам
    monthly_data = df.group_by("month").agg(pl.col("revenue").sum()).sort("month")
    fig.add_trace(
        go.Scatter(
            x=monthly_data["month"], 
            y=monthly_data["revenue"],
            name="Выручка"
        ),
        row=1, col=1
    )
    
    # 2. Продажи по регионам
    regional_data = df.group_by("region").agg(pl.col("revenue").sum())
    fig.add_trace(
        go.Bar(
            x=regional_data["region"],
            y=regional_data["revenue"],
            name="По регионам"
        ),
        row=1, col=2
    )
    
    # 3. Распределение по продуктам
    product_data = df.group_by("product").agg(pl.col("revenue").sum())
    fig.add_trace(
        go.Pie(
            labels=product_data["product"],
            values=product_data["revenue"],
            name="Продукты"
        ),
        row=2, col=1
    )
    
    # 4. Тренд продаж
    daily_trend = df.group_by("date").agg(pl.col("revenue").sum()).sort("date")
    fig.add_trace(
        go.Scatter(
            x=daily_trend["date"],
            y=daily_trend["revenue"],
            name="Тренд"
        ),
        row=2, col=2
    )
    
    fig.update_layout(
        height=800,
        title_text=f"Дашборд продаж | Общая выручка: {total_revenue:,.0f}",
        showlegend=False
    )
    
    return fig

# Создание дашборда
dashboard_fig = create_simple_dashboard(sales_df)
dashboard_fig.show()
```

### Интерактивный дашборд

```python
def create_interactive_dashboard(df: pl.DataFrame):
    """Создание интерактивного дашборда с фильтрами"""
    
    # Создание нескольких графиков
    figures = {}
    
    # 1. График с фильтрацией
    fig_filtered = px.scatter(
        df.to_pandas(),
        x="sales",
        y="revenue",
        color="region",
        size="quantity",
        hover_data=["product", "date"],
        title="Продажи vs Выручка"
    )
    
    # Добавление фильтров
    fig_filtered.update_layout(
        updatemenus=[
            dict(
                type="dropdown",
                direction="down",
                x=0.1,
                y=1.15,
                buttons=[
                    dict(
                        args=[{"color": [df["region"].to_pandas()]}],
                        label="По регионам",
                        method="restyle"
                    ),
                    dict(
                        args=[{"color": [df["product"].to_pandas()]}],
                        label="По продуктам",
                        method="restyle"
                    )
                ]
            )
        ]
    )
    
    figures['main'] = fig_filtered
    
    # 2. Heatmap корреляций
    numeric_cols = ['sales', 'revenue', 'quantity', 'price']
    corr_matrix = df.select(numeric_cols).to_pandas().corr()
    
    fig_heatmap = px.imshow(
        corr_matrix,
        text_auto=True,
        title="Корреляция показателей"
    )
    figures['correlation'] = fig_heatmap
    
    return figures

# Создание интерактивного дашборда
interactive_figures = create_interactive_dashboard(sales_df)
for name, fig in interactive_figures.items():
    fig.show()
```

## Оптимизация производительности

### Работа с большими данными

```python
def optimize_large_data_processing(df: pl.DataFrame):
    """Оптимизация обработки больших данных"""
    
    # 1. Использование LazyFrame для отложенных вычислений
    lazy_df = df.lazy()
    
    # Цепочка отложенных операций
    optimized_result = (lazy_df
        .filter(pl.col("revenue") > 1000)
        .group_by("region", "product")
        .agg([
            pl.col("revenue").sum().alias("total_revenue"),
            pl.col("quantity").mean().alias("avg_quantity")
        ])
        .sort("total_revenue", descending=True)
        .collect()  # Выполнение вычислений
    )
    
    # 2. Оптимизация типов данных
    optimized_types = df.with_columns([
        pl.col("region").cast(pl.Categorical),
        pl.col("product").cast(pl.Categorical),
        pl.col("category").cast(pl.Categorical)
    ])
    
    # 3. Использование эффективных агрегаций
    efficient_agg = df.group_by("region").agg([
        pl.col("revenue").sum().alias("total_revenue"),
        pl.col("sales").mean().alias("avg_sales"),
        pl.col("quantity").count().alias("transaction_count")
    ])
    
    return {
        'lazy_result': optimized_result,
        'optimized_types': optimized_types,
        'efficient_agg': efficient_agg
    }

# Стратегии для визуализации больших данных
def visualize_large_data(df: pl.DataFrame, strategy: str = 'aggregate'):
    """Визуализация больших данных с различными стратегиями"""
    
    if strategy == 'aggregate':
        # Агрегация перед визуализацией
        plot_df = df.group_by("date").agg([
            pl.col("revenue").sum().alias("daily_revenue")
        ]).to_pandas()
        
    elif strategy == 'sample':
        # Случайная выборка
        plot_df = df.sample(min(10000, len(df))).to_pandas()
        
    elif strategy == 'bin':
        # Биннинг для непрерывных данных
        plot_df = df.with_columns([
            (pl.col("revenue") // 1000 * 1000).alias("revenue_bin")
        ]).group_by("revenue_bin").agg([
            pl.count().alias("frequency")
        ]).to_pandas()
    
    return plot_df
```

### Мониторинг производительности

```python
import time
from contextlib import contextmanager

@contextmanager
def timer(operation_name: str):
    """Контекстный менеджер для измерения времени выполнения"""
    start_time = time.time()
    try:
        yield
    finally:
        end_time = time.time()
        print(f"{operation_name} заняло {end_time - start_time:.2f} секунд")

# Пример использования
with timer("Обработка данных"):
    result = optimize_large_data_processing(sales_df)

# Профилирование операций
def profile_operations(df: pl.DataFrame):
    """Профилирование различных операций"""
    
    operations = {
        "Фильтрация": lambda: df.filter(pl.col("revenue") > 1000),
        "Группировка": lambda: df.group_by("region").agg(pl.col("revenue").sum()),
        "Сортировка": lambda: df.sort("revenue", descending=True),
        "Агрегация": lambda: df.select(pl.col("revenue").sum())
    }
    
    for op_name, op_func in operations.items():
        with timer(op_name):
            result = op_func()
    
    return "Профилирование завершено"

# Запуск профилирования
profile_results = profile_operations(sales_df)
```

## Полезные функции и утилиты

### Утилиты для анализа данных

```python
def data_quality_report(df: pl.DataFrame) -> pl.DataFrame:
    """Создание отчета о качестве данных"""
    
    report_data = []
    
    for col in df.columns:
        col_data = {
            'column': col,
            'dtype': str(df[col].dtype),
            'total_count': df[col].len(),
            'null_count': df[col].null_count(),
            'null_percentage': (df[col].null_count() / df[col].len()) * 100,
            'unique_count': df[col].n_unique()
        }
        
        # Для числовых колонок добавляем статистики
        if df[col].dtype in [pl.Int64, pl.Float64]:
            col_data.update({
                'mean': df[col].mean(),
                'std': df[col].std(),
                'min': df[col].min(),
                'max': df[col].max()
            })
        
        report_data.append(col_data)
    
    return pl.DataFrame(report_data)

def detect_outliers(df: pl.DataFrame, column: str, method: str = 'iqr') -> pl.DataFrame:
    """Обнаружение выбросов в данных"""
    
    if method == 'iqr':
        # Метод межквартильного размаха
        q1 = df[column].quantile(0.25)
        q3 = df[column].quantile(0.75)
        iqr = q3 - q1
        lower_bound = q1 - 1.5 * iqr
        upper_bound = q3 + 1.5 * iqr
        
        outliers = df.filter(
            (pl.col(column) < lower_bound) | (pl.col(column) > upper_bound)
        )
    
    elif method == 'zscore':
        # Метод Z-score
        mean_val = df[column].mean()
        std_val = df[column].std()
        outliers = df.filter(
            pl.col(column).abs() > 3  # Z-score > 3
        )
    
    return outliers

# Создание отчета о качестве данных
quality_report = data_quality_report(sales_df)
print("Отчет о качестве данных:")
print(quality_report)

# Обнаружение выбросов
outliers = detect_outliers(sales_df, "revenue")
print(f"Найдено {len(outliers)} выбросов в выручке")
```

### Утилиты для визуализации

```python
def save_plotly_figure(fig: go.Figure, filename: str, formats: List[str] = ['html', 'png']):
    """Сохранение графиков в различных форматах"""
    
    for fmt in formats:
        if fmt == 'html':
            fig.write_html(f"{filename}.html")
        elif fmt == 'png':
            fig.write_image(f"{filename}.png", scale=2)
        elif fmt == 'pdf':
            fig.write_image(f"{filename}.pdf")
        elif fmt == 'json':
            fig.write_json(f"{filename}.json")
    
    print(f"График сохранен в форматах: {formats}")

def create_color_palette(n_colors: int, palette_name: str = 'viridis') -> List[str]:
    """Создание цветовой палитры"""
    
    import plotly.express as px
    
    if n_colors <= 10:
        # Использование встроенных палитр для небольшого количества цветов
        if palette_name == 'qualitative':
            return px.colors.qualitative.Set3[:n_colors]
        else:
            return px.colors.sequential.Viridis[:n_colors]
    else:
        # Генерация палитры для большого количества цветов
        return px.colors.sample_colorscale(
            px.colors.sequential.Viridis,
            np.linspace(0, 1, n_colors)
        )

# Функция для создания аннотаций
def add_annotations(fig: go.Figure, annotations: List[Dict[str, Any]]):
    """Добавление аннотаций на график"""
    
    for annotation in annotations:
        fig.add_annotation(**annotation)
    
    return fig

# Пример использования утилит
example_fig = px.scatter(sales_df.to_pandas(), x="sales", y="revenue")
save_plotly_figure(example_fig, "example_plot", ['html', 'png'])

# Создание кастомной цветовой палитры
custom_colors = create_color_palette(5, 'qualitative')
print("Кастомная палитра:", custom_colors)
```