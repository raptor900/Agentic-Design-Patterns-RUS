# Глава 8: Управление памятью

Эффективное управление памятью критически важно для интеллектуальных агентов, которым необходимо сохранять информацию. Агентам, подобно людям, требуются различные типы памяти для эффективной работы. Эта глава рассматривает управление памятью, конкретно addressing непосредственные (краткосрочные) и персистентные (долгосрочные) потребности агентов в памяти.

В агентных системах память refers to способность агента сохранять и использовать информацию из прошлых взаимодействий, наблюдений и опыта обучения. Эта возможность позволяет агентам принимать информированные решения, поддерживать контекст диалога и улучшаться со временем. Память агента generally классифицируется на два основных типа:

* **Краткосрочная память (Контекстная память):** Similar to рабочей памяти, она хранит информацию, которая обрабатывается в данный момент или недавно accessed. Для агентов, использующих большие языковые модели (LLM), краткосрочная память primarily существует в контекстном окне. Это окно содержит последние сообщения, ответы агента, результаты использования инструментов и рефлексии агента из текущего взаимодействия, всё из которых informs последующие ответы и действия LLM. Контекстное окно имеет ограниченную ёмкость, restricting количество недавней информации, к которой агент может напрямую получить доступ. Эффективное управление краткосрочной памятью involves сохранение наиболее релевантной информации в этом ограниченном пространстве, possibly через техники, такие как обобщение старых сегментов диалога или emphasizing ключевые детали. Появление моделей с «длинным контекстным окном» simply расширяет размер этой краткосрочной памяти, allowing больше информации в рамках одного взаимодействия. However, этот контекст по-прежнему эфемерен — он теряется по завершении сессии, и его обработка может быть costly и неэффективной каждый раз. Consequently, агентам требуются отдельные типы памяти для достижения истинной персистентности, recall информации из прошлых взаимодействий и построения lasting базы знаний.
* **Долгосрочная память (Персистентная память):** Она acts as репозиторий информации, которую агенты должны сохранять между various взаимодействиями, задачами или длительными периодами, akin to долгосрочным базам знаний. Данные typically хранятся outside среды непосредственной обработки агента, often в базах данных, графах знаний или векторных базах данных. В векторных БД информация преобразуется в числовые векторы и хранится, enabling агентам извлекать данные на основе семантического сходства rather than точного совпадения ключевых слов — процесс, known as семантический поиск. Когда агенту нужна информация из долгосрочной памяти, он запрашивает внешнее хранилище, извлекает релевантные данные и интегрирует их в краткосрочный контекст для immediate использования, thus combining prior знание с текущим взаимодействием.

## Практические применения и сценарии использования

Управление памятью vitally важно для агентов, которым необходимо отслеживать информацию и работать интеллектуально со временем. Это essential для агентов, чтобы surpassing базовые возможности ответа на вопросы. Применения включают:

* **Чатботы и разговорный ИИ:** Поддержание потока диалога relies on краткосрочную память. Чатботы require запоминания предыдущих пользовательских вводов для предоставления связных ответов. Долгосрочная память enables чатботам вспоминать предпочтения пользователя, прошлые проблемы или предыдущие обсуждения, offering персонализированные и непрерывные взаимодействия.
* **Задачно-ориентированные агенты:** Агенты, управляющие многошаговыми задачами, нуждаются в краткосрочной памяти для отслеживания предыдущих шагов, текущего прогресса и общих целей. Эта информация might resides в контексте задачи или временном хранилище. Долгосрочная память crucial для доступа к специфическим пользовательским данным, отсутствующим в непосредственном контексте.
* **Персонализированный опыт:** Агенты, offering tailored взаимодействия, используют долгосрочную память для хранения и извлечения предпочтений пользователя, прошлого поведения и личной информации. Это allows агентам адаптировать свои ответы и предложения.
* **Обучение и улучшение:** Агенты могут refine свою производительность, обучаясь на прошлых взаимодействиях. Успешные стратегии, ошибки и новая информация сохраняются в долгосрочной памяти, facilitating будущие адаптации. Агенты с обучением с подкреплением хранят выученные стратегии или знания этим способом.
* **Извлечение информации (RAG):** Агенты, designed для ответа на вопросы, обращаются к базе знаний — their долгосрочной памяти, often реализованной within Retrieval Augmented Generation (RAG). Агент извлекает релевантные документы или данные для информирования своих ответов.
* **Автономные системы:** Роботы или беспилотники require память для карт, маршрутов, местоположений объектов и выученных поведений. This involves краткосрочную память для immediate окружения и долгосрочную память для общих environmental знаний.

