# Глава 10: Протокол контекста модели

Чтобы LLM эффективно функционировали как агенты, их возможности должны выходить за рамки мультимодальной генерации. Необходимо взаимодействие с внешней средой: доступ к актуальным данным, использование внешнего ПО и выполнение операционных задач. Протокол контекста модели (Model Context Protocol, MCP) решает эту потребность, предоставляя стандартизированный интерфейс для взаимодействия LLM с внешними ресурсами.

## Обзор паттерна MCP

Представьте универсальный адаптер, позволяющий любой LLM подключиться к любой внешней системе. По сути, именно это и представляет собой MCP — открытый стандарт для унификации коммуникации LLM (Gemini, GPT, Claude) с внешними приложениями и инструментами.

MCP работает по клиент-серверной архитектуре, определяя, как данные (resources), промпты и инструменты (tools) экспонируются сервером и потребляются MCP-клиентом (LLM-хост или AI-агент).

Однако MCP является контрактом для «агентного интерфейса», и его эффективность зависит от дизайна underlying API. Если API позволяет извлекать только полные данные по одному элементу, агент будет медленным при больших объёмах. Для эффективности API должен быть улучшен детерминированными функциями.

MCP также может оборачивать API, формат данных которого не является понятным для агента. Если MCP-сервер возвращает PDF, но агент не может его парсить — это бесполезно. Лучший подход — API, возвращающий текстовую версию (Markdown).

## MCP vs. Вызов функций инструментов

MCP и вызов функций — различные механизмы. Вызов функций — прямой запрос LLM к конкретному инструменту (модель «один-к-одному»). MCP — стандартизированный интерфейс для обнаружения и использования внешних возможностей (клиент-серверная архитектура).

| Возможность | Вызов функций | MCP |
|---|---|---|
| Стандартизация | Проприетарный | Открытый стандарт |
| Область | Прямой вызов функции | Фреймворк обнаружения и коммуникации |
| Архитектура | Один-к-одному | Клиент-серверная |
| Обнаружение | Явное | Динамическое |
| Переиспользуемость | Coupled | Автономные серверы |

## Дополнительные соображения по MCP

* **Tool vs. Resource vs. Prompt:** Resource — статические данные, Tool — исполняемая функция, Prompt — шаблон взаимодействия.
* **Обнаруживаемость:** Клиент может динамически запрашивать сервер о возможностях.
* **Безопасность:** Аутентификация и авторизация.
* **Реализация:** SDK от Anthropic, FastMCP упрощают разработку.
* **Обработка ошибок:** Протокол определяет передачу ошибок LLM.
* **Локальный vs. удалённый сервер:** Выбор зависит от требований.
* **Транспорт:** JSON-RPC через STDIO (локально), Streamable HTTP и SSE (удалённо).

Компоненты MCP:

1. **LLM:** Основной интеллект — обработка запросов, планирование, решение о внешних действиях.
2. **MCP Client:** Посредник, переводящий намерения LLM в MCP-запросы.
3. **MCP Server:** Шлюз к внешнему миру — экспонирует инструменты, ресурсы, промпты.
4. **Сторонний сервис:** Фактический внешний инструмент или источник данных.

Поток: обнаружение → формулирование запроса → коммуникация клиента → выполнение сервера → ответ и обновление контекста.

## Практические применения и сценарии использования

* **Интеграция с БД:** Доступ к структурированным данным через естественный язык.
* **Оркестрация генеративных медиа:** Imagen, Veo, Chirp 3 HD, Lyria.
* **Взаимодействие с внешними API:** Погода, курсы, email, CRM.
* **Извлечение информации на основе рассуждений:** Точный поиск по документам.
* **Разработка пользовательских инструментов:** FastMCP для экспонирования внутренних функций.
* **Стандартизированная коммуникация LLM-приложение:** Снижение накладных расходов.
* **Оркестрация сложных рабочих процессов:** Многошаговые процессы с несколькими MCP-сервисами.
* **Управление IoT-устройствами:** Естественно-языковое управление.
* **Автоматизация финансовых сервисов:** Анализ, торговля, отчётность.

## Практический пример кода с ADK

