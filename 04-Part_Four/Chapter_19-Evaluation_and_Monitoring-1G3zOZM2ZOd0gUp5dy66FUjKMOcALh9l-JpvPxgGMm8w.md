# Глава 19: Оценка и мониторинг

Эта глава рассматривает методологии, позволяющие интеллектуальным агентам систематически оценивать свою производительность, отслеживать прогресс к целям и выявлять операционные аномалии. Хотя Глава 11 описывает постановку целей и мониторинг, а Глава 17 — механизмы рассуждения, эта глава фокусируется на непрерывном, часто внешнем измерении эффективности агента, его соответствия требованиям.

![Мониторинг и оценка производительности агента](../assets/Monitoring_and_Evaluating_Agent_Performance.png)

Рис. 1: Best practices для оценки и мониторинга

## Практические применения и сценарии использования

* **Отслеживание производительности в продакшне:** Непрерывный мониторинг точности, латентности, потребления ресурсов развёрнутого агента.
* **A/B-тестирование:** Сравнение версий агента для определения оптимальных подходов.
* **Аудиты соответствия и безопасности:** Автоматические отчёты по соответствию этическим, регуляторным и safety-требованиям.
* **Enterprise-системы:** AI «Contract» — динамическое соглашение, кодифицирующее цели, правила и контроли для делегированных задач.
* **Детекция дрейфа:** Отслеживание снижения точности из-за изменений в распределении входных данных.
* **Детекция аномалий в поведении:** Выявление неожиданных действий агента.
* **Оценка прогресса обучения:** Кривая обучения, улучшение навыков, генерализация.

## Практический пример кода

**Оценка ответов агента:** Ключевой процесс для оценки качества и точности вывода. Метрики: фактическая корректность, беглость, соответствие цели.

```python
def evaluate_response_accuracy(agent_output: str, expected_output: str) -> float:
    return 1.0 if agent_output.strip().lower() == expected_output.strip().lower() else 0.0

score = evaluate_response_accuracy("The capital of France is Paris.", "Paris is the capital of France.")
print(f"Response accuracy: {score}")
```

Проблема: точное сравнение строк не улавливает семантическое сходство. Более эффективные метрики: сходство строк (Levenshtein), семантическое сходство (cosine similarity с эмбеддингами), LLM-as-a-Judge, RAG-метрики (faithfulness, relevance).

**Мониторинг латентности:** Измерение времени отклика. Для production — логирование в persistent storage (JSON, InfluxDB, Prometheus, Datadog).

**Отслеживание использования токенов:** Критично для управления стоимостью LLM.

```python
class LLMInteractionMonitor:
    def __init__(self):
        self.total_input_tokens = 0
        self.total_output_tokens = 0

    def record_interaction(self, prompt: str, response: str):
        input_tokens = len(prompt.split())
        output_tokens = len(response.split())
        self.total_input_tokens += input_tokens
        self.total_output_tokens += output_tokens
```

**LLM-as-a-Judge для «полезности»:** Оценка субъективных качеств через LLM-оценщика. Используя продвинутые лингвистические способности LLM, метод предлагает нюансированные оценки.

```python
import os, json, logging
import google.generativeai as genai

LEGAL_SURVEY_RUBRIC = """
You are an expert legal survey methodologist. Evaluate survey questions on:
1. Clarity & Precision (1-5)
2. Neutrality & Bias (1-5)
3. Relevance & Focus (1-5)
4. Completeness (1-5)
5. Appropriateness for Audience (1-5)
Output JSON: overall_score, rationale, detailed_feedback, concerns, recommended_action.
"""

class LLMJudgeForLegalSurvey:
    def __init__(self, model_name='gemini-1.5-flash-latest', temperature=0.2):
        self.model = genai.GenerativeModel(model_name)
        self.temperature = temperature

    def judge_survey_question(self, question: str):
        prompt = f"{LEGAL_SURVEY_RUBRIC}\n\nQuestion: {question}"
        response = self.model.generate_content(prompt, generation_config=genai.types.GenerationConfig(temperature=self.temperature, response_mime_type="application/json"))
        return json.loads(response.text)
```

