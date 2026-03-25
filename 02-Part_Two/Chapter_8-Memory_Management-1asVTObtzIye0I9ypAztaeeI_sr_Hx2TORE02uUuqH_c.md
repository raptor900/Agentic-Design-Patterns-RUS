# Глава 8: Управление памятью

Эффективное управление памятью критически важно для интеллектуальных агентов, которым необходимо сохранять информацию. Агентам, подобно людям, требуются различные типы памяти для эффективной работы. Эта глава посвящена управлению памятью, в частности — непосредственной (краткосрочной) и постоянной (долгосрочной) памяти агентов.

В агентных системах память означает способность агента сохранять и использовать информацию из прошлых взаимодействий, наблюдений и опыта обучения. Эта возможность позволяет агентам принимать обоснованные решения, поддерживать контекст диалога и улучшаться со временем. Память агента обычно делится на два основных типа:

* **Краткосрочная память (контекстная память):** Аналог рабочей памяти — хранит информацию, которая обрабатывается или недавно использовалась. Для агентов на основе LLM краткосрочная память существует преимущественно в контекстном окне. Это окно содержит последние сообщения, ответы агента, результаты использования инструментов и рефлексии из текущего взаимодействия. Контекстное окно ограничено по ёмкости. Эффективное управление краткосрочной памятью включает сохранение наиболее релевантной информации в этом ограниченном пространстве — через обобщение старых сегментов или выделение ключевых деталей. Модели с «длинным контекстом» расширяют размер этой краткосрочной памяти, но контекст по-прежнему эфемерен — он теряется при завершении сессии и может быть затратным в обработке. Поэтому агентам нужны отдельные типы памяти для истинной персистентности.
* **Долгосрочная память (персистентная память):** Репозиторий информации, которую агенты должны сохранять между взаимодействиями, задачами или длительными периодами. Данные обычно хранятся вне среды обработки — в базах данных, графах знаний или векторных БД. В векторных БД информация конвертируется в числовые векторы, позволяя агентам извлекать данные на основе семантического сходства (семантический поиск). При необходимости агент запрашивает внешнее хранилище, извлекает релевантные данные и интегрирует их в краткосрочный контекст.
```python
# Example: Using InMemorySessionService 
# This is suitable for local development and testing where data 
# persistence across application restarts is not required. 
from google.adk.sessions import InMemorySessionService
session_service = InMemorySessionService()
```
## Практические применения и сценарии использования

Управление памятью жизненно важно для агентов, которым необходимо отслеживать информацию и работать интеллектуально со временем. Применения включают:

* **Чатботы и разговорный ИИ:** Краткосрочная память поддерживает поток диалога. Долгосрочная память позволяет вспоминать предпочтения пользователя, прошлые проблемы, обеспечивая персонализированные непрерывные взаимодействия.
* **Задачно-ориентированные агенты:** Краткосрочная память для отслеживания предыдущих шагов, прогресса и целей. Долгосрочная — для доступа к пользовательским данным, отсутствующим в непосредственном контексте.
* **Персонализированный опыт:** Долгосрочная память хранит предпочтения, поведение, личную информацию для адаптации ответов.
* **Обучение и улучшение:** Агенты совершенствуются, сохраняя успешные стратегии и ошибки в долгосрочной памяти.
* **Извлечение информации (RAG):** Агенты обращаются к базе знаний (долгосрочная память) для информирования своих ответов.
* **Автономные системы:** Роботы и беспилотники нуждаются в памяти для карт, маршрутов, местоположений объектов.

Память позволяет агентам вести историю, учиться, персонализировать взаимодействия и решать сложные временнЫе задачи.
```python
# Example: Using DatabaseSessionService 
# This is suitable for production or development requiring persistent storage. 
# You need to configure a database URL (e.g., for SQLite, PostgreSQL, etc.). 
# Requires: pip install google-adk[sqlalchemy] and a database driver (e.g., psycopg2 for PostgreSQL) 
from google.adk.sessions import DatabaseSessionService 
# Example using a local SQLite file: 
db_url = "sqlite:///./my_agent_data.db"
session_service = DatabaseSessionService(db_url=db_url)
```

