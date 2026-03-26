# Глава 5: Использование инструментов (Вызов функций)

## Обзор паттерна «Использование инструментов»

В предыдущих главах мы обсуждали агентные паттерны, primarily включающие оркестрацию взаимодействий между языковыми моделями и управление потоком информации (цепочки, маршрутизация, параллелизация, рефлексия). Однако для того чтобы агенты были действительно полезны и взаимодействовали с реальным миром, им необходима способность использовать **инструменты**.

Паттерн «Использование инструментов», часто реализуемый через механизм вызова функций (function calling), позволяет агенту взаимодействовать с внешними API, базами данных, сервисами или выполнять код. Он позволяет LLM решать, когда и как использовать конкретную внешнюю функцию на основе запроса пользователя.

Процесс обычно включает:

1. **Определение инструмента:** Внешние функции определяются и описываются для LLM с указанием назначения, имени и параметров.
2. **Решение LLM:** LLM получает запрос пользователя и доступные инструменты. На основе понимания решает, нужно ли вызывать инструмент.
3. **Генерация вызова функции:** LLM генерирует структурированный вывод (обычно JSON) с именем инструмента и аргументами.
4. **Выполнение инструмента:** Агентный фреймворк перехватывает вывод, выполняет функцию.
5. **Наблюдение/Результат:** Результат возвращается агенту.
6. **Обработка LLM:** LLM получает вывод инструмента как контекст и формулирует финальный ответ.

Этот паттерн фундаментален — он преодолевает ограничения обучающих данных LLM и позволяет получать актуальную информацию, выполнять вычисления, взаимодействовать с данными.

Хотя «вызов функций» describes вызов конкретных функций, полезно рассмотреть более широкое понятие «вызова инструментов»: инструментом может быть API-эндпоинт, запрос к БД или инструкция другому агенту.

Фреймворки LangChain, LangGraph и Google ADK предоставляют надёжную поддержку определения и интеграции инструментов.

Использование инструментов — краеугольный паттерн для построения мощных интерактивных агентов.

## Практические применения и сценарии использования

### 1. Получение информации из внешних источников

Доступ к данным в реальном времени или информации, отсутствующей в обучающих данных LLM.

* **Сценарий:** Агент погоды.
  * **Инструмент:** API погоды, принимающий локацию и возвращающий текущие условия.
  * **Поток:** Пользователь спрашивает «Какая погода в Лондоне?», LLM вызывает инструмент с «London», получает данные, форматирует ответ.

### 2. Взаимодействие с базами данных и API

Выполнение запросов, обновлений и других операций со структурированными данными.

* **Сценарий:** E-commerce агент.
  * **Инструменты:** API-вызовы для проверки наличия, статуса заказа, обработки платежей.
  * **Поток:** Пользователь: «Есть ли товар X в наличии?», LLM вызывает API инвентаризации.

### 3. Выполнение вычислений и анализ данных

Использование внешних калькуляторов и инструментов анализа.

* **Сценарий:** Финансовый агент.
  * **Инструменты:** Калькулятор, API биржевых данных.
  * **Поток:** Пользователь спрашивает про цену акций и прибыль, LLM вызывает API и калькулятор.

### 4. Отправка сообщений

Отправка писем, сообщений через внешние сервисы.

* **Сценарий:** Персональный ассистент.
  * **Инструмент:** Email API.
  * **Поток:** «Напиши письмо Джону о встрече» — LLM извлекает данные и вызывает инструмент.

### 5. Выполнение кода

Запуск фрагментов кода в безопасной среде.

* **Сценарий:** Ассистент программиста.
  * **Инструмент:** Интерпретатор кода.
  * **Поток:** Пользователь предоставляет Python-фрагмент, LLM запускает и анализирует.

### 6. Управление другими системами или устройствами

Взаимодействие с умным домом, IoT-платформами.

* **Сценарий:** Агент умного дома.
  * **Инструмент:** API управления освещением.
  * **Поток:** «Выключи свет в гостиной» — LLM вызывает инструмент.