## Траектории агентов

Оценка траекторий агентов критически важна. Стандартные тесты ПО недостаточны для вероятностных агентов. Нужна качественная оценка как финального вывода, так и последовательности шагов.

Методы сравнения: точное совпадение (exact match), совпадение по порядку (in-order), совпадение в любом порядке (any-order), precision, recall.

**Файлы оценки:**
* **Test files (JSON):** Одна сессия, несколько ходов. Идеальны для unit-тестирования.
* **Evalset files:** Множество сессий с длинными диалогами. Для интеграционных тестов.

**Многоагентные системы:** Проверка кооперации (передаёт ли Flight-Agent правильные данные Hotel-Agent?), планирования (соблюдает ли порядок?), выбора агента (используется ли правильный для задачи?), масштабируемости (улучшает ли добавление агентов?).

## От агентов к продвинутым контракторам

Предложена эволюция от простых AI-агентов к «контракторам» — детерминированным и подотчётным системам на основе четырёх столпов:

1. **Формализованный контракт:** Детальная спецификация deliverables, источников данных, области, ожидаемой стоимости и времени.
2. **Динамический жизненный цикл переговоров:** Контрактор анализирует условия, ведёт диалог, выявляет неоднозначности до начала выполнения.
3. **Итеративное выполнение с фокусом на качестве:** Self-validation и коррекция. Для генерации кода: несколько подходов, компиляция, unit-тесты, оценка.
4. **Иерархическая декомпозиция через субконтракты:** Первичный контрактор-менеджер разбивает сложную задачу на субконтракты для специализированных агентов.

![Пример выполнения контракта между агентами](../assets/Contract_Execution_Example_Among_Agents.png)

Рис. 2: Пример выполнения контракта между агентами

## Google ADK

Google ADK поддерживает три метода оценки: веб-UI (`adk web`), программная интеграция через pytest, CLI (`adk eval`).

![Поддержка оценки в Google ADK](../assets/Evaluation_Support_for_Google_ADK.png)

Рис. 3: Поддержка оценки в Google ADK

## Краткий обзор

**Что:** Агентные системы работают в динамичных средах. Традиционного тестирования ПО недостаточно. Нужны адаптивные методы оценки и метрики.

**Почему:** Фреймворк оценки включает метрики точности, латентности, использования токенов, анализ траекторий, LLM-as-a-Judge.

**Когда использовать:** При развёртывании в продакшн, A/B-тестировании, аудитах соответствия, детекции дрейфа, оценке сложного поведения.

**Визуальное резюме:**

![Паттерн оценки и мониторинга](../assets/Evaluation_and_Monitoring_Design_Pattern.png)

Рис. 4: Паттерн «Оценка и мониторинг»

## Ключевые выводы

* Оценка агентов выходит за рамки простых тестов к непрепывной многофакторной оценке.
* Практические применения: мониторинг в продакшн, A/B, аудиты, детекция дрейфа.
* Траектории агентов — критический элемент оценки.
* ADK: test files (unit) и evalset files (integration), три метода запуска.
* «Контракторы» — эволюция агентов с формальными соглашениями, переговорами, self-validation и декомпозицией.

## Заключение

Эффективная оценка AI-агентов требует многофакторного подхода: мониторинга латентности и токенов, анализа траекторий, LLM-as-a-Judge. Для production-агентов парадигма смещается к формальным «контрактам» с верифицируемыми deliverables. Это превращает агентов из непредсказуемых инструментов в подотчётные системы.

## Ссылки

1. ADK Web: [https://github.com/google/adk-web](https://github.com/google/adk-web)
2. ADK Evaluate: [https://google.github.io/adk-docs/evaluate/](https://google.github.io/adk-docs/evaluate/)
3. Survey on Evaluation of LLM-based Agents: [https://arxiv.org/abs/2503.16416](https://arxiv.org/abs/2503.16416)
4. Agent-as-a-Judge: [https://arxiv.org/abs/2410.10934](https://arxiv.org/abs/2410.10934)
5. Agent Companion: [https://www.kaggle.com/whitepaper-agent-companion](https://www.kaggle.com/whitepaper-agent-companion)
