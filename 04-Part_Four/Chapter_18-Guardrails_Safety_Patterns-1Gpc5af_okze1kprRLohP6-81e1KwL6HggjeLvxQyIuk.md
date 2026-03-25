# Глава 18: Ограничители/Паттерны безопасности

Ограничители (guardrails), также известные как паттерны безопасности, — критически важные механизмы, обеспечивающие безопасную, этичную и целенаправленную работу интеллектуальных агентов. По мере роста автономности и интеграции в критические системы они выступают защитным слоем, направляющим поведение и вывод агента для предотвращения вредоносных, предвзятых, нерелевантных или иных нежелательных ответов.

Ограничители могут реализовываться на различных этапах: валидация и санитизация входа (фильтрация вредоносного контента), фильтрация выхода (анализ сгенерированных ответов на токсичность или смещение), поведенческие ограничения (уровень промпта), ограничения использования инструментов, внешние API модерации, контроль человека (Human-in-the-Loop).

Основная цель — не ограничить возможности агента, а обеспечить его надёжную, заслуживающую доверия и полезную работу. Без ограничителей AI-система может быть непредсказуемой и потенциально опасной. Для дополнительного снижения рисков менее вычислительно затратная модель может использоваться как быстрая дополнительная защита для предварительной проверки входов или перепроверки выходов основной модели.

## Практические применения и сценарии использования

* **Чатботы клиентской поддержки:** Предотвращение генерации оскорбительного языка, вредных советов (медицинских, юридических), off-topic ответов.
* **Системы генерации контента:** Соответствие правилам, юридическим и этическим стандартам, избегание hate speech, дезинформации.
* **Обучающие репетиторы:** Предотвращение неверных ответов, предвзятых точек зрения, неуместных диалогов.
* **Юридические ассистенты:** Предотвращение предоставления определённых юридических советов, направление к лицензированным юристам.
* **Инструменты найма и HR:** Обеспечение справедливости, предотвращение смещений при отборе кандидатов.
* **Модерация соцсетей:** Автоматическое выявление hate speech, дезинформации, откровенного контента.
* **Научные ассистенты:** Предотвращение фабрикации данных или необоснованных выводов.

## Практический пример кода (CrewAI)

Реализация ограничителей в CrewAI — многоаспектный подход, требующий многоуровневой защиты. Процесс начинается с санитизации и валидации входа, включая API модерации и Pydantic-валидацию.

Мониторинг и наблюдаемость: логирование всех действий, метрики (латентность, успешность, ошибки), трассировка.

Обработка ошибок: try-except, retry с exponential backoff, чёткие сообщения.

Конфигурация агента — дополнительный слой: определение ролей, целей, backstory. Безопасное управление API-ключами.

```python
import os, json, logging
from typing import Tuple, Any, List
from crewai import Agent, Task, Crew, Process, LLM
from pydantic import BaseModel, Field, ValidationError
from crewai.tasks.task_output import TaskOutput
from crewai.crews.crew_output import CrewOutput

logging.basicConfig(level=logging.ERROR, format='%(asctime)s - %(levelname)s - %(message)s')

CONTENT_POLICY_MODEL = "gemini/gemini-2.0-flash"

SAFETY_GUARDRAIL_PROMPT = """
You are an AI Content Policy Enforcer...
"""

class PolicyEvaluation(BaseModel):
   compliance_status: str = Field(description="'compliant' or 'non-compliant'")
   evaluation_summary: str = Field(description="Brief explanation")
   triggered_policies: List[str] = Field(description="List of triggered policies")

def validate_policy_evaluation(output: Any) -> Tuple[bool, Any]:
   # Validates LLM output against PolicyEvaluation Pydantic model
   # Returns (True, evaluation) or (False, error_message)
   ...

policy_enforcer_agent = Agent(
   role='AI Content Policy Enforcer',
   goal='Screen user inputs against safety policies',
   verbose=False, allow_delegation=False,
   llm=LLM(model=CONTENT_POLICY_MODEL, temperature=0.0)
)

evaluate_input_task = Task(
   description=f"{SAFETY_GUARDRAIL_PROMPT}\nUser Input: '{{user_input}}'",
   expected_output="JSON conforming to PolicyEvaluation schema",
   agent=policy_enforcer_agent,
   guardrail=validate_policy_evaluation,
   output_pydantic=PolicyEvaluation,
)

crew = Crew(agents=[policy_enforcer_agent], tasks=[evaluate_input_task], process=Process.sequential)
```

