# Глава 7: Многоагентное взаимодействие

Хотя монолитная архитектура агента может быть эффективна для чётко определённых задач, её возможности часто ограничены при работе со сложными междоменными задачами. Для преодоления этого Межагентная Коммуникация (A2A) позволяет различным AI-агентам, потенциально построенным на разных фреймворках, эффективно сотрудничать. Это сотрудничество включает бесшовную координацию, делегирование задач и обмен информацией.

## Обзор паттерна «Многоагентное взаимодействие»

Паттерн «Многоагентное взаимодействие» предполагает проектирование систем, где несколько независимых или полуавтономных агентов работают совместно для достижения общей цели. Каждый агент обычно имеет определённую роль, цели и доступ к различным инструментам или базам знаний.

Взаимодействие может принимать различные формы:

* **Последовательная передача:** Один агент завершает задачу и передаёт вывод другому.
* **Параллельная обработка:** Несколько агентов работают над разными частями одновременно.
* **Дебаты и консенсус:** Агенты с различными перспективами обсуждают варианты.
* **Иерархические структуры:** Агент-менеджер динамически делегирует задачи.
* **Команды экспертов:** Специализированные агенты (исследователь, писатель, редактор) сотрудничают.
* **Критик-Рецензент:** Первые агенты генерируют, вторые критически оценивают.

Многоагентная система (см. Рис. 1) включает определение ролей, каналов коммуникации и протоколов взаимодействия.

![Многоагентная система](../assets/Multi_Agent_System.png)

Рис. 1: Пример многоагентной системы

## Практические применения и сценарии использования

* **Сложные исследования:** Команда агентов: поиск, обобщение, анализ, синтез.
* **Разработка ПО:** Агенты: аналитик, генератор кода, тестировщик, автор документации.
* **Креативный контент:** Маркетинговая кампания: исследование рынка, копирайтинг, дизайн.
* **Финансовый анализ:** Агенты: биржевые данные, тональность, технический анализ, рекомендации.
* **Клиентская поддержка:** Фронтлайн → специалист при сложных вопросах.
* **Оптимизация цепочки поставок:** Агенты представляют узлы и оптимизируют логистику.

## Многоагентное взаимодействие: исследование взаимосвязей и коммуникационных структур

Понимание того, как агенты взаимодействуют и общаются, фундаментально для проектирования эффективных многоагентных систем. Как показано на Рис. 2, существует спектр моделей взаимодействия, от простейшей до сложных пользовательских фреймворков.

### 1. Один агент

На базовом уровне «один агент» работает автономно без прямого взаимодействия. Прост в реализации, но ограничен собственными возможностями.

### 2. Сеть

«Сеть» — несколько агентов взаимодействуют децентрализованно. Устойчивость к отказам, но сложно управлять коммуникацией в крупных сетях.

### 3. Супервизор

«Супервизор» — выделенный агент координирует подчинённых. Чёткие линии авторитета, но — единая точка отказа.

### 4. Супервизор как инструмент

«Супервизор как инструмент» — расширение: супервизор предоставляет ресурсы и guidance вместо директивного управления.

### 5. Иерархия

«Иерархия» — многоуровневая структура с супервизорами разных уровней. Подходит для сложных задач.

### 6. Пользовательская

«Пользовательская» — максимальная гибкость: уникальные структуры, комбинирующие элементы вышеуказанных моделей.

![Агенты взаимодействуют различными способами](../assets/Agents_Communicate_and_Interact_in_Various_Ways.png)

Рис. 2: Агенты взаимодействуют различными способами.

## Практический пример кода (CrewAI)

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
    """
    Initializes and runs the AI crew for content creation using the latest Gemini model.
    """
    setup_environment()

    # Define the language model to use.
    # Updated to a model from the Gemini 2.0 series for better performance and features.
    # For cutting-edge (preview) capabilities, you could use "gemini-2.5-flash".
    llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash")

    # Define Agents with specific roles and goals
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

    # Define Tasks for the agents
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

    # Create the Crew
    blog_creation_crew = Crew(
        agents=[researcher, writer],
        tasks=[research_task, writing_task],
        process=Process.sequential,
        llm=llm,
        verbose=2,  # Set verbosity for detailed crew execution logs
    )

    # Execute the Crew
    print("## Running the blog creation crew with Gemini 2.0 Flash... ##")
    try:
        result = blog_creation_crew.kickoff()
        print("\n------------------\n")
        print("## Crew Final Output ##")
        print(result)
    except Exception as e:
        print(f"\nAn unexpected error occurred: {e}")


if __name__ == "__main__":
    main()
