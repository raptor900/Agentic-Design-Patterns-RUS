# Глава 16: Ресурсно-ориентированная оптимизация

Ресурсно-ориентированная оптимизация позволяет интеллектуальным агентам динамически отслеживать и управлять вычислительными, временными и финансовыми ресурсами в процессе работы. В отличие от простого планирования, фокусирующегося на последовательности действий, ресурсно-ориентированная оптимизация требует от агентов принятия решений об выполнении действий для достижения целей в рамках бюджета ресурсов или для оптимизации эффективности. Это включает выбор между более точными, но дорогими моделями и более быстрыми, дешёвыми, или решение о выделении дополнительных вычислений для более качественного ответа versus возврат более быстрого, менее детального.

Например, агент, анализирующий большой набор данных для финансового аналитика. Если аналитику нужен предварительный отчёт немедленно, агент использует быструю и доступную модель для обобщения трендов. Если нужен высокоточный прогноз для критического инвестиционного решения с большим бюджетом — агент выделяет ресурсы для использования мощной, медленной, но точной предиктивной модели. Ключевая стратегия — fallback-механизм, действующий как страховка при недоступности предпочтительной модели (перегрузка, троттлинг). Для graceful degradation система автоматически переключается на модель по умолчанию.

## Практические применения и сценарии использования

* **Оптимизация стоимости LLM:** Выбор между дорогой LLM для сложных задач и доступной — для простых.
* **Операции с чувствительной латентностью:** Выбор более быстрого пути рассуждения в реальном времени.
* **Энергоэффективность:** Оптимизация обработки на edge-устройствах.
* **Fallback для надёжности:** Автоматическое переключение на резервную модель.
* **Управление использованием данных:** Использование обобщённых данных вместо полных наборов.
* **Адаптивное распределение задач:** Самоназначение задач в многоагентных системах на основе нагрузки.

## Практический пример кода

Система оценивает сложность вопроса. Для простых — Gemini Flash (быстро, дёшево). Для сложных — Gemini Pro (точно, дорого). Решение зависит от бюджета и времени.

Например, планировщик путешествий с иерархическим агентом: высокоуровневое планирование (понимание сложного запроса, составление маршрута) — мощной LLM. Отдельные задачи (поиск цен, проверка наличия) — быстрой и доступной моделью.

Google ADK поддерживает подход через многоагентную архитектуру: модульность, гибкость моделей (Gemini Pro/Flash, другие через LiteLLM), динамическая маршрутизация, встроенная оценка.

Два агента с одинаковой настройкой, но разными моделями:

```python
from google.adk.agents import Agent

gemini_pro_agent = Agent(
    name="GeminiProAgent",
    model="gemini-2.5-pro",
    description="A highly capable agent for complex queries.",
    instruction="You are an expert assistant for complex problem-solving.",
)

gemini_flash_agent = Agent(
    name="GeminiFlashAgent",
    model="gemini-2.5-flash",
    description="A fast and efficient agent for simple queries.",
    instruction="You are a quick assistant for straightforward questions.",
)
```

Агент-маршрутизатор может направлять запросы на основе метрик: короткие — на дешёвую модель, длинные — на продвинутую. Более sophisticated Router использует LLM или ML-модели для анализа нюансов и сложности запроса. Оптимизация: prompt tuning, fine-tuning маршрутизатора.

```python
import asyncio
from typing import AsyncGenerator
from google.adk.agents import Agent, BaseAgent
from google.adk.events import Event
from google.adk.agents.invocation_context import InvocationContext

class QueryRouterAgent(BaseAgent):
    name: str = "QueryRouter"
    description: str = "Routes user queries to the appropriate LLM agent based on complexity."

    async def _run_async_impl(self, context: InvocationContext) -> AsyncGenerator[Event, None]:
        user_query = context.current_message.text
        query_length = len(user_query.split())

        if query_length < 20:
            response = await gemini_flash_agent.run_async(context.current_message)
            yield Event(author=self.name, content=f"Flash Agent processed: {response}")
        else:
            response = await gemini_pro_agent.run_async(context.current_message)
            yield Event(author=self.name, content=f"Pro Agent processed: {response}")
```

**Агент-критик** оценивает ответы: для самокоррекции (выявление ошибок, доработка), мониторинга производительности, сигналов для RL/fine-tuning, косвенного управления бюджетом (идентификация неоптимальных маршрутизационных решений).

```
CRITIC_SYSTEM_PROMPT = """
You are the Critic Agent, the quality assurance arm of the research system.
Your function is to meticulously review and challenge information, guaranteeing
accuracy, completeness, and unbiased presentation.
All criticism must be constructive. Structure your feedback clearly.
"""
```

## Практический пример кода с OpenAI

Система классифицирует запросы на три категории:

* **simple:** Простые фактические вопросы.
* **reasoning:** Логика, математика, многошаговые выводы.
* **internet_search:** Текущие события, данные вне обучающей выборки.

