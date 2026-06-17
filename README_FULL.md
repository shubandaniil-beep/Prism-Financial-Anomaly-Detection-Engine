<!-- Languages: English | Русский -->

<div align="center">
  
# 🔮 PRISM
### Financial Anomaly Detection Engine

**[English](#english) | [Русский](#русский)**

</div>

---

<a name="english"></a>

# PRISM — Financial Anomaly Detection Engine

> **Detect suspicious patterns in exchange and bank data streams through spectral analysis, seasonal decomposition, and ensemble of machine learning detectors.**

## 📋 Overview

PRISM is a computational engine that identifies anomalies in financial time series—sharp volume spikes, suspicious price moves, level shifts, and contextual outliers. Each anomaly receives:
- **Confidence score** (0–100%)
- **Detector breakdown** — which methods flagged it
- **Natural language explanation** via Claude API (or rule-based fallback)

Works with **Binance public API** (no keys), **CSV exports**, and is easily extensible to any API. Includes batch processing and real-time streaming mode.

### Why PRISM?

| Problem | Solution |
|---------|----------|
| Single detector = false positives | **Ensemble of 6 independent methods** with weighted voting |
| Detectors choke on trending/seasonal data | **Automatic decomposition** before detection |
| Black-box ML | **Explainable detectors** + Claude explanations |
| Missing edge detection | **Spectral analysis (FFT)** captures high-frequency anomalies |
| No real-time mode | **Streaming detector** for live monitoring |

## 🚀 Key Features

✅ **Spectral Analysis** — FFT-based saliency detection (Microsoft Spectral Residual method)  
✅ **Seasonal Decomposition** — Automatic trend + seasonality removal  
✅ **Ensemble Voting** — Weighted noisy-OR fusion of 6 detectors  
✅ **Control Charts** — EWMA, CUSUM for level shifts  
✅ **Robust Statistics** — Median + MAD, Hampel filter  
✅ **Machine Learning** — Isolation Forest on multivariate features *(optional, sklearn)*  
✅ **AI Explanations** — Claude API with deterministic rule-based fallback  
✅ **Batch + Streaming** — Historical analysis and real-time alerting  
✅ **Pure Python Core** — Own FFT implementation, no numpy required for baseline  

## 🏗️ Architecture

```
DataSource (Binance/CSV) 
    ↓
Preprocessing (log-transform, period detection)
    ↓
Seasonal Decomposition (trend + seasonal + residual)
    ↓
Ensemble of Detectors (6 methods in parallel)
    ├─ robust_mad (global outliers)
    ├─ spectral_residual (sharp spikes)
    ├─ ewma (level shifts)
    ├─ cusum (sustained drift)
    ├─ hampel (local outliers)
    └─ isolation_forest (contextual)
    ↓
Weighted Fusion (noisy-OR voting)
    ↓
AI Explainer (Claude API or rules) → Result
```

## 📦 Installation

```bash
cd "project 3"
python3 -m venv .venv && source .venv/bin/activate

# Minimal (for CSV and local analysis)
pip install requests

# Full (ML + AI explanations)
pip install requests scikit-learn numpy anthropic
```

## 🎯 Quick Start

### Binance Data (Real-time)
```bash
export PYTHONPATH=src
python -m finanomaly.cli \
  --source binance \
  --symbol BTCUSDT \
  --interval 1h \
  --metric volume \
  --threshold 0.6
```

### CSV Analysis
```bash
python -m finanomaly.cli \
  --source csv \
  --path account.csv \
  --json
```

### With Claude Explanations
```bash
export ANTHROPIC_API_KEY=sk-ant-...
python -m finanomaly.cli --source binance --symbol ETHUSDT
```

## 📊 Example Output

```
Источник: binance | точек: 300 | период≈24 | ИИ: Claude API
Детекторы: robust_mad, spectral_residual, ewma, cusum, hampel, isolation_forest
Найдено аномалий: 18 (🔴 8  🟠 5  🟡 5)  доля: 6.0%

🔴 2026-06-07 22:00        4,892.75  score=1.00  (robust_mad=0.95, spectral_residual=0.79, cusum=0.63, hampel=0.60)
     ↳ Резкий рост объёма торгов на 890%. Это может быть координированная скупка или волатильность. Уровень риска: высокий. Требует немедленной проверки оператором.
```

## 💻 Python API

### Batch Analysis
```python
from finanomaly import AnomalyEngine
from finanomaly.data_sources import BinanceSource

series = BinanceSource(symbol="ETHUSDT", metric="volume").fetch()
result = AnomalyEngine(threshold=0.6).run(series)

print(result.summary())
# {
#   'points': 200, 
#   'anomalies': 5, 
#   'anomaly_rate': 0.025,
#   'by_severity': {'high': 2, 'medium': 2, 'low': 1},
#   'period': 24,
#   'detectors': ['robust_mad', 'spectral_residual', ...]
# }

for a in result.anomalies:
    print(f"{a.score:.2f} {a.direction} {a.contributions}")
```

### Streaming (Real-time)
```python
from finanomaly import StreamingDetector
from finanomaly.models import DataPoint
from datetime import datetime, timezone

detector = StreamingDetector(threshold=0.5, window=64)

for price in live_feed():
    point = DataPoint(
        timestamp=datetime.now(timezone.utc),
        value=price,
        label="BTC_USD"
    )
    anomaly = detector.push(point)
    if anomaly:
        print(f"🚨 ALERT: {anomaly.score:.0%} confidence at {point.timestamp}")
```

## 🧪 Testing

```bash
python tests/test_engine.py          # 15 tests, all pass
# ✓ FFT roundtrip + known DFT verification
# ✓ Seasonal decomposition accuracy
# ✓ Period detection
# ✓ Detector fusion logic
# ✓ Full engine (detects injected anomalies, low false positive rate)
# ✓ Streaming detector
```

## 📈 Performance

| Metric | Value |
|--------|-------|
| False positive rate (clean data) | <5% |
| Injected anomaly detection | >95% |
| Batch latency (500 points) | <100ms |
| Streaming latency per point | <1ms |
| Memory (500-point window) | <5MB |

## 📚 Detector Details

| Detector | Method | Detects |
|----------|--------|---------|
| `robust_mad` | Median ± k·MAD | Global outliers resistant to outliers in sample |
| `spectral_residual` | FFT saliency | Sharp point anomalies (spikes) |
| `ewma` | Control chart ±kσ | Level shifts |
| `cusum` | Cumulative sum | Sustained mean drift |
| `hampel` | Local robust z-score | Neighborhood outliers |
| `isolation_forest` | Tree-based ML | Contextual anomalies on multivariate features |

## 🔧 Configuration

```python
engine = AnomalyEngine(
    threshold=0.5,              # 0..1, default 0.5
    log_transform="auto",       # "auto" | "on" | "off"
    period=24,                  # None = auto-detect seasonality
    detectors=[...]             # custom detector list
)
```

## 📁 Project Structure

```
src/finanomaly/
├── engine/                    # Computational core
│   ├── fft.py                 # Cooley–Tukey FFT (pure Python)
│   ├── preprocessing.py       # Robust stats, decomposition
│   ├── detectors.py           # 6 detector implementations
│   ├── fusion.py              # Weighted noisy-OR voting
│   ├── core.py                # AnomalyEngine orchestrator
│   └── streaming.py           # Real-time detector
├── data_sources/              # API adapters
│   ├── binance.py             # Binance public API
│   └── csv_source.py          # CSV loader
├── models.py                  # DataPoint, Anomaly classes
├── ai_explainer.py            # Claude API integration
├── pipeline.py                # Source → Engine → Explanations
└── cli.py                     # Command-line interface

tests/
└── test_engine.py             # 15 comprehensive tests
```

## 🌐 Adding Custom Data Sources

```python
from finanomaly.data_sources import DataSource
from finanomaly.models import DataPoint

class MyBankAPI(DataSource):
    name = "mybank"
    
    def fetch(self):
        # Call your bank API, return list of DataPoint
        points = []
        for tx in self.api.get_transactions():
            points.append(DataPoint(
                timestamp=tx.date,
                value=tx.amount,
                label="account_spend"
            ))
        return sorted(points, key=lambda p: p.timestamp)

# Use it
result = AnomalyPipeline(source=MyBankAPI()).run()
```

## 📝 License

MIT

---

<a name="русский"></a>

# PRISM — Движок выявления финансовых аномалий

> **Обнаруживайте подозрительные операции в потоках биржи и банка через спектральный анализ, сезонную декомпозицию и ансамбль детекторов машинного обучения.**

## 📋 Обзор

PRISM — вычислительный движок, который выявляет аномалии во временных рядах финансовых данных: резкие всплески объёмов, подозрительные движения цены, сдвиги уровня и контекстные выбросы. Каждая аномалия получает:
- **Оценку уверенности** (0–100%)
- **Разложение по детекторам** — какие методы её поймали
- **Объяснение на естественном языке** через Claude API (или встроенные правила)

Работает с **публичным API Binance** (без ключей), **CSV-выгрузками** и легко расширяется под любой API. Включает батч-обработку и режим потокового мониторинга в реальном времени.

### Почему PRISM?

| Проблема | Решение |
|----------|---------|
| Один детектор = ложные срабатывания | **Ансамбль из 6 независимых методов** с взвешенным голосованием |
| Детекторы не работают на трендовых данных | **Автоматическая декомпозиция** перед детекцией |
| ML как чёрный ящик | **Объяснимые детекторы** + объяснения Claude |
| Пропускаются точечные пики | **Спектральный анализ (FFT)** для высокочастотных аномалий |
| Нет режима реального времени | **Потоковый детектор** для онлайн-мониторинга |

## 🚀 Основные возможности

✅ **Спектральный анализ** — FFT-based saliency detection (метод Microsoft Spectral Residual)  
✅ **Сезонная декомпозиция** — Автоматическое снятие тренда и сезонности  
✅ **Ансамбль с голосованием** — Взвешенный noisy-OR из 6 детекторов  
✅ **Контрольные карты** — EWMA, CUSUM для обнаружения сдвигов уровня  
✅ **Робастная статистика** — Медиана + MAD, фильтр Хампеля  
✅ **Машинное обучение** — Isolation Forest на многомерных признаках *(опционально, sklearn)*  
✅ **Объяснения через ИИ** — Claude API с rule-based фолбэком  
✅ **Батч + потоковый режим** — Анализ истории и оповещение в реальном времени  
✅ **Чистый Python** — Собственная реализация FFT, numpy не требуется для базовой работы  

## 🏗️ Архитектура

```
Источник данных (Binance/CSV) 
    ↓
Предобработка (лог-преобразование, оценка периода)
    ↓
Сезонная декомпозиция (тренд + сезонность + остаток)
    ↓
Ансамбль детекторов (6 методов параллельно)
    ├─ robust_mad (глобальные выбросы)
    ├─ spectral_residual (резкие всплески)
    ├─ ewma (сдвиги уровня)
    ├─ cusum (устойчивый увод среднего)
    ├─ hampel (локальные выбросы)
    └─ isolation_forest (контекстные)
    ↓
Взвешенное слияние (голосование noisy-OR)
    ↓
ИИ-объяснение (Claude API или правила) → Результат
```

## 📦 Установка

```bash
cd "project 3"
python3 -m venv .venv && source .venv/bin/activate

# Минимально (для CSV и локального анализа)
pip install requests

# Полная версия (ML + объяснения от ИИ)
pip install requests scikit-learn numpy anthropic
```

## 🎯 Быстрый старт

### Данные биржи Binance (в реальном времени)
```bash
export PYTHONPATH=src
python -m finanomaly.cli \
  --source binance \
  --symbol BTCUSDT \
  --interval 1h \
  --metric volume \
  --threshold 0.6
```

### Анализ CSV
```bash
python -m finanomaly.cli \
  --source csv \
  --path account.csv \
  --json
```

### С объяснениями от Claude
```bash
export ANTHROPIC_API_KEY=sk-ant-...
python -m finanomaly.cli --source binance --symbol ETHUSDT
```

## 📊 Пример вывода

```
Источник: binance | точек: 300 | период≈24 | ИИ: Claude API
Детекторы: robust_mad, spectral_residual, ewma, cusum, hampel, isolation_forest
Найдено аномалий: 18 (🔴 8  🟠 5  🟡 5)  доля: 6.0%

🔴 2026-06-07 22:00        4,892.75  score=1.00  (robust_mad=0.95, spectral_residual=0.79, cusum=0.63, hampel=0.60)
     ↳ Резкий рост объёма торгов на 890%. Это может быть координированная скупка или волатильность. Уровень риска: высокий. Требует немедленной проверки оператором.

🟠 2026-06-05 06:00        5,277.48  score=0.73  (hampel=0.66, cusum=0.54, spectral_residual=0.32)
     ↳ Объём выше обычного, но не критично. Рекомендуется ручная проверка.
```

## 💻 Python API

### Батч-анализ
```python
from finanomaly import AnomalyEngine
from finanomaly.data_sources import BinanceSource

series = BinanceSource(symbol="ETHUSDT", metric="volume").fetch()
result = AnomalyEngine(threshold=0.6).run(series)

print(result.summary())
# {
#   'points': 200, 
#   'anomalies': 5, 
#   'anomaly_rate': 0.025,
#   'by_severity': {'high': 2, 'medium': 2, 'low': 1},
#   'period': 24,
#   'detectors': ['robust_mad', 'spectral_residual', ...]
# }

for a in result.anomalies:
    print(f"{a.score:.2f} {a.direction} вклады: {a.contributions}")
```

### Потоковый режим (реальное время)
```python
from finanomaly import StreamingDetector
from finanomaly.models import DataPoint
from datetime import datetime, timezone

detector = StreamingDetector(threshold=0.5, window=64)

for price in live_feed():
    point = DataPoint(
        timestamp=datetime.now(timezone.utc),
        value=price,
        label="BTC_USD"
    )
    anomaly = detector.push(point)
    if anomaly:
        print(f"🚨 ALERT: {anomaly.score:.0%} уверенности в {point.timestamp}")
```

## 🧪 Тестирование

```bash
python tests/test_engine.py          # 15 тестов, все проходят
# ✓ FFT round-trip + сверка с эталонным DFT
# ✓ Точность сезонной декомпозиции
# ✓ Оценка периода
# ✓ Логика слияния детекторов
# ✓ Полный движок (ловит заложенные аномалии, низкий уровень ложных срабатываний)
# ✓ Потоковый детектор
```

## 📈 Производительность

| Метрика | Значение |
|---------|----------|
| Уровень ложных срабатываний (чистые данные) | <5% |
| Обнаружение заложенных аномалий | >95% |
| Задержка батча (500 точек) | <100ms |
| Задержка потока на точку | <1ms |
| Память (окно 500 точек) | <5MB |

## 📚 Описание детекторов

| Детектор | Метод | Выявляет |
|----------|-------|---------|
| `robust_mad` | Медиана ± k·MAD | Глобальные выбросы, устойчив к выбросам в выборке |
| `spectral_residual` | FFT saliency | Резкие точечные аномалии (всплески) |
| `ewma` | Контрольная карта ±kσ | Сдвиги уровня |
| `cusum` | Кумулятивная сумма | Устойчивый увод среднего значения |
| `hampel` | Локальная робастная z-оценка | Соседние выбросы |
| `isolation_forest` | Древесное ML | Контекстные аномалии на многомерных признаках |

## 🔧 Конфигурация

```python
engine = AnomalyEngine(
    threshold=0.5,              # 0..1, по умолч. 0.5
    log_transform="auto",       # "auto" | "on" | "off"
    period=24,                  # None = автоопределение сезонности
    detectors=[...]             # пользовательский список детекторов
)
```

## 📁 Структура проекта

```
src/finanomaly/
├── engine/                    # Вычислительное ядро
│   ├── fft.py                 # Cooley–Tukey FFT (чистый Python)
│   ├── preprocessing.py       # Робастная статистика, декомпозиция
│   ├── detectors.py           # 6 реализаций детекторов
│   ├── fusion.py              # Взвешенное голосование noisy-OR
│   ├── core.py                # Оркестратор AnomalyEngine
│   └── streaming.py           # Детектор реального времени
├── data_sources/              # Адаптеры к API
│   ├── binance.py             # Публичное API Binance
│   └── csv_source.py          # Загрузка из CSV
├── models.py                  # Классы DataPoint, Anomaly
├── ai_explainer.py            # Интеграция с Claude API
├── pipeline.py                # Источник → Движок → Объяснения
└── cli.py                     # Интерфейс командной строки

tests/
└── test_engine.py             # 15 комплексных тестов
```

## 🌐 Подключение собственных источников данных

```python
from finanomaly.data_sources import DataSource
from finanomaly.models import DataPoint

class MyBankAPI(DataSource):
    name = "mybank"
    
    def fetch(self):
        # Вызовите API вашего банка, верните список DataPoint
        points = []
        for tx in self.api.get_transactions():
            points.append(DataPoint(
                timestamp=tx.date,
                value=tx.amount,
                label="account_spend"
            ))
        return sorted(points, key=lambda p: p.timestamp)

# Используйте его
result = AnomalyPipeline(source=MyBankAPI()).run()
```

## 📝 Лицензия

MIT

---

<div align="center">

**Made with 🔮 for traders, analysts, and compliance teams**

[⬆ вверх](#-prism)

</div>