## Практический пример кода: Управление памятью в Google ADK

Google Agent Developer Kit (ADK) предлагает структурированный метод управления контекстом и памятью. Понимание Session, State и Memory в ADK жизненно важно для построения агентов, которые сохраняют информацию.

Как и в человеческом взаимодействии, агенты должны помнить предыдущие обмены для связных диалогов. ADK упрощает управление контекстом через три базовых понятия:

* **Session:** Отдельная ветка диалога, логирующая сообщения и действия (Event), а также хранящая временные данные (State) для этого взаимодействия.
* **State (`session.state`):** Данные в сессии, относящиеся только к текущему активному диалогу.
* **Memory:** Поисковый репозиторий информации из прошлых диалогов или внешних источников.

ADK предоставляет специализированные сервисы: SessionService управляет потоками чата, MemoryService — хранением и извлечением долгосрочных знаний.

Оба сервиса предлагают различные варианты хранения. InMemory — для тестирования (данные не сохраняются между перезапусками). Для персистентности и масштабируемости ADK поддерживает базы данных и облачные сервисы.
```python
# Example: Using VertexAiSessionService
# This is suitable for scalable production on Google Cloud Platform, leveraging
# Vertex AI infrastructure for session management.
# Requires: pip install google-adk[vertexai] and GCP setup/authentication

from google.adk.sessions import VertexAiSessionService


PROJECT_ID = "your-gcp-project-id"  # Replace with your GCP project ID
LOCATION = "us-central1"  # Replace with your desired GCP location

# The app_name used with this service should correspond to the Reasoning Engine ID or name
REASONING_ENGINE_APP_NAME = (
    "projects/your-gcp-project-id/locations/us-central1/reasoningEngines/your-engine-id"
)  # Replace with your Reasoning Engine resource name

session_service = VertexAiSessionService(project=PROJECT_ID, location=LOCATION)

# When using this service, pass REASONING_ENGINE_APP_NAME to service methods:
# session_service.create_session(app_name=REASONING_ENGINE_APP_NAME, ...)
# session_service.get_session(app_name=REASONING_ENGINE_APP_NAME, ...)
# session_service.append_event(session, event, app_name=REASONING_ENGINE_APP_NAME)
# session_service.delete_session(app_name=REASONING_ENGINE_APP_NAME, ...)
```

### Session: Отслеживание каждого диалога

Session в ADK отслеживает и управляет отдельными потоками чата. При начале взаимодействия SessionService создаёт объект Session (`google.adk.sessions.Session`), включающий уникальные идентификаторы, хронологию событий, хранилище временных данных (state) и метку последнего обновления. Разработчики обычно взаимодействуют с Session через SessionService, отвечающий за жизненный цикл: создание, возобновление, запись активности, управление удалением.

ADK предоставляет несколько реализаций SessionService: InMemorySessionService (тестирование, без персистентности), DatabaseSessionService (надёжное сохранение в БД), VertexAiSessionService (масштабируемое решение на Google Cloud).

```python
# Import necessary classes from the Google Agent Developer Kit (ADK)
from google.adk.agents import LlmAgent
from google.adk.sessions import InMemorySessionService, Session
from google.adk.runners import Runner
from google.genai.types import Content, Part


# Define an LlmAgent with an output_key.
greeting_agent = LlmAgent(
 name="Greeter",
 model="gemini-2.0-flash",
 instruction="Generate a short, friendly greeting.",
 output_key="last_greeting",
)


# --- Setup Runner and Session ---
app_name, user_id, session_id = "state_app", "user1", "session1"

session_service = InMemorySessionService()

runner = Runner(
    agent=greeting_agent,
    app_name=app_name,
    session_service=session_service,
)

session = session_service.create_session(
    app_name=app_name,
    user_id=user_id,
    session_id=session_id,
)

print(f"Initial state: {session.state}")


# --- Run the Agent ---
user_message = Content(parts=[Part(text="Hello")])

print("\n--- Running the agent ---")
for event in runner.run(
    user_id=user_id,
    session_id=session_id,
    new_message=user_message,
):
    if event.is_final_response():
        print("Agent responded.")


# --- Check Updated State ---
# Correctly check the state after the runner has finished processing all events.
updated_session = session_service.get_session(app_name, user_id, session_id)
print(f"\nState after agent run: {updated_session.state}")
```

