# Глава 2: Маршрутизация

## Обзор паттерна «Маршрутизация»

Хотя последовательная обработка через цепочку промптов является базовой техникой выполнения детерминированных, линейных рабочих процессов с языковыми моделями, её применимость ограничена в сценариях, требующих адаптивных ответов. Реальные агентные системы часто должны выбирать между несколькими возможными действиями на основе контингентных факторов: состояния среды, пользовательского ввода или результата предыдущей операции. Эта способность к динамическому принятию решений, управляющая потоком управления к различным специализированным функциям, инструментам или подпроцессам, реализуется через механизм, известный как маршрутизация (routing).

Маршрутизация вносит условную логику в операционную систему агента, обеспечивая переход от фиксированного пути выполнения к модели, где агент динамически оценивает конкретные критерии для выбора из набора возможных последующих действий. Это позволяет системе работать более гибко и контекстно-ориентировано.

Например, агент для обработки клиентских обращений, оснащённый функцией маршрутизации, сначала классифицирует входящий запрос для определения намерения пользователя. На основе классификации он направляет запрос к специализированному агенту для прямого ответа, к инструменту извлечения данных из базы для информации об аккаунте или к процедуре эскалации для сложных вопросов. Таким образом, более продвинутый агент с маршрутизацией может:

1. Проанализировать запрос пользователя.
2. **Направить** запрос на основе его *намерения*:
   * Если намерение — «проверить статус заказа», направить к подагенту или цепочке инструментов, взаимодействующей с базой заказов.
   * Если намерение — «информация о продукте», направить к подагенту или цепочке поиска по каталогу продуктов.
   * Если намерение — «техподдержка», направить к другой цепочке, обращающейся к руководствам по устранению неполадок или к человеку.
   * Если намерение неясно, направить к подагенту уточнения или цепочке промптов.

Базовый компонент паттерна маршрутизации — механизм, выполняющий оценку и направляющий поток. Этот механизм может быть реализован несколькими способами:

* **Маршрутизация на основе LLM:** Саму языковую модель можно использовать для анализа входа и вывода конкретного идентификатора или инструкции.
* **Маршрутизация на основе эмбеддингов:** Входной запрос преобразуется в векторное представление, которое сравнивается с эмбеддингами маршрутов. Полезно для семантической маршрутизации.
* **Правиловая маршрутизация:** Предопределённые правила (if-else) на основе ключевых слов. Быстрее и детерминированнее, но менее гибко.
* **Маршрутизация на основе ML-модели:** Дискриминативная модель, обученная на размеченных данных. Логика маршрутизации закодирована в обученных весах.

Механизмы маршрутизации могут применяться на различных этапах операционного цикла агента. Фреймворки LangChain, LangGraph и Google ADK предоставляют явные конструкции для определения и управления такой условной логикой.

Реализация маршрутизации позволяет системе выйти за рамки детерминированной последовательной обработки, способствуя разработке более адаптивных потоков выполнения.

## Практические применения и сценарии использования

Паттерн маршрутизации — критический механизм управления в дизайне адаптивных агентных систем, позволяющий им динамически изменять путь выполнения в ответ на переменные входы и внутренние состояния.

В **человеко-компьютерном взаимодействии** маршрутизация интерпретирует намерение пользователя и определяет наиболее подходящее последующее действие.

В **автоматизированных конвейерах обработки данных** маршрутизация выступает функцией классификации и распределения входящих данных.

В **сложных системах со специализированными инструментами** маршрутизация действует как диспетчер высокого уровня, назначающий задачи наиболее подходящему агенту.

В конечном счёте, маршрутизация обеспечивает способность к логическому арбитражу, необходимую для создания функционально разнообразных и контекстно-ориентированных систем.

## Практический пример кода (LangChain)

Реализация использования инструментов в LangChain — двухэтапный процесс. Сначала определяются инструменты, затем они привязываются к языковой модели.

```bash
pip install langchain langgraph google-cloud-aiplatform langchain-google-genai google-adk deprecated pydantic
```

Также необходимо настроить среду с ключом API выбранной языковой модели.