Использование инструментов превращает языковую модель из генератора текста в агента, способного воспринимать, рассуждать и действовать.

![Примеры использования инструментов](../assets/Some_Examples_of_an_Agent_Using_Tool.png)

Рис. 1: Примеры использования инструментов агентом

## Практический пример кода (LangChain)

Реализация использования инструментов в LangChain — двухэтапный процесс: определение инструментов (обычно через оборачивание Python-функций) и привязка к языковой модели.

```python
import os
import getpass
import asyncio
import nest_asyncio
from typing import List
from dotenv import load_dotenv
import logging

from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.tools import tool as langchain_tool
from langchain.agents import create_tool_calling_agent, AgentExecutor


# UNCOMMENT
# Prompt the user securely and set API keys as environment variables
os.environ["GOOGLE_API_KEY"] = getpass.getpass("Enter your Google API key: ")
os.environ["OPENAI_API_KEY"] = getpass.getpass("Enter your OpenAI API key: ")

try:
    # A model with function/tool calling capabilities is required.
    llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash", temperature=0)
    print(f"✅ Language model initialized: {llm.model}")
except Exception as e:
    print(f"🛑 Error initializing language model: {e}")
    llm = None


# --- Define a Tool ---
@langchain_tool
def search_information(query: str) -> str:
    """
    Provides factual information on a given topic. Use this tool to find answers to phrases
    like 'capital of France' or 'weather in London?'.
    """
    print(f"\n--- 🛠️ Tool Called: search_information with query: '{query}' ---")

    # Simulate a search tool with a dictionary of predefined results.
    simulated_results = {
        "weather in london": "The weather in London is currently cloudy with a temperature of 15°C.",
        "capital of france": "The capital of France is Paris.",
        "population of earth": "The estimated population of Earth is around 8 billion people.",
        "tallest mountain": "Mount Everest is the tallest mountain above sea level.",
        "default": f"Simulated search result for '{query}': No specific information found, but the topic seems interesting.",
    }
    result = simulated_results.get(query.lower(), simulated_results["default"])
    print(f"--- TOOL RESULT: {result} ---")
    return result


tools = [search_information]


# --- Create a Tool-Calling Agent ---
if llm:
    # This prompt template requires an `agent_scratchpad` placeholder for the agent's internal steps.
    agent_prompt = ChatPromptTemplate.from_messages([
        ("system", "You are a helpful assistant."),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}"),
    ])

    # Create the agent, binding the LLM, tools, and prompt together.
    agent = create_tool_calling_agent(llm, tools, agent_prompt)

    # AgentExecutor is the runtime that invokes the agent and executes the chosen tools.
    # The 'tools' argument is not needed here as they are already bound to the agent.
    agent_executor = AgentExecutor(agent=agent, verbose=True, tools=tools)


async def run_agent_with_tool(query: str):
    """Invokes the agent executor with a query and prints the final response."""
    print(f"\n--- 🏃 Running Agent with Query: '{query}' ---")
    try:
        response = await agent_executor.ainvoke({"input": query})
        print("\n--- ✅ Final Agent Response ---")
        print(response["output"])
    except Exception as e:
        print(f"\n🛑 An error occurred during agent execution: {e}")


async def main():
    """Runs all agent queries concurrently."""
    tasks = [
        run_agent_with_tool("What is the capital of France?"),
        run_agent_with_tool("What's the weather like in London?"),
        run_agent_with_tool("Tell me something about dogs."),  # Should trigger the default tool response
    ]
    await asyncio.gather(*tasks)


nest_asyncio.apply()
asyncio.run(main())

```

Как упоминалось, этот код на Python строит простую агентную систему с использованием LangChain и Google Generative AI. Определяется инструмент `search_information`, имитирующий предоставление фактической информации. Имеет предопределённые ответы и ответ по умолчанию. Инициализируется ChatGoogleGenerativeAI с возможностями вызова инструментов. Создаётся ChatPromptTemplate, затем `create_tool_calling_agent` и AgentExecutor. Запросы проверяют как конкретные, так и ответы по умолчанию.

