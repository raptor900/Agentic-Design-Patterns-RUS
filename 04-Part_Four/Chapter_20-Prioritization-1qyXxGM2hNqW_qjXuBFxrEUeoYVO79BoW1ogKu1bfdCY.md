# Глава 20: Приоритизация

В сложных динамичных окружениях агенты часто сталкиваются с множеством потенциальных действий, конфликтующими целями и ограниченными ресурсами. Без определённого процесса определения следующего действия агенты могут испытывать снижение эффективности, задержки или неспособность достичь ключевых целей. Паттерн «Приоритизация» решает эту проблему, позволяя агентам оценивать и ранжировать задачи, цели или действия по их значимости, срочности, зависимостям и установленным критериям. Это гарантирует, что агенты концентрируют усилия на наиболее критических задачах, повышая эффективность и согласованность с целями.

## Обзор паттерна «Приоритизация»

Агенты используют приоритизацию для эффективного управления задачами, целями и подцелями, направляя последующие действия. Этот процесс обеспечивает обоснованное принятие решений при работе с несколькими запросами, ставя важные или срочные действия выше менее критических. Он особенно актуален в реальных сценариях, где ресурсы ограничены, время ограничено, а цели могут конфликтовать.

Фундаментальные аспекты агентной приоритизации обычно включают несколько элементов. Во-первых, определение критериев устанавливает правила или метрики для оценки задач: срочность (временная чувствительность задачи), важность (влияние на основную цель), зависимости (является ли задача предшественником для других), доступность ресурсов (готовность необходимых инструментов или информации), баланс усилий и ожидаемого результата, а также пользовательские предпочтения для персонализированных агентов. Во-вторых, оценка задач — анализ каждой потенциальной задачи по этим критериям методами от простых правил до сложного скоринга или рассуждений LLM. В-третьих, логика планирования или выбора — алгоритм, который на основе оценок выбирает оптимальное следующее действие или последовательность задач, потенциально используя очередь или продвинутый компонент планирования. Наконец, динамическая переприоритизация позволяет агенту изменять приоритеты по мере изменения обстоятельств — появления нового критического события или приближения дедлайна — обеспечивая адаптивность и отзывчивость.

Приоритизация может происходить на различных уровнях: выбор общей цели (приоритизация высокоуровневых целей), упорядочивание шагов внутри плана (приоритизация подзадач) или выбор немедленного действия из доступных вариантов (выбор действия). Эффективная приоритизация позволяет агентам демонстрировать более интеллектуальное, эффективное и устойчивое поведение, особенно в сложных многоцелевых окружениях. Это отражает организацию человеческих команд, где менеджеры приоритизируют задачи, учитывая вклад всех участников.

## Практические применения и сценарии

В различных реальных применениях агенты искусственного интеллекта демонстрируют расширенное использование приоритизации для своевременного и эффективного принятия решений.

* **Автоматизированная клиентская поддержка и сервис:** Агенты приоритизируют срочные запросы вроде сообщений о системных сбоях над рутинными — например, сбросом пароля. Они также могут отдавать приоритет VIP-клиентам.
* **Облачные вычисления:** AI управляет и планирует ресурсы, выделяя их критическим приложениям в часы пиковой нагрузки, откладывая менее срочные пакетные задачи на непиковое время для оптимизации затрат.
* **Системы автономного вождения:** Постоянно приоритизируют действия для обеспечения безопасности и эффективности: торможение для предотвращения столкновения важнее, чем удержание в полосе или экономия топлива.
* **Финансовый трейдинг:** Боты приоритизируют сделки, анализируя рыночные условия, толерантность к риску, маржу прибыли и новости в реальном времени, обеспечивая немедленное исполнение высокоприоритетных транзакций.
* **Управление проектами:** AI-агенты приоритизируют задачи на проектной доске на основе дедлайнов, зависимостей, доступности команды и стратегической важности.
* **Кибербезопасность:** Агенты, анализирующие сетевой трафик, приоритизируют оповещения по severity угрозы, потенциальному воздействию и критичности активов, обеспечивая немедленный ответ на самые опасные угрозы.
* **Персональные AI-ассистенты:** Используют приоритизацию для управления повседневной жизнью организуя события календаря, напоминания и уведомления по определяемой пользователем важности, ближайшим дедлайнам и текущему контексту.

Эти примеры коллективно иллюстрируют, как способность приоритизировать лежит в основе улучшенной производительности и возможностей принятия решений AI-агентов across широкого спектра ситуаций.