Память enables агентам поддерживать историю, учиться, персонализировать взаимодействия и управлять сложными, зависящими от времени задачами.
```python
# Example: Using InMemorySessionService 
# This is suitable for local development and testing where data 
# persistence across application restarts is not required. 
from google.adk.sessions import InMemorySessionService
session_service = InMemorySessionService()
```## Практический пример кода: Управление памятью в Google ADK

Google Agent Developer Kit (ADK) предлагает структурированный метод управления контекстом и памятью, including компоненты для практического применения. Твёрдое понимание Session, State и Memory в ADK vitally важно для построения агентов, которым необходимо сохранять информацию.

Подобно человеческим взаимодействиям, агентам требуется ability вспоминать предыдущие обмены для ведения связных и естественных диалогов. ADK упрощает управление контекстом через три базовых concept и их associated сервисы.

Каждое взаимодействие с агентом может рассматриваться как уникальная ветка диалога. Агентам might потребуется доступ к данным из более ранних взаимодействий. ADK структурирует это следующим образом:

* **Session:** Отдельная ветка чата, логирующая сообщения и действия (Events) для specific взаимодействия, а также хранящая временные данные (State), relevant к этому диалогу.
* **State (`session.state`):** Данные, stored within Session, содержащие информацию, relevant только для текущей активной ветки чата.
* **Memory:** Поисковый репозиторий информации, sourced from various прошлых чатов или внешних источников, serving как ресурс для извлечения данных beyond непосредственного диалога.

ADK provides выделенные сервисы для управления critical компонентами, essential для построения сложных, stateful и контекстно-ориентированных агентов. SessionService manages ветки чата (объекты Session), handling их инициацию, запись и завершение, while MemoryService oversees хранение и извлечение долгосрочных знаний (Memory).

Both SessionService и MemoryService предлагают various опции конфигурации, allowing пользователям выбирать методы хранения based on потребностей приложения. In-memory опции available для тестирования, though данные не сохраняются между перезапусками. Для persistent хранения и масштабируемости ADK также supports базы данных и облачные сервисы.

### Session: Отслеживание каждого диалога

Объект Session в ADK designed для отслеживания и управления отдельными ветками чата. При инициации диалога с агентом SessionService генерирует объект Session, represented as `google.adk.sessions.Session`. Этот объект encapsulates все данные, relevant к конкретной ветке чата, including уникальные идентификаторы (`id`, `app_name`, `user_id`), хронологическую запись событий как объектов Event, хранилище для сессионно-специфичных временных данных, known as state, и временную метку, indicating последнее обновление (`last_update_time`). Разработчики typically взаимодействуют с объектами Session indirectly через SessionService. SessionService responsible for управление жизненным циклом сессий диалога, which включает инициацию новых сессий, возобновление предыдущих, запись активности сессии (including обновления состояния), идентификацию активных сессий и управление удалением данных сессии. ADK provides несколько реализаций SessionService с varying механизмами хранения для истории сессий и временных данных, such as InMemorySessionService, который suitable для тестирования, но не обеспечивает персистентность данных между перезапусками приложения.

```python
# Example: Using DatabaseSessionService 
# This is suitable for production or development requiring persistent storage. 
# You need to configure a database URL (e.g., for SQLite, PostgreSQL, etc.). 
# Requires: pip install google-adk[sqlalchemy] and a database driver (e.g., psycopg2 for PostgreSQL) 
from google.adk.sessions import DatabaseSessionService 
# Example using a local SQLite file: 
db_url = "sqlite:///./my_agent_data.db"
session_service = DatabaseSessionService(db_url=db_url)
```Затем DatabaseSessionService, если вам нужно надёжное сохранение в базу данных, которой вы управляете.

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
```Кроме того, есть VertexAiSessionService, использующий инфраструктуру Vertex AI для масштабируемого production на Google Cloud.

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
```Выбор подходящего SessionService is crucial, так как он определяет, как хранятся история взаимодействий агента и временные данные, а также их персистентность.

Каждый обмен сообщениями involves циклический процесс: Сообщение received, Runner извлекает или устанавливает Session с помощью SessionService, агент обрабатывает сообщение using контекст Session (state и исторические взаимодействия), агент генерирует ответ и may обновить state, Runner encapsulates это как Event, и метод `session_service.append_event` записывает новое событие и обновляет состояние в хранилище. Затем Session ожидает следующее сообщение. Ideally, метод `delete_session` используется для завершения сессии по окончании взаимодействия. Этот процесс illustrates, как SessionService поддерживает континуальность, управляя историей Session и временными данными.

### State: Черновик сессии

В ADK каждая Session, representing ветку чата, включает компонент state, akin to временной рабочей памяти агента на duration конкретного диалога. While session.events логирует всю историю чата, session.state хранит и обновляет динамические точки данных, relevant активному чату.

Fundamentally, session.state работает как словарь, хранящий данные в формате ключ-значение. Его core функция — enabling агенту сохранять и управлять деталями, essential для связного диалога, such as предпочтения пользователя, прогресс задачи, инкрементальный сбор данных или условные флаги, influencing последующие действия агента.

Структура состояния comprises строковые ключи, paired с значениями сериализуемых Python-типов, including строки, числа, булевы значения, списки и словари, containing эти базовые типы. State является динамическим, evolving throughout диалог. Постоянство этих изменений depends on настроенного SessionService.

Организация state может достигаться using префиксы ключей для определения scope и persistence данных. Ключи без префиксов are сессионно-специфичными.

* Префикс `user:` ассоциирует данные с ID пользователя across все сессии.
* Префикс `app:` обозначает данные, shared среди всех пользователей приложения.
* Префикс `temp:` indicates данные, valid только для текущего хода обработки, и не сохраняются persistent.

Агент обращается ко всем данным state through единый словарь session.state. SessionService handles извлечение данных, слияние и персистентность. State should обновляться при добавлении Event в историю сессии через `session_service.append_event()`. This ensures accurate отслеживание, proper сохранение в persistent сервисах и safe handling изменений состояния.

#### 1. Простой способ: Использование `output_key` (для текстовых ответов агента)

This is самый простой метод, если вы просто хотите сохранить финальный текстовый ответ агента directly в state. When вы настраиваете LlmAgent, simply укажите output_key, который хотите использовать. Runner видит это и automatically создаёт необходимые действия для сохранения ответа в state, когда добавляет event. Рассмотрим пример кода, демонстрирующий обновление состояния через `output_key`.

```python
# Example: Using InMemoryMemoryService
# This is suitable for local development and testing where data
# persistence across application restarts is not required.
# Memory content is lost when the app stops.

from google.adk.memory import InMemoryMemoryService

memory_service = InMemoryMemoryService()
```Behind the scenes, Runner видит ваш `output_key` и automatically создаёт необходимые действия с `state_delta` при вызове `append_event`.

#### 2. Стандартный способ: Использование `EventActions.state_delta` (для более сложных обновлений)