```python
# Copyright (c) 2025 Marco Fago
# https://www.linkedin.com/in/marco-fago/
#
# This code is licensed under the MIT License.
# See the LICENSE file in the repository for the full license text.

from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnablePassthrough, RunnableBranch


# --- Configuration ---
# Ensure your API key environment variable is set (e.g., GOOGLE_API_KEY)
try:
    llm = ChatGoogleGenerativeAI(model="gemini-2.5-flash", temperature=0)
    print(f"Language model initialized: {llm.model}")
except Exception as e:
    print(f"Error initializing language model: {e}")
    llm = None


# --- Define Simulated Sub-Agent Handlers (equivalent to ADK sub_agents) ---
def booking_handler(request: str) -> str:
    """Simulates the Booking Agent handling a request."""
    print("\n--- DELEGATING TO BOOKING HANDLER ---")
    return f"Booking Handler processed request: '{request}'. Result: Simulated booking action."


def info_handler(request: str) -> str:
    """Simulates the Info Agent handling a request."""
    print("\n--- DELEGATING TO INFO HANDLER ---")
    return f"Info Handler processed request: '{request}'. Result: Simulated information retrieval."


def unclear_handler(request: str) -> str:
    """Handles requests that couldn't be delegated."""
    print("\n--- HANDLING UNCLEAR REQUEST ---")
    return f"Coordinator could not delegate request: '{request}'. Please clarify."


# --- Define Coordinator Router Chain (equivalent to ADK coordinator's instruction) ---
# This chain decides which handler to delegate to.
coordinator_router_prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        """Analyze the user's request and determine which specialist handler should process it.
        - If the request is related to booking flights or hotels,
           output 'booker'.
        - For all other general information questions, output 'info'.
        - If the request is unclear or doesn't fit either category,
           output 'unclear'.
        ONLY output one word: 'booker', 'info', or 'unclear'."""
    ),
    ("user", "{request}")
])

if llm:
    coordinator_router_chain = coordinator_router_prompt | llm | StrOutputParser()


# --- Define the Delegation Logic (equivalent to ADK's Auto-Flow based on sub_agents) ---
# Use RunnableBranch to route based on the router chain's output.

# Define the branches for the RunnableBranch
branches = {
    "booker": RunnablePassthrough.assign(
        output=lambda x: booking_handler(x['request']['request'])
    ),
    "info": RunnablePassthrough.assign(
        output=lambda x: info_handler(x['request']['request'])
    ),
    "unclear": RunnablePassthrough.assign(
        output=lambda x: unclear_handler(x['request']['request'])
    ),
}

# Create the RunnableBranch. It takes the output of the router chain
# and routes the original input ('request') to the corresponding handler.
delegation_branch = RunnableBranch(
    (lambda x: x['decision'].strip() == 'booker', branches["booker"]),  # Added .strip()
    (lambda x: x['decision'].strip() == 'info', branches["info"]),      # Added .strip()
    branches["unclear"]  # Default branch for 'unclear' or any other output
)

# Combine the router chain and the delegation branch into a single runnable
# The router chain's output ('decision') is passed along with the original input ('request')
# to the delegation_branch.
coordinator_agent = {
    "decision": coordinator_router_chain,
    "request": RunnablePassthrough()
} | delegation_branch | (lambda x: x['output'])  # Extract the final output


# --- Example Usage ---
def main():
    if not llm:
        print("\nSkipping execution due to LLM initialization failure.")
        return

    print("--- Running with a booking request ---")
    request_a = "Book me a flight to London."
    result_a = coordinator_agent.invoke({"request": request_a})
    print(f"Final Result A: {result_a}")

    print("\n--- Running with an info request ---")
    request_b = "What is the capital of Italy?"
    result_b = coordinator_agent.invoke({"request": request_b})
    print(f"Final Result B: {result_b}")

    print("\n--- Running with an unclear request ---")
    request_c = "Tell me about quantum physics."
    result_c = coordinator_agent.invoke({"request": request_c})
    print(f"Final Result C: {result_c}")


if __name__ == "__main__":
    main()
```