# Практический пример кода (CrewAI)

```python
# pip install crewai langchain-openai

import os
from crewai import Agent, Task, Crew
from crewai.tools import tool
import logging


# --- Best Practice: Configure Logging ---
# A basic logging setup helps in debugging and tracking the crew's execution.
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')


# --- Set up your API Key ---
# For production, it's recommended to use a more secure method for key management
# like environment variables loaded at runtime or a secret manager.
#
# Set the environment variable for your chosen LLM provider (e.g., OPENAI_API_KEY)
# os.environ["OPENAI_API_KEY"] = "YOUR_API_KEY"
# os.environ["OPENAI_MODEL_NAME"] = "gpt-4o"


# --- 1. Refactored Tool: Returns Clean Data ---
# The tool now returns raw data (a float) or raises a standard Python error.
# This makes it more reusable and forces the agent to handle outcomes properly.
@tool("Stock Price Lookup Tool")
def get_stock_price(ticker: str) -> float:
    """
    Fetches the latest simulated stock price for a given stock ticker symbol.
    Returns the price as a float. Raises a ValueError if the ticker is not found.
    """
    logging.info(f"Tool Call: get_stock_price for ticker '{ticker}'")
    simulated_prices = {
        "AAPL": 178.15,
        "GOOGL": 1750.30,
        "MSFT": 425.50,
    }
    price = simulated_prices.get(ticker.upper())
    if price is not None:
        return price
    else:
        # Raising a specific error is better than returning a string.
        # The agent is equipped to handle exceptions and can decide on the next action.
        raise ValueError(f"Simulated price for ticker '{ticker.upper()}' not found.")


# --- 2. Define the Agent ---
# The agent definition remains the same, but it will now leverage the improved tool.
financial_analyst_agent = Agent(
    role='Senior Financial Analyst',
    goal='Analyze stock data using provided tools and report key prices.',
    backstory="You are an experienced financial analyst adept at using data sources to find stock information. You provide clear, direct answers.",
    verbose=True,
    tools=[get_stock_price],
    # Allowing delegation can be useful, but is not necessary for this simple task.
    allow_delegation=False,
)


# --- 3. Refined Task: Clearer Instructions and Error Handling ---
# The task description is more specific and guides the agent on how to react
# to both successful data retrieval and potential errors.
analyze_aapl_task = Task(
    description=(
        "What is the current simulated stock price for Apple (ticker: AAPL)? "
        "Use the 'Stock Price Lookup Tool' to find it. "
        "If the ticker is not found, you must report that you were unable to retrieve the price."
    ),
    expected_output=(
        "A single, clear sentence stating the simulated stock price for AAPL. "
        "For example: 'The simulated stock price for AAPL is $178.15.' "
        "If the price cannot be found, state that clearly."
    ),
    agent=financial_analyst_agent,
)


# --- 4. Formulate the Crew ---
# The crew orchestrates how the agent and task work together.
financial_crew = Crew(
    agents=[financial_analyst_agent],
    tasks=[analyze_aapl_task],
    verbose=True  # Set to False for less detailed logs in production
)


# --- 5. Run the Crew within a Main Execution Block ---
# Using a __name__ == "__main__": block is a standard Python best practice.
def main():
    """Main function to run the crew."""
    # Check for API key before starting to avoid runtime errors.
    if not os.environ.get("OPENAI_API_KEY"):
        print("ERROR: The OPENAI_API_KEY environment variable is not set.")
        print("Please set it before running the script.")
        return

    print("\n## Starting the Financial Crew...")
    print("---------------------------------")

    # The kickoff method starts the execution.
    result = financial_crew.kickoff()

    print("\n---------------------------------")
    print("## Crew execution finished.")
    print("\nFinal Result:\n", result)


if __name__ == "__main__":
    main()
```

