---
title: AI Job Hunter
emoji: 🤖
colorFrom: blue
colorTo: purple
sdk: docker
app_port: 7860
pinned: false
license: mit
short_description: Автономный AI-агент для поиска вакансий на hh.ru и Habr Career
---

# AI Job Hunter Agent

> Автономный Python-агент для поиска AI-вакансий на **hh.ru и Habr Career**, анализа через LLM и адаптации резюме под каждую позицию.

**Автор:** Слава · [@ysiSevera](https://t.me/ysiSevera)  
**Стек:** Python 3.x · LangGraph · LangChain · OpenAI API · BeautifulSoup · Pydantic · fpdf2

![CI](https://github.com/slwvw1234-hue/ai_job_hunter/actions/workflows/ci.yml/badge.svg)

---

## Что умеет агент

### Автономный режим `/auto` (рекомендуется)

Одна команда — полный цикл без участия человека:

```
agent> /auto prompt engineer
```

Агент (LangGraph ReAct) самостоятельно:
1. Ищет вакансии на **hh.ru + Habr Career** по 4 связанным запросам (~200+ вакансий)
2. **Keyword pre-filter** без LLM — мгновенно отсеивает нерелевантное и senior-позиции
3. Анализирует топ-кандидатов через LLM (score 0–100, APPLY/MAYBE/SKIP)
4. Адаптирует резюме под лучшую вакансию
5. Генерирует сопроводительное письмо
6. Создаёт итоговый Markdown-отчёт
7. Экспортирует **PDF-пакет** (отчёт + резюме + письмо)

### Ручные команды CLI

| Команда | Что делает |
|---------|-----------|
| `/search` | Парсит hh.ru + Habr Career по запросам выбранной профессии |
| `/analyze [N]` | Анализирует N вакансий: score 0–100, APPLY / MAYBE / SKIP |
| `/adapt N` | Адаптирует резюме под конкретную вакансию |
| `/cover N [тон]` | Сопроводительное письмо (professional / friendly / concise) |
| `/resume N` | Экспортирует адаптированное резюме в Markdown и PDF |
| `/report` | Создаёт Markdown-отчёт по всем проанализированным вакансиям |
| `/open N` | Открывает вакансию в браузере |
| `/list [фильтр]` | Список вакансий: `apply`, `maybe`, `skip`, `top5`, `all` |

---

## Архитектура

```
ai_job_hunter/
├── agent.py                    # CLI-интерфейс, точка входа, все команды
├── config.py                   # Настройки, загрузка .env
├── run.bat                     # Запуск CLI (двойной клик на Windows)
├── memory/
│   └── base_resume.json        # Долгосрочная память — базовое резюме
├── modules/
│   ├── autonomous_agent.py     # LangGraph ReAct-агент (/auto)
│   ├── tools.py                # 7 LangChain @tool инструментов
│   ├── searcher.py             # Парсинг hh.ru + Habr Career, фильтрация
│   ├── analyzer.py             # LLM-анализ, keyword pre-filter, скоринг
│   ├── resume_adapter.py       # Адаптация резюме под вакансию
│   ├── cover_letter.py         # Генерация сопроводительных писем
│   └── exporter.py             # Экспорт в Markdown + PDF (3 типа)
├── tests/
│   ├── test_searcher.py        # 17 юнит-тестов (Vacancy, filters, keyword score)
│   └── test_evaluator.py
├── output/                     # Результаты работы агента (gitignored)
├── .env.example
├── pytest.ini
└── requirements.txt
```

### Как это работает (автономный режим)

```mermaid
flowchart TD
    U["/auto запрос"] --> A[LangGraph ReAct Agent]
    A --> B["search_vacancies\nhh.ru + Habr Career\n4 запроса · ~200+ вакансий"]
    B --> C["keyword pre-filter\nбез LLM · мгновенно\n200 → топ-60"]
    C --> D["analyze_vacancies\nLLM score + APPLY/MAYBE/SKIP\nтоп-50 из отфильтрованных"]
    D --> E[adapt_resume\nАдаптация резюме]
    E --> F[generate_cover_letter\nСопроводительное письмо]
    F --> G[export_report\nMarkdown отчёт]
    G --> H[export_all_pdf\nPDF пакет]
    H --> I[output/\nотчёт + резюме + письмо]

    style A fill:#1e66f5,color:#fff
    style C fill:#fe640b,color:#fff
    style H fill:#40a02b,color:#fff
    style I fill:#313244,color:#cdd6f4
```

---

## Быстрый старт

### 1. Клонируй репозиторий

```bash
git clone https://github.com/slwvw1234-hue/ai_job_hunter.git
cd ai_job_hunter
```

### 2. Создай виртуальное окружение

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# Linux / macOS
source .venv/bin/activate
```

### 3. Установи зависимости

```bash
pip install -r requirements.txt
```

### 4. Настрой `.env`

```bash
# Windows
copy .env.example .env

# Linux / macOS
cp .env.example .env
```

Открой `.env` и заполни:

```env
OPENAI_API_KEY=sk-...        # OpenAI ключ
LLM_MODEL=gpt-4o-mini        # Модель (gpt-4o-mini рекомендуется)
SEARCH_AREA=113              # 113 = вся Россия, 1 = Москва
RELEVANCE_THRESHOLD=45       # Минимальный score для отображения
```

Получить ключ OpenAI: [platform.openai.com/api-keys](https://platform.openai.com/api-keys)

> **Для пользователей из России:** если OpenAI недоступен напрямую, укажи прокси:
> ```env
> HTTPS_PROXY=http://127.0.0.1:10809
> ```

### 5. Заполни своё резюме

Открой `memory/base_resume.json` и замени данные на свои: имя, навыки, проекты, целевые роли.

### 6. Запусти агента

```bash
# Windows — двойной клик на run.bat

# CLI вручную:
python agent.py
```

---

## Пример сессии

```
agent> /auto prompt engineer

[AUTO AGENT] Запускаю автономный поиск: 'prompt engineer'
  [TOOL] search_vacancies → Найдено 211 вакансий (hh.ru: 130, Habr Career: 81)
  [TOOL] analyze_vacancies
    [PRE-FILTER] 211 → топ-60 по keyword-скорингу (без LLM)
    [LLM]        анализ топ-20 из отфильтрованных
    Прошли порог: 8 из 20
  [TOOL] adapt_resume → Резюме адаптировано под: LLM/Agent Engineer | ФордевиндAI
  [TOOL] generate_cover_letter → Письмо готово
  [TOOL] export_report → output/report_20260512_1945.md
  [TOOL] export_all_pdf → PDF-пакет готов:
      Отчёт:  output/report_20260512_1945.pdf
      Резюме: output/resume_Fordwind_20260512_1945.pdf
      Письмо: output/cover_Fordwind_20260512_1945.pdf

Топ-3 вакансии:
1. LLM/Agent Engineer | ФордевиндAI    | Score: 65 | MAYBE
2. AI / LLM Engineer (0→1 продукт)     | Score: 55 | MAYBE
3. Prompt Engineer (AI)                | Score: 45 | MAYBE
```

---

## Настройки

### `.env` переменные

| Переменная | Описание | По умолчанию |
|-----------|----------|-------------|
| `OPENAI_API_KEY` | OpenAI API ключ | — |
| `LLM_MODEL` | Модель OpenAI | `gpt-4o-mini` |
| `SEARCH_AREA` | Регион поиска (113 = РФ) | `113` |
| `RELEVANCE_THRESHOLD` | Минимальный score для отчёта | `45` |
| `HTTPS_PROXY` | Прокси если OpenAI недоступен | — |

---

## Тесты

```bash
# Запуск всех тестов
pytest

# Только быстрые unit-тесты (без сети)
pytest tests/test_searcher.py
```

17 юнит-тестов покрывают: модель `Vacancy`, парсинг зарплат с Habr, keyword pre-filter, конфигурацию.

---

## Зависимости

| Библиотека | Назначение |
|------------|-----------|
| `langchain` + `langgraph` | ReAct-агент, @tool инструменты |
| `langchain-openai` | ChatOpenAI интеграция |
| `requests` + `urllib3` | HTTP-запросы |
| `beautifulsoup4` + `lxml` | Парсинг hh.ru и Habr Career |
| `pydantic` | Валидация и структуры данных |
| `python-dotenv` | Загрузка .env |
| `fpdf2` | Генерация PDF |

---

## Roadmap

- [x] Парсинг hh.ru с фильтрацией Senior/мусора
- [x] Парсинг Habr Career
- [x] Анализ через LLM (OpenAI) — score 0–100, APPLY/MAYBE/SKIP
- [x] Keyword pre-filter — быстрый отбор без LLM
- [x] Адаптация резюме под вакансию
- [x] Генерация сопроводительных писем
- [x] Экспорт в PDF (резюме + письмо + отчёт)
- [x] **LangGraph ReAct-агент (`/auto`) — полный цикл одной командой**
- [x] **7 LangChain @tool инструментов**
- [x] **PDF-пакет: три документа автоматически**
- [x] 17 юнит-тестов

---

## Этот проект как портфолио

Демонстрирует:

- **LangGraph / LangChain** — ReAct-агент, @tool инструменты, `create_react_agent`
- **Prompt Engineering** — многоэтапные промпты, JSON-first, anti-hallucination, управление температурой
- **LLM API** — OpenAI-совместимая интеграция, `ChatOpenAI`, структурированный вывод через Pydantic
- **Web Scraping** — парсинг hh.ru и Habr Career, обход антиботовых мер, кэширование
- **Python** — модульная архитектура, Pydantic-валидация, PDF-генерация, юнит-тесты
- **Системное мышление** — двухэтапный пайплайн: keyword-фильтр → LLM → PDF-пакет

---

*Создан как портфолио-проект для поиска позиций Prompt Engineer / AI Engineer на российском рынке*