Затем DatabaseSessionService для надёжного сохранения в базу данных.

```python
# Example: Using InMemoryMemoryService
# This is suitable for local development and testing where data
# persistence across application restarts is not required.
# Memory content is lost when the app stops.

from google.adk.memory import InMemoryMemoryService

memory_service = InMemoryMemoryService()
```

Также доступен VertexAiSessionService для масштабируемой продакшн-среды на Google Cloud.

```python
# Example: Using VertexAiRagMemoryService
# This is suitable for scalable production on GCP, leveraging
# Vertex AI RAG (Retrieval Augmented Generation) for persistent,
# searchable memory.
# Requires: pip install google-adk[vertexai], GCP
# setup/authentication, and a Vertex AI RAG Corpus.

from google.adk.memory import VertexAiRagMemoryService


# The resource name of your Vertex AI RAG Corpus
RAG_CORPUS_RESOURCE_NAME = (
    "projects/your-gcp-project-id/locations/us-central1/ragCorpora/your-corpus-id"
)  # Replace with your Corpus resource name

# Optional configuration for retrieval behavior
SIMILARITY_TOP_K = 5  # Number of top results to retrieve
VECTOR_DISTANCE_THRESHOLD = 0.7  # Threshold for vector similarity

memory_service = VertexAiRagMemoryService(
    rag_corpus=RAG_CORPUS_RESOURCE_NAME,
    similarity_top_k=SIMILARITY_TOP_K,
    vector_distance_threshold=VECTOR_DISTANCE_THRESHOLD,
)

# When using this service, methods like add_session_to_memory
# and search_memory will interact with the specified Vertex AI
# RAG Corpus.
```

Выбор SessionService критичен — он определяет, как хранятся история взаимодействий и временные данные, и их персистентность.

Каждый обмен сообщениями — циклический процесс: сообщение получено → Runner извлекает или создаёт Session → агент обрабатывает сообщение с контекстом Session → генерирует ответ и обновляет state → Runner оборачивает в Event → `session_service.append_event` записывает событие и обновляет состояние. Session ожидает следующее сообщение. При завершении взаимодействия вызывается `delete_session`.

### State: Черновик сессии

В ADK каждая Session включает компонент state — временную рабочую память агента для конкретного диалога. В отличие от session.events (лог всей истории), session.state хранит и обновляет динамические данные.

По сути, session.state — словарь «ключ-значение». Основная функция — позволять агенту запоминать детали для связного диалога: предпочтения пользователя, прогресс задач, инкрементальные данные, условные флаги.

Структура: строковые ключи + значения сериализуемых Python-типов. Организация — через префиксы ключей:

* Без префикса — данные, специфичные для сессии.
* Префикс `user:` — данные пользователя во всех сессиях.
* Префикс `app:` — данные, общие для всех пользователей приложения.
* Префикс `temp:` — данные только для текущего хода обработки, не сохраняются.

Агент обращается ко всем данным state через единый словарь session.state. SessionService обрабатывает извлечение, слияние и персистентность. State обновляется при добавлении Event через `session_service.append_event()`.

#### 1. Простой способ: `output_key` (для текстовых ответов агента)

Самый простой метод сохранения финального текстового ответа агента в state. При настройке LlmAgent указывается `output_key`. Runner автоматически создаёт необходимые действия для сохранения.

```python
from langchain.memory import ChatMessageHistory


# Initialize the history object
history = ChatMessageHistory()

# Add user and AI messages
history.add_user_message("I'm heading to New York next week.")
history.add_ai_message("Great! It's a fantastic city.")

# Access the list of messages
print(history.messages)
```

За кулисами Runner видит `output_key` и автоматически создаёт действия с `state_delta` при вызове `append_event`.