## Практический пример кода (LangChain)

Следующий код демонстрирует разработку агента «менеджер проекта» с использованием LangChain. Этот агент позволяет создавать, приоритизировать и назначать задачи членам команды, иллюстрируя применение больших языковых моделей с пользовательскими инструментами для автоматизированного управления проектами.
```python
import os
import asyncio
from typing import List, Optional, Dict, Type

from dotenv import load_dotenv
from pydantic import BaseModel, Field
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import Tool
from langchain_openai import ChatOpenAI
from langchain.agents import AgentExecutor, create_react_agent
from langchain.memory import ConversationBufferMemory


# --- 0. Конфигурация и настройка ---
# Загружает OPENAI_API_KEY из файла .env.
load_dotenv()

# Клиент ChatOpenAI автоматически получает API ключ из окружения.
llm = ChatOpenAI(temperature=0.5, model="gpt-4o-mini")


# --- 1. Система управления задачами ---
class Task(BaseModel):
    """Представляет отдельную задачу в системе."""
    id: str
    description: str
    priority: Optional[str] = None  # P0, P1, P2
    assigned_to: Optional[str] = None  # Имя исполнителя


class SuperSimpleTaskManager:
    """Эффективный и надёжный менеджер задач в памяти."""

    def __init__(self):
        # Используем словарь для поиска, обновления и удаления за O(1).
        self.tasks: Dict[str, Task] = {}
        self.next_task_id = 1

    def create_task(self, description: str) -> Task:
        """Создаёт и сохраняет новую задачу."""
        task_id = f"TASK-{self.next_task_id:03d}"
        new_task = Task(id=task_id, description=description)
        self.tasks[task_id] = new_task
        self.next_task_id += 1
        print(f"DEBUG: Task created - {task_id}: {description}")
        return new_task

    def update_task(self, task_id: str, **kwargs) -> Optional[Task]:
        """Безопасное обновление задачи через model_copy из Pydantic."""
        task = self.tasks.get(task_id)
        if task:
            # Используем model_copy для типобезопасного обновления.
            update_data = {k: v for k, v in kwargs.items() if v is not None}
            updated_task = task.model_copy(update=update_data)
            self.tasks[task_id] = updated_task
            print(f"DEBUG: Task {task_id} updated with {update_data}")
            return updated_task

        print(f"DEBUG: Task {task_id} not found for update.")
        return None

    def list_all_tasks(self) -> str:
        """Выводит список всех задач в системе."""
        if not self.tasks:
            return "No tasks in the system."

        task_strings = []
        for task in self.tasks.values():
            task_strings.append(
                f"ID: {task.id}, Desc: '{task.description}', "
                f"Priority: {task.priority or 'N/A'}, "
                f"Assigned To: {task.assigned_to or 'N/A'}"
            )
        return "Current Tasks:\n" + "\n".join(task_strings)


task_manager = SuperSimpleTaskManager()


# --- 2. Инструменты для агента менеджера проекта ---
# Используем модели Pydantic для аргументов инструментов — лучшая валидация и ясность.
class CreateTaskArgs(BaseModel):
    description: str = Field(description="A detailed description of the task.")


class PriorityArgs(BaseModel):
    task_id: str = Field(description="The ID of the task to update, e.g., 'TASK-001'.")
    priority: str = Field(description="The priority to set. Must be one of: 'P0', 'P1', 'P2'.")


class AssignWorkerArgs(BaseModel):
    task_id: str = Field(description="The ID of the task to update, e.g., 'TASK-001'.")
    worker_name: str = Field(description="The name of the worker to assign the task to.")


def create_new_task_tool(description: str) -> str:
    """Создаёт новую задачу проекта с заданным описанием."""
    task = task_manager.create_task(description)
    return f"Created task {task.id}: '{task.description}'."


def assign_priority_to_task_tool(task_id: str, priority: str) -> str:
    """Назначает приоритет (P0, P1, P2) задаче по её ID."""
    if priority not in ["P0", "P1", "P2"]:
        return "Invalid priority. Must be P0, P1, or P2."
    task = task_manager.update_task(task_id, priority=priority)
    return f"Assigned priority {priority} to task {task.id}." if task else f"Task {task_id} not found."


def assign_task_to_worker_tool(task_id: str, worker_name: str) -> str:
    """Назначает задачу конкретному исполнителю."""
    task = task_manager.update_task(task_id, assigned_to=worker_name)
    return f"Assigned task {task.id} to {worker_name}." if task else f"Task {task_id} not found."


# Все инструменты, доступные агенту PM
pm_tools = [
    Tool(
        name="create_new_task",
        func=create_new_task_tool,
        description="Use this first to create a new task and get its ID.",
        args_schema=CreateTaskArgs
    ),
    Tool(
        name="assign_priority_to_task",
        func=assign_priority_to_task_tool,
        description="Use this to assign a priority to a task after it has been created.",
        args_schema=PriorityArgs
    ),
    Tool(
        name="assign_task_to_worker",
        func=assign_task_to_worker_tool,
        description="Use this to assign a task to a specific worker after it has been created.",
        args_schema=AssignWorkerArgs
    ),
    Tool(
        name="list_all_tasks",
        func=task_manager.list_all_tasks,
        description="Use this to list all current tasks and their status."
    ),
]


# --- 3. Определение агента менеджера проекта ---
pm_prompt_template = ChatPromptTemplate.from_messages([
    ("system", """You are a focused Project Manager LLM agent. Your goal is to manage project tasks efficiently.
      When you receive a new task request, follow these steps:
    1.  First, create the task with the given description using the `create_new_task` tool. You must do this first to get a `task_id`.
    2.  Next, analyze the user's request to see if a priority or an assignee is mentioned.
        - If a priority is mentioned (e.g., "urgent", "ASAP", "critical"), map it to P0. Use `assign_priority_to_task`.
        - If a worker is mentioned, use `assign_task_to_worker`.
    3.  If any information (priority, assignee) is missing, you must make a reasonable default assignment (e.g., assign P1 priority and assign to 'Worker A').
    4.  Once the task is fully processed, use `list_all_tasks` to show the final state.

    Available workers: 'Worker A', 'Worker B', 'Review Team'
    Priority levels: P0 (highest), P1 (medium), P2 (lowest)
    """),
    ("placeholder", "{chat_history}"),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}")
])

# Создаём исполнителя агента
pm_agent = create_react_agent(llm, pm_tools, pm_prompt_template)
pm_agent_executor = AgentExecutor(
    agent=pm_agent,
    tools=pm_tools,
    verbose=True,
    handle_parsing_errors=True,
    memory=ConversationBufferMemory(memory_key="chat_history", return_messages=True)
)


# --- 4. Простой поток взаимодействия ---
async def run_simulation():
    print("--- Project Manager Simulation ---")

    # Сценарий 1: Обработка нового срочного запроса на функцию
    print("\n[User Request] I need a new login system implemented ASAP. It should be assigned to Worker B.")
    await pm_agent_executor.ainvoke({"input": "Create a task to implement a new login system. It's urgent and should be assigned to Worker B."})

    print("\n" + "-" * 60 + "\n")

    # Сценарий 2: Обработка менее срочного обновления контента с меньшим количеством деталей
    print("[User Request] We need to review the marketing website content.")
    await pm_agent_executor.ainvoke({"input": "Manage a new task: Review marketing website content."})

    print("\n--- Simulation Complete ---")


# Запуск симуляции
if __name__ == "__main__":
    asyncio.run(run_simulation())

```

