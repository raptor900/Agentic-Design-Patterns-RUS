# Глава 5: Использование инструментов (Вызов функций)

## Обзор паттерна «Использование инструментов»

До сих пор мы обсуждали агентные паттерны, включающие оркестрацию взаимодействий между языковыми моделями и управление потоком информации внутри рабочего процесса агента (цепочка, маршрутизация, параллелизация, рефлексия). Однако для того чтобы агенты были действительно полезны и взаимодействовали с реальным миром или внешними системами, им необходима способность использовать **инструменты**.

Паттерн «Использование инструментов», часто реализуемый через механизм вызова функций (function calling), позволяет агенту взаимодействовать с внешними API, базами данных, сервисами или даже выполнять код. Он позволяет LLM, стоящей в основе агента, решать, когда и как использовать конкретную внешнюю функцию на основе запроса пользователя или текущего состояния задачи.

Процесс обычно включает:

1. **Определение инструмента:** Внешние функции или возможности определяются и описываются для LLM. Описание включает назначение функции, её имя и принимаемые параметры с типами.
2. **Решение LLM:** LLM получает запрос пользователя и доступные определения инструментов. На основе понимания запроса и инструментов LLM решает, нужно ли вызывать один или несколько инструментов.
3. **Генерация вызова функции:** Если LLM решает использовать инструмент, она генерирует структурированный вывод (обычно JSON-объект), указывающий имя инструмента и аргументы, извлечённые из запроса пользователя.
4. **Выполнение инструмента:** Агентный фреймворк или уровень оркестрации перехватывает этот структурированный вывод, идентифицирует запрошенный инструмент и выполняет фактическую внешнюю функцию с предоставленными аргументами.
5. **Наблюдение/Результат:** Вывод или результат выполнения инструмента возвращается агенту.
6. **Обработка LLM (необязательно, но типично):** LLM получает вывод инструмента как контекст и использует его для формулирования финального ответа или определения следующего шага (который может включать вызов другого инструмента, рефлексию или предоставление ответа).

Этот паттерн фундаментален, поскольку преодолевает ограничения обучающих данных LLM и позволяет получать актуальную информацию, выполнять вычисления, взаимодействовать с пользовательскими данными или запускать реальные действия. Вызов функций — технический механизм, соединяющий рассуждающие способности LLM с широким спектром внешних функций.

Хотя «вызов функций» хорошо описывает вызов конкретных предопределённых кодовых функций, полезно рассмотреть более широкое понятие «вызова инструментов». Инструментом может быть традиционная функция, но также — сложный API-эндпоинт, запрос к базе данных или инструкция, направленная другому специализированному агенту. Этот взгляд позволяет проектировать системы, где основной агент делегирует сложный анализ данных выделенному «агенту-аналитику» или запрашивает внешнюю базу знаний через её API.

Фреймворки вроде LangChain, LangGraph и Google ADK предоставляют надёжную поддержку определения инструментов и их интеграции в рабочие процессы, часто используя нативные возможности вызова функций современных LLM.

Использование инструментов — краеугольный паттерн для построения мощных, интерактивных и внешне осведомлённых агентов.

## Практические применения и сценарии использования

Паттерн «Использование инструментов» применим практически везде, где агенту нужно выйти за рамки генерации текста:

### 1. Получение информации из внешних источников

Доступ к данным реального времени, отсутствующим в обучающих данных LLM.

* **Сценарий:** Агент погоды.
  * **Инструмент:** API погоды, принимающий локацию и возвращающий текущие условия.
  * **Поток:** Пользователь спрашивает «Какая погода в Лондоне?», LLM идентифицирует потребность в инструменте погоды, вызывает его с «London», инструмент возвращает данные, LLM форматирует ответ.

### 2. Взаимодействие с базами данных и API

Выполнение запросов, обновлений и других операций со структурированными данными.

* **Сценарий:** E-commerce агент.
  * **Инструменты:** API-вызовы для проверки наличия товаров, статуса заказа, обработки платежей.
  * **Поток:** Пользователь спрашивает «Есть ли товар X в наличии?», LLM вызывает API инвентаризации, инструмент возвращает количество.

### 3. Выполнение вычислений и анализ данных

Использование внешних калькуляторов, библиотек анализа или статистических инструментов.