Как упоминалось, этот код на Python строит простую агентную систему с использованием LangChain и Google Generative AI. Он определяет три имитированных обработчика подагентов: `booking_handler`, `info_handler` и `unclear_handler`. Базовый компонент — `coordinator_router_chain`, использующий ChatPromptTemplate для категоризации запросов. Результат используется RunnableBranch для делегирования. Код демонстрирует три примера запросов, показывая маршрутизацию разных входов.

## Практический пример кода (Google ADK)

Agent Development Kit (ADK) — фреймворк для разработки агентных систем, предоставляющий структурированную среду для определения возможностей и поведения агента. В отличие от архитектур на основе явных вычислительных графов, маршрутизация в парадигме ADK обычно реализуется через определение дискретного набора «инструментов», представляющих функции агента. Выбор подходящего инструмента в ответ на запрос пользователя управляется внутренней логикой фреймворка, использующей underlying модель для сопоставления намерения с правильным обработчиком.

Следующий код демонстрирует пример приложения ADK с использованием библиотеки Google ADK. Он настраивает агента-«Координатора», маршрутизирующего запросы пользователей к специализированным подагентам («Booker» для бронирований и «Info» для общей информации) на основе определённых инструкций.

```python
# Copyright (c) 2025 Marco Fago
#
# This code is licensed under the MIT License.
# See the LICENSE file in the repository for the full license text.

import uuid
from typing import Dict, Any, Optional

from google.adk.agents import Agent
from google.adk.runners import InMemoryRunner
from google.adk.tools import FunctionTool
from google.genai import types
from google.adk.events import Event


# --- Define Tool Functions ---
# These functions simulate the actions of the specialist agents.
def booking_handler(request: str) -> str:
    """
    Handles booking requests for flights and hotels.

    Args:
        request: The user's request for a booking.

    Returns:
        A confirmation message that the booking was handled.
    """
    print("-------------------------- Booking Handler Called ----------------------------")
    return f"Booking action for '{request}' has been simulated."


def info_handler(request: str) -> str:
    """
    Handles general information requests.

    Args:
        request: The user's question.

    Returns:
        A message indicating the information request was handled.
    """
    print("-------------------------- Info Handler Called ----------------------------")
    return f"Information request for '{request}'. Result: Simulated information retrieval."


def unclear_handler(request: str) -> str:
    """Handles requests that couldn't be delegated."""
    return f"Coordinator could not delegate request: '{request}'. Please clarify."


# --- Create Tools from Functions ---
booking_tool = FunctionTool(booking_handler)
info_tool = FunctionTool(info_handler)

# Define specialized sub-agents equipped with their respective tools
booking_agent = Agent(
    name="Booker",
    model="gemini-2.0-flash",
    description="A specialized agent that handles all flight "
                "and hotel booking requests by calling the booking tool.",
    tools=[booking_tool],
)

info_agent = Agent(
    name="Info",
    model="gemini-2.0-flash",
    description="A specialized agent that provides general information "
                "and answers user questions by calling the info tool.",
    tools=[info_tool],
)

# Define the parent agent with explicit delegation instructions
coordinator = Agent(
    name="Coordinator",
    model="gemini-2.0-flash",
    instruction=(
        "You are the main coordinator. Your only task is to analyze "
        "incoming user requests "
        "and delegate them to the appropriate specialist agent. Do not try to answer the user directly.\n"
        "- For any requests related to booking flights or hotels, delegate to the 'Booker' agent.\n"
        "- For all other general information questions, delegate to the 'Info' agent."
    ),
    description="A coordinator that routes user requests to the correct specialist agent.",
    # The presence of sub_agents enables LLM-driven delegation (Auto-Flow) by default.
    sub_agents=[booking_agent, info_agent],
)


# --- Execution Logic ---
async def run_coordinator(runner: InMemoryRunner, request: str):
    """Runs the coordinator agent with a given request and delegates."""
    print(f"\n--- Running Coordinator with request: '{request}' ---")
    final_result = ""
    try:
        user_id = "user_123"
        session_id = str(uuid.uuid4())

        await runner.session_service.create_session(
            app_name=runner.app_name,
            user_id=user_id,
            session_id=session_id,
        )

        for event in runner.run(
            user_id=user_id,
            session_id=session_id,
            new_message=types.Content(
                role='user',
                parts=[types.Part(text=request)],
            ),
        ):
            if event.is_final_response() and event.content:
                # Try to get text directly from event.content to avoid iterating parts
                if hasattr(event.content, 'text') and event.content.text:
                    final_result = event.content.text
                elif event.content.parts:
                    # Fallback: Iterate through parts and extract text (might trigger warning)
                    text_parts = [part.text for part in event.content.parts if getattr(part, "text", None)]
                    final_result = "".join(text_parts)
                # Assuming the loop should break after the final response
                break

        print(f"Coordinator Final Response: {final_result}")
        return final_result

    except Exception as e:
        print(f"An error occurred while processing your request: {e}")
        return f"An error occurred while processing your request: {e}"


async def main():
    """Main function to run the ADK example."""
    print("--- Google ADK Routing Example (ADK Auto-Flow Style) ---")
    print("Note: This requires Google ADK installed and authenticated.")

    runner = InMemoryRunner(coordinator)

    # Example Usage
    result_a = await run_coordinator(runner, "Book me a hotel in Paris.")
    print(f"Final Output A: {result_a}")

    result_b = await run_coordinator(runner, "What is the highest mountain in the world?")
    print(f"Final Output B: {result_b}")

    result_c = await run_coordinator(runner, "Tell me a random fact.")  # Should go to Info
    print(f"Final Output C: {result_c}")

    result_d = await run_coordinator(runner, "Find flights to Tokyo next month.")  # Should go to Booker
    print(f"Final Output D: {result_d}")


if __name__ == "__main__":
    import nest_asyncio

    nest_asyncio.apply()
    await main()
```