Данный фрагмент кода реализует полноценную систему управления задачами с использованием Python и LangChain, предназначенную для имитации агента менеджера проекта на базе большой языковой модели.

Система использует класс SuperSimpleTaskManager для эффективного управления задачами в памяти, применяя структуру словаря для быстрого извлечения данных. Каждая задача представлена моделью Pydantic Task, которая включает атрибуты: уникальный идентификатор, описательный текст, необязательный уровень приоритета (P0, P1, P2) и необязательного исполнителя. Использование памяти варьируется в зависимости от типа задач, количества работников и других факторов. Менеджер задач предоставляет методы создания, изменения и получения списка всех задач.

Агент взаимодействует с менеджером задач через определённый набор инструментов (Tools). Эти инструменты позволяют создавать новые задачи, назначать приоритеты, распределять задачи по персоналу и выводить список всех задач. Каждый инструмент инкапсулирован для взаимодействия с экземпляром SuperSimpleTaskManager. Модели Pydantic используются для определения требуемых аргументов инструментов, обеспечивая валидацию данных.

AgentExecutor настраивается с языковой моделью, набором инструментов и компонентом памяти диалога для сохранения контекста. Определяется специальный ChatPromptTemplate, направляющий поведение агента в его роли менеджера проекта. Промпт инструктирует агента начать с создания задачи, затем назначить приоритет и персонал, и завершить полным списком задач. В промпте заданы значения по умолчанию — приоритет P1 и «Worker A» — для случаев, когда информация отсутствует.