* **Сценарий:** Финансовый агент.
  * **Инструменты:** Функция калькулятора, API биржевых данных, инструмент электронных таблиц.
  * **Поток:** Пользователь спрашивает про цену акций и потенциальную прибыль, LLM вызывает API акций, затем калькулятор.

### 4. Отправка сообщений

Отправка писем, сообщений через внешние сервисы.

* **Сценарий:** Персональный ассистент.
  * **Инструмент:** API отправки email.
  * **Поток:** Пользователь: «Напиши письмо Джону о завтрашней встрече», LLM вызывает инструмент email с получателем, темой и текстом.

### 5. Выполнение кода

Запуск фрагментов кода в безопасной среде.

* **Сценарий:** Ассистент программиста.
  * **Инструмент:** Интерпретатор кода.
  * **Поток:** Пользователь предоставляет Python-фрагмент, LLM использует инструмент для запуска и анализа.

### 6. Управление другими системами и устройствами

Взаимодействие с умным домом, IoT-платформами.

* **Сценарий:** Агент умного дома.
  * **Инструмент:** API управления освещением.
  * **Поток:** «Выключи свет в гостиной» — LLM вызывает инструмент умного дома с командой и целевым устройством.

Использование инструментов превращает языковую модель из генератора текста в агента, способного воспринимать, рассуждать и действовать в цифровом или физическом мире (см. Рис. 1).

![Примеры использования инструментов агентом](../assets/Some_Examples_of_an_Agent_Using_Tool.png)

Рис. 1: Примеры использования инструментов агентом

## Практический пример кода (LangChain)

Реализация использования инструментов в LangChain — двухэтапный процесс. Сначала определяются инструменты (обычно через оборачивание существующих Python-функций). Затем они привязываются к языковой модели, давая ей возможность генерировать структурированные запросы к инструментам.

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

os.environ["GOOGLE_API_KEY"] = getpass.getpass("Enter your Google API key: ")
os.environ["OPENAI_API_KEY"] = getpass.getpass("Enter your OpenAI API key: ")

try:
    llm = ChatGoogleGenerativeAI(model="gemini-2.0-flash", temperature=0)
    print(f"Language model initialized: {llm.model}")
except Exception as e:
    print(f"Error initializing language model: {e}")
    llm = None


@langchain_tool
def search_information(query: str) -> str:
    """
    Provides factual information on a given topic. Use this tool to find answers to phrases
    like 'capital of France' or 'weather in London?'.
    """
    print(f"\n--- Tool Called: search_information with query: '{query}' ---")

    simulated_results = {
        "weather in london": "The weather in London is currently cloudy with a temperature of 15°C.",
        "capital of france": "The capital of France is Paris.",
        "population of earth": "The estimated population of Earth is around 8 billion people.",
        "tallest mountain": "Mount Everest is the tallest mountain above sea level.",
        "default": f"Simulated search result for '{query}': No specific information found.",
    }
    result = simulated_results.get(query.lower(), simulated_results["default"])
    print(f"--- TOOL RESULT: {result} ---")
    return result


tools = [search_information]

if llm:
    agent_prompt = ChatPromptTemplate.from_messages([
        ("system", "You are a helpful assistant."),
        ("human", "{input}"),
        ("placeholder", "{agent_scratchpad}"),
    ])

    agent = create_tool_calling_agent(llm, tools, agent_prompt)
    agent_executor = AgentExecutor(agent=agent, verbose=True, tools=tools)


async def run_agent_with_tool(query: str):
    print(f"\n--- Running Agent with Query: '{query}' ---")
    try:
        response = await agent_executor.ainvoke({"input": query})
        print("\n--- Final Agent Response ---")
        print(response["output"])
    except Exception as e:
        print(f"Error during agent execution: {e}")


async def main():
    tasks = [
        run_agent_with_tool("What is the capital of France?"),
        run_agent_with_tool("What's the weather like in London?"),
        run_agent_with_tool("Tell me something about dogs."),
    ]
    await asyncio.gather(*tasks)


