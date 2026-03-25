# Глава 14: Извлечение знаний (RAG)

LLM демонстрируют значительные возможности в генерации человекоподобного текста. Однако их база знаний обычно ограничена данными, на которых они обучались, что ограничивает доступ к информации в реальном времени, специфическим корпоративным данным или узкоспециальным деталям. Извлечение знаний (RAG, Retrieval Augmented Generation) решает это ограничение. RAG позволяет LLM получать доступ к внешней, актуальной и контекстно-специфичной информации, повышая точность, релевантность и фактическую обоснованность их выводов.

Для AI-агентов это критически важно: они могут grounding свои действия и ответы в актуальных проверяемых данных, выходящих за рамки статического обучения. Эта возможность позволяет точно выполнять сложные задачи: доступ к последним корпоративным политикам, проверка текущих запасов перед размещением заказа.

## Обзор паттерна «Извлечение знаний (RAG)»

Паттерн RAG значительно расширяет возможности LLM, предоставляя доступ к внешним базам знаний перед генерацией ответа. Вместо полагания исключительно на внутренние предобученные знания, RAG позволяет LLM «искать» информацию, подобно тому как человек обращается к книге или поиску в интернете.

Когда пользователь задаёт вопрос системе с RAG, запрос не отправляется напрямую к LLM. Сначала система просматривает обширную внешнюю базу знаний — высокоорганизованную библиотеку документов, БД или веб-страниц — в поисках релевантной информации. Поиск не простое совпадение ключевых слов — это «семантический поиск», понимающий намерение пользователя и смысл слов. Найденные фрагменты «дополняют» (augment) исходный промпт. Затем расширенный промпт отправляется LLM, которая генерирует ответ, factually grounded в полученных данных.

Преимущества RAG: доступ к актуальной информации, снижение риска «галлюцинаций», использование специализированных знаний, возможность «цитирования» источников.

### Эмбеддинги

В контексте LLM эмбеддинги — числовые представления текста (слов, фраз, документов) в виде векторов. Ключевая идея — захват семантического смысла и отношений в математическом пространстве. Схожие по смыслу тексты имеют близкие эмбеддинги. Например, «кошечка» и «котёнок» будут рядом (2, 3) и (2.1, 3.1), а «машина» — далеко (8, 1). В реальности эмбеддинги имеют сотни или тысячи измерений.

### Текстовое сходство

Текстовое сходство — мера схожести двух фрагментов текста. Может быть на поверхностном уровне (лексическое сходство) или на глубоком уровне смысла. В RAG текстовое сходство критически важно для поиска наиболее релевантной информации. Например: «Какова столица Франции?» и «Какой город является столицей Франции?» — разные формулировки, один вопрос. Хорошая модель текстового сходства назначит высокий балл.

### Семантическое сходство и расстояние

Семантическое сходство — продвинутая форма текстового сходства, фокусирующаяся на значении и контексте, а не на словах. Семантическое расстояние — обратная величина: высокое сходство = малое расстояние. В RAG семантический поиск находит документы с наименьшим семантическим расстоянием. Например: «пушистый кот-компаньон» и «домашняя кошка» — общих слов нет, но семантическая модель распознает тождество понятий.

![Базовые концепции RAG: чанкинг, эмбеддинги, векторная БД](../assets/RAG_Core_Concepts_Chunking_Embeddings_and_Vector_Database.png)

Рис. 1: Базовые концепции RAG: чанкинг, эмбеддинги и векторная БД

### Чанкинг документов

Чанкинг — процесс разбиения больших документов на более мелкие управляемые фрагменты (chunks). RAG-система не может подавать целиком большие документы в LLM. Способ чанкинга важен для сохранения контекста. Например, 50-страничное руководство пользователя разбивается на секции, параграфы или предложения. Раздел «Устранение неполадок» — отдельный чанк от «Руководства по установке». Когда пользователь спрашивает о конкретной проблеме, RAG извлекает релевантный чанк по устранению неполадок, а не всё руководство.

После чанкинга RAG применяет технику извлечения. Основной метод — векторный поиск (с эмбеддингами и семантическим расстоянием). Традиционный BM25 — алгоритм на частоте ключевых слов. Гибридный поиск объединяет точность BM25 с контекстуальным пониманием семантического поиска.

### Векторные БД

Векторная БД — специализированный тип БД для эффективного хранения и запросов эмбеддингов. Традиционные методы (ключевые слова) не понимают смысл. Векторные БД специально созданы для семантического поиска: хранят текст как числовые векторы и находят результаты на основе концептуального значения. Когда запрос пользователя тоже конвертируется в вектор, БД использует оптимизированные алгоритмы (например, HNSW — Hierarchical Navigable Small World) для быстрого поиска среди миллионов векторов.

