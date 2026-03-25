# Глава 20: Приоритизация

В сложных динамичных средах агенты часто сталкиваются с многочисленными возможными действиями, конфликтующими целями и ограниченными ресурсами. Без определённого процесса определения следующего действия эффективность снижается, возникают задержки или невыполнение ключевых целей. Паттерн «Приоритизация» решает эту задачу, позволяя агентам оценивать и ранжировать задачи, цели и действия на основе их значимости, срочности, зависимостей и установленных критериев.

## Обзор паттерна «Приоритизация»

Агенты используют приоритизацию для управления задачами, целями и подцелями. Процесс включает:

* **Определение критериев:** Правила или метрики для оценки задач: срочность, важность, зависимости, доступность ресурсов, анализ затрат/выгод, предпочтения пользователя.
* **Оценка задач:** Сравнение каждой задачи с критериями (от простых правил до LLM-рассуждений).
* **Логика выбора:** Алгоритм, выбирающий оптимальное действие или последовательность.
* **Динамическая переприоритизация:** Изменение приоритетов при изменении обстоятельств (новые события, приближающиеся дедлайны).

Приоритизация может происходить на разных уровнях: выбор общей цели, упорядочивание шагов плана, выбор следующего действия.

## Практические применения и сценарии использования

* **Автоматизированная клиентская поддержка:** Приоритет срочных запросов (сбои системы) над рутинными (сброс пароля).
* **Облачные вычисления:** Приоритетное выделение ресурсов критичным приложениям.
* **Беспилотные автомобили:** Торможение для избежания столкновения > соблюдение полосы > оптимизация расхода топлива.
* **Финансовый трейдинг:** Приоритет сделок по рыночным условиям, риску, прибыли.
* **Управление проектами:** Приоритет задач по дедлайнам, зависимостям, стратегической важности.
* **Кибербезопасность:** Приоритет алертов по серьёзности угрозы и критичности активов.
* **Персональные ассистенты:** Организация событий, напоминаний, уведомлений по важности.

## Практический пример кода

```python
import os, asyncio
from typing import List, Optional, Dict
from dotenv import load_dotenv
from pydantic import BaseModel, Field
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import Tool
from langchain_openai import ChatOpenAI
from langchain.agents import AgentExecutor, create_react_agent
from langchain.memory import ConversationBufferMemory

load_dotenv()
llm = ChatOpenAI(temperature=0.5, model="gpt-4o-mini")

class Task(BaseModel):
    id: str
    description: str
    priority: Optional[str] = None
    assigned_to: Optional[str] = None

class SuperSimpleTaskManager:
    def __init__(self):
        self.tasks: Dict[str, Task] = {}
        self.next_task_id = 1

    def create_task(self, description: str) -> Task:
        task_id = f"TASK-{self.next_task_id:03d}"
        new_task = Task(id=task_id, description=description)
        self.tasks[task_id] = new_task
        self.next_task_id += 1
        return new_task

    def update_task(self, task_id: str, **kwargs) -> Optional[Task]:
        task = self.tasks.get(task_id)
        if task:
            update_data = {k: v for k, v in kwargs.items() if v is not None}
            updated_task = task.model_copy(update=update_data)
            self.tasks[task_id] = updated_task
            return updated_task
        return None

    def list_all_tasks(self) -> str:
        if not self.tasks:
            return "No tasks in the system."
        return "Current Tasks:\n" + "\n".join(
            f"ID: {t.id}, Desc: '{t.description}', Priority: {t.priority or 'N/A'}, Assigned: {t.assigned_to or 'N/A'}"
            for t in self.tasks.values()
        )

task_manager = SuperSimpleTaskManager()

pm_tools = [
    Tool(name="create_new_task", func=lambda d: f"Created {task_manager.create_task(d).id}", description="Create task"),
    Tool(name="assign_priority", func=lambda tid, p: f"Priority {p} set for {tid}" if task_manager.update_task(tid, priority=p) else "Not found", description="Set priority P0/P1/P2"),
    Tool(name="assign_worker", func=lambda tid, w: f"Assigned to {w}" if task_manager.update_task(tid, assigned_to=w) else "Not found", description="Assign worker"),
    Tool(name="list_all_tasks", func=task_manager.list_all_tasks, description="List tasks"),
]

pm_prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a Project Manager. Create task, assign priority and worker, then list all tasks. Defaults: P1, Worker A."),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}"),
])

pm_agent = create_react_agent(llm, pm_tools, pm_prompt)
pm_agent_executor = AgentExecutor(agent=pm_agent, tools=pm_tools, verbose=True, handle_parsing_errors=True, memory=ConversationBufferMemory(memory_key="chat_history", return_messages=True))

async def run_simulation():
    await pm_agent_executor.ainvoke({"input": "Create a task to implement a new login system. It's urgent and should be assigned to Worker B."})
    await pm_agent_executor.ainvoke({"input": "Manage a new task: Review marketing website content."})

if __name__ == "__main__":
    asyncio.run(run_simulation())
```

Код реализует систему управления задачами с AI-агентом-менеджером. SuperSimpleTaskManager хранит задачи в словаре для быстрого доступа. Инструменты: создание, назначение приоритета, назначение исполнителя, список задач. Промпт инструктирует агента: создать задачу → назначить приоритет и исполнителя → показать список. Агент самостоятельно интерпретирует «urgent» как P0 и распределяет задачи.

## Краткий обзор

**Что:** Агенты в сложных средах сталкиваются с множеством действий и конфликтующих целей. Без приоритизации — неэффективность и невыполнение целей.

**Почему:** Приоритизация позволяет ранжировать задачи по срочности, важности, зависимостям. Динамическая переприоритизация обеспечивает адаптивность.

**Когда использовать:** При автономном управлении несколькими конфликтующими задачами в ограниченных ресурсах.

**Визуальное резюме:**

![Паттерн приоритизации](../assets/Prioritization_Design_Pattern.png)

Рис. 1: Паттерн «Приоритизация»

## Ключевые выводы

* Приоритизация позволяет агентам работать эффективно в сложных средах.
* Критерии: срочность, важность, зависимости, стоимость.
* Динамическая переприоритизация — адаптация к изменениям в реальном времени.
* Приоритизация на разных уровнях: стратегическом и тактическом.

## Заключение

Паттерн приоритизации — краеугольный камень эффективных агентных систем. Он позволяет автономно оценивать конфликтующие задачи, принимать обоснованные решения и адаптировать фокус в реальном времени. Это превращает агента из простого исполнителя в проактивного стратегического decision-maker.

## Ссылки

1. AI-driven Project Scheduling: [https://www.irejournals.com/paper-details/1706160](https://www.irejournals.com/paper-details/1706160)
2. AI-Driven Decision Support in Agile PM: [https://www.mdpi.com/2079-8954/13/3/208](https://www.mdpi.com/2079-8954/13/3/208)