```python
import os, json, requests
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()
client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

def classify_prompt(prompt: str) -> dict:
    response = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "Classify into: simple, reasoning, internet_search. Return JSON: {\"classification\": \"...\"}"},
            {"role": "user", "content": prompt},
        ],
    )
    return json.loads(response.choices[0].message.content)

def google_search(query: str) -> list:
    params = {"key": os.getenv("GOOGLE_CUSTOM_SEARCH_API_KEY"), "cx": os.getenv("GOOGLE_CSE_ID"), "q": query, "num": 1}
    r = requests.get("https://www.googleapis.com/customsearch/v1", params=params)
    return [{"title": i.get("title"), "snippet": i.get("snippet")} for i in r.json().get("items", [])]

def handle_prompt(prompt: str) -> dict:
    cls = classify_prompt(prompt)["classification"]
    model = {"simple": "gpt-4o-mini", "reasoning": "o4-mini", "internet_search": "gpt-4o"}.get(cls, "gpt-4o")
    ctx = google_search(prompt) if cls == "internet_search" else None
    full = f"Web results:\n{ctx}\nQuery: {prompt}" if ctx else prompt
    resp = client.chat.completions.create(model=model, messages=[{"role": "user", "content": full}])
    return {"classification": cls, "response": resp.choices[0].message.content, "model": model}
```

## OpenRouter

OpenRouter — единый интерфейс к сотням AI-моделей. Автоматический failover, оптимизация затрат, интеграция через SDK.

```python
import json, requests
response = requests.post(
    url="https://openrouter.ai/api/v1/chat/completions",
    headers={"Authorization": "Bearer <OPENROUTER_API_KEY>"},
    data=json.dumps({"model": "openai/gpt-4o", "messages": [{"role": "user", "content": "What is the meaning of life?"}]}),
)
```

OpenRouter предлагает:

* **Автоматический выбор модели:** `model: "openrouter/auto"` — маршрутизация к оптимизированной модели.
* **Последовательный fallback:** `models: ["anthropic/claude-3.5-sonnet", "gryphe/mythomax-l2-13b"]` — первая модель fails → следующая.

Рейтинг моделей: [https://openrouter.ai/rankings](https://openrouter.ai/rankings).

## Спектр оптимизаций агентных ресурсов

* **Динамическое переключение моделей:** Стратегический выбор LLM на основе сложности задачи.
* **Адаптивное использование инструментов:** Выбор наиболее эффективного инструмента с учётом стоимости API, латентности, времени выполнения.
* **Контекстное сокращение и обобщение:** Минимизация токенов через обобщение и выборочное сохранение релевантной информации.
* **Проактивное предсказание ресурсов:** Прогнозирование нагрузок для proactive аллокации.
* **Ресурсно-чувствительное исследование:** Учёт коммуникационных затрат в многоагентных системах.
* **Энергоэффективное развёртывание:** Минимизация энергопотребления.
* **Параллелизация и распределённые вычисления:** Распределение нагрузки.
* **Обучаемые политики распределения ресурсов:** Адаптация стратегий на основе обратной связи.
* **Graceful degradation и fallback:** Работа при severe ограничениях ресурсов.

## Краткий обзор

**Что:** Управление вычислительными, временными и финансовыми ресурсами. LLM-приложения дороги и медленны; без динамического управления системы не могут адаптироваться.

**Почему:** Агентная система с «Router Agent» классифицирует сложность и маршрутизирует к подходящей LLM. «Critique Agent» оценивает качество и улучшает логику маршрутизации.

**Когда использовать:** При жёстких бюджетах, latency-критичных приложениях, edge-устройствах, балансировке качества и стоимости.

**Визуальное резюме:**

![Паттерн ресурсно-ориентированной оптимизации](../assets/Resource_Aware_Optimization_Design_Pattern.png)

Рис. 2: Паттерн ресурсно-ориентированной оптимизации

## Ключевые выводы

* Ресурсно-ориентированная оптимизация необходима для динамического управления ресурсами.
* Многоагентная архитектура ADK обеспечивает модульность.
* LLM-маршрутизатор направляет запросы к оптимальной модели.
* Агент-критик предоставляет обратную связь для самокоррекции и оптимизации логики маршрутизации.
* Дополнительные оптимизации: адаптивное использование инструментов, контекстное сокращение, предсказание ресурсов, graceful degradation.

## Заключение

Ресурсно-ориентированная оптимизация необходима для эффективной работы интеллектуальных агентов в реальных ограничениях. Техники: динамическое переключение моделей, адаптивное использование инструментов, контекстное сокращение. Продвинутые стратегии: обучаемые политики, graceful degradation. Интеграция этих принципов — основа для масштабируемых, надёжных и устойчивых AI-систем.

## Ссылки

1. Google ADK: [https://google.github.io/adk-docs/](https://google.github.io/adk-docs/)
2. Gemini Flash 2.5 & Pro: [https://aistudio.google.com/](https://aistudio.google.com/)
3. OpenRouter: [https://openrouter.ai/docs/quickstart](https://openrouter.ai/docs/quickstart)
