# Глава 8: Управление памятью

Эффективное управление памятью критически важно для интеллектуальных агентов, которым необходимо сохранять информацию. Агентам, подобно людям, для эффективной работы требуются различные типы памяти. Эта глава рассматривает управление памятью, в частности — непосредственные (краткосрочные) и персистентные (долгосрочные) потребности агентов в памяти.

В агентных системах память означает способность агента сохранять и использовать информацию из прошлых взаимодействий, наблюдений и опыта обучения. Эта возможность позволяет агентам принимать обоснованные решения, поддерживать контекст диалога и совершенствоваться со временем. Память агента обычно классифицируется на два основных типа:

* **Краткосрочная память (Контекстная память):** Аналогично рабочей памяти человека, она хранит информацию, которая обрабатывается в данный момент или была недавно использована. Для агентов на основе больших языковых моделей (LLM) краткосрочная память преимущественно существует в контекстном окне. Это окно содержит последние сообщения, ответы агента, результаты использования инструментов и рефлексии агента из текущего взаимодействия — всё, что информирует последующие ответы и действия LLM. Контекстное окно имеет ограниченную ёмкость, что ограничивает объём недавней информации, к которой агент может получить прямой доступ. Эффективное управление краткосрочной памятью предполагает сохранение наиболее релевантной информации в этом ограниченном пространстве, возможно, с помощью таких техник, как обобщение старых сегментов диалога или выделение ключевых деталей. Появление моделей с «длинным контекстным окном» просто расширяет размер этой краткосрочной памяти, позволяя хранить больше информации в рамках одного взаимодействия. Тем не менее, этот контекст по-прежнему эфемерен — он теряется по завершении сессии, а его обработка каждый раз может быть затратной и неэффективной. Следовательно, агентам требуются отдельные типы памяти для достижения истинной персистентности, извлечения информации из прошлых взаимодействий и построения прочной базы знаний.
* **Долгосрочная память (Персистентная память):** Выступает репозиторием информации, которую агенты должны сохранять между различными взаимодействиями, задачами или длительными периодами времени — подобно долгосрочным базам знаний. Данные обычно хранятся вне среды непосредственной обработки агента, как правило, в базах данных, графах знаний или векторных базах данных. В векторных БД информация преобразуется в числовые векторы, что позволяет агентам извлекать данные на основе семантического сходства, а не точного совпадения ключевых слов — процесс, известный как семантический поиск. Когда агенту требуется информация из долгосрочной памяти, он обращается к внешнему хранилищу, извлекает релевантные данные и интегрирует их в краткосрочный контекст для непосредственного использования, объединяя имеющиеся знания с текущим взаимодействием.

## Практические применения и сценарии использования

Управление памятью жизненно важно для агентов, которым необходимо отслеживать информацию и работать интеллектуально со временем. Это необходимо, чтобы агенты могли выходить за рамки базовых возможностей ответа на вопросы. Применения включают:

* **Чатботы и разговорный ИИ:** Поддержание потока диалога зависит от краткосрочной памяти. Чатботы должны помнить предыдущие вводы пользователя для предоставления связных ответов. Долгосрочная память позволяет чатботам вспоминать предпочтения пользователя, прошлые проблемы или предыдущие обсуждения, обеспечивая персонализированные и непрерывные взаимодействия.
* **Задачно-ориентированные агенты:** Агенты, управляющие многошаговыми задачами, нуждаются в краткосрочной памяти для отслеживания предыдущих шагов, текущего прогресса и общих целей. Эта информация может находиться в контексте задачи или временном хранилище. Долгосрочная память критически важна для доступа к специфическим пользовательским данным, отсутствующим в непосредственном контексте.
* **Персонализированный опыт:** Агенты, обеспечивающие индивидуальные взаимодействия, используют долгосрочную память для хранения и извлечения предпочтений пользователя, прошлого поведения и личной информации. Это позволяет агентам адаптировать свои ответы и предложения.
* **Обучение и улучшение:** Агенты могут совершенствовать свою производительность, обучаясь на прошлых взаимодействиях. Успешные стратегии, ошибки и новая информация сохраняются в долгосрочной памяти, что облегчает будущие адаптации. Агенты с обучением с подкреплением хранят выученные стратегии и знания подобным образом.
* **Извлечение информации (RAG):** Агенты, предназначенные для ответа на вопросы, обращаются к базе знаний — своей долгосрочной памяти, часто реализованной в рамках Retrieval Augmented Generation (RAG). Агент извлекает релевантные документы или данные для обоснования своих ответов.
* **Автономные системы:** Роботы или беспилотники нуждаются в памяти для хранения карт, маршрутов, местоположений объектов и выученных моделей поведения. Это включает краткосрочную память для непосредственного окружения и долгосрочную память для общих знаний об окружающей среде.

