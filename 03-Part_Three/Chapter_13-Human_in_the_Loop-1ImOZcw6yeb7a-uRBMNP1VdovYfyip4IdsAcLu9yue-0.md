# Глава 13: Человек в контуре (Human-in-the-Loop)

Паттерн «Человек в контуре» (HITL) представляет собой ключевую стратегию в разработке и внедрении агентов. Он целенаправленно переплетает уникальные преимущества человеческого познания — суждение, креативность, нюансированное понимание — с вычислительной мощностью и эффективностью ИИ. Эта стратегическая интеграция часто является необходимостью, особенно по мере внедрения AI-систем в критические процессы принятия решений.

Базовый принцип HITL — обеспечение работы ИИ в этических границах, соблюдение протоколов безопасности и достижение целей с максимальной эффективностью. Это особенно важно в доменах, характеризующихся сложностью, неоднозначностью или значительным риском.

Подход HITL вращается вокруг идеи синергии искусственного и человеческого интеллекта. ИИ позиционируется как инструмент, расширяющий человеческие возможности.

На практике HITL реализуется: валидация и рецензирование, активное управление в реальном времени, совместное решение проблем через интерактивный диалог.

## Обзор паттерна «Человек в контуре»

HITL интегрирует ИИ с человеческим вводом. Оптимальная производительность часто требует сочетания автоматизированной обработки и человеческой интуиции.

Аспекты:

* **Человеческий надзор:** Мониторинг производительности и вывода агента.
* **Вмешательство и коррекция:** Агент запрашивает вмешательство при ошибках; оператор исправляет, восполняет пробелы, направляет.
* **Обратная связь для обучения:** Сбор и использование для уточнения моделей (RLHF).
* **Аугментация решений:** Агент предоставляет анализ, человек принимает финальное решение.
* **Человеко-агентное сотрудничество:** Кооперативное взаимодействие с разделением обязанностей.
* **Политики эскалации:** Протоколы передачи задач операторам.

> **Оговорки:** HITL имеет ограничения: недостаточная масштабируемость (человеческий надзор не справляется с миллионами задач), зависимость от квалифицированных экспертов, вопросы приватности (данные должны анонимизироваться).

## Практические применения и сценарии использования

* **Модерация контента:** ИИ фильтрует, неоднозначные случаи — эскалация модераторам.
* **Беспилотные автомобили:** Автономное управление с передачей контроля человеку.
* **Детекция мошенничества:** ИИ флагирует, аналитики расследуют.
* **Юридический анализ:** ИИ сканирует, юристы проверяют.
* **Поддержка клиентов:** Чатбот → оператор при сложных проблемах.
* **Разметка данных:** Люди размечают для обучения моделей.
* **Доработка генеративного ИИ:** Редакторы проверяют вывод LLM.
* **Автономные сети:** ИИ анализирует алерты, критические решения эскалируются.

«Человек на контуре» (Human-on-the-loop) — вариация: эксперты задают политику, ИИ выполняет немедленные действия (трейдинг, маршрутизация звонков).

## Практический пример кода

```python
from typing import Optional

from google.adk.agents import Agent
from google.adk.tools.tool_context import ToolContext
from google.adk.callbacks import CallbackContext
from google.adk.models.llm import LlmRequest
from google.genai import types


# Placeholder for tools (replace with actual implementations if needed)
def troubleshoot_issue(issue: str) -> dict:
    return {"status": "success", "report": f"Troubleshooting steps for {issue}."}


def create_ticket(issue_type: str, details: str) -> dict:
    return {"status": "success", "ticket_id": "TICKET123"}


def escalate_to_human(issue_type: str) -> dict:
    # This would typically transfer to a human queue in a real system
    return {"status": "success", "message": f"Escalated {issue_type} to a human specialist."}


technical_support_agent = Agent(
    name="technical_support_specialist",
    model="gemini-2.0-flash-exp",
    instruction="""
    You are a technical support specialist for our electronics company.
    FIRST, check if the user has a support history in state["customer_info"]["support_history"].
    If they do, reference this history in your responses.

    For technical issues:
    1. Use the troubleshoot_issue tool to analyze the problem.
    2. Guide the user through basic troubleshooting steps.
    3. If the issue persists, use create_ticket to log the issue.

    For complex issues beyond basic troubleshooting:
    1. Use escalate_to_human to transfer to a human specialist.

    Maintain a professional but empathetic tone. Acknowledge the frustration technical issues can cause,
    while providing clear steps toward resolution.
    """,
    tools=[troubleshoot_issue, create_ticket, escalate_to_human],
)


def personalization_callback(
    callback_context: CallbackContext, llm_request: LlmRequest
) -> Optional[LlmRequest]:
    """Adds personalization information to the LLM request."""
    # Get customer info from state
    customer_info = callback_context.state.get("customer_info")
    if customer_info:
        customer_name = customer_info.get("name", "valued customer")
        customer_tier = customer_info.get("tier", "standard")
        recent_purchases = customer_info.get("recent_purchases", [])

        personalization_note = (
            f"\nIMPORTANT PERSONALIZATION:\n"
            f"Customer Name: {customer_name}\n"
            f"Customer Tier: {customer_tier}\n"
        )
        if recent_purchases:
            personalization_note += f"Recent Purchases: {', '.join(recent_purchases)}\n"

        if llm_request.contents:
            # Add as a system message before the first content
            system_content = types.Content(
                role="system",
                parts=[types.Part(text=personalization_note)],
            )
            llm_request.contents.insert(0, system_content)

    return None  # Return None to continue with the modified request
```

## Краткий обзор

**Что:** AI-агенты struggle с нюансированными суждениями и этическим рассуждением. Полная автономия несёт риски.

**Почему:** HITL стратегически интегрирует человеческий надзор. Симбиотическое партнёрство: ИИ — вычисления, человек — валидация и вмешательство.

**Когда использования:** В доменах с серьёзными последствиями ошибок. Для задач с неоднозначностью. Для continuous improvement.

**Визуальное резюме:**

![Паттерн «Человек в контуре»](../assets/Human_in_the_Loop_Design_Pattern.png)

Рис. 1: Паттерн «Человек в контуре»

## Ключевые выводы

* HITL интегрирует человеческий интеллект и суждение в AI-рабочие процессы.
* Критически важен для безопасности, этики и эффективности.
* Аспекты: надзор, вмешательство, обратная связь, аугментация решений.
* Политики эскалации необходимы для передачи задач.
* Ограничения: масштабируемость, зависимость от экспертов, приватность.

## Заключение

HITL — ключевой паттерн для создания устойчивых, безопасных и этичных AI-систем. Интеграция надзора, вмешательства и обратной связи повышает надёжность. По мере развития ИИ HITL остаётся краеугольным камнем ответственной разработки.

## Ссылки

1. A Survey of Human-in-the-loop for Machine Learning: [https://arxiv.org/abs/2108.00941](https://arxiv.org/abs/2108.00941)
