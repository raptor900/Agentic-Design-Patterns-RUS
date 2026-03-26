# Глава 15: Межагентная коммуникация (A2A)

Отдельные AI-агенты часто сталкиваются с ограничениями при решении сложных многоаспектных задач. Для преодоления этого межагентная коммуникация (A2A) позволяет различным AI-агентам, потенциально построенным на разных фреймворках, эффективно сотрудничать.

## Обзор паттерна «Межагентная коммуникация»

Протокол Agent2Agent (A2A) — открытый стандарт для коммуникации и сотрудничества между различными фреймворками AI-агентов. Поддерживается Atlassian, Box, LangChain, MongoDB, Salesforce, SAP, ServiceNow. Microsoft планирует интегрировать A2A в Azure AI Foundry и Copilot Studio.

## Базовые концепции A2A

**Основные субъекты:**

* **User:** Инициирует запросы.
* **A2A Client:** Приложение или AI-агент, действующий от имени пользователя.
* **A2A Server:** AI-агент с HTTP-эндпоинтом, обрабатывающим запросы. Работает как «непрозрачная» система.

**Agent Card:** Цифровая идентичность агента — JSON-файл с информацией для взаимодействия и автоматического обнаружения: идентичность, URL, версия, возможности (streaming, push-уведомления), навыки, режимы ввода/вывода, требования аутентификации.

```json
{
    "name": "WeatherBot",
    "description": "Provides accurate weather forecasts and historical data.",
    "url": "http://weather-service.example.com/a2a",
    "version": "1.0.0",
    "capabilities": {
        "streaming": true,
        "pushNotifications": false,
        "stateTransitionHistory": true
    },
    "authentication": {
        "schemes": [
            "apiKey"
        ]
    },
    "defaultInputModes": [
        "text"
    ],
    "defaultOutputModes": [
        "text"
    ],
    "skills": [
        {
            "id": "get_current_weather",
            "name": "Get Current Weather",
            "description": "Retrieve real-time weather for any location.",
            "inputModes": [
                "text"
            ],
            "outputModes": [
                "text"
            ],
            "examples": [
                "What's the weather in Paris?",
                "Current conditions in Tokyo"
            ],
            "tags": [
                "weather",
                "current",
                "real-time"
            ]
        },
        {
            "id": "get_forecast",
            "name": "Get Forecast",
            "description": "Get 5-day weather predictions.",
            "inputModes": [
                "text"
            ],
            "outputModes": [
                "text"
            ],
            "examples": [
                "5-day forecast for New York",
                "Will it rain in London this weekend?"
            ],
            "tags": [
                "weather",
                "forecast",
                "prediction"
            ]
        }
    ]
}
```

**Обнаружение агентов:** Стратегии:

* **Well-Known URI:** Стандартизированный путь (`/.well-known/agent.json`). Широкая доступность.
* **Курируемые реестры:** Централизованный каталог.
* **Прямая конфигурация:** Для closely coupled систем.

**Коммуникация и задачи:** Асинхронные задачи — базовые единицы работы. Каждая получает уникальный ID и проходит состояния. Коммуникация через Message с атрибутами и частями. Результаты — artifacts. Протокол — JSON-RPC 2.0 через HTTP(S).

**Механизмы взаимодействия:**

* **Синхронный запрос/ответ:** Клиент отправляет и ждёт полный ответ.
* **Асинхронный поллинг:** Сервер подтверждает «working», клиент опрашивает.
* **Streaming (SSE):** Постоянное соединение для инкрементальных обновлений.
* **Push-уведомления (Webhooks):** Для очень долгих задач.

```json
# Synchronous Request Example 
{
    "jsonrpc": "2.0",
    "id": "1",
    "method": "sendTask",
    "params": {
        "id": "task-001",
        "sessionId": "session-001",
        "message": {
            "role": "user",
            "parts": [
                {
                    "type": "text",
                    "text": "What is the exchange rate from USD to EUR?"
                }
            ]
        },
        "acceptedOutputModes": [
            "text/plain"
        ],
        "historyLength": 5
    }
}
```

Синхронный запрос использует метод `sendTask`, стриминговый — `sendTaskSubscribe`.

```json
# Streaming Request Example 
{
    "jsonrpc": "2.0",
    "id": "2",
    "method": "sendTaskSubscribe",
    "params": {
        "id": "task-002",
        "sessionId": "session-001",
        "message": {
            "role": "user",
            "parts": [
                {
                    "type": "text",
                    "text": "What's the exchange rate for JPY to GBP today?"
                }
            ]
        },
        "acceptedOutputModes": [
            "text/plain"
        ],
        "historyLength": 5
    }
}
```

**Безопасность:** Взаимная TLS, аудит-логи, требования аутентификации в Agent Card, OAuth 2.0/API-ключи через HTTP-заголовки.

## A2A vs. MCP

A2A дополняет MCP (см. Рис. 1). MCP — структурирование контекста и взаимодействие с данными/инструментами. A2A — координация и коммуникация между агентами, делегирование задач.

![Сравнение протоколов A2A и MCP](../assets/Comparison_A2A_and_MCP_Protocols.png)

Рис. 1: Сравнение протоколов A2A и MCP

## Практические применения и сценарии использования

* **Многофреймворковое сотрудничество:** Агенты на разных фреймворках взаимодействуют.
* **Оркестрация рабочих процессов:** Enterprise-делегирование и координация.
* **Динамическое извлечение информации:** Обмен данными в реальном времени.

## Практический пример кода