```

Этот код определяет многоагентную систему CrewAI для генерации поста блога о трендах ИИ. Два агента: исследователь трендов и писатель. Задача исследования — первая, задача написания зависит от результата.

## Практический пример кода (Google ADK)

Теперь рассмотрим примеры в Google ADK с фокусом на иерархическую, параллельную и последовательную координацию.

```python
from typing import AsyncGenerator

from google.adk.agents import LlmAgent, BaseAgent
from google.adk.agents.invocation_context import InvocationContext
from google.adk.events import Event


# Correctly implement a custom agent by extending BaseAgent
class TaskExecutor(BaseAgent):
    """A specialized agent with custom, non-LLM behavior."""
    name: str = "TaskExecutor"
    description: str = "Executes a predefined task."

    async def _run_async_impl(self, context: InvocationContext) -> AsyncGenerator[Event, None]:
        """Custom implementation logic for the task."""
        # This is where your custom logic would go.
        # For this example, we'll just yield a simple event.
        yield Event(author=self.name, content="Task finished successfully.")


# Define individual agents with proper initialization
# LlmAgent requires a model to be specified.
greeter = LlmAgent(
    name="Greeter",
    model="gemini-2.0-flash-exp",
    instruction="You are a friendly greeter.",
)

# Instantiate our concrete custom agent
task_doer = TaskExecutor()

# Create a parent agent and assign its sub-agents
# The parent agent's description and instructions should guide its delegation logic.
coordinator = LlmAgent(
    name="Coordinator",
    model="gemini-2.0-flash-exp",
    description="A coordinator that can greet users and execute tasks.",
    instruction="When asked to greet, delegate to the Greeter. When asked to perform a task, delegate to the TaskExecutor.",
    sub_agents=[
        greeter,
        task_doer,
    ],
)

# The ADK framework automatically establishes the parent-child relationships.
# These assertions will pass if checked after initialization.
assert greeter.parent_agent == coordinator
assert task_doer.parent_agent == coordinator

print("Agent hierarchy created successfully.")
```

Этот фрагмент демонстрирует использование LoopAgent для итеративных workflow.

```python
import asyncio
from typing import AsyncGenerator

from google.adk.agents import LoopAgent, LlmAgent, BaseAgent
from google.adk.events import Event, EventActions
from google.adk.agents.invocation_context import InvocationContext


# Best Practice: Define custom agents as complete, self-describing classes.
class ConditionChecker(BaseAgent):
    """A custom agent that checks for a 'completed' status in the session state."""
    name: str = "ConditionChecker"
    description: str = "Checks if a process is complete and signals the loop to stop."

    async def _run_async_impl(
        self, context: InvocationContext
    ) -> AsyncGenerator[Event, None]:
        """Checks state and yields an event to either continue or stop the loop."""
        status = context.session.state.get("status", "pending")
        is_done = status == "completed"

        if is_done:
            # Escalate to terminate the loop when the condition is met.
            yield Event(author=self.name, actions=EventActions(escalate=True))
        else:
            # Yield a simple event to continue the loop.
            yield Event(author=self.name, content="Condition not met, continuing loop.")


# Correction: The LlmAgent must have a model and clear instructions.
process_step = LlmAgent(
    name="ProcessingStep",
    model="gemini-2.0-flash-exp",
    instruction=(
        "You are a step in a longer process. Perform your task. "
        "If you are the final step, update session state by setting 'status' to 'completed'."
    ),
)


# The LoopAgent orchestrates the workflow.
poller = LoopAgent(
    name="StatusPoller",
    max_iterations=10,
    sub_agents=[
        process_step,
        ConditionChecker(),  # Instantiating the well-defined custom agent.
    ],
)

# This poller will now execute 'process_step'
# and then 'ConditionChecker' repeatedly until the status is 'completed'
# or 10 iterations have passed.
```

Этот фрагмент демонстрирует SequentialAgent — линейные рабочие процессы.

```python
from google.adk.agents import SequentialAgent, Agent


# This agent's output will be saved to session.state["data"]
step1 = Agent(
    name="Step1_Fetch",
    output_key="data",
)

# This agent will use the data from the previous step.
# We instruct it on how to find and use this data.
step2 = Agent(
    name="Step2_Process",
    instruction="Analyze the information found in state['data'] and provide a summary.",
)

pipeline = SequentialAgent(
    name="MyPipeline",
    sub_agents=[step1, step2],
)

# When the pipeline is run with an initial input, Step1 will execute,
# its response will be stored in session.state["data"], and then
# Step2 will execute, using the information from the state as instructed.
```

Следующий фрагмент демонстрирует ParallelAgent — параллельное выполнение подагентов.

```python
from google.adk.agents import Agent, ParallelAgent