Этот код демонстрирует простое приложение Crew.ai для имитации финансового анализа. Определяется инструмент `get_stock_price`, имитирующий поиск цен акций. Создаётся агент-аналитик с ролью старшего финансового аналиста и инструментом. Задача инструктирует найти цену AAPL. Crew оркестрирует выполнение.

## Практический пример кода (Google ADK)

Google ADK включает библиотеку нативно интегрированных инструментов.

**Google Search:** Пример инструмента, предоставляющего прямой интерфейс к Google Search.

```python
from google.adk.agents import Agent as ADKAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.tools import google_search
from google.genai import types
import nest_asyncio
import asyncio


# Define variables required for Session setup and Agent execution
APP_NAME = "Google Search Agent"
USER_ID = "user1234"
SESSION_ID = "1234"


# Define Agent with access to search tool
root_agent = ADKAgent(
    name="basic_search_agent",
    model="gemini-2.0-flash-exp",
    description="Agent to answer questions using Google Search.",
    instruction="I can answer your questions by searching the internet. Just ask me anything!",
    tools=[google_search],  # Google Search is a pre-built tool to perform Google searches.
)


# Agent Interaction
async def call_agent(query: str):
    """
    Helper function to call the agent with a query.
    """
    # Session and Runner
    session_service = InMemorySessionService()
    await session_service.create_session(
        app_name=APP_NAME,
        user_id=USER_ID,
        session_id=SESSION_ID,
    )

    runner = Runner(agent=root_agent, app_name=APP_NAME, session_service=session_service)

    content = types.Content(role='user', parts=[types.Part(text=query)])
    events = runner.run(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    for event in events:
        if event.is_final_response() and event.content:
            # Safely extract text from the final response
            if hasattr(event.content, "text") and event.content.text:
                final_response = event.content.text
            elif event.content.parts:
                final_response = "".join(
                    part.text for part in event.content.parts if getattr(part, "text", None)
                )
            else:
                final_response = ""
            print("Agent Response:", final_response)


nest_asyncio.apply()
asyncio.run(call_agent("what's the latest ai news?"))
```

Этот код создаёт агента Google ADK с инструментом Google Search. Агент отвечает на вопросы, выполняя веб-поиск. Определяется InMemorySessionService, Runner и функция `call_agent`.

**Выполнение кода:** Google ADK включает компонент для динамического выполнения кода. Инструмент `built_in_code_execution` предоставляет агенту песочницу Python-интерпретатора.

```python
import os
import getpass
import asyncio
import nest_asyncio
from typing import List
from dotenv import load_dotenv
import logging

from google.adk.agents import Agent as ADKAgent, LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.tools import google_search
from google.adk.code_executors import BuiltInCodeExecutor
from google.genai import types


# Define variables required for Session setup and Agent execution
APP_NAME = "calculator"
USER_ID = "user1234"
SESSION_ID = "session_code_exec_async"


# Agent Definition
code_agent = LlmAgent(
    name="calculator_agent",
    model="gemini-2.0-flash",
    code_executor=BuiltInCodeExecutor(),
    instruction="""You are a calculator agent.
    When given a mathematical expression, write and execute Python code to calculate the result.
    Return only the final numerical result as plain text, without markdown or code blocks.
    """,
    description="Executes Python code to perform calculations.",
)


# Agent Interaction (Async)
async def call_agent_async(query: str):
    # Session and Runner
    session_service = InMemorySessionService()
    await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)

    runner = Runner(agent=code_agent, app_name=APP_NAME, session_service=session_service)

    content = types.Content(role='user', parts=[types.Part(text=query)])
    print(f"\n--- Running Query: {query} ---")

    try:
        # Use run_async
        async for event in runner.run_async(user_id=USER_ID, session_id=SESSION_ID, new_message=content):
            print(f"Event ID: {event.id}, Author: {event.author}")

            if event.content and event.content.parts and event.is_final_response():
                for part in event.content.parts:  # Iterate through all parts
                    if getattr(part, "executable_code", None):
                        # Access the actual code string via .code
                        print(f"  Debug: Agent generated code:\n```