For случаев, когда вам нужно делать более сложные вещи — such as обновлять несколько ключей одновременно, сохранять things, которые не являются просто текстом, targeting специфические scope вроде `user:` или `app:`, или делать обновления, которые не привязаны к финальному текстовому ответу агента — вы будете manually строить словарь ваших изменений состояния (the `state_delta`) и включать его в EventActions Event, который вы добавляете. Рассмотрим пример:

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
```Этот код demonstrates подход на основе инструментов для управления состоянием сессии пользователя в приложении. Он определяет функцию *log_user_login*, которая acts as инструмент. This tool responsible for обновления состояния сессии при входе пользователя.
Функция принимает объект ToolContext, предоставленный ADK, для доступа и модификации словаря состояния сессии. Inside инструмент, она increment `user:login_count`, устанавливает `task_status` в "active", записывает `user:last_login_ts` (timestamp) и добавляет временный флаг `temp:validation_needed`.

Демонстрационная часть кода simulates, как этот инструмент would быть использован. Она настраивает in-memory сервис сессий и создаёт начальную сессию с some предопределённым состоянием. ToolContext затем manually создаётся для mimic среды, в которой ADK Runner would выполнять инструмент. Функция `log_user_login` вызывается с этим mock контекстом. Finally, код извлекает сессию снова, чтобы показать, что состояние было updated в результате выполнения инструмента.

Обратите внимание, что прямая модификация словаря `session.state` после извлечения сессии strongly discouraged, так как это bypasses стандартный механизм обработки событий. Such прямые изменения не будут recorded в истории событий сессии, may не быть persisted выбранным `SessionService`, could привести к проблемам concurrency и не обновит essential метаданные, such as временные метки. Рекомендуемые методы обновления состояния сессии: использование параметра `output_key` на `LlmAgent` (specifically для финальных текстовых ответов агента) или включение изменений состояния в `EventActions.state_delta` при добавлении события через `session_service.append_event()`. `session.state` should primarily использоваться для чтения существующих данных.

Итого: при проектировании состояния — простота, базовые типы данных, чистые имена ключей с правильными префиксами, избегание глубокой вложенности и always обновление состояния через процесс append_event.

## Memory: Долгосрочная база знаний через MemoryService

В агентных системах компонент Session maintains запись текущей истории чата (events) и временных данных (state), specific к одному диалогу. However, для агентов, которым необходимо сохранять информацию across multiple взаимодействий или получать доступ к внешним данным, необходимо управление долгосрочными знаниями. This facilitated MemoryService.

```python
from langchain.memory import ChatMessageHistory


# Initialize the history object
history = ChatMessageHistory()

# Add user and AI messages
history.add_user_message("I'm heading to New York next week.")
history.add_ai_message("Great! It's a fantastic city.")

# Access the list of messages
print(history.messages)
```Session и State могут быть conceptualized как краткосрочная память для одной сессии чата, while Долгосрочные Знания, managed MemoryService, function как persistent и поисковый репозиторий. This репозиторий may содержать информацию из multiple прошлых взаимодействий или внешних источников. MemoryService, as определённый интерфейсом BaseMemoryService, establishes стандарт для управления этим поисковым, долгосрочным знанием. Его primary функции включают добавление информации — which involves извлечение содержимого из сессии и его сохранение using метод `add_session_to_memory` — и извлечение информации — which позволяет агенту запрашивать хранилище и получать релевантные данные using метод `search_memory`.

ADK offers несколько реализаций для создания этого хранилища долгосрочных знаний. InMemoryMemoryService provides временное решение хранения, suitable для тестирования, but данные не сохраняются между перезапусками приложения. Для production сред VertexAiRagMemoryService typically используется. This сервис leverages Google Cloud Retrieval Augmented Generation (RAG), enabling масштабируемые, persistent и семантические возможности поиска (также см. Главу 14 о RAG).

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
```## Практический пример кода: Управление памятью в LangChain и LangGraph

В LangChain и LangGraph память — critical компонент для создания интеллектуальных и естественно-звучных разговорных приложений. Она позволяет AI-агенту запоминать информацию из прошлых взаимодействий, учиться на обратной связи и адаптироваться к предпочтениям пользователя. Функция памяти LangChain provides основу для этого, referencing сохранённую историю для обогащения текущих промптов и затем записи последнего обмена для будущего использования. По мере того как агенты обрабатывают более сложные задачи, эта возможность becomes essential для both эффективности и удовлетворённости пользователя.

**Краткосрочная память:** Это привязано к ветке (thread-scoped), meaning оно отслеживает текущий диалог within одной сессии или ветки. Оно provides immediate контекст, но полная история может challenging контекстное окно LLM, potentially leading к ошибкам или плохой производительности. LangGraph manages краткосрочную память как часть состояния агента, which persisted через чекпоинтер, allowing ветку быть resumed в любое время.

**Долгосрочная память:** Она хранит пользовательские или прикладные данные across сессиями и shared между ветками диалога. Она saved в кастомных «namespace» и может быть recalled в любое время в любой ветке. LangGraph provides хранилища для сохранения и извлечения долгосрочных воспоминаний, enabling агентам сохранять знания indefinitely.

LangChain provides несколько инструментов для управления историей диалога, ranging от ручного контроля до автоматизированной интеграции within цепочками.

**ChatMessageHistory: Ручное управление памятью.** Для прямого и простого контроля над историей диалога outside формальной цепочки класс ChatMessageHistory is идеален. Он позволяет manually отслеживать обмены диалога.

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
```**ConversationBufferMemory: Автоматизированная память для цепочек.** Для интеграции памяти directly в цепочки ConversationBufferMemory is общепринятый выбор. Он хранит буфер диалога и делает его available вашему промпту. Его поведение может быть customized с помощью двух ключевых параметров:

* `memory_key`: Строка, specifying имя переменной в вашем промпте, которая будет содержать историю чата. По умолчанию — "history".
* `return_messages`: Boolean, определяющий формат истории.
  * Если `False` (по умолчанию), возвращает одну отформатированную строку, ideal для стандартных LLM.
  * Если `True`, возвращает список объектов сообщений, recommended формат для Chat-моделей.

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
```Интеграция этой памяти в LLMChain позволяет модели обращаться к истории диалога и provides контекстно-релевантные ответы.

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
```Для улучшенной эффективности с chat-моделями рекомендуется использовать структурированный список объектов сообщений, установив `return_messages=True`.

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
```**Типы долгосрочной памяти:** Долгосрочная память позволяет системам сохранять информацию across different диалогами, providing более глубокий уровень контекста и персонализации. Она может быть разбита на три типа, analogous к человеческой памяти:

* **Семантическая память: Запоминание фактов:** Включает сохранение конкретных фактов и концепций, such as предпочтения пользователя или доменные знания. Используется для grounding ответов агента, leading к более персонализированным и релевантным взаимодействиям. Эта информация может быть managed как continuously обновляемый «профиль» пользователя (JSON-документ) или как «коллекция» отдельных factual документов.
* **Эпизодическая память: Запоминание опыта:** Включает recall прошлых событий или действий. Для AI-агентов эпизодическая память often используется для запоминания, как accomplish задачу. На практике frequently реализуется через few-shot prompting, where агент учится из past успешных последовательностей взаимодействия для правильного выполнения задач.
* **Процедурная память: Запоминание правил:** Это память о том, как выполнять задачи — базовые инструкции и поведения агента, often contained в его системном промпте. Обычно для агентов модифицировать свои собственные промпты для адаптации и улучшения. Эффективная техника — «Рефлексия», where агент получает свои текущие инструкции и недавние взаимодействия, затем asked улучшить свои собственные инструкции.

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
```Ниже приведён псевдокод, demonstrating как агент might использовать рефлексию для обновления своей процедурной памяти, stored в LangGraph BaseStore.