#### 2. Стандартный способ: `EventActions.state_delta` (для сложных обновлений)

Для обновления нескольких ключей, сохранения нетекстовых данных, целевых скоупов (`user:`, `app:`) или обновлений, не связанных с финальным текстовым ответом — вручную строится словарь изменений `state_delta` и включается в EventActions добавляемого Event.

```python
from langchain.memory import ConversationBufferMemory


# Initialize memory
memory = ConversationBufferMemory()

# Save a conversation turn
memory.save_context(
    {"input": "What's the weather like?"},
    {"output": "It's sunny today."},
)

# Load the memory as a string
print(memory.load_memory_variables({}))
```

Этот код демонстрирует подход на основе инструментов для управления состоянием сессии. Функция `log_user_login` — инструмент, обновляющий состояние при входе пользователя: увеличивает счётчик, устанавливает статус задачи, записывает метку времени. Код демонстрации создаёт in-memory сессию, имитирует вызов инструмента и проверяет обновлённое состояние.

Прямое изменение `session.state` после извлечения сессии настоятельно не рекомендуется — это обходит стандартный механизм обработки событий и может привести к проблемам согласованности. Рекомендуемые методы: `output_key` на LlmAgent или `EventActions.state_delta` при `append_event`.

Итого: простые типы данных, чистые имена ключей с префиксами, избегание вложенных структур, обновление state только через append_event.

## Memory: Долгосрочная база знаний через MemoryService

Session хранит текущую историю чата и временные данные для одного диалога. Для сохранения информации между взаимодействиями или доступа к внешним данным необходима долгосрочная память — MemoryService.

```python
from langchain_openai import OpenAI
from langchain.chains import LLMChain
from langchain.prompts import PromptTemplate
from langchain.memory import ConversationBufferMemory


# 1. Define LLM and Prompt
llm = OpenAI(temperature=0)

template = """You are a helpful travel agent.
Previous conversation: {history}
New question: {question}
Response:"""
prompt = PromptTemplate.from_template(template)

# 2. Configure Memory
# The memory_key "history" matches the variable in the prompt
memory = ConversationBufferMemory(memory_key="history")

# 3. Build the Chain
conversation = LLMChain(llm=llm, prompt=prompt, memory=memory)

# 4. Run the Conversation
response = conversation.predict(question="I want to book a flight.")
print(response)

response = conversation.predict(question="My name is Sam, by the way.")
print(response)

response = conversation.predict(question="What was my name again?")
print(response)
```

Session и State — краткосрочная память для одного чата. Долгосрочное хранилище MemoryService — персистентный поисковый репозиторий из нескольких прошлых взаимодействий или внешних источников. MemoryService (интерфейс BaseMemoryService) стандартизирует управление: добавление информации (`add_session_to_memory`) и извлечение (`search_memory`).

ADK предлагает несколько реализаций. InMemoryMemoryService — временное хранилище для тестирования. Для продакшна — VertexAiRagMemoryService, использующий Google Cloud RAG для масштабируемого семантического поиска.

```python
from langchain_openai import ChatOpenAI
from langchain.chains import LLMChain
from langchain.memory import ConversationBufferMemory
from langchain_core.prompts import (
    ChatPromptTemplate,
    MessagesPlaceholder,
    SystemMessagePromptTemplate,
    HumanMessagePromptTemplate,
)


# 1. Define Chat Model and Prompt
llm = ChatOpenAI()

prompt = ChatPromptTemplate(
    messages=[
        SystemMessagePromptTemplate.from_template("You are a friendly assistant."),
        MessagesPlaceholder(variable_name="chat_history"),
        HumanMessagePromptTemplate.from_template("{question}"),
    ]
)

# 2. Configure Memory
# return_messages=True is essential for chat models
memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)

# 3. Build the Chain
conversation = LLMChain(llm=llm, prompt=prompt, memory=memory)

# 4. Run the Conversation
response = conversation.predict(question="Hi, I'm Jane.")
print(response)

response = conversation.predict(question="Do you remember my name?")
print(response)
```

