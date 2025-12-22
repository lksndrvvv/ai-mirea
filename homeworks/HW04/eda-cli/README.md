# HW04 — eda-cli: HTTP-сервис качества датасетов

Проект HW04 является частичным продолжением HW03 и представляет собой HTTP-сервис
для оценки качества CSV-датасетов на основе эвристик качества данных.

Сервис реализован поверх проекта `eda-cli` и использует:
- EDA-ядро из HW03 (`summarize_dataset`, `missing_table`, `compute_quality_flags`);
- FastAPI для HTTP-слоя;
- uv и uvicorn для управления окружением и запуска сервиса.

Проект выполняется в рамках Семинара 04 курса «Инженерия ИИ».

## Требования

- Python 3.11+
- [uv](https://docs.astral.sh/uv/) установлен в систему

## Инициализация проекта

Из папки проекта `homeworks/HW04/eda-cli`:

```bash
uv sync
```

Эта команда:

- создаст виртуальное окружение `.venv`;
- установит зависимости из `pyproject.toml`;
- установит сам проект `eda-cli` в окружение.

## Запуск CLI

### Краткий обзор

```bash
uv run eda-cli overview data/example.csv
```

Параметры:

- `--sep` – разделитель (по умолчанию `,`);
- `--encoding` – кодировка (по умолчанию `utf-8`).

### Полный EDA-отчёт

```bash
uv run eda-cli report data/example.csv --out-dir reports
```

В результате в каталоге `reports/` появятся:

- `report.md` – основной отчёт в Markdown;
- `summary.csv` – таблица по колонкам;
- `missing.csv` – пропуски по колонкам;
- `correlation.csv` – корреляционная матрица (если есть числовые признаки);
- `top_categories/*.csv` – top-k категорий по строковым признакам;
- `hist_*.png` – гистограммы числовых колонок;
- `missing_matrix.png` – визуализация пропусков;
- `correlation_heatmap.png` – тепловая карта корреляций.

## Тесты

```bash
uv run pytest -q
```

## Дополнения по HW03 (добавлено мной)

В функции compute_quality_flags я добавила две дополнительные эвристики:

1. has_constant_columns — проверяет, есть ли колонки с одинаковыми значениями.
2. has_high_cardinality_categoricals — проверяет, есть ли категориальные признаки
   с очень большим числом уникальных значений.

В отчёт добавлены строки, которые выводят результаты этих проверок.

Также добавлены тесты, которые проверяют работу этих эвристик.



## HTTP-сервис (FastAPI)
### Запуск сервиса
Из папки homeworks/HW04/eda-cli:

```bash
uv run uvicorn eda_cli.api:app --reload --port 8000
```

После запуска доступна интерактивная документация Swagger UI:
http://127.0.0.1:8000/docs

### Реализованные эндпоинты
#### GET /health
Системный эндпоинт для проверки состояния сервиса.

#### POST /quality
Эндпоинт-заглушка, принимающий агрегированные признаки датасета
(n_rows, n_cols, max_missing_share и др.) и возвращающий
эвристическую оценку качества данных.

#### POST /quality-from-csv
Эндпоинт принимает CSV-файл, использует EDA-ядро из HW03
(summarize_dataset, missing_table, compute_quality_flags)
и возвращает:

- quality_score
- ok_for_model
- набор булевых флагов качества
- время обработки запроса (latency_ms)

Корректно обрабатывает ошибки чтения CSV и пустые файлы.

#### POST /quality-flags-from-csv
Дополнительный эндпоинт, реализованный в рамках HW04.

Эндпоинт:
- принимает CSV-файл;
- использует функции summarize_dataset, missing_table,
compute_quality_flags;
- возвращает полный набор флагов качества, вычисляемых EDA-ядром.

В ответе присутствуют эвристики, добавленные в HW03, включая:
- has_constant_columns
- has_high_cardinality_categoricals

Пример ответа:
{
  "flags": {
    "too_few_rows": true,
    "too_many_missing": false,
    "has_constant_columns": false,
    "has_high_cardinality_categoricals": true,
    "quality_score": 0.78
  }
}