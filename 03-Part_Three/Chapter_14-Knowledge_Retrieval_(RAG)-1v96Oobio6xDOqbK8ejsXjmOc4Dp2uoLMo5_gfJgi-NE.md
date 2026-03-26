# Глава 14: Извлечение знаний (RAG)

LLM демонстрируют значительные возможности в генерации человекоподобного текста. Однако их база знаний обычно ограничена обучающими данными. Извлечение знаний (RAG, Retrieval Augmented Generation) решает это ограничение, позволяя LLM получать доступ к внешней, актуальной и контекстно-специфичной информации.

Для AI-агентов это критически важно: они могут grounding свои действия в актуальных проверяемых данных, выходящих за рамки статического обучения.

## Обзор паттерна «Извлечение знаний (RAG)»

RAG значительно расширяет возможности LLM, предоставляя доступ к внешним базам знаний перед генерацией ответа. Вместо полагания исключительно на внутренние знания, RAG позволяет LLM «искать» информацию.

Когда пользователь задаёт вопрос системе с RAG, запрос не отправляется напрямую к LLM. Сначала система выполняет «семантический поиск» по внешней базе знаний, находя релевантные фрагменты. Затем эти фрагменты «дополняют» (augment) исходный промпт. Расширенный промпт отправляется LLM, которая генерирует ответ, grounded в полученных данных.

Преимущества: доступ к актуальной информации, снижение галлюцинаций, использование специализированных знаний, цитирование источников.

### Эмбеддинги

В контексте LLM эмбеддинги — числовые представления текста в виде векторов. Схожие по смыслу тексты имеют близкие эмбеддинги. Например, «кошка» и «котёнок» рядом, «машина» далеко. В реальности — сотни или тысячи измерений.

### Текстовое сходство

Мера схожести двух фрагментов текста. В RAG — для поиска наиболее релевантной информации. «Какова столица Франции?» и «Какой город является столицей Франции?» — разные формулировки, один вопрос. Хорошая модель назначит высокий балл.

### Семантическое сходство и расстояние

Продвинутая форма, фокусирующаяся на значении и контексте. В RAG — поиск документов с наименьшим семантическим расстоянием.

![Базовые концепции RAG](../assets/RAG_Core_Concepts_Chunking_Embeddings_and_Vector_Database.png)

Рис. 1: Базовые концепции RAG: чанкинг, эмбеддинги, векторная БД

### Чанкинг документов

Разбиение больших документов на мелкие управляемые фрагменты (chunks). Способ чанкинга важен для сохранения контекста. После чанкинга — векторный поиск (с эмбеддингами) или BM25 (ключевые слова), или гибридный поиск.

### Векторные БД

Специализированные БД для хранения и запросов эмбеддингов. Семантический поиск вместо ключевых слов. Алгоритмы вроде HNSW для быстрого поиска среди миллионов векторов.

Реализации: Pinecone, Weaviate, Chroma DB, Milvus, Qdrant, Redis, Elasticsearch, Postgres (pgvector). Библиотеки: FAISS, ScaNN.

### Проблемы RAG

* Информация разбросана по нескольким чанкам/документам.
* Зависимость от качества чанкинга.
* Синтез противоречивых источников.
* Требуется предварительная обработка.
* Периодическая актуализация.
* Влияние на производительность.

### Graph RAG

Продвинутая форма с графом знаний вместо векторной БД. Синтез ответов из фрагментированной информации. Сложность и стоимость поддержки.

### Agentic RAG

Введение слоя рассуждения. Агент — gatekeeper и refiner знаний:

1. **Рефлексия и валидация:** Анализ метаданных, отбрасывание устаревших.
2. **Reconciliation конфликтов:** Приоритизация наиболее надёжного источника.
3. **Многошаговое рассуждение:** Декомпозиция сложных запросов.
4. **Пробелы и внешние инструменты:** Веб-поиск для восполнения.

![Agentic RAG](../assets/Agentic_RAG_Introduces_Reasoning_Agent.png)

Рис. 2: Agentic RAG вводит агента рассуждения.

### Проблемы Agentic RAG

Рост сложности, стоимости, латентности. Агент может стать источником ошибок.

### Итого

RAG — powerful паттерн для превращения LLM в более knowledgeable и reliable системы. Agentic RAG и GraphRAG повышают глубину и trust.

## Практические применения и сценарии использования

* **Корпоративный поиск и Q&A:** Внутренние чатботы по документам.
* **Поддержка клиентов:** Доступ к мануалам, FAQ, тикетам.
* **Персонализированные рекомендации:** Семантически связанный контент.
* **Обзоры новостей:** LLM + ленты → актуальные обобщения.

## Практический пример кода (ADK)

```python
from google.adk.tools import google_search
from google.adk.agents import Agent


search_agent = Agent(
    name="research_assistant",
    model="gemini-2.0-flash-exp",
    instruction="You help users research topics. When asked, use the Google Search tool",
    tools=[google_search],
)
```

Использование Vertex AI RAG в ADK:

```python
# Import the necessary VertexAiRagMemoryService class from the google.adk.memory module.
from google.adk.memory import VertexAiRagMemoryService


RAG_CORPUS_RESOURCE_NAME = "projects/your-gcp-project-id/locations/us-central1/ragCorpora/your-corpus-id"

# Define an optional parameter for the number of top similar results to retrieve.
# This controls how many relevant document chunks the RAG service will return.
SIMILARITY_TOP_K = 5

# Define an optional parameter for the vector distance threshold.
# This threshold determines the maximum semantic distance allowed for retrieved results;
# results with a distance greater than this value might be filtered out.
VECTOR_DISTANCE_THRESHOLD = 0.7

# Initialize an instance of VertexAiRagMemoryService.
# This sets up the connection to your Vertex AI RAG Corpus.
# - rag_corpus: Specifies the unique identifier for your RAG Corpus.
# - similarity_top_k: Sets the maximum number of similar results to fetch.
# - vector_distance_threshold: Defines the similarity threshold for filtering results.
memory_service = VertexAiRagMemoryService(
    rag_corpus=RAG_CORPUS_RESOURCE_NAME,
    similarity_top_k=SIMILARITY_TOP_K,
    vector_distance_threshold=VECTOR_DISTANCE_THRESHOLD,
)
```

## Практический пример кода (LangChain)

```python
import os
import requests
from typing import List, Dict, Any, TypedDict

from langchain_community.document_loaders import TextLoader
from langchain_core.documents import Document
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_community.embeddings import OpenAIEmbeddings
from langchain_community.vectorstores import Weaviate
from langchain_openai import ChatOpenAI
from langchain.text_splitter import CharacterTextSplitter
from langchain.schema.runnable import RunnablePassthrough
from langgraph.graph import StateGraph, END

import weaviate
from weaviate.embedded import EmbeddedOptions
import dotenv


# Load environment variables (e.g., OPENAI_API_KEY)
dotenv.load_dotenv()

# Set your OpenAI API key (ensure it's loaded from .env or set here)
# os.environ["OPENAI_API_KEY"] = "YOUR_OPENAI_API_KEY"


# --- 1. Data Preparation (Preprocessing) ---

# Load data
url = "https://github.com/langchain-ai/langchain/blob/master/docs/docs/how_to/state_of_the_union.txt"
res = requests.get(url)
with open("state_of_the_union.txt", "w") as f:
    f.write(res.text)

loader = TextLoader("./state_of_the_union.txt")
documents = loader.load()

# Chunk documents
text_splitter = CharacterTextSplitter(chunk_size=500, chunk_overlap=50)
chunks = text_splitter.split_documents(documents)

# Embed and store chunks in Weaviate
client = weaviate.Client(embedded_options=EmbeddedOptions())

vectorstore = Weaviate.from_documents(
    client=client,
    documents=chunks,
    embedding=OpenAIEmbeddings(),
    by_text=False,
)

# Define the retriever
retriever = vectorstore.as_retriever()

# Initialize LLM
llm = ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0)


# --- 2. Define the State for LangGraph ---
class RAGGraphState(TypedDict):
    question: str
    documents: List[Document]
    generation: str


# --- 3. Define the Nodes (Functions) ---
def retrieve_documents_node(state: RAGGraphState) -> RAGGraphState:
    """Retrieves documents based on the user's question."""
    question = state["question"]
    documents = retriever.invoke(question)
    return {"documents": documents, "question": question, "generation": ""}


def generate_response_node(state: RAGGraphState) -> RAGGraphState:
    """Generates a response using the LLM based on retrieved documents."""
    question = state["question"]
    documents = state["documents"]

    # Prompt template from the PDF
    template = """You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.
Question: {question}
Context: {context}
Answer: """
    prompt = ChatPromptTemplate.from_template(template)

    # Format the context from the documents
    context = "\n\n".join([doc.page_content for doc in documents])

    # Create the RAG chain
    rag_chain = prompt | llm | StrOutputParser()

    # Invoke the chain
    generation = rag_chain.invoke({"context": context, "question": question})

    return {"question": question, "documents": documents, "generation": generation}


# --- 4. Build the LangGraph Graph ---
workflow = StateGraph(RAGGraphState)

# Add nodes
workflow.add_node("retrieve", retrieve_documents_node)
workflow.add_node("generate", generate_response_node)

# Set the entry point
workflow.set_entry_point("retrieve")

# Add edges (transitions)
workflow.add_edge("retrieve", "generate")
workflow.add_edge("generate", END)

# Compile the graph
app = workflow.compile()


# --- 5. Run the RAG Application ---
if __name__ == "__main__":
    print("\n--- Running RAG Query ---")
    query = "What did the president say about Justice Breyer"
    inputs = {"question": query}
    for s in app.stream(inputs):
        print(s)

    print("\n--- Running another RAG Query ---")
    query_2 = "What did the president say about the economy?"
    inputs_2 = {"question": query_2}
    for s in app.stream(inputs_2):
        print(s)
```

## Краткий обзор

**Что:** LLM ограничены статичными данными. Нет доступа к реальному времени или специфическим данным.

**Почему:** RAG подключает к внешним источникам: извлечение фрагментов → дополнение промпта → grounded ответ.

**Когда использования:** Когда LLM нужна специфическая, актуальная или проприетарная информация.

**Визуальное резюме:**

![Паттерн извлечения знаний — БД](../assets/Knowledge_Retrieval_Pattern_Database.png)

Паттерн извлечения знаний: AI-агент запрашивает структурированные БД

![Паттерн извлечения знаний — поиск](../assets/Knowledge_Retrieval_Pattern_Search.png)

Рис. 3: Паттерн извлечения знаний: AI-агент ищет информацию из интернета.

## Ключевые выводы

* RAG расширяет LLM доступом к внешней информации.
* Процесс: извлечение + дополнение промпта.
* Преодолевает устаревшие данные, снижает галлюцинации.
* GraphRAG — граф знаний для сложных запросов.
* Agentic RAG — слой рассуждения для валидации и синтеза.
* Применения: корпоративный поиск, поддержка, рекомендации.

## Заключение

RAG решает ключевое ограничение статичных знаний LLM. Процесс: извлечение → дополнение → генерация. Технологии: эмбеддинги, семантический поиск, векторные БД. Agentic RAG и GraphRAG повышают глубину. RAG — критический паттерн для превращения LLM в powerful инструменты рассуждения.

## Ссылки

1. Lewis, P., et al. (2020). Retrieval-Augmented Generation: [https://arxiv.org/abs/2005.11401](https://arxiv.org/abs/2005.11401)
2. Google RAG: [https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/rag-overview](https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/rag-overview)
3. GraphRAG: [https://arxiv.org/abs/2501.00309](https://arxiv.org/abs/2501.00309)
4. LangChain RAG: [https://medium.com/data-science/retrieval-augmented-generation-rag-from-theory-to-langchain-implementation-4e9bd5f6a4f2](https://medium.com/data-science/retrieval-augmented-generation-rag-from-theory-to-langchain-implementation-4e9bd5f6a4f2)
5. Vertex AI RAG: [https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/manage-your-rag-corpus#corpus-management](https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/manage-your-rag-corpus#corpus-management)