## Практический пример кода: Управление памятью в LangChain и LangGraph

В LangChain и LangGraph память — критический компонент для создания интеллектуальных разговорных приложений. Она позволяет ИИ-агенту помнить информацию прошлых взаимодействий, учиться на обратной связи и адаптироваться к предпочтениям пользователя.

**Краткосрочная память:** Привязана к ветке диалога, отслеживает текущую беседу в рамках сессии. Предоставляет непосредственный контекст, но полная история может переполнять контекстное окно LLM. LangGraph управляет краткосрочной памятью как частью состояния агента с персистентностью через чекпоинтер.

**Долгосрочная память:** Хранит пользовательские или прикладные данные между сессиями, общие между потоками. Сохраняется в пользовательских «namespace» и доступна в любом потоке.

LangChain предоставляет инструменты для управления историей диалога:

**ChatMessageHistory: Ручное управление памятью.** Для прямого контроля вне формальной цепочки:

```python
# Node that updates the agent's instructions
def update_instructions(state: State, store: BaseStore):
    namespace = ("instructions",)

    # Get the current instructions from the store
    current_instructions = store.search(namespace)[0]

    # Create a prompt to ask the LLM to reflect on the conversation
    # and generate new, improved instructions
    prompt = prompt_template.format(
        instructions=current_instructions.value["instructions"],
        conversation=state["messages"],
    )

    # Get the new instructions from the LLM
    output = llm.invoke(prompt)
    new_instructions = output["new_instructions"]

    # Save the updated instructions back to the store
    store.put(("agent_instructions",), "agent_a", {"instructions": new_instructions})


# Node that uses the instructions to generate a response
def call_model(state: State, store: BaseStore):
    namespace = ("agent_instructions",)

    # Retrieve the latest instructions from the store
    instructions = store.get(namespace, key="agent_a")[0]

    # Use the retrieved instructions to format the prompt
    prompt = prompt_template.format(
        instructions=instructions.value["instructions"]
    )
    # ... application logic continues
```

**ConversationBufferMemory: Автоматизированная память для цепочек.** Для интеграции памяти в цепочки. Параметры:

* `memory_key`: Имя переменной в промпте для хранения истории (по умолчанию «history»).
* `return_messages`: Формат истории — `False` (строка) или `True` (список сообщений, рекомендуется для Chat-моделей).

```python
from langgraph.store.memory import InMemoryStore


# A placeholder for a real embedding function
def embed(texts: list[str]) -> list[list[float]]:
    # In a real application, use a proper embedding model
    return [[1.0, 2.0] for _ in texts]


# Initialize an in-memory store. For production, use a database-backed store.
store = InMemoryStore(index={"embed": embed, "dims": 2})

# Define a namespace for a specific user and application context
user_id = "my-user"
application_context = "chitchat"
namespace = (user_id, application_context)

# 1. Put a memory into the store
store.put(
    namespace,
    "a-memory",  # The key for this memory
    {
        "rules": [
            "User likes short, direct language",
            "User only speaks English & python",
        ],
        "my-key": "my-value",
    },
)

# 2. Get the memory by its namespace and key
item = store.get(namespace, "a-memory")
print("Retrieved Item:", item)

# 3. Search for memories within the namespace, filtering by content
# and sorting by vector similarity to the query.
items = store.search(
    namespace,
    filter={"my-key": "my-value"},
    query="language preferences",
)
print("Search Results:", items)
```

Интеграция памяти в LLMChain позволяет модели обращаться к истории диалога и отвечать контекстно.

```python
from google.adk.memory import VertexAiMemoryBankService


agent_engine_id = agent_engine.api_resource.name.split("/")[-1]

memory_service = VertexAiMemoryBankService(
    project="PROJECT_ID",
    location="LOCATION",
    agent_engine_id=agent_engine_id,
)

session = await session_service.get_session(
    app_name=app_name,
    user_id="USER_ID",
    session_id=session.id,
)

await memory_service.add_session_to_memory(session)
```

Для chat-моделей рекомендуется использовать структурированный список сообщений с `return_messages=True`.