nest_asyncio.apply()
asyncio.run(main())
```

Код настраивает агента с вызовом инструментов на LangChain и Google Gemini. Определяется инструмент `search_information`, имитирующий предоставление фактических ответов на конкретные запросы. ChatGoogleGenerativeAI инициализируется с возможностями вызова инструментов. ChatPromptTemplate задаёт взаимодействие. Функция `create_tool_calling_agent` объединяет модель, инструменты и промпт. AgentExecutor управляет выполнением. Запросы запускаются асинхронно через `ainvoke`.

## Практический пример кода (CrewAI)

```python
import os
from crewai import Agent, Task, Crew
from crewai.tools import tool
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')

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
        raise ValueError(f"Simulated price for ticker '{ticker.upper()}' not found.")

financial_analyst_agent = Agent(
    role='Senior Financial Analyst',
    goal='Analyze stock data using provided tools and report key prices.',
    backstory="You are an experienced financial analyst adept at using data sources to find stock information.",
    verbose=True,
    tools=[get_stock_price],
    allow_delegation=False,
)

analyze_aapl_task = Task(
    description=(
        "What is the current simulated stock price for Apple (ticker: AAPL)? "
        "Use the 'Stock Price Lookup Tool' to find it. "
        "If the ticker is not found, report that you were unable to retrieve the price."
    ),
    expected_output=(
        "A single, clear sentence stating the simulated stock price for AAPL. "
        "For example: 'The simulated stock price for AAPL is $178.15.' "
        "If the price cannot be found, state that clearly."
    ),
    agent=financial_analyst_agent,
)

financial_crew = Crew(
    agents=[financial_analyst_agent],
    tasks=[analyze_aapl_task],
    verbose=True
)

def main():
    if not os.environ.get("OPENAI_API_KEY"):
        print("ERROR: The OPENAI_API_KEY environment variable is not set.")
        return

    print("\n## Starting the Financial Crew...")
    result = financial_crew.kickoff()
    print("\nFinal Result:\n", result)

if __name__ == "__main__":
    main()
```

Этот код демонстрирует приложение Crew.ai для имитации финансового анализа. Определяется инструмент `get_stock_price`, имитирующий поиск цен акций для предопределённых тикеров. Создаётся агент-аналитик с ролью старшего финансового аналиста и инструментом. Задача `analyze_aapl_task` инструктирует найти цену AAPL. Crew оркестрирует выполнение.

## Практический пример кода (Google ADK)

Google ADK включает библиотеку нативно интегрированных инструментов.

**Google Search:**

```python
from google.adk.agents import Agent as ADKAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.tools import google_search
from google.genai import types
import nest_asyncio
import asyncio

APP_NAME = "Google Search Agent"
USER_ID = "user1234"
SESSION_ID = "1234"

root_agent = ADKAgent(
    name="basic_search_agent",
    model="gemini-2.0-flash-exp",
    description="Agent to answer questions using Google Search.",
    instruction="I can answer your questions by searching the internet. Just ask me anything!",
    tools=[google_search],
)

async def call_agent(query: str):
    session_service = InMemorySessionService()
    await session_service.create_session(
        app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID,
    )
    runner = Runner(agent=root_agent, app_name=APP_NAME, session_service=session_service)
    content = types.Content(role='user', parts=[types.Part(text=query)])
    events = runner.run(user_id=USER_ID, session_id=SESSION_ID, new_message=content)

    for event in events:
        if event.is_final_response() and event.content:
            if hasattr(event.content, "text") and event.content.text:
                final_response = event.content.text
            elif event.content.parts:
                final_response = "".join(part.text for part in event.content.parts if getattr(part, "text", None))
            else:
                final_response = ""
            print("Agent Response:", final_response)

nest_asyncio.apply()
asyncio.run(call_agent("what's the latest ai news?"))
```

Этот код создаёт агента Google ADK с инструментом Google Search. Агент отвечает на вопросы, выполняя веб-поиск. Определяется InMemorySessionService, Runner и функция `call_agent` для отправки запросов.

**Выполнение кода:**

Google ADK включает компонент для динамического выполнения кода. Инструмент `built_in_code_execution` предоставляет агенту песочницу Python-интерпретатора, позволяющую модели писать и запускать код для вычислительных задач, манипуляций со структурами данных и процедурных скриптов.

```python
from google.adk.agents import LlmAgent
from google.adk.runners import Runner
from google.adk.sessions import InMemorySessionService
from google.adk.code_executors import BuiltInCodeExecutor
from google.genai import types
import nest_asyncio
import asyncio