Этот раздел описывает подключение к локальному MCP-серверу файловой системы.

### Настройка агента с MCPToolset

```python
import os

from google.adk.agents import LlmAgent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset, StdioServerParameters


# Create a reliable absolute path to a folder named 'mcp_managed_files'
# within the same directory as this agent script.
# This ensures the agent works out-of-the-box for demonstration.
# For production, you would point this to a more persistent and secure location.
TARGET_FOLDER_PATH = os.path.join(
    os.path.dirname(os.path.abspath(__file__)),
    "mcp_managed_files",
)

# Ensure the target directory exists before the agent needs it.
os.makedirs(TARGET_FOLDER_PATH, exist_ok=True)

root_agent = LlmAgent(
    model="gemini-2.0-flash",
    name="filesystem_assistant_agent",
    instruction=(
        "Help the user manage their files. You can list files, read files, and write files. "
        f"You are operating in the following directory: {TARGET_FOLDER_PATH}"
    ),
    tools=[
        MCPToolset(
            connection_params=StdioServerParameters(
                command="npx",
                args=[
                    "-y",  # Argument for npx to auto-confirm install
                    "@modelcontextprotocol/server-filesystem",
                    # This MUST be an absolute path to a folder.
                    TARGET_FOLDER_PATH,
                ],
            ),
            # Optional: You can filter which tools from the MCP server are exposed.
            # For example, to only allow reading:
            # tool_filter=['list_directory', 'read_file']
        )
    ],
)
```

npx (Node Package Execute) — утилита для выполнения Node.js-пакетов из реестра npm. Широко используется для запуска community MCP-серверов.

```python
# ./adk_agent_samples/mcp_agent/__init__.py 
from . import agent
```

Создание файла __init__.py необходимо для recognition agent.py как discoverable Python-пакета для ADK.

```python
connection_params = StdioConnectionParams(
    server_params={
        "command": "python3",
        "args": ["./agent/mcp_server.py"],
        "env": {
            "SERVICE_ACCOUNT_PATH": SERVICE_ACCOUNT_PATH,
            "DRIVE_FOLDER_ID": DRIVE_FOLDER_ID,
        },
    }
)
```

UVX — командная строка, использующая uv для выполнения команд в изолированной Python-среде.

```python
connection_params = StdioConnectionParams(
    server_params={
        "command": "uvx",
        "args": ["mcp-google-sheets@latest"],
        "env": {
            "SERVICE_ACCOUNT_PATH": SERVICE_ACCOUNT_PATH,
            "DRIVE_FOLDER_ID": DRIVE_FOLDER_ID,
        },
    }
)
```

После создания MCP-сервера следующий шаг — подключение.

## Подключение MCP-сервера через ADK Web

Выполните `adk web`, перейдите в родительскую директорию и выберите агента в UI.

## Создание MCP-сервера с FastMCP

## Server setup with FastMCP

FastMCP — высокоуровневый Python-фреймворк для упрощённой разработки MCP-серверов. Позволяет быстро определять инструменты, ресурсы и промпты с помощью декораторов. Автоматическая генерация схем из сигнатур функций.

```python
cd ./adk_agent_samples # Or your equivalent parent directory 
adk web
```

## To illustrate, consider a basic "greet" tool provided by the server. ADK agents and other MCP clients can interact with this tool using HTTP once it is active

Этот Python-скрипт определяет функцию greet, принимающую имя и возвращающую приветствие. Декоратор @tool() регистрирует её как MCP-инструмент. При запуске сервер слушает на localhost:8000.

## Использование FastMCP-сервера с ADK-агентом

ADK-агент может быть настроен как MCP-клиент через HttpServerParameters. Параметр tool_filter ограничивает доступные инструменты.

