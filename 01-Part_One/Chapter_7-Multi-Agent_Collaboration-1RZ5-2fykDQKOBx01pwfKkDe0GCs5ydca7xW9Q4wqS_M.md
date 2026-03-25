# Глава 7: Многоагентное взаимодействие

Хотя монолитная архитектура агента может быть эффективна для чётко определённых задач, её возможности часто ограничены при работе со сложными междоменными задачами. Паттерн «Многоагентное взаимодействие» решает эти ограничения, структурируя систему как кооперативный ансамбль различных специализированных агентов. Этот подход основан на принципе декомпозиции задач, когда высокоуровневая цель разбивается на дискретные подзадачи, и каждой назначается агент с соответствующими инструментами и возможностями.

Например, сложный исследовательский запрос может быть декомпозирован между агентом-исследователем (поиск информации), агентом-аналитиком (статистическая обработка) и агентом-синтезатором (генерация финального отчёта). Эффективность такой системы определяется не только разделением труда, но и критически зависит от механизмов межагентной коммуникации — стандартизированного протокола и общей онтологии, позволяющих агентам обмениваться данными, делегировать подзадачи и координировать действия.

Распределённая архитектура даёт преимущества: модульность, масштабируемость, надёжность (отказ одного агента не обязательно приводит к общему сбою). Взаимодействие создаёт синергетический эффект, превосходящий возможности любого отдельного агента.

## Обзор паттерна «Многоагентное взаимодействие»

Паттерн предполагает проектирование систем, где несколько независимых или полуавтономных агентов работают совместно. У каждого — определённая роль, цели, свои инструменты или базы знаний. Сила паттерна — в синергии между агентами.

Взаимодействие может принимать различные формы:

* **Последовательная передача:** Один агент завершает задачу и передаёт результат другому (аналогично паттерну планирования, но с разными агентами).
* **Параллельная обработка:** Несколько агентов работают над разными частями задачи одновременно, результаты затем объединяются.
* **Дискуссия и консенсус:** Агенты с различными перспективами обсуждают варианты для принятия более обоснованных решений.
* **Иерархическая структура:** Агент-менеджер динамически делегирует задачи рабочим агентам на основе их инструментов и синтезирует результаты. Каждый агент может обрабатывать свою группу инструментов.
* **Команды экспертов:** Специализированные агенты (исследователь, писатель, редактор) сотрудничают для создания сложного результата.
* **Критик-рецензент:** Первые агенты генерируют вывод (планы, черновики), вторые критически оценивают его на соответствие политикам, качество, корректность. Автор дорабочает на основе обратной связи.

Многоагентная система (см. Рис. 1) включает определение ролей и ответственности, установку каналов коммуникации и формулирование протокола взаимодействия.

![Многоагентная система](../assets/Multi_Agent_System.png)

Рис. 1: Пример многоагентной системы

Фреймворки вроде Crew AI и Google ADK разработаны для этого парадигмы, предоставляя структуры для спецификации агентов, задач и их интерактивных процедур. Подход эффективен для задач, требующих разнообразных специальных знаний, включающих множество этапов или использующих преимущества параллельной обработки.

## Практические применения и сценарии использования

* **Сложные исследования и анализ:** Команда агентов: поиск в академических базах, обобщение находок, выявление трендов, синтез отчёта.
* **Разработка ПО:** Агенты: аналитик требований, генератор кода, тестировщик, автор документации. Передают выводы друг другу для сборки и проверки компонентов.
* **Генерация креативного контента:** Маркетинговая кампания: агент-исследователь рынка, агент-копирайтер, агент-дизайнер (с генерацией изображений), агент планирования соцсетей.
* **Финансовый анализ:** Агенты специализируются на получении биржевых данных, анализе тональности новостей, техническом анализе и рекомендациях.
* **Эскалация клиентской поддержки:** Фронтлайн-агент → специализированный агент (техэксперт, биллинг) при сложных вопросах.
* **Оптимизация цепочки поставок:** Агенты представляют узлы цепочки (поставщики, производители, дистрибьюторы) и оптимизируют запасы и логистику.
* **Анализ и устранение неполадок сети:** Несколько агентов совместно диагностируют проблемы и предлагают оптимальные действия.

## Структуры взаимосвязей и коммуникации

Как показано на Рис. 2, существует спектр моделей взаимодействия:

### 1. Один агент

Агент действует автономно, без прямого взаимодействия с другими. Прост в реализации, но ограничен собственными возможностями.

### 2. Сеть

Агенты взаимодействуют децентрализованно, «точка-точка». Устойчивость к отказам, но сложно управлять коммуникацией в крупной сети.

### 3. Супервизор

Выделенный агент-супервизор координирует подчинённых. Центральный узел коммуникации, распределения задач, разрешения конфликтов. Чёткие линии авторитета, но — единая точка отказа.