```python
import datetime

from google.adk.agents import LlmAgent  # type: ignore[import-untyped]
from google.adk.tools.google_api_tool import CalendarToolset  # type: ignore[import-untyped]


async def create_agent(client_id: str, client_secret: str) -> LlmAgent:
    """Constructs the ADK agent."""
    toolset = CalendarToolset(client_id=client_id, client_secret=client_secret)
    return LlmAgent(
        model="gemini-2.0-flash-001",
        name="calendar_agent",
        description="An agent that can help manage a user's calendar",
        instruction=(
            f""" You are an agent that can help manage a user's calendar. Users will request information about the state of their calendar """
            f""" or to make changes to their calendar. Use the provided tools for interacting with the calendar API. """
            f""" If not specified, assume the calendar the user wants is the 'primary' calendar. """
            f""" When using the Calendar API tools, use well-formed RFC3339 timestamps. Today is {datetime.datetime.now()}. """
        ),
        tools=await toolset.get_tools(),
    )
```

Код определяет асинхронную функцию `create_agent`, создающую ADK-агента для Google Calendar. CalendarToolset с учётными данными. LlmAgent с Gemini и инструментами календаря.

```python
def main(host: str = "0.0.0.0", port: int = 8000):
    # Verify an API key is set.
    # Not required if using Vertex AI APIs.
    if os.getenv("GOOGLE_GENAI_USE_VERTEXAI") != "TRUE" and not os.getenv("GOOGLE_API_KEY"):
        raise ValueError(
            "GOOGLE_API_KEY environment variable not set and "
            "GOOGLE_GENAI_USE_VERTEXAI is not TRUE."
        )

    skill = AgentSkill(
        id="check_availability",
        name="Check Availability",
        description="Checks a user's availability for a time using their Google Calendar",
        tags=["calendar"],
        examples=["Am I free from 10am to 11am tomorrow?"],
    )

    agent_card = AgentCard(
        name="Calendar Agent",
        description="An agent that can manage a user's calendar",
        url=f"http://{host}:{port}/",
        version="1.0.0",
        defaultInputModes=["text"],
        defaultOutputModes=["text"],
        capabilities=AgentCapabilities(streaming=True),
        skills=[skill],
    )

    adk_agent = asyncio.run(
        create_agent(
            client_id=os.getenv("GOOGLE_CLIENT_ID"),
            client_secret=os.getenv("GOOGLE_CLIENT_SECRET"),
        )
    )

    runner = Runner(
        app_name=agent_card.name,
        agent=adk_agent,
        artifact_service=InMemoryArtifactService(),
        session_service=InMemorySessionService(),
        memory_service=InMemoryMemoryService(),
    )
    agent_executor = ADKAgentExecutor(runner, agent_card)

    async def handle_auth(request: Request) -> PlainTextResponse:
        await agent_executor.on_auth_callback(
            str(request.query_params.get("state")),
            str(request.url),
        )
        return PlainTextResponse("Authentication successful.")

    request_handler = DefaultRequestHandler(
        agent_executor=agent_executor,
        task_store=InMemoryTaskStore(),
    )

    a2a_app = A2AStarletteApplication(
        agent_card=agent_card,
        http_handler=request_handler,
    )
    routes = a2a_app.routes()
    routes.append(
        Route(
            path="/authenticate",
            methods=["GET"],
            endpoint=handle_auth,
        )
    )
    app = Starlette(routes=routes)

    uvicorn.run(app, host=host, port=port)


if __name__ == "__main__":
    main()
```

## Краткий обзор

**Что:** Отдельные AI-агенты struggle со сложными задачами. Стандартизированный язык коммуникации отсутствует.

**Почему:** A2A — открытый HTTP-стандарт интероперабельности. Agent Card описывает возможности. Механизмы: синхронные, асинхронные, стриминговые.

**Когда использования:** При оркестрации агентов на разных фреймворках.

**Визуальное резюме:**

![Паттерн межагентной коммуникации A2A](../assets/A2A_Inter-Agent_Communication_Pattern.png)

Рис. 2: Паттерн межагентной коммуникации A2A

## Ключевые выводы

* A2A — открытый HTTP-стандарт для коммуникации AI-агентов.
* AgentCard — цифровой идентификатор с описанием возможностей.
* Синхронные и стриминговые взаимодействия.
* Многоходовые диалоги и контекст.
* Модульная архитектура со специализированными агентами.
* A2A — высокоуровневый протокол; MCP — стандартизированный интерфейс для ресурсов.

## Заключение

A2A устанавливает жизненно важный открытый стандарт для преодоления изоляции агентов. Agent Card — цифровая идентичность. Гибкость: синхронные, асинхронные, стриминговые паттерны. Основа для модульных, масштабируемых многоагентных систем.

## Ссылки

1. Chen, B. (2025). How to Build Your First Google A2A Project: [https://www.trickle.so/blog/how-to-build-google-a2a-project](https://www.trickle.so/blog/how-to-build-google-a2a-project)
2. Google A2A GitHub: [https://github.com/google-a2a/A2A](https://github.com/google-a2a/A2A)
3. Google ADK: [https://google.github.io/adk-docs/](https://google.github.io/adk-docs/)
4. A2A Codelabs: [https://codelabs.developers.google.com/intro-a2a-purchasing-concierge#0](https://codelabs.developers.google.com/intro-a2a-purchasing-concierge#0)
5. A2A Protocol: [https://a2a-protocol.org/latest/](https://a2a-protocol.org/latest/)
6. Multi-Agent Systems with A2A: [https://www.oreilly.com/radar/designing-collaborative-multi-agent-systems-with-the-a2a-protocol/](https://www.oreilly.com/radar/designing-collaborative-multi-agent-systems-with-the-a2a-protocol/)