```python
# fastmcp_server.py
# This script demonstrates how to create a simple MCP server using FastMCP.
# It exposes a single tool that generates a greeting.
# 1. Make sure you have FastMCP installed:
# pip install fastmcp

from fastmcp import FastMCP, Client


# Initialize the FastMCP server.
mcp_server = FastMCP()


# Define a simple tool function.
# The `@mcp_server.tool` decorator registers this Python function as an MCP tool.
# The docstring becomes the tool's description for the LLM.
@mcp_server.tool
def greet(name: str) -> str:
    """
    Generates a personalized greeting.

    Args:
        name: The name of the person to greet.

    Returns:
        A greeting string.
    """
    return f"Hello, {name}! Nice to meet you."


# Or if you want to run it from the script:
if __name__ == "__main__":
    mcp_server.run(
        transport="http",
        host="127.0.0.1",
        port=8000,
    )
```

Скрипт определяет Agent с именем fastmcp_greeter_agent. Агент оснащён MCPToolset для подключения к FastMCP-серверу на localhost:8000.

Для запуска: `python fastmcp_server.py` → `adk web` → выберите агента.

## Краткий обзор

**Что:** LLM должны выходить за рамки генерации текста. Без стандартизированной коммуникации каждая интеграция — кастомная и непереиспользуемая.

**Почему:** MCP — универсальный интерфейс клиент-серверной архитектуры для экспонирования инструментов и ресурсов.

**Когда использования:** При построении сложных, масштабируемых агентных систем с разнообразными внешними инструментами.

**Визуальное резюме:**

![Протокол контекста модели](../assets/Model_Context_Protocol.png)

Рис. 1: Протокол контекста модели

## Ключевые выводы

* MCP — открытый стандарт для коммуникации LLM с внешними системами.
* Клиент-серверная архитектура для ресурсов, промптов и инструментов.
* ADK поддерживает MCP-серверы и экспонирование своих инструментов.
* FastMCP упрощает разработку MCP-серверов.
* MCP Tools for Genmedia — Imagen, Veo, Chirp 3 HD, Lyria.
* MCP позволяет взаимодействовать с реальными системами за пределами генерации текста.

## Заключение

MCP — открытый стандарт для коммуникации LLM с внешними системами. Использует клиент-серверную архитектуру для доступа к ресурсам и выполнения действий. Позволяет взаимодействовать с БД, медиа, IoT, финансовыми сервисами. Ключевой компонент для создания интерактивных AI-агентов.

## Ссылки