### 4. Супервизор как инструмент

Расширение концепции: супервизор предоставляет ресурсы, guidance или аналитическую поддержку, а не директивное управление.

### 5. Иерархия

Многоуровневая структура: супервизоры высшего уровня → низшего → операционные агенты. Подходит для сложных задач, разложимых на подзадачи.

### 6. Пользовательская

Максимальная гибкость: уникальные структуры, комбинирующие элементы вышеуказанных моделей или полностью новые.

![Агенты взаимодействуют различными способами](../assets/Agents_Communicate_and_Interact_in_Various_Ways.png)

Рис. 2: Агенты взаимодействуют различными способами.

Выбор модели — критическое архитектурное решение, зависящее от сложности задачи, количества агентов, требуемого уровня автономии и допустимых накладных расходов на коммуникацию.

## Практический пример кода (Crew AI)

```python
import os
from dotenv import load_dotenv
from crewai import Agent, Task, Crew, Process
from langchain_google_genai import ChatGoogleGenerativeAI


def setup_environment():
    """Loads environment variables and checks for the required API key."""
    load_dotenv()
    if not os.getenv("GOOGLE_API_KEY"):
        raise ValueError("GOOGLE_API_KEY not found. Please set it in your .env file.")


def main():
    setup_environment()

    llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")

    researcher = Agent(
        role='Senior Research Analyst',
        goal='Find and summarize the latest trends in AI.',
        backstory="You are an experienced research analyst with a knack for identifying key trends and synthesizing information.",
        verbose=True,
        allow_delegation=False,
    )

    writer = Agent(
        role='Technical Content Writer',
        goal='Write a clear and engaging blog post based on research findings.',
        backstory="You are a skilled writer who can translate complex technical topics into accessible content.",
        verbose=True,
        allow_delegation=False,
    )

    research_task = Task(
        description="Research the top 3 emerging trends in Artificial Intelligence in 2024-2025. Focus on practical applications and potential impact.",
        expected_output="A detailed summary of the top 3 AI trends, including key points and sources.",
        agent=researcher,
    )

    writing_task = Task(
        description="Write a 500-word blog post based on the research findings. The post should be engaging and easy for a general audience to understand.",
        expected_output="A complete 500-word blog post about the latest AI trends.",
        agent=writer,
        context=[research_task],
    )

    blog_creation_crew = Crew(
        agents=[researcher, writer],
        tasks=[research_task, writing_task],
        process=Process.sequential,
        llm=llm,
        verbose=2,
    )

    print("## Running the blog creation crew with Gemini 2.0 Flash... ##")
    try:
        result = blog_creation_crew.kickoff()
        print("\n------------------\n")
        print("## Crew Final Output ##")
        print(result)
    except Exception as e:
        print(f"An unexpected error occurred: {e}")


if __name__ == "__main__":
    main()
```

Код определяет двух агентов CrewAI: исследователя трендов ИИ и писателя для блога. Задача исследования — первая в последовательности, задача написания зависит от её результата. Crew обрабатывает задачи последовательно.

## Практический пример кода (Google ADK)

### Иерархическая структура

```python
from typing import AsyncGenerator
from google.adk.agents import LlmAgent, BaseAgent
from google.adk.agents.invocation_context import InvocationContext
from google.adk.events import Event


class TaskExecutor(BaseAgent):
    """Специализированный агент с пользовательским, не-LLM поведением."""
    name: str = "TaskExecutor"
    description: str = "Executes a predefined task."

    async def _run_async_impl(self, context: InvocationContext) -> AsyncGenerator[Event, None]:
        yield Event(author=self.name, content="Task finished successfully.")


greeter = LlmAgent(
    name="Greeter",
    model="gemini-2.0-flash-exp",
    instruction="You are a friendly greeter.",
)

task_doer = TaskExecutor()

coordinator = LlmAgent(
    name="Coordinator",
    model="gemini-2.0-flash-exp",
    description="A coordinator that can greet users and execute tasks.",
    instruction="When asked to greet, delegate to the Greeter. When asked to perform a task, delegate to the TaskExecutor.",
    sub_agents=[greeter, task_doer],
)

assert greeter.parent_agent == coordinator
assert task_doer.parent_agent == coordinator
print("Agent hierarchy created successfully.")
```

### Циклический агент (LoopAgent)