Рассматриваемый код также включает функцию асинхронной симуляции `run_simulation` для демонстрации рабочих возможностей агента. Симуляция выполняет два сценария: управление срочной задачей с назначенным персоналом и управление менее срочной задачей с минимальным набором входных данных. Действия и логические процессы агента выводятся в консоль благодаря параметру verbose=True в AgentExecutor.

# Краткий обзор

**Что:** AI-агенты в сложных окружениях сталкиваются с множеством потенциальных действий, конфликтующими целями и ограниченными ресурсами. Без чёткого метода определения следующего хода агенты рискуют стать неэффективными. Это может привести к значительным operational задержкам или полному провалу достижения главных целей. Ключевая задача — управлять этим overwhelming количеством выборов, чтобы агент действовал целенаправленно и логично.

**Почему:** Паттерн «Приоритизация» предоставляет стандартизированное решение, позволяя агентам ранжировать задачи и цели через установление чётких критериев: срочность, важность, зависимости и стоимость ресурсов. Агент оценивает каждое потенциальное действие по этим критериям для определения наиболее критического и своевременного курса действий. Эта агентная способность позволяет системе динамически адаптироваться к изменяющимся обстоятельствам и эффективно управлять ограниченными ресурсами. Фокусируясь на наивысших приоритетах, поведение агента становится более интеллектуальным, устойчивым и aligned со стратегическими целями.

**Правило:** Используйте паттерн «Приоритизация», когда агентная система должна автономно управлять множественными, часто конфликтующими задачами или целями при ограниченных ресурсах для эффективной работы в динамичном окружении.

**Визуальное резюме:**

![Паттерн приоритизации](../assets/Prioritization_Design_Pattern.png )

Рис. 1: Паттерн проектирования приоритизации

# Ключевые выводы

* Приоритизация позволяет AI-агентам эффективно функционировать в сложных многоаспектных окружениях.
* Агенты используют установленные критерии — срочность, важность, зависимости — для оценки и ранжирования задач.
* Динамическая переприоритизация позволяет агентам корректировать фокус в ответ на изменения в реальном времени.
* Приоритизация происходит на различных уровнях — от стратегических overarching целей до непосредственных тактических решений.
* Эффективная приоритизация ведёт к повышению производительности и улучшенной operational надёжности AI-агентов.

# Заключение

В заключение, паттерн приоритизации — краеугольный камень эффективного агентного AI, обеспечивающий системам способность навигировать в сложностях динамичных окружений с целью и интеллектом. Он позволяет агенту автономно оценивать множество конфликтующих задач и целей, принимая обоснованные решения о том, куда направить ограниченные ресурсы. Эта агентная способность выходит за рамки простого выполнения задач, позволяя системе действовать как проактивный стратегический принимающий решений. Взвешивая критерии срочности, важности и зависимостей, агент демонстрирует сложное рассуждение, близкое к человеческому.

Ключевая особенность этого агентного поведения — динамическая переприоритизация, которая даёт агенту автономию адаптировать фокус в реальном времени по мере изменения условий. Как показывает пример кода, агент интерпретирует неоднозначные запросы, автономно выбирает и использует подходящие инструменты и логически выстраивает последовательность действий для достижения целей. Эта способность self-manage свой рабочий процесс — то, что отличает истинную агентную систему от простого автоматизированного скрипта. В конечном счёте, освоение приоритизации фундаментально и критически важно для создания robust и интеллектуальных агентов, способных эффективно и надёжно работать в любом сложном сценарии реального мира, обеспечивая автономию, адаптивность и стратегическое мышление на уровне, приближённом к человеческому.

# Ссылки

1. Examining the Security of Artificial Intelligence in Project Management: A Case Study of AI-driven Project Scheduling and Resource Allocation in Information Systems Projects; [https://www.irejournals.com/paper-details/1706160](https://www.irejournals.com/paper-details/1706160)
2. AI-Driven Decision Support Systems in Agile Software Project Management: Enhancing Risk Mitigation and Resource Allocation; [https://www.mdpi.com/2079-8954/13/3/208](https://www.mdpi.com/2079-8954/13/3/208)