Память позволяет агентам поддерживать историю, учиться, персонализировать взаимодействия и управлять сложными, зависящими от времени задачами.

## Практический пример кода: Управление памятью в Google ADK

Google Agent Developer Kit (ADK) предлагает структурированный метод управления контекстом и памятью, включая компоненты для практического применения. Твёрдое понимание сессий (Session), состояния (State) и памяти (Memory) в ADK жизненно важно для построения агентов, которым необходимо сохранять информацию.

Подобно человеческим взаимодействиям, агентам требуется способность вспоминать предыдущие обмены для ведения связных и естественных диалогов. ADK упрощает управление контекстом через три базовых понятия и связанные с ними сервисы.

Каждое взаимодействие с агентом можно рассматривать как уникальную ветку диалога. Агентам может потребоваться доступ к данным из более ранних взаимодействий. ADK структурирует это следующим образом:

* **Session:** Отдельная ветка чата, логирующая сообщения и действия (Events) для конкретного взаимодействия, а также хранящая временные данные (State), относящиеся к этому диалогу.
* **State (session.state):** Данные, хранящиеся в рамках Session и содержащие информацию, относящуюся только к текущей активной ветке чата.
* **Memory:** Поисковый репозиторий информации, полученной из различных прошлых чатов или внешних источников, служащий ресурсом для извлечения данных за пределами непосредственного диалога.

ADK предоставляет выделенные сервисы для управления критически важными компонентами, необходимыми для построения сложных, сохраняющих состояние и контекстно-ориентированных агентов. SessionService управляет ветками чата (объектами Session), обрабатывая их инициацию, запись и завершение, тогда как MemoryService отвечает за хранение и извлечение долгосрочных знаний (Memory).

И SessionService, и MemoryService предлагают различные опции конфигурации, позволяя пользователям выбирать методы хранения в зависимости от потребностей приложения. In-memory варианты доступны для целей тестирования, хотя данные не сохраняются между перезапусками. Для персистентного хранения и масштабируемости ADK также поддерживает базы данных и облачные сервисы.

### Session: Отслеживание каждого диалога

Объект Session в ADK предназначен для отслеживания и управления отдельными ветками чата. При инициации диалога с агентом SessionService генерирует объект Session, представленный как `google.adk.sessions.Session`. Этот объект включает все данные, относящиеся к конкретной ветке чата: уникальные идентификаторы (`id`, `app_name`, `user_id`), хронологическую запись событий в виде объектов Event, хранилище для сессионно-специфичных временных данных, известное как state, и временную метку последнего обновления (`last_update_time`). Разработчики обычно взаимодействуют с объектами Session косвенно через SessionService. SessionService отвечает за управление жизненным циклом сессий диалога, включая инициацию новых сессий, возобновление предыдущих, запись активности сессии (включая обновления состояния), идентификацию активных сессий и управление удалением данных сессии. ADK предоставляет несколько реализаций SessionService с различными механизмами хранения для истории сессий и временных данных, таких как InMemorySessionService, который подходит для тестирования, но не обеспечивает персистентность данных между перезапусками приложения.

```python
# Example: Using InMemorySessionService 
# This is suitable for local development and testing where data 
# persistence across application restarts is not required. 
from google.adk.sessions import InMemorySessionService
session_service = InMemorySessionService()
```

Затем DatabaseSessionService — если вам нужно надёжное сохранение в базу данных, которой вы управляете.

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

Также доступен VertexAiSessionService, использующий инфраструктуру Vertex AI для масштабируемой продакшн-среды на Google Cloud.

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

Выбор подходящего SessionService критически важен, поскольку он определяет, как хранятся история взаимодействий агента и временные данные, а также степень их персистентности.

Каждый обмен сообщениями представляет собой циклический процесс: сообщение получено, Runner извлекает или устанавливает Session с помощью SessionService, агент обрабатывает сообщение, используя контекст Session (состояние и исторические взаимодействия), агент генерирует ответ и может обновить состояние, Runner оборачивает это в Event, а метод `session_service.append_event` записывает новое событие и обновляет состояние в хранилище. Затем Session ожидает следующее сообщение. В идеале, метод `delete_session` используется для завершения сессии по окончании взаимодействия. Этот процесс показывает, как SessionService поддерживает континуальность, управляя историей Session и временными данными.

### State: Черновик сессии

В ADK каждая Session, представляющая ветку чата, включает компонент state, аналогичный временной рабочей памяти агента на время конкретного диалога. В то время как session.events логирует всю историю чата, session.state хранит и обновляет динамические данные, относящиеся к активному чату.

По сути, session.state работает как словарь, хранящий данные в формате ключ-значение. Его основная функция — позволять агенту сохранять и управлять деталями, необходимыми для связного диалога: предпочтениями пользователя, прогрессом задачи, инкрементальным сбором данных или условными флагами, влияющими на последующие действия агента.

Структура состояния включает строковые ключи в паре со значениями сериализуемых Python-типов, включая строки, числа, булевы значения, списки и словари, содержащие эти базовые типы. Состояние динамично и эволюционирует на протяжении диалога. Постоянство этих изменений зависит от настроенного SessionService.

Организация состояния может достигаться с помощью префиксов ключей для определения области видимости и персистентности данных. Ключи без префиксов специфичны для сессии.

* Префикс `user:` связывает данные с идентификатором пользователя во всех сессиях.
* Префикс `app:` обозначает данные, общие для всех пользователей приложения.
* Префикс `temp:` указывает на данные, действительные только для текущего хода обработки, которые не сохраняются персистентно.

Агент обращается ко всем данным состояния через единый словарь session.state. SessionService обрабатывает извлечение данных, их слияние и персистентность. Состояние должно обновляться при добавлении Event в историю сессии через `session_service.append_event()`. Это обеспечивает корректное отслеживание, правильное сохранение в персистентных сервисах и безопасную обработку изменений состояния.

#### 1. Простой способ: Использование output_key (для текстовых ответов агента)

Это самый простой метод, если вы хотите просто сохранить финальный текстовый ответ агента непосредственно в состояние. При настройке LlmAgent укажите нужный вам output_key. Runner увидит это и автоматически создаст необходимые действия для сохранения ответа в состоянии при добавлении события. Рассмотрим пример кода, демонстрирующий обновление состояния через output_key.

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

За кулисами Runner видит ваш `output_key` и автоматически создаёт необходимые действия с `state_delta` при вызове `append_event`.

#### 2. Стандартный способ: Использование EventActions.state_delta (для более сложных обновлений)

Для случаев, когда вам нужно делать более сложные вещи — обновлять несколько ключей одновременно, сохранять данные, которые не являются просто текстом, обращаться к конкретным областям вроде `user:` или `app:`, или делать обновления, не привязанные к финальному текстовому ответу агента — вы будете вручную строить словарь изменений состояния (state_delta) и включать его в EventActions добавляемого Event. Рассмотрим пример:

```python
import time

from google.adk.tools.tool_context import ToolContext
from google.adk.sessions import InMemorySessionService


# --- Define the Recommended Tool-Based Approach ---
def log_user_login(tool_context: ToolContext) -> dict:
    """
    Updates the session state upon a user login event.
    This tool encapsulates all state changes related to a user login.

    Args:
        tool_context: Automatically provided by ADK, gives access to session state.

    Returns:
        A dictionary confirming the action was successful.
    """
    # Access the state directly through the provided context.
    state = tool_context.state

    # Get current values or defaults, then update the state.
    # This is much cleaner and co-locates the logic.
    login_count = state.get("user:login_count", 0) + 1
    state["user:login_count"] = login_count
    state["task_status"] = "active"
    state["user:last_login_ts"] = time.time()
    state["temp:validation_needed"] = True

    print("State updated from within the `log_user_login` tool.")

    return {
        "status": "success",
        "message": f"User login tracked. Total logins: {login_count}.",
    }


# --- Demonstration of Usage ---
# In a real application, an LLM Agent would decide to call this tool.
# Here, we simulate a direct call for demonstration purposes.

# 1. Setup
session_service = InMemorySessionService()
app_name, user_id, session_id = "state_app_tool", "user3", "session3"

session = session_service.create_session(
    app_name=app_name,
    user_id=user_id,
    session_id=session_id,
    state={"user:login_count": 0, "task_status": "idle"},
)

print(f"Initial state: {session.state}")

# 2. Simulate a tool call (in a real app, the ADK Runner does this)
# We create a ToolContext manually just for this standalone example.
from google.adk.tools.tool_context import InvocationContext

mock_context = ToolContext(
    invocation_context=InvocationContext(
        app_name=app_name,
        user_id=user_id,
        session_id=session_id,
        session=session,
        session_service=session_service,
    )
)

# 3. Execute the tool
log_user_login(mock_context)

# 4. Check the updated state
updated_session = session_service.get_session(app_name, user_id, session_id)
print(f"State after tool execution: {updated_session.state}")

# Expected output will show the same state change as the "Before" case,
# but the code organization is significantly cleaner and more robust.
```

Этот код демонстрирует подход на основе инструментов для управления состоянием сессии пользователя в приложении. Он определяет функцию *log_user_login*, которая выступает в роли инструмента. Этот инструмент отвечает за обновление состояния сессии при входе пользователя.
Функция принимает объект ToolContext, предоставляемый ADK, для доступа и модификации словаря состояния сессии. Внутри инструмента она увеличивает счётчик `user:login_count`, устанавливает `task_status` в «active», записывает `user:last_login_ts` (метку времени) и добавляет временный флаг `temp:validation_needed`.

Демонстрационная часть кода показывает, как этот инструмент может быть использован. Она настраивает in-memory сервис сессий и создаёт начальную сессию с предопределённым состоянием. Затем вручную создаётся ToolContext, имитирующий среду, в которой ADK Runner выполнил бы инструмент. Функция `log_user_login` вызывается с этим mock-контекстом. Наконец, код извлекает сессию снова, чтобы показать, что состояние было обновлено в результате выполнения инструмента. Цель — продемонстрировать, как инкапсуляция изменений состояния внутри инструментов делает код более чистым и организованным по сравнению с прямой манипуляцией состоянием вне инструментов.

Обратите внимание, что прямая модификация словаря `session.state` после извлечения сессии настоятельно не рекомендуется, поскольку это обходит стандартный механизм обработки событий. Такие прямые изменения не будут записаны в историю событий сессии, могут не быть сохранены выбранным `SessionService`, могут привести к проблемам конкурентности и не обновят важные метаданные, такие как временные метки. Рекомендуемые методы обновления состояния сессии — использование параметра `output_key` на `LlmAgent` (для финальных текстовых ответов агента) или включение изменений состояния в `EventActions.state_delta` при добавлении события через `session_service.append_event()`. `session.state` следует использовать в первую очередь для чтения существующих данных.

Итого: при проектировании состояния придерживайтесь простоты, используйте базовые типы данных, давайте ключам чистые имена с правильными префиксами, избегайте глубокой вложенности и всегда обновляйте состояние через процесс append_event.

## Memory: Долгосрочная база знаний через MemoryService

В агентных системах компонент Session хранит запись текущей истории чата (events) и временных данных (state), специфичных для одного диалога. Однако для агентов, которым необходимо сохранять информацию между множественными взаимодействиями или получать доступ к внешним данным, необходимо управление долгосрочными знаниями. Это обеспечивается MemoryService.

```python
# Example: Using InMemoryMemoryService
# This is suitable for local development and testing where data
# persistence across application restarts is not required.
# Memory content is lost when the app stops.

from google.adk.memory import InMemoryMemoryService

memory_service = InMemoryMemoryService()
```