```python
import asyncio
from typing import AsyncGenerator
from google.adk.agents import LoopAgent, LlmAgent, BaseAgent
from google.adk.events import Event, EventActions
from google.adk.agents.invocation_context import InvocationContext


class ConditionChecker(BaseAgent):
    """Проверяет завершение процесса в состоянии сессии."""
    name: str = "ConditionChecker"
    description: str = "Checks if a process is complete and signals the loop to stop."

    async def _run_async_impl(self, context: InvocationContext) -> AsyncGenerator[Event, None]:
        status = context.session.state.get("status", "pending")
        is_done = status == "completed"
        if is_done:
            yield Event(author=self.name, actions=EventActions(escalate=True))
        else:
            yield Event(author=self.name, content="Condition not met, continuing loop.")


process_step = LlmAgent(
    name="ProcessingStep",
    model="gemini-2.0-flash-exp",
    instruction="You are a step in a longer process. Perform your task. If you are the final step, update session state by setting 'status' to 'completed'.",
)

poller = LoopAgent(
    name="StatusPoller",
    max_iterations=10,
    sub_agents=[process_step, ConditionChecker()],
)
```

### Последовательный агент (SequentialAgent)

```python
from google.adk.agents import SequentialAgent, Agent

step1 = Agent(
    name="Step1_Fetch",
    output_key="data",
)

step2 = Agent(
    name="Step2_Process",
    instruction="Analyze the information found in state['data'] and provide a summary.",
)

pipeline = SequentialAgent(
    name="MyPipeline",
    sub_agents=[step1, step2],
)
```

### Параллельный агент (ParallelAgent)

```python
from google.adk.agents import Agent, ParallelAgent

weather_fetcher = Agent(
    name="weather_fetcher",
    model="gemini-2.0-flash-exp",
    instruction="Fetch the weather for the given location and return only the weather report.",
    output_key="weather_data",
)

news_fetcher = Agent(
    name="news_fetcher",
    model="gemini-2.0-flash-exp",
    instruction="Fetch the top news story for the given topic and return only that story.",
    output_key="news_data",
)

data_gatherer = ParallelAgent(
    name="data_gatherer",
    sub_agents=[weather_fetcher, news_fetcher],
)
```

### Агент как инструмент (Agent as a Tool)

```python
from google.adk.agents import LlmAgent
from google.adk.tools import agent_tool
from google.genai import types


def generate_image(prompt: str) -> dict:
    """Генерирует изображение по текстовому описанию."""
    print(f"TOOL: Generating image for prompt: '{prompt}'")
    mock_image_bytes = b"mock_image_data_for_a_cat_wearing_a_hat"
    return {
        "status": "success",
        "image_bytes": mock_image_bytes,
        "mime_type": "image/png",
    }


image_generator_agent = LlmAgent(
    name="ImageGen",
    model="gemini-2.0-flash",
    description="Generates an image based on a detailed text prompt.",
    instruction="You are an image generation specialist. Take the user's request and use the `generate_image` tool. After the tool returns the image bytes, you MUST output the image.",
    tools=[generate_image],
)

image_tool = agent_tool.AgentTool(
    agent=image_generator_agent,
    description="Use this tool to generate an image. The input should be a descriptive prompt.",
)

artist_agent = LlmAgent(
    name="Artist",
    model="gemini-2.0-flash",
    instruction="You are a creative artist. First, invent a creative prompt for an image. Then, use the `ImageGen` tool to generate the image.",
    tools=[image_tool],
)
```

## Краткий обзор

**Что:** Сложные задачи превышают возможности монолитного LLM-агента. Единственный агент может не обладать разнообразными навыками или инструментами.

**Почему:** Многоагентное взаимодействие создаёт систему кооперирующихся агентов. Каждый решает свою подзадачу с нужными инструментами. Взаимодействие через протоколы коммуникации: последовательная передача, параллельные потоки, иерархическое делегирование.

**Когда использовать:** Когда задача слишком сложна для одного агента и разложима на подзадачи. Идеально для исследований, разработки ПО, создания контента.

**Визуальное резюме:**

![Многоагентный дизайн-паттерн](../assets/Multi_Agent_Design_Pattern.png)

Рис. 3: Многоагентный дизайн-паттерн

## Ключевые выводы

* Многоагентное взаимодействие — несколько агентов работают совместно для достижения общей цели.
* Использует специализированные роли, распределённые задачи и межагентную коммуникацию.
* Формы: последовательная передача, параллельная обработка, дискуссия, иерархические структуры.
* Идеально для сложных задач, требующих разнообразной экспертизы или множества этапов.

## Заключение

Эта глава рассмотрела паттерн «Многоагентное взаимодействие», демонстрируя преимущества оркестрации нескольких специализированных агентов. Мы изучили различные модели взаимодействия и их критическую роль в решении сложных задач.

## Ссылки

1. Multi-Agent Collaboration Mechanisms: A Survey of LLMs: [https://arxiv.org/abs/2501.06322](https://arxiv.org/abs/2501.06322)
2. Multi-Agent System — The Power of Collaboration: [https://aravindakumar.medium.com/introducing-multi-agent-frameworks-the-power-of-collaboration-e9db31bba1b6](https://aravindakumar.medium.com/introducing-multi-agent-frameworks-the-power-of-collaboration-e9db31bba1b6)