Код создаёт систему принудительной контентной политики. `SAFETY_GUARDRAIL_PROMPT` определяет политики: попытки jailbreak, запрещённый контент (дискриминация, опасные действия, откровенный материал), off-topic дискуссии, обсуждение конкурентов. Pydantic-модель PolicyEvaluation структурирует вывод. Функция-ограничитель валидирует и логирует. Агент с моделью Gemini Flash обеспечивает детерминированную проверку.

## Практический пример кода (Vertex AI)

Google Cloud Vertex AI предлагает многоаспектный подход: идентификация и авторизация, фильтрация входов/выходов, встроенные функции безопасности Gemini, валидация через callbacks.

Практики: менее затратная модель как дополнительная защита, изолированная среда выполнения кода, оценка и мониторинг, ограничение в сетевых границах (VPC).

```python
from google.adk.agents import Agent
from google.adk.tools.base_tool import BaseTool
from google.adk.tools.tool_context import ToolContext
from typing import Optional, Dict, Any

def validate_tool_params(tool: BaseTool, args: Dict[str, Any], tool_context: ToolContext) -> Optional[Dict]:
    expected_user_id = tool_context.state.get("session_user_id")
    actual_user_id_in_args = args.get("user_id_param")
    if actual_user_id_in_args and actual_user_id_in_args != expected_user_id:
        return {"status": "error", "error_message": "Tool call blocked: User ID validation failed."}
    return None

root_agent = Agent(
    model='gemini-2.0-flash-exp',
    name='root_agent',
    instruction="You are a root agent that validates tool calls.",
    before_tool_callback=validate_tool_params,
)
```

Callback валидирует аргументы инструмента перед выполнением: сравнивает user_id из аргументов с session_user_id. При несовпадении — блокировка.

## Инженерия надёжных агентов

Построение надёжных AI-агентов требует применения тех же принципов, что и в традиционной разработке ПО:

* **Checkpoint и rollback:** Реализация чекпоинтов — аналог транзакционных систем с commit и rollback. Каждый чекпоинт — валидное состояние.
* **Модульность и разделение ответственности:** Специализированные агенты/инструменты вместо монолитных. Улучшает масштабируемость, устойчивость к сбоям.
* **Наблюдаемость через структурированное логирование:** Полная цепочка рассуждений агента: инструменты, данные, reasoning, confidence scores.
* **Принцип минимальных привилегий:** Агенту даются только минимально необходимые права.
* **Изоляция выполнения:** Песочница для запуска кода.

## Краткий обзор

**Что:** Автономные AI-агенты могут генерировать вредоносные, предвзятые или фактически неверные выводы. Уязвимы к атакам типа jailbreak.

**Почему:** Ограничители — многоуровневый защитный механизм на всех этапах: валидация входа, фильтрация выхода, ограничение инструментов, контроль человека.

**Когда использовать:** В любых приложениях, где вывод AI-агента влияет на пользователей, системы или репутацию. Критически важно для customer-facing ролей, генерации контента, чувствительных данных.

**Визуальное резюме:**

![Паттерн ограничителей](../assets/Guardrail_Design_Pattern.png)

Рис. 1: Паттерн «Ограничители»

## Ключевые выводы

* Ограничители необходимы для построения ответственных, этичных и безопасных агентов.
* Реализуются на всех этапах: валидация входа, фильтрация выхода, поведенческие промпты, ограничения инструментов, внешняя модерация.
* Комбинация техник даёт наиболее надёжную защиту.
* Требуют непрерывного мониторинга и доработки.
* Для production-агентов применяйте инженерные best practices: устойчивость к сбоям, управление состоянием, тестирование.

## Заключение

Реализация эффективных ограничителей — ядро ответственной разработки ИИ. Стратегическое применение паттернов безопасности позволяет строить надёжных, эффективных и заслуживающих доверия агентов. Многоуровневая защита от валидации входа до контроля человека создаёт устойчивую систему.

## Ссылки

1. Принципы безопасности Google AI: [https://ai.google/principles/](https://ai.google/principles/)
2. Руководство OpenAI по модерации: [https://platform.openai.com/docs/guides/moderation](https://platform.openai.com/docs/guides/moderation)
3. Prompt injection: [https://en.wikipedia.org/wiki/Prompt_injection](https://en.wikipedia.org/wiki/Prompt_injection)