Реализации: Pinecone, Weaviate (управляемые), Chroma DB, Milvus, Qdrant (open-source), Redis, Elasticsearch, Postgres (pgvector). Библиотеки: FAISS (Meta AI), ScaNN (Google Research).

### Проблемы RAG

* Информация может быть разбросана по нескольким чанкам или документам.
* Зависимость от качества чанкинга и извлечения; нерелевантные чанки вносят шум.
* Синтез противоречивых источников.
* Требуется предварительная обработка и хранение в специализированных БД.
* Периодическая актуализация знаний.
* Влияние на производительность: латентность, стоимость, количество токенов.

### Graph RAG

GraphRAG — продвинутая форма RAG, использующая граф знаний вместо простой векторной БД. Отвечает на сложные запросы, навигируя по явным связям (рёбра) между сущностями (узлами). Преимущество: синтез ответов из фрагментированной информации. Недостатки: сложность, стоимость построения и поддержки графа знаний, менее гибкий, более высокая латентность.

### Agentic RAG

Эволюция паттерна — **Agentic RAG** (см. Рис. 2) — вводит слой рассуждения и принятия решений. Вместо простого извлечения и дополнения «агент» выступает критическим gatekeeper и refiner знаний.

Во-первых, агент выполняет рефлексию и валидацию источников: анализирует метаданные, распознаёт наиболее актуальные и авторитетные документы, отбрасывает устаревшие.

![Agentic RAG вводит агента рассуждения](../assets/Agentic_RAG_Introduces_Reasoning_Agent.png)

Рис. 2: Agentic RAG вводит агента рассуждения, активно оценивающего и уточняющего извлечённую информацию.

Во-вторых, агент reconciles конфликты знаний: при обнаружении противоречий (разные суммы бюджета в разных документах) приоритизирует наиболее надёжный источник.

В-третьих, агент выполняет многошаговое рассуждение: декомпозирует сложный запрос на подзапросы, собирает информацию по частям, синтезирует структурированный контекст.

В-четвёртых, агент выявляет пробелы в знаниях и использует внешние инструменты (веб-поиск) для восполнения.

### Проблемы Agentic RAG

Основной недостаток — рост сложности и стоимости. Циклы рефлексии и многошагового рассуждения увеличивают латентность. Агент сам может стать источником ошибок: бесконечные циклы, неверная интерпретация, отбрасывание релевантной информации.

### Итого

Agentic RAG — sophisticated эволюция стандартного паттерна извлечения, превращающая пассивный конвейер данных в активную проблемно-решающую систему. Расширенные методы (GraphRAG, Agentic RAG) повышают глубину и доверие, но добавляют сложность, латентность и стоимость.

## Практические применения и сценарии использования

* **Корпоративный поиск и Q&A:** Внутренние чатботы, отвечающие по документам (HR-политики, техруководства, спецификации).
* **Поддержка клиентов:** Системы на RAG с доступом к мануалам, FAQ, тикетам.
* **Персонализированные рекомендации:** Семантически связанный контент вместо ключевых слов.
* **Обзоры новостей:** LLM + ленты новостей → актуальные обобщения.

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
from google.adk.memory import VertexAiRagMemoryService

RAG_CORPUS_RESOURCE_NAME = "projects/your-gcp-project-id/locations/us-central1/ragCorpora/your-corpus-id"
SIMILARITY_TOP_K = 5
VECTOR_DISTANCE_THRESHOLD = 0.7

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
from langgraph.graph import StateGraph, END
import weaviate
from weaviate.embedded import EmbeddedOptions
import dotenv

dotenv.load_dotenv()

url = "https://github.com/langchain-ai/langchain/blob/master/docs/docs/how_to/state_of_the_union.txt"
res = requests.get(url)
with open("state_of_the_union.txt", "w") as f:
    f.write(res.text)