1. Документация MCP: [https://google.github.io/adk-docs/mcp/](https://google.github.io/adk-docs/mcp/)
2. FastMCP: [https://github.com/jlowin/fastmcp](https://github.com/jlowin/fastmcp)
3. MCP Tools for Genmedia: [https://google.github.io/adk-docs/mcp/#mcp-servers-for-google-cloud-genmedia](https://google.github.io/adk-docs/mcp/#mcp-servers-for-google-cloud-genmedia)
4. MCP Toolbox for Databases: [https://google.github.io/adk-docs/mcp/databases/](https://google.github.io/adk-docs/mcp/databases/)
```python
# ./adk_agent_samples/fastmcp_client_agent/agent.py
import os

from google.adk.agents import LlmAgent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset, HttpServerParameters


# Define the FastMCP server's address.
# Make sure your fastmcp_server.py (defined previously) is running on this port.
FASTMCP_SERVER_URL = "http://localhost:8000"

root_agent = LlmAgent(
    model="gemini-2.0-flash",  # Or your preferred model
    name="fastmcp_greeter_agent",
    instruction='You are a friendly assistant that can greet people by their name. Use the "greet" tool.',
    tools=[
        MCPToolset(
            connection_params=HttpServerParameters(
                url=FASTMCP_SERVER_URL,
            ),
            # Optional: Filter which tools from the MCP server are exposed
            # For this example, we're expecting only 'greet'
            tool_filter=["greet"],
        )
    ],
)
```



The script defines an Agent named `fastmcp_greeter_agent` that uses a Gemini language model. It's given a specific instruction to act as a friendly assistant whose purpose is to greet people. Crucially, the code equips this agent with a tool to perform its task. It configures an MCPToolset to connect to a separate server running on localhost:8000, which is expected to be the FastMCP server from the previous example. The agent is specifically granted access to the greet tool hosted on that server. In essence, this code sets up the client side of the system, creating an intelligent agent that understands its goal is to greet people and knows exactly which external tool to use to accomplish it.

Creating an `__init__.py` file within the `fastmcp_client_agent` directory is necessary. This ensures the agent is recognized as a discoverable Python package for the ADK.

To begin, open a new terminal and run `python fastmcp_server.py` to start the FastMCP server. Next, go to the parent directory of `fastmcp_client_agent` (for example, `adk_agent_samples`) in your terminal and execute `adk web`. Once the ADK Web UI loads in your browser, select the `fastmcp_greeter_agent` from the agent menu. You can then test it by entering a prompt like "Greet John Doe." The agent will use the `greet` tool on your FastMCP server to create a response.


**What:** To function as effective agents, LLMs must move beyond simple text generation. They require the ability to interact with the external environment to access current data and utilize external software. Without a standardized communication method, each integration between an LLM and an external tool or data source becomes a custom, complex, and non-reusable effort. This ad-hoc approach hinders scalability and makes building complex, interconnected AI systems difficult and inefficient.

**Why:** The Model Context Protocol (MCP) offers a standardized solution by acting as a universal interface between LLMs and external systems. It establishes an open, standardized protocol that defines how external capabilities are discovered and used. Operating on a client-server model, MCP allows servers to expose tools, data resources, and interactive prompts to any compliant client. LLM-powered applications act as these clients, dynamically discovering and interacting with available resources in a predictable manner. This standardized approach fosters an ecosystem of interoperable and reusable components, dramatically simplifying the development of complex agentic workflows.

**Rule of thumb:** Use the Model Context Protocol (MCP) when building complex, scalable, or enterprise-grade agentic systems that need to interact with a diverse and evolving set of external tools, data sources, and APIs. It is ideal when interoperability between different LLMs and tools is a priority, and when agents require the ability to dynamically discover new capabilities without being redeployed. For simpler applications with a fixed and limited number of predefined functions, direct tool function calling may be sufficient.

**Visual summary:**

![Model Context Protocol](../assets/Model_Context_Protocol.png)

Fig.1: Model Context protocol


These are the key takeaways:

* The Model Context Protocol (MCP) is an open standard facilitating standardized communication between LLMs and external applications, data sources, and tools.  
* It employs a client-server architecture, defining the methods for exposing and consuming resources, prompts, and tools.  
* The Agent Development Kit (ADK) supports both utilizing existing MCP servers and exposing ADK tools via an MCP server.  
* FastMCP simplifies the development and management of MCP servers, particularly for exposing tools implemented in Python.  
* MCP Tools for Genmedia Services allows agents to integrate with Google Cloud's generative media capabilities (Imagen, Veo, Chirp 3 HD, Lyria).  
* MCP enables LLMs and agents to interact with real-world systems, access dynamic information, and perform actions beyond text generation.


The Model Context Protocol (MCP) is an open standard that facilitates communication between Large Language Models (LLMs) and external systems. It employs a client-server architecture, enabling LLMs to access resources, utilize prompts, and execute actions through standardized tools. MCP allows LLMs to interact with databases, manage generative media workflows, control IoT devices, and automate financial services. Practical examples demonstrate setting up agents to communicate with MCP servers, including filesystem servers and servers built with FastMCP, illustrating its integration with the Agent Development Kit (ADK). MCP is a key component for developing interactive AI agents that extend beyond basic language capabilities.


1. Model Context Protocol (MCP) Documentation. (Latest). *Model Context Protocol (MCP)*. [https://google.github.io/adk-docs/mcp/](https://google.github.io/adk-docs/mcp/)  
2. FastMCP Documentation. FastMCP. [https://github.com/jlowin/fastmcp](https://github.com/jlowin/fastmcp)  
3. MCP Tools for Genmedia Services. *MCP Tools for Genmedia Services*. [https://google.github.io/adk-docs/mcp/\#mcp-servers-for-google-cloud-genmedia](https://google.github.io/adk-docs/mcp/#mcp-servers-for-google-cloud-genmedia)  
4. MCP Toolbox for Databases Documentation. (Latest). *MCP Toolbox for Databases*. [https://google.github.io/adk-docs/mcp/databases/](https://google.github.io/adk-docs/mcp/databases/)
