# Глава 15: Межагентная коммуникация (A2A)

Отдельные AI-агенты часто сталкиваются с ограничениями при решении сложных многоаспектных задач, даже с продвинутыми возможностями. Для преодоления этого межагентная коммуникация (A2A) позволяет различным AI-агентам, потенциально построенным на разных фреймворках, эффективно сотрудничать. Это сотрудничество включает бесшовную координацию, делегирование задач и обмен информацией.

Протокол A2A от Google — открытый стандарт для универсальной коммуникации. Эта глава рассматривает A2A, его практические применения и реализацию в Google ADK.

## Обзор паттерна «Межагентная коммуникация»

Протокол Agent2Agent (A2A) — открытый стандарт для коммуникации и сотрудничества между различными фреймворками AI-агентов. Он обеспечивает интероперабельность: AI-агенты на LangGraph, CrewAI или Google ADK могут работать совместно независимо от происхождения.

A2A поддерживают Atlassian, Box, LangChain, MongoDB, Salesforce, SAP, ServiceNow. Microsoft планирует интегрировать A2A в Azure AI Foundry и Copilot Studio. Auth0 и SAP интегрируют поддержку A2A в свои платформы.

## Базовые концепции A2A

Протокол обеспечивает структурированный подход к взаимодействию агентов на основе нескольких ключевых концепций.

**Основные субъекты:** A2A включает три сущности:

* **User:** Инициирует запросы.
* **A2A Client (Client Agent):** Приложение или AI-агент, действующий от имени пользователя.
* **A2A Server (Remote Agent):** AI-агент или система с HTTP-эндпоинтом, обрабатывающим запросы клиента. Работает как «непрозрачная» система — клиент не обязан понимать внутренние детали.

**Agent Card:** Цифровая идентичность агента — JSON-файл с информацией для клиентского взаимодействия и автоматического обнаружения: идентичность, URL-эндпоинт, версия, поддерживаемые возможности (streaming, push-уведомления), навыки, режимы ввода/вывода, требования аутентификации.

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
        "schemes": ["apiKey"]
    },
    "defaultInputModes": ["text"],
    "defaultOutputModes": ["text"],
    "skills": [
        {
            "id": "get_current_weather",
            "name": "Get Current Weather",
            "description": "Retrieve real-time weather for any location.",
            "tags": ["weather", "current", "real-time"]
        },
        {
            "id": "get_forecast",
            "name": "Get Forecast",
            "description": "Get 5-day weather predictions.",
            "tags": ["weather", "forecast", "prediction"]
        }
    ]
}
```

**Обнаружение агентов:** клиенты находят Agent Card, описывающие возможности доступных A2A-серверов. Стратегии:

* **Well-Known URI:** Agent Card размещается по стандартизированному пути (`/.well-known/agent.json`). Широкая доступность.
* **Курируемые реестры:** Централизованный каталог для enterprise-сред.
* **Прямая конфигурация:** Внедрение или приватное распространение. Для closely coupled систем.

**Коммуникация и задачи:** В A2A коммуникация строится на асинхронных задачах — базовых единицах работы для долгих процессов. Каждая задача получает уникальный идентификатор и проходит состояния: submitted, working, completed. Коммуникация между агентами — через Message, содержащий атрибуты (метаданные) и части (содержимое: текст, файлы, JSON). Результаты агента — artifacts (тоже из частей, могут стримиться). Протокол — HTTP(S) с JSON-RPC 2.0. Для континуальности используется server-generated contextId.

**Механизмы взаимодействия:**

* **Синхронный запрос/ответ:** Клиент отправляет и ждёт полный ответ.
* **Асинхронный поллинг:** Сервер подтверждает «working» с task ID, клиент опрашивает статус.
* **Streaming (Server-Sent Events):** Постоянное однонаправленное соединение для инкрементальных обновлений.
* **Push-уведомления (Webhooks):** Для очень долгих задач — клиент регистрирует webhook URL, сервер отправляет уведомление.

```json
// Синхронный запрос
{
    "jsonrpc": "2.0",
    "id": "1",
    "method": "sendTask",
    "params": {
        "id": "task-001",
        "message": {
            "role": "user",
            "parts": [{"type": "text", "text": "What is the exchange rate from USD to EUR?"}]
        },
        "acceptedOutputModes": ["text/plain"],
        "historyLength": 5
    }
}
```

```json
// Стриминговый запрос
{
    "jsonrpc": "2.0",
    "id": "2",
    "method": "sendTaskSubscribe",
    "params": {
        "id": "task-002",
        "message": {
            "role": "user",
            "parts": [{"type": "text", "text": "What's the exchange rate for JPY to GBP today?"}]
        },
        "acceptedOutputModes": ["text/plain"],
        "historyLength": 5
    }
}
```

**Безопасность:** Взаимная TLS (mTLS) для шифрования и аутентификации, полные аудит-логи, явное объявление требований аутентификации в Agent Card, OAuth 2.0 токены и API-ключи через HTTP-заголовки.

## A2A vs. MCP

A2A — протокол, дополняющий Model Context Protocol (MCP) (см. Рис. 1). MCP фокусируется на структурировании контекста для агентов и их взаимодействии с внешними данными и инструментами. A2A обеспечивает координацию и коммуникацию между агентами, делегирование задач и сотрудничество.

![Сравнение протоколов A2A и MCP](../assets/Comparison_A2A_and_MCP_Protocols.png)

Рис. 1: Сравнение протоколов A2A и MCP

## Практические применения и сценарии использования

* **Многофреймворковое сотрудничество:** A2A позволяет независимым агентам на разных фреймворках (ADK, LangChain, CrewAI) взаимодействовать.
* **Оркестрация автоматизированных рабочих процессов:** Enterprise-сценарии с делегированием и координацией задач.
* **Динамическое извлечение информации:** Агенты запрашивают и обмениваются данными в реальном времени.

## Практический пример кода

Репозиторий [https://github.com/google-a2a/a2a-samples/tree/main/samples](https://github.com/google-a2a/a2a-samples/tree/main/samples) содержит примеры на Java, Go и Python для LangGraph, CrewAI, Azure AI Foundry и AG2.

```python
import datetime
from google.adk.agents import LlmAgent
from google.adk.tools.google_api_tool import CalendarToolset