## Краткий обзор

**Что:** Агентные системы должны реагировать на широкий спектр входов и ситуаций, которые не могут быть обработаны одним линейным процессом. Без механизма выбора правильного инструмента или подпроцесса для конкретной задачи система остаётся жёсткой и неадаптивной.

**Почему:** Маршрутизация стандартизированно решает эту проблему, внося условную логику в операционную систему агента. Она позволяет сначала анализировать входящий запрос для определения намерения, а затем динамически направлять поток управления к наиболее подходящему инструменту. Решение может приниматься через LLM, предопределённые правила или семантическое сходство.

**Когда использовать:** Когда агент должен выбирать между несколькими рабочими процессами, инструментами или подагентами на основе пользовательского ввода или текущего состояния. Необходимо для приложений, классифицирующих входящие запросы.

**Визуальное резюме:**

![Паттерн маршрутизатора с использованием LLM](../assets/Router_Pattern_Using_LLM_as_a_Router.png)

Рис. 1: Паттерн маршрутизатора с использованием LLM

## Ключевые выводы

* Маршрутизация позволяет агентам принимать динамические решения о следующем шаге рабочего процесса на основе условий.
* Она позволяет обрабатывать разнообразные входы и адаптировать поведение, выходя за рамки линейного выполнения.
* Логика маршрутизации может реализовываться через LLM, правиловые системы или семантическое сходство эмбеддингов.
* Фреймворки LangGraph и Google ADK предоставляют структурированные способы определения и управления маршрутизацией.

## Заключение

Паттерн маршрутизации — критический шаг к построению по-настоящему динамичных и отзывчивых агентных систем. Мы рассмотрели применение в различных доменах: от чатботов клиентской поддержки до сложных конвейеров обработки данных. Примеры кода с LangChain и Google ADK продемонстрировали два разных, но эффективных подхода. LangGraph обеспечивает визуальное и явное определение состояний и переходов, Google ADK фокусируется на определении набора возможностей и полагается на способность фреймворка маршрутизировать запросы. Освоение маршрутизации необходимо для создания универсальных и надёжных агентных приложений.

## Ссылки

1. Документация LangGraph: [https://www.langchain.com/](https://www.langchain.com/)
2. Документация Google ADK: [https://google.github.io/adk-docs/](https://google.github.io/adk-docs/)