В этом фрагменте внутри блока кода — вывод отладочной информации.

```")
                    elif getattr(part, "code_execution_result", None):
                        # Access outcome and output correctly
                        print(
                            "  Debug: Code Execution Result: "
                            f"{part.code_execution_result.outcome} - Output:\n{part.code_execution_result.output}"
                        )
                    elif getattr(part, "text", None) and not part.text.isspace():
                        # Also print any text parts found in any event for debugging
                        print(f"  Text: '{part.text.strip()}'")

                # --- Check for final response AFTER specific parts ---
                text_parts = [part.text for part in event.content.parts if getattr(part, "text", None)]
                final_result = "".join(text_parts)
                print(f"==> Final Agent Response: {final_result}")

    except Exception as e:
        print(f"ERROR during agent run: {e}")

    print("-" * 30)


# Main async function to run the examples
async def main():
    await call_agent_async("Calculate the value of (5 + 7) * 3")
    await call_agent_async("What is 10 factorial?")


# Execute the main async function
try:
    nest_asyncio.apply()
    asyncio.run(main())
except RuntimeError as e:
    # Handle specific error when running asyncio.run in an already running loop (like Jupyter/Colab)
    if "cannot be called from a running event loop" in str(e):
        print("\nRunning in an existing event loop (like Colab/Jupyter).")
        print("Please run `await main()` in a notebook cell instead.")
        # If in an interactive environment like a notebook, you might need to run:
        # await main()
    else:
        raise e  # Re-raise other runtime errors
```

Этот скрипт использует Google Agent Development Kit (ADK) для создания агента, решающего математические задачи путём написания и выполнения Python-кода. Определяется LlmAgent с инструкцией выступать калькулятором и инструментом `built_in_code_execution`. Функция `call_agent_async` отправляет запрос пользователя к раннеру и обрабатывает события. Внутри неё асинхронный цикл выводит сгенерированный код и результат выполнения, carefully различая промежуточные шаги и финальный ответ с числом.

```python
import asyncio
import os

from google.genai import types
from google.adk import agents
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService


# --- Configuration ---
# Ensure you have set your GOOGLE_API_KEY and DATASTORE_ID environment variables
# For example:
# os.environ["GOOGLE_API_KEY"] = "YOUR_API_KEY"
# os.environ["DATASTORE_ID"] = "YOUR_DATASTORE_ID"
DATASTORE_ID = os.environ.get("DATASTORE_ID")


# --- Application Constants ---
APP_NAME = "vsearch_app"
USER_ID = "user_123"  # Example User ID
SESSION_ID = "session_456"  # Example Session ID


# --- Agent Definition (Updated with the newer model from the guide) ---
vsearch_agent = agents.VSearchAgent(
    name="q2_strategy_vsearch_agent",
    description="Answers questions about Q2 strategy documents using Vertex AI Search.",
    model="gemini-2.0-flash-exp",  # Updated model based on the guide's examples
    datastore_id=DATASTORE_ID,
    model_parameters={"temperature": 0.0},
)


# --- Runner and Session Initialization ---
runner = Runner(
    agent=vsearch_agent,
    app_name=APP_NAME,
    session_service=InMemorySessionService(),
)


# --- Agent Invocation Logic ---
async def call_vsearch_agent_async(query: str):
    """Initializes a session and streams the agent's response."""
    print(f"User: {query}")
    print("Agent: ", end="", flush=True)
    try:
        # Construct the message content correctly
        content = types.Content(role='user', parts=[types.Part(text=query)])

        # Process events as they arrive from the asynchronous runner
        async for event in runner.run_async(
            user_id=USER_ID,
            session_id=SESSION_ID,
            new_message=content,
        ):
            # For token-by-token streaming of the response text
            if hasattr(event, "content_part_delta") and event.content_part_delta:
                print(event.content_part_delta.text, end="", flush=True)

            # Process the final response and its associated metadata
            if event.is_final_response():
                print()  # Newline after the streaming response
                if getattr(event, "grounding_metadata", None):
                    print(
                        f"  (Source Attributions: "
                        f"{len(event.grounding_metadata.grounding_attributions)} sources found)"
                    )
                else:
                    print("  (No grounding metadata found)")
                print("-" * 30)
    except Exception as e:
        print(f"\nAn error occurred: {e}")
        print("Please ensure your datastore ID is correct and that the service account has the necessary permissions.")
        print("-" * 30)


# --- Run Example ---
async def run_vsearch_example():
    # Replace with a question relevant to YOUR datastore content
    await call_vsearch_agent_async("Summarize the main points about the Q2 strategy document.")
    await call_vsearch_agent_async("What safety procedures are mentioned for lab X?")


# --- Execution ---
if __name__ == "__main__":
    if not DATASTORE_ID:
        print("Error: DATASTORE_ID environment variable is not set.")
    else:
        try:
            asyncio.run(run_vsearch_example())
        except RuntimeError as e:
            # This handles cases where asyncio.run is called in an environment
            # that already has a running event loop (like a Jupyter notebook).
            if "cannot be called from a running event loop" in str(e):
                print("Skipping execution in a running event loop. Please run this script directly.")
            else:
                raise e
```