async def create_agent(client_id: str, client_secret: str) -> LlmAgent:
    toolset = CalendarToolset(client_id=client_id, client_secret=client_secret)
    return LlmAgent(
        model="gemini-2.0-flash-001",
        name="calendar_agent",
        description="An agent that can help manage a user's calendar",
        instruction=(
            "You are an agent that can help manage a user's calendar. "
            "Use the provided tools for interacting with the calendar API. "
            f"Today is {datetime.datetime.now()}."
        ),
        tools=await toolset.get_tools(),
    )
```

Код определяет асинхронную функцию `create_agent`, создающую ADK-агента для управления Google Calendar. Инициализируется CalendarToolset с учётными данными. LlmAgent настраивается с моделью Gemini, описанием и инструментами календаря.

Запуск сервера:

```python
def main(host: str = "0.0.0.0", port: int = 8000):
    skill = AgentSkill(
        id="check_availability",
        name="Check Availability",
        description="Checks a user's availability using Google Calendar",
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

    adk_agent = asyncio.run(create_agent(...))
    runner = Runner(app_name=agent_card.name, agent=adk_agent, ...)
    agent_executor = ADKAgentExecutor(runner, agent_card)

    request_handler = DefaultRequestHandler(agent_executor=agent_executor, task_store=InMemoryTaskStore())
    a2a_app = A2AStarletteApplication(agent_card=agent_card, http_handler=request_handler)
    app = Starlette(routes=a2a_app.routes())
    uvicorn.run(app, host=host, port=port)
```

Код настраивает A2A-совместимого агента с навыком «check_availability», AgentCard и запуском веб-приложения через Uvicorn.

## Краткий обзор

**Что:** Отдельные AI-агенты, особенно на разных фреймворках, struggle со сложными задачами. Изолированность препятствует созданию многоагентных систем.

**Почему:** Протокол A2A — открытый HTTP-стандарт для интероперабельности. Agent Card описывает возможности. Механизмы: синхронные, асинхронные и стриминговые взаимодействия.

**Когда использовать:** При оркестрации двух или более агентов, особенно на разных фреймворках. Для модульных приложений со специализированными агентами.

**Визуальное резюме:**

![Паттерн межагентной коммуникации A2A](../assets/A2A_Inter-Agent_Communication_Pattern.png)

Рис. 2: Паттерн межагентной коммуникации A2A

## Ключевые выводы

* A2A — открытый HTTP-стандарт для коммуникации AI-агентов на разных фреймворках.
* AgentCard — цифровой идентификатор с описанием возможностей.
* Синхронные (`sendTask`) и стриминговые (`sendTaskSubscribe`) взаимодействия.
* Поддержка многоходовых диалогов и контекста.
* Модульная архитектура: специализированные агенты на разных портах.
* A2A — высокоуровневый протокол для задач и рабочих процессов; MCP — стандартизированный интерфейс для внешних ресурсов.

## Заключение

Протокол A2A устанавливает жизненно важный открытый стандарт для преодоления изоляции отдельных AI-агентов. Agent Card служит цифровой идентичностью. Гибкость поддерживает синхронные, асинхронные и стриминговые паттерны. Это создаёт основу для модульных, масштабируемых и интеллектуальных многоагентных систем.

## Ссылки

1. Chen, B. (2025). How to Build Your First Google A2A Project: [https://www.trickle.so/blog/how-to-build-google-a2a-project](https://www.trickle.so/blog/how-to-build-google-a2a-project)
2. Google A2A GitHub: [https://github.com/google-a2a/A2A](https://github.com/google-a2a/A2A)
3. Google ADK: [https://google.github.io/adk-docs/](https://google.github.io/adk-docs/)
4. A2A Codelabs: [https://codelabs.developers.google.com/intro-a2a-purchasing-concierge#0](https://codelabs.developers.google.com/intro-a2a-purchasing-concierge#0)
5. A2A Protocol: [https://a2a-protocol.org/latest/](https://a2a-protocol.org/latest/)
6. Multi-Agent Systems with A2A: [https://www.oreilly.com/radar/designing-collaborative-multi-agent-systems-with-the-a2a-protocol/](https://www.oreilly.com/radar/designing-collaborative-multi-agent-systems-with-the-a2a-protocol/)