loader = TextLoader("./state_of_the_union.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=500, chunk_overlap=50)
chunks = text_splitter.split_documents(documents)

client = weaviate.Client(embedded_options=EmbeddedOptions())
vectorstore = Weaviate.from_documents(client=client, documents=chunks, embedding=OpenAIEmbeddings(), by_text=False)
retriever = vectorstore.as_retriever()
llm = ChatOpenAI(model_name="gpt-3.5-turbo", temperature=0)

class RAGGraphState(TypedDict):
    question: str
    documents: List[Document]
    generation: str

def retrieve_documents_node(state: RAGGraphState) -> RAGGraphState:
    question = state["question"]
    documents = retriever.invoke(question)
    return {"documents": documents, "question": question, "generation": ""}

def generate_response_node(state: RAGGraphState) -> RAGGraphState:
    question = state["question"]
    documents = state["documents"]
    template = """You are an assistant for question-answering tasks. Use the following pieces of retrieved context to answer the question. If you don't know the answer, just say that you don't know. Use three sentences maximum and keep the answer concise.
Question: {question}
Context: {context}
Answer: """
    prompt = ChatPromptTemplate.from_template(template)
    context = "\n\n".join([doc.page_content for doc in documents])
    rag_chain = prompt | llm | StrOutputParser()
    generation = rag_chain.invoke({"context": context, "question": question})
    return {"question": question, "documents": documents, "generation": generation}

workflow = StateGraph(RAGGraphState)
workflow.add_node("retrieve", retrieve_documents_node)
workflow.add_node("generate", generate_response_node)
workflow.set_entry_point("retrieve")
workflow.add_edge("retrieve", "generate")
workflow.add_edge("generate", END)
app = workflow.compile()

if __name__ == "__main__":
    query = "What did the president say about Justice Breyer"
    for s in app.stream({"question": query}):
        print(s)
```

Код демонстрирует RAG-пайплайн на LangChain и LangGraph. Начинается с создания базы знаний из текстового документа: сегментация на чанки, трансформация в эмбеддинги, хранение в векторном хранилище Weaviate. StateGraph управляет рабочим процессом: `retrieve_documents_node` запрашивает векторное хранилище, `generate_response_node` генерирует ответ через LLM.

## Краткий обзор

**Что:** LLM ограничены статичными обучающими данными: нет доступа к реальным временем, приватным или специализированным данным.

**Почему:** RAG подключает LLM к внешним источникам знаний: извлечение релевантных фрагментов → дополнение промпта → генерация точного контекстного ответа.

**Когда использовать:** Когда LLM нужно отвечать на основе специфической, актуальной или проприетарной информации. Идеально для Q&A по внутренним документам, ботов поддержки, fact-based приложений.

**Визуальное резюме:**

![Паттерн извлечения знаний — БД](../assets/Knowledge_Retrieval_Pattern_Database.png)

Паттерн извлечения знаний: AI-агент запрашивает структурированные БД

![Паттерн извлечения знаний — поиск](../assets/Knowledge_Retrieval_Pattern_Search.png)

Рис. 3: Паттерн извлечения знаний: AI-агент ищет и синтезирует информацию из интернета

## Ключевые выводы

* RAG расширяет LLM доступом к внешней, актуальной и специфичной информации.
* Процесс включает извлечение (поиск релевантных фрагментов) и дополнение (добавление в промпт LLM).
* RAG преодолевает ограничения устаревших данных, снижает галлюцинации, позволяет интегрировать доменные знания.
* Позволяет атрибутивные ответы с цитированием источников.
* GraphRAG использует граф знаний для навигации по связям между сущностями.
* Agentic RAG вводит слой рассуждения: валидация, reconciliation конфликтов, декомпозиция запросов, внешние инструменты.
* Применения: корпоративный поиск, поддержка, юридический анализ, рекомендации.

## Заключение

RAG решает ключевое ограничение статичных знаний LLM через подключение к внешним источникам. Процесс: извлечение → дополнение промпта → генерация контекстного ответа. Технологии: эмбеддинги, семантический поиск, векторные БД. Agentic RAG вводит слой рассуждения для повышения надёжности. GraphRAG использует графы знаний для сложных взаимосвязанных запросов. RAG — критический паттерн для превращения LLM из «закрытых книг» в мощные «открытые» инструменты рассуждения.

## Ссылки

1. Lewis, P., et al. (2020). *Retrieval-Augmented Generation for Knowledge-Intensive NLP Tasks*: [https://arxiv.org/abs/2005.11401](https://arxiv.org/abs/2005.11401)
2. Google AI: RAG: [https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/rag-overview](https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/rag-overview)
3. GraphRAG: [https://arxiv.org/abs/2501.00309](https://arxiv.org/abs/2501.00309)
4. LangChain RAG: [https://medium.com/data-science/retrieval-augmented-generation-rag-from-theory-to-langchain-implementation-4e9bd5f6a4f2](https://medium.com/data-science/retrieval-augmented-generation-rag-from-theory-to-langchain-implementation-4e9bd5f6a4f2)
5. Vertex AI RAG Corpus: [https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/manage-your-rag-corpus#corpus-management](https://cloud.google.com/vertex-ai/generative-ai/docs/rag-engine/manage-your-rag-corpus#corpus-management)