В целом, этот код provides базовый фреймворк для построения conversation-ориентированного агента, способного отслеживать историю и использовать инструменты для предоставления информированных ответов. Он включает несколько классов: LLMInteractionMonitor для отслеживания использования токенов, ToolExecutor для имитации выполнения инструментов, LLMJudge для оценки ответов и CodeVerifier для проверки кода. Каждый класс отвечает за конкретный аспект мониторинга и оценки производительности агента.

## Краткий обзор

**Что:** LLM — мощные генераторы текста, но отключены от внешнего мира. Без стандартизированной коммуникации каждая интеграция — кастомная и непереиспользуемая.

**Почему:** Паттерн «Использование инструментов» подключает LLM к внешним системам через описание доступных функций. На основе запроса агентная LLM генерирует структурированный вызов.

**Когда использования:** Когда агенту нужно выйти за рамки внутренних знаний LLM: для данных в реальном времени, доступа к проприетарной информации, вычислений, выполнения кода.

**Визуальное резюме:**

![Паттерн использования инструментов](../assets/Tool_Use_Design_Pattern.png)

Рис. 2: Паттерн «Использование инструментов»

## Ключевые выводы

* Использование инструментов позволяет агентам взаимодействовать с внешними системами.
* Включает определение инструментов с чёткими описаниями и параметрами.
* LLM решает, когда использовать, и генерирует структурированные вызовы.
* Фреймворки выполняют фактические вызовы и возвращают результаты.
* LangChain: декоратор @tool, `create_tool_calling_agent`, AgentExecutor.
* Google ADK: предустановленные инструменты (Google Search, Code Execution, Vertex AI Search).

## Заключение

Паттерн «Использование инструментов» — критический архитектурный принцип для расширения функционального охвата LLM. Фреймворки LangChain, Google ADK и Crew AI предоставляют абстракции для интеграции, управляя предоставлением спецификаций и парсингом запросов.

## Ссылки

1. Документация LangChain (Tools): [https://python.langchain.com/docs/integrations/tools/](https://python.langchain.com/docs/integrations/tools/)
2. Документация Google ADK (Tools): [https://google.github.io/adk-docs/tools/](https://google.github.io/adk-docs/tools/)
3. Документация OpenAI Function Calling: [https://platform.openai.com/docs/guides/function-calling](https://platform.openai.com/docs/guides/function-calling)
4. Документация CrewAI (Tools): [https://docs.crewai.com/concepts/tools](https://docs.crewai.com/concepts/tools)