Session и State можно рассматривать как краткосрочную память для одной сессии чата, тогда как Долгосрочные Знания, управляемые MemoryService, функционируют как персистентный и поисковый репозиторий. Этот репозиторий может содержать информацию из множества прошлых взаимодействий или внешних источников. MemoryService, определяемый интерфейсом BaseMemoryService, устанавливает стандарт для управления этим поисковым долгосрочным знанием. Его основные функции включают добавление информации — извлечение содержимого из сессии и его сохранение с помощью метода `add_session_to_memory` — и извлечение информации — позволяющее агенту запрашивать хранилище и получать релевантные данные с помощью метода `search_memory`.

ADK предлагает несколько реализаций для создания этого хранилища долгосрочных знаний. InMemoryMemoryService предоставляет временное решение хранения, подходящее для тестирования, но данные не сохраняются между перезапусками приложения. Для production-сред обычно используется VertexAiRagMemoryService. Этот сервис использует Google Cloud Retrieval Augmented Generation (RAG), обеспечивая масштабируемые, персистентные и семантические возможности поиска (см. также Главу 14 о RAG).

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


## Практический пример кода: Управление памятью в LangChain и LangGraph

В LangChain и LangGraph память является критическим компонентом для создания интеллектуальных и естественно-звучных разговорных приложений. Она позволяет AI-агенту запоминать информацию из прошлых взаимодействий, учиться на обратной связи и адаптироваться к предпочтениям пользователя. Функция памяти LangChain обеспечивает основу для этого, обращаясь к сохранённой истории для обогащения текущих промптов и записи последнего обмена для будущего использования. По мере того как агенты обрабатывают более сложные задачи, эта возможность становится необходимой как для эффективности, так и для удовлетворённости пользователя.

**Краткосрочная память:** Привязана к ветке (thread-scoped), что означает отслеживание текущего диалога в рамках одной сессии или ветки. Она обеспечивает непосредственный контекст, но полная история может создавать нагрузку на контекстное окно LLM, что потенциально приводит к ошибкам или снижению производительности. LangGraph управляет краткосрочной памятью как частью состояния агента, которая сохраняется через чекпоинтер, позволяя ветке быть возобновлённой в любое время.

**Долгосрочная память:** Хранит пользовательские или прикладные данные между сессиями и является общей между ветками диалога. Сохраняется в пользовательских «namespace» и может быть извлечена в любое время в любой ветке. LangGraph предоставляет хранилища для сохранения и извлечения долгосрочных воспоминаний, позволяя агентам сохранять знания бессрочно.

LangChain предоставляет несколько инструментов для управления историей диалога, от ручного контроля до автоматизированной интеграции в цепочки.

**ChatMessageHistory: Ручное управление памятью.** Для прямого и простого контроля над историей диалога вне формальной цепочки класс ChatMessageHistory является идеальным. Он позволяет вручную отслеживать обмены диалога.

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

**ConversationBufferMemory: Автоматизированная память для цепочек.** Для интеграции памяти непосредственно в цепочки ConversationBufferMemory является распространённым выбором. Он хранит буфер диалога и делает его доступным вашему промпту. Его поведение может быть настроено с помощью двух ключевых параметров:

* `memory_key`: Строка, определяющая имя переменной в вашем промпте, которая будет содержать историю чата. По умолчанию — «history».
* `return_messages`: Логическое значение, определяющее формат истории.
  * Если `False` (по умолчанию), возвращает одну отформатированную строку, что идеально для стандартных LLM.
  * Если `True`, возвращает список объектов сообщений — рекомендуемый формат для Chat-моделей.

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

Интеграция этой памяти в LLMChain позволяет модели обращаться к истории диалога и предоставлять контекстно-релевантные ответы.

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

Для повышения эффективности при работе с chat-моделями рекомендуется использовать структурированный список объектов сообщений, установив `return_messages=True`.

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

**Типы долгосрочной памяти:** Долгосрочная память позволяет системам сохранять информацию между различными диалогами, обеспечивая более глубокий уровень контекста и персонализации. Она может быть разбита на три типа, аналогичных человеческой памяти:

* **Семантическая память: Запоминание фактов.** Предполагает сохранение конкретных фактов и концепций, таких как предпочтения пользователя или доменные знания. Используется для обоснования ответов агента, что приводит к более персонализированным и релевантным взаимодействиям. Эта информация может управляться как постоянно обновляемый «профиль» пользователя (JSON-документ) или как «коллекция» отдельных фактических документов.
* **Эпизодическая память: Запоминание опыта.** Предполагает воспроизведение прошлых событий или действий. Для AI-агентов эпизодическая память часто используется для запоминания того, как accomplish задачу. На практике frequently реализуется через few-shot prompting, где агент учится из прошлых успешных последовательностей взаимодействия для правильного выполнения задач.
* **Процедурная память: Запоминание правил.** Это память о том, как выполнять задачи — базовые инструкции и поведения агента, часто содержащиеся в его системном промпте. Агентам свойственно модифицировать свои собственные промпты для адаптации и улучшения. Эффективная техника — «Рефлексия», при которой агент получает свои текущие инструкции и недавние взаимодействия, а затем получает задание улучшить свои собственные инструкции.

Ниже приведён псевдокод, демонстрирующий, как агент может использовать рефлексию для обновления своей процедурной памяти, хранящейся в LangGraph BaseStore.

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

LangGraph хранит долгосрочные воспоминания как JSON-документы в хранилище. Каждое воспоминание организуется под пользовательским namespace (подобно папке) и уникальным ключом (подобно имени файла). Эта иерархическая структура позволяет легко организовывать и извлекать информацию. Следующий код демонстрирует использование InMemoryStore для записи, получения и поиска воспоминаний.

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




## Vertex Memory Bank

Memory Bank — управляемый сервис в Vertex AI Agent Engine, обеспечивающий агентов персистентной долгосрочной памятью. Сервис использует модели Gemini для асинхронного анализа историй диалогов и извлечения ключевых фактов и предпочтений пользователя.

Эта информация сохраняется персистентно, организуется по определённому scope (например, ID пользователя) и интеллектуально обновляется для консолидации новых данных и разрешения противоречий. При начале новой сессии агент извлекает relevant воспоминания либо через полное извлечение данных, либо через семантический поиск с использованием эмбеддингов. Этот процесс позволяет агенту поддерживать континуальность между сессиями и персонализировать ответы на основе извлечённой информации.

Runner агента взаимодействует с VertexAiMemoryBankService, который инициализируется первым. Этот сервис отвечает за автоматическое хранение воспоминаний, генерируемых во время диалогов агента. Каждое воспоминание маркируется уникальным USER_ID и APP_NAME, обеспечивая точное извлечение в будущем.

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

Memory Bank предлагает бесшовную интеграцию с Google ADK, обеспечивая немедленный out-of-the-box опыт. Для пользователей других агентных фреймворков, таких как LangGraph и CrewAI, Memory Bank также предлагает поддержку через прямые вызовы API. Примеры кода в сети, демонстрирующие эти интеграции, readily available для заинтересованных читателей.


## Краткий обзор

**Что:** Агентным системам необходимо запоминать информацию из прошлых взаимодействий для выполнения сложных задач и обеспечения связного пользовательского опыта. Без механизма памяти агенты являются stateless — неспособными поддерживать контекст диалога, учиться на опыте или персонализировать ответы. Это фундаментально ограничивает их простыми одноразовыми взаимодействиями, не позволяя обрабатывать многошаговые процессы или изменяющиеся потребности пользователя. Основная проблема — как эффективно управлять как непосредственной временной информацией одного диалога, так и обширными персистентными знаниями, накопленными со временем.

**Почему:** Стандартизированное решение — реализация двухкомпонентной системы памяти, различающей краткосрочное и долгосрочное хранение. Краткосрочная контекстная память хранит данные недавних взаимодействий в контекстном окне LLM для поддержания потока диалога. Для информации, которая должна сохраняться, решения долгосрочной памяти используют внешние базы данных, часто векторные хранилища, для эффективного семантического извлечения. Агентные фреймворки, такие как Google ADK, предоставляют специфические компоненты для управления этим: Session для ветки диалога и State для её временных данных. Выделенный MemoryService используется для интерфейса с базой долгосрочных знаний, позволяя агенту извлекать и включать relevant прошлую информацию в текущий контекст.