# It's better to define the fetching logic as tools for the agents.
# For simplicity in this example, we'll embed the logic in the agent's instruction.
# In a real-world scenario, you would use tools.

# Define the individual agents that will run in parallel
weather_fetcher = Agent(
    name="weather_fetcher",
    model="gemini-2.0-flash-exp",
    instruction="Fetch the weather for the given location and return only the weather report.",
    output_key="weather_data",  # The result will be stored in session.state["weather_data"]
)

news_fetcher = Agent(
    name="news_fetcher",
    model="gemini-2.0-flash-exp",
    instruction="Fetch the top news story for the given topic and return only that story.",
    output_key="news_data",  # The result will be stored in session.state["news_data"]
)

# Create the ParallelAgent to orchestrate the sub-agents
data_gatherer = ParallelAgent(
    name="data_gatherer",
    sub_agents=[
        weather_fetcher,
        news_fetcher,
    ],
)
```

Этот фрагмент демонстрирует паттерн «Агент как инструмент» (Agent as a Tool).

```python
from google.adk.agents import LlmAgent
from google.adk.tools import agent_tool
from google.genai import types


# 1. A simple function tool for the core capability.
# This follows the best practice of separating actions from reasoning.
def generate_image(prompt: str) -> dict:
    """
    Generates an image based on a textual prompt.

    Args:
        prompt: A detailed description of the image to generate.

    Returns:
        A dictionary with the status and the generated image bytes.
    """
    print(f"TOOL: Generating image for prompt: '{prompt}'")
    # In a real implementation, this would call an image generation API.
    # For this example, we return mock image data.
    mock_image_bytes = b"mock_image_data_for_a_cat_wearing_a_hat"
    return {
        "status": "success",
        # The tool returns the raw bytes, the agent will handle the Part creation.
        "image_bytes": mock_image_bytes,
        "mime_type": "image/png",
    }


# 2. Refactor the ImageGeneratorAgent into an LlmAgent.
# It now correctly uses the input passed to it.
image_generator_agent = LlmAgent(
    name="ImageGen",
    model="gemini-2.0-flash",
    description="Generates an image based on a detailed text prompt.",
    instruction=(
        "You are an image generation specialist. Your task is to take the user's request "
        "and use the `generate_image` tool to create the image. "
        "The user's entire request should be used as the 'prompt' argument for the tool. "
        "After the tool returns the image bytes, you MUST output the image."
    ),
    tools=[generate_image],
)


# 3. Wrap the corrected agent in an AgentTool.
# The description here is what the parent agent sees.
image_tool = agent_tool.AgentTool(
    agent=image_generator_agent,
    description="Use this tool to generate an image. The input should be a descriptive prompt of the desired image.",
)


# 4. The parent agent remains unchanged. Its logic was correct.
artist_agent = LlmAgent(
    name="Artist",
    model="gemini-2.0-flash",
    instruction=(
        "You are a creative artist. First, invent a creative and descriptive prompt for an image. "
        "Then, use the `ImageGen` tool to generate the image using your prompt."
    ),
    tools=[image_tool],
)
```

## Краткий обзор

**Что:** Отдельные AI-агенты struggle со сложными задачами. Без стандартизированного языка создание многоагентных систем затруднено.

**Почему:** A2A протокол — открытый HTTP-стандарт для интероперабельности. Agent Card описывает возможности. Механизмы: синхронные, асинхронные и стриминговые взаимодействия.

**Когда использования:** При оркестрации двух или более агентов, особенно на разных фреймворках.

**Визуальное резюме:**

![Паттерн многоагентного дизайна](../assets/Multi_Agent_Design_Pattern.png)

Рис. 2: Многоагентный дизайн-паттерн

## Ключевые выводы

* Многоагентное взаимодействие — несколько агентов работают совместно.
* Использует специализированные роли, распределённые задачи, межагентную коммуникацию.
* Формы: последовательная передача, параллельная обработка, дискуссия, иерархия.
* Идеально для задач с разнообразной экспертизой или множеством этапов.

## Заключение

Эта глава рассмотрела паттерн «Многоагентное взаимодействие». Мы изучили различные модели и их роль в решении сложных задач. Понимание агентного сотрудничества ведёт к изучению их взаимодействия с внешней средой.

## Ссылки

1. Multi-Agent Collaboration Mechanisms: [https://arxiv.org/abs/2501.06322](https://arxiv.org/abs/2501.06322)
2. Multi-Agent System — The Power of Collaboration: [https://aravindakumar.medium.com/introducing-multi-agent-frameworks-the-power-of-collaboration-e9db31bba1b6](https://aravindakumar.medium.com/introducing-multi-agent-frameworks-the-power-of-collaboration-e9db31bba1b6)