APP_NAME = "calculator"
USER_ID = "user1234"
SESSION_ID = "session_code_exec_async"

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

async def call_agent_async(query: str):
    session_service = InMemorySessionService()
    await session_service.create_session(app_name=APP_NAME, user_id=USER_ID, session_id=SESSION_ID)
    runner = Runner(agent=code_agent, app_name=APP_NAME, session_service=session_service)
    content = types.Content(role='user', parts=[types.Part(text=query)])
    print(f"\n--- Running Query: {query} ---")

    try:
        async for event in runner.run_async(user_id=USER_ID, session_id=SESSION_ID, new_message=content):
            if event.content and event.content.parts and event.is_final_response():
                for part in event.content.parts:
                    if getattr(part, "executable_code", None):
                        print(f"  Debug: Agent generated code:\n```python\n{part.executable_code.code}\n```")
                    elif getattr(part, "code_execution_result", None):
                        print(f"  Debug: Code Execution Result: {part.code_execution_result.outcome} - Output:\n{part.code_execution_result.output}")
                    elif getattr(part, "text", None) and not part.text.isspace():
                        print(f"  Text: '{part.text.strip()}'")
                text_parts = [part.text for part in event.content.parts if getattr(part, "text", None)]
                final_result = "".join(text_parts)
                print(f"==> Final Agent Response: {final_result}")
    except Exception as e:
        print(f"ERROR during agent run: {e}")

async def main():
    await call_agent_async("Calculate the value of (5 + 7) * 3")
    await call_agent_async("What is 10 factorial?")

try:
    nest_asyncio.apply()
    asyncio.run(main())
except RuntimeError as e:
    if "cannot be called from a running event loop" in str(e):
        print("Running in an existing event loop.")
    else:
        raise e
```

## Краткий обзор

**Что:** LLM — мощные генераторы текста, но фундаментально отключены от внешнего мира. Их знания статичны, и они не могут выполнять действия или получать данные в реальном времени.

**Почему:** Паттерн «Использование инструментов» решает эту проблему через описание доступных внешних функций в понятном LLM формате. На основе запроса агентная LLM генерирует структурированный вызов. Уровень оркестрации выполняет вызов, возвращает результат LLM, которая включает актуальную внешнюю информацию в финальный ответ.

**Когда использовать:** Когда агенту нужно выйти за рамки внутренних знаний LLM: для данных в реальном времени, доступа к проприетарной информации, точных вычислений, выполнения кода или действий в других системах.

**Визуальное резюме:**

![Паттерн «Использование инструментов»](../assets/Tool_Use_Design_Pattern.png)

Рис. 2: Паттерн «Использование инструментов»

## Ключевые выводы

* Использование инструментов (вызов функций) позволяет агентам взаимодействовать с внешними системами и получать динамическую информацию.
* Включает определение инструментов с чёткими описаниями и параметрами, понятными LLM.
* LLM решает, когда использовать инструмент, и генерирует структурированные вызовы.
* Агентные фреймворки выполняют фактические вызовы и возвращают результаты LLM.
* Необходимо для агентов, выполняющих реальные действия и предоставляющих актуальную информацию.
* LangChain упрощает определение инструментов через декоратор @tool и предоставляет `create_tool_calling_agent` и AgentExecutor.
* Google ADK имеет полезные предустановленные инструменты: Google Search, Code Execution и Vertex AI Search.

## Заключение

Паттерн «Использование инструментов» — критический архитектурный принцип для расширения функционального охвата LLM за пределы внутренней генерации текста. Фреймворки LangChain, Google ADK и Crew AI предоставляют структурированные абстракции для интеграции внешних инструментов, управляя процессом предоставления спецификаций инструментов модели и парсинга последующих запросов.

## Ссылки

1. Документация LangChain (Tools): [https://python.langchain.com/docs/integrations/tools/](https://python.langchain.com/docs/integrations/tools/)
2. Документация Google ADK (Tools): [https://google.github.io/adk-docs/tools/](https://google.github.io/adk-docs/tools/)
3. Документация OpenAI Function Calling: [https://platform.openai.com/docs/guides/function-calling](https://platform.openai.com/docs/guides/function-calling)
4. Документация CrewAI (Tools): [https://docs.crewai.com/concepts/tools](https://docs.crewai.com/concepts/tools)