**Когда использовать:** Когда агенту нужно делать больше, чем отвечать на один вопрос. Это необходимо для агентов, которые должны поддерживать контекст на протяжении диалога, отслеживать прогресс в многошаговых задачах или персонализировать взаимодействия, вспоминая предпочтения пользователя и историю. Реализуйте управление памятью, когда от агента ожидается обучение или адаптация на основе прошлых успехов, неудач или новой приобретённой информации.

**Визуальное резюме:**

![Паттерн управления памятью](../assets/Memory_Management_Design_Pattern.png)

Рис. 1: Паттерн «Управление памятью»

## Ключевые выводы

Для быстрого обзора основных моментов об управлении памятью:

* Память чрезвычайно важна для агентов, чтобы отслеживать вещи, учиться и персонализировать взаимодействия.
* Разговорный ИИ опирается как на краткосрочную память для непосредственного контекста в рамках одного чата, так и на долгосрочную для персистентных знаний между множественными сессиями.
* Краткосрочная память (непосредственная информация) временна, часто ограничена контекстным окном LLM или тем, как фреймворк передаёт контекст.
* Долгосрочная память (постоянная информация) сохраняет данные между различными чатами, используя внешние хранилища вроде векторных баз данных, и доступна через поиск.
* Фреймворки, такие как ADK, имеют специфические компоненты: Session (ветка чата), State (временные данные чата) и MemoryService (поисковое долгосрочное знание) для управления памятью.
* SessionService в ADK управляет полным жизненным циклом сессии чата, включая её историю (events) и временные данные (state).
* session.state в ADK — словарь для временных данных чата. Префиксы (user:, app:, temp:) указывают, к какой области принадлежат данные и сохраняются ли они.
* В ADK следует обновлять состояние с помощью EventActions.state_delta или output_key при добавлении событий, а не изменять словарь state напрямую.
* MemoryService в ADK предназначен для помещения информации в долгосрочное хранилище и предоставления агентам возможности искать её, часто с помощью инструментов.
* LangChain предлагает практические инструменты, такие как ConversationBufferMemory, для автоматической инъекции истории одного диалога в промпт, позволяя агенту извлекать непосредственный контекст.
* LangGraph обеспечивает продвинутую долгосрочную память с помощью хранилища для сохранения и извлечения семантических фактов, эпизодического опыта или даже обновляемых процедурных правил между различными пользовательскими сессиями.
* Memory Bank — управляемый сервис, обеспечивающий агентов персистентной долгосрочной памятью через автоматическое извлечение, хранение и извлечение пользовательской информации для персонализированных непрерывных диалогов в рамках фреймворков, таких как Google ADK, LangGraph и CrewAI.

## Заключение

Эта глава углубилась в действительно важную задачу управления памятью для агентных систем, показывая разницу между краткосрочным контекстом и знаниями, сохраняющимися на длительное время. Мы обсудили, как эти типы памяти устроены и где они применяются при построении более умных агентов, способных запоминать. Мы подробно рассмотрели, как Google ADK предоставляет конкретные компоненты — Session, State и MemoryService — для управления этим. Теперь, когда мы рассмотрели, как агенты могут запоминать — как в краткосрочной, так и в долгосрочной перспективе — мы можем перейти к тому, как они могут учиться и адаптироваться. Следующий паттерн «Обучение и адаптация» посвящён тому, как агент изменяет свой способ мышления, действий или свои знания на основе нового опыта или данных.

## Ссылки

1. ADK Memory: [https://google.github.io/adk-docs/sessions/memory/](https://google.github.io/adk-docs/sessions/memory/)
2. LangGraph Memory: [https://langchain-ai.github.io/langgraph/concepts/memory/](https://langchain-ai.github.io/langgraph/concepts/memory/)
3. Vertex AI Agent Engine Memory Bank: [https://cloud.google.com/blog/products/ai-machine-learning/vertex-ai-memory-bank-in-public-preview](https://cloud.google.com/blog/products/ai-machine-learning/vertex-ai-memory-bank-in-public-preview)
