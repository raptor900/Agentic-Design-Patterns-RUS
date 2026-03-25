# Глава 3: Параллелизация

## Обзор паттерна «Параллелизация»

В предыдущих главах мы рассмотрели цепочку промптов для последовательных рабочих процессов и маршрутизацию для динамического принятия решений и переходов между различными путями. Хотя эти паттерны важны, многие сложные агентные задачи включают несколько подзадач, которые могут выполняться *одновременно*, а не одна за другой. Именно здесь паттерн **«Параллелизация»** становится ключевым.

Параллелизация подразумевает одновременное выполнение нескольких компонентов — вызовов LLM, использования инструментов или даже целых подагентов (см. Рис. 1). Вместо ожидания завершения одного шага перед запуском следующего параллельное выполнение позволяет независимым задачам работать одновременно, существенно сокращая общее время выполнения задач, которые можно разбить на независимые части.

Рассмотрим агента, задача которого — провести исследование по теме и обобщить результаты. Последовательный подход может выглядеть так:

1. Поиск по источнику A.
2. Обобщение источника A.
3. Поиск по источнику B.
4. Обобщение источника B.
5. Синтез финального ответа на основе обобщений A и B.

Параллельный подход вместо этого:

1. Поиск по источнику A *и* поиск по источнику B одновременно.
2. Как только оба поиска завершены — обобщение источника A *и* обобщение источника B одновременно.
3. Синтез финального ответа на основе обобщений A и B (этот шаг обычно последовательный, ожидает завершения параллельных шагов).

Основная идея — выявить части рабочего процесса, не зависящие от вывода других частей, и выполнить их параллельно. Это особенно эффективно при работе с внешними сервисами (API, базы данных), имеющими задержку, — вы можете отправлять несколько запросов одновременно.

Реализация параллелизации часто требует фреймворков с поддержкой асинхронного выполнения или многопоточности/мультпроцессинга. Современные агентные фреймворки спроектированы с учётом асинхронных операций, позволяя легко определять шаги, выполняемые параллельно.

![Параллелизация с подагентами](../assets/Parallelization_with_Sub_Agents.png)

Рис. 1. Пример параллелизации с подагентами

Фреймворки вроде LangChain, LangGraph и Google ADK предоставляют механизмы параллельного выполнения. В LangChain Expression Language (LCEL) параллелизация достигается через объединение исполняемых объектов с помощью операторов (например, `|` для последовательных связей) и структурирования цепочек или графов с ветвями, выполняющимися параллельно. LangGraph с его графовой структурой позволяет определять несколько узлов, запускаемых из одного перехода состояния, эффективно включая параллельные ветви в рабочем процессе. Google ADK предоставляет нативные механизмы для параллельного выполнения агентов.

Паттерн параллелизации критически важен для повышения эффективности и отзывчивости агентных систем, особенно при задачах с множеством независимых обращений к внешним сервисам.

## Практические применения и сценарии использования

### 1. Сбор информации и исследования

Сбор данных из нескольких источников одновременно — классический сценарий.

* **Сценарий:** Агент исследует компанию.
  * **Параллельные задачи:** Поиск новостных статей, загрузка биржевых данных, проверка упоминаний в соцсетях, запрос к базе данных компаний — всё одновременно.
  * **Выгода:** Полная картина формируется значительно быстрее последовательных запросов.

### 2. Обработка и анализ данных

Применение различных техник анализа или обработка разных сегментов данных параллельно.

* **Сценарий:** Агент анализирует отзывы клиентов.
  * **Параллельные задачи:** Анализ тональности, извлечение ключевых слов, категоризация отзывов, выявление срочных проблем — одновременно по пакету отзывов.
  * **Выгода:** Многоаспектный анализ за короткое время.

### 3. Взаимодействие с несколькими API или инструментами

Вызов нескольких независимых API для сбора разных типов информации или выполнения различных действий.

* **Сценарий:** Агент планирования путешествий.
  * **Параллельные задачи:** Проверка цен на авиабилеты, поиск доступности отелей, поиск местных мероприятий, рекомендации ресторанов — одновременно.
  * **Выгода:** Полный план путешествия формируется быстрее.

### 4. Генерация контента с несколькими компонентами

Генерация разных частей сложного контента параллельно.

* **Сценарий:** Агент создаёт маркетинговое письмо.
  * **Параллельные задачи:** Генерация темы, черновик тела письма, поиск подходящего изображения, создание текста кнопки призыва к действию — одновременно.
  * **Выгода:** Финальное письмо собирается эффективнее.

### 5. Валидация и проверка

Проведение нескольких независимых проверок параллельно.

* **Сценарий:** Агент проверяет пользовательский ввод.
  * **Параллельные задачи:** Проверка формата email, валидация номера телефона, проверка адреса по базе данных, фильтрация ненормативной лексики — одновременно.
  * **Выгода:** Быстрая обратная связь по корректности ввода.

### 6. Мультимодальная обработка

Параллельная обработка различных модальностей (текст, изображение, аудио) одного входа.

* **Сценарий:** Агент анализирует пост в соцсети с текстом и изображением.
  * **Параллельные задачи:** Анализ текста на тональность и ключевые слова *и* анализ изображения на объекты и описание сцены — одновременно.
  * **Выгода:** Быстрая интеграция инсайтов из разных модальностей.

### 7. A/B-тестирование или генерация вариантов

Генерация нескольких вариантов ответа параллельно для выбора лучшего.

* **Сценарий:** Агент генерирует различные варианты креативного текста.
  * **Параллельные задачи:** Генерация трёх разных заголовков для статьи одновременно с использованием слегка разных промптов или моделей.
  * **Выгода:** Быстрое сравнение и выбор лучшего варианта.

Параллелизация — фундаментальная техника оптимизации в агентном дизайне, позволяющая разработчикам создавать более производительные приложения за счёт одновременного выполнения независимых задач.

## Практический пример кода (LangChain)

Параллельное выполнение в LangChain реализуется через LangChain Expression Language (LCEL). Основной метод — структурирование нескольких исполняемых компонентов в словарь или список. При передаче такой коллекции следующему компоненту цепочки среда выполнения LCEL запускает вложенные исполняемые объекты одновременно.

В LangGraph этот принцип применяется к топологии графа. Параллельные рабочие процессы определяются архитектурой графа, в которой несколько узлов без прямых последовательных зависимостей могут быть запущены из одного общего узла.

Ниже представлен пример параллельного рабочего процесса на LangChain, выполняющего две независимые операции одновременно в ответ на один пользовательский запрос.

```python
import os
import asyncio
from typing import Optional

from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import Runnable, RunnableParallel, RunnablePassthrough


# --- Конфигурация ---
try:
    llm: Optional[ChatOpenAI] = ChatOpenAI(model="gpt-4o-mini", temperature=0.7)
except Exception as e:
    print(f"Ошибка инициализации языковой модели: {e}")
    llm = None


# --- Определение независимых цепочек ---
# Эти три цепочки представляют независимые задачи, выполняемые параллельно.
summarize_chain: Runnable = (
    ChatPromptTemplate.from_messages([
        ("system", "Summarize the following topic concisely:"),
        ("user", "{topic}"),
    ])
    | llm
    | StrOutputParser()
)

questions_chain: Runnable = (
    ChatPromptTemplate.from_messages([
        ("system", "Generate three interesting questions about the following topic:"),
        ("user", "{topic}"),
    ])
    | llm
    | StrOutputParser()
)

terms_chain: Runnable = (
    ChatPromptTemplate.from_messages([
        ("system", "Identify 5-10 key terms from the following topic, separated by commas:"),
        ("user", "{topic}"),
    ])
    | llm
    | StrOutputParser()
)


# --- Построение параллельной + синтезирующей цепочки ---
# 1. Блок задач для параллельного выполнения.
map_chain = RunnableParallel(
    {
        "summary": summarize_chain,
        "questions": questions_chain,
        "key_terms": terms_chain,
        "topic": RunnablePassthrough(),
    }
)

# 2. Финальный промпт синтеза, объединяющий параллельные результаты.
synthesis_prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        "Based on the following information:\n"
        "Summary: {summary}\n"
        "Related Questions: {questions}\n"
        "Key Terms: {key_terms}\n"
        "Synthesize a comprehensive answer."
    ),
    ("user", "Original topic: {topic}"),
])

# 3. Полная цепочка: параллельные результаты → промпт синтеза → LLM → парсер.
full_parallel_chain = map_chain | synthesis_prompt | llm | StrOutputParser()


# --- Запуск цепочки ---
async def run_parallel_example(topic: str) -> None:
    """Асинхронный вызов цепочки параллельной обработки."""
    if not llm:
        print("LLM не инициализирована.")
        return

    print(f"\n--- Параллельный пример LangChain для темы: '{topic}' ---")
    try:
        response = await full_parallel_chain.ainvoke(topic)
        print("\n--- Финальный ответ ---")
        print(response)
    except Exception as e:
        print(f"\nОшибка при выполнении цепочки: {e}")


if __name__ == "__main__":
    test_topic = "The history of space exploration"
    asyncio.run(run_parallel_example(test_topic))
```

Данный код реализует приложение LangChain для эффективной обработки заданной темы за счёт параллельного выполнения. Обратите внимание, что asyncio обеспечивает конкурентность, а не истинный параллелизм. Он достигает этого на одном потоке с помощью цикла событий, переключающегося между задачами при простое (например, при сетевом запросе). Это создаёт эффект одновременного выполнения нескольких задач, но код по-прежнему выполняется одним потоком, ограниченным Global Interpreter Lock (GIL) Python.

Код импортирует модули из `langchain_openai` и `langchain_core`. Пытается инициализировать ChatOpenAI с моделью «gpt-4o-mini». Определяются три независимые цепочки LangChain: для обобщения темы, генерации трёх вопросов и извлечения 5–10 ключевых терминов. Каждая цепочка состоит из ChatPromptTemplate, языковой модели и StrOutputParser.

Блок RunnableParallel объединяет эти три цепочки для одновременного выполнения, включая RunnablePassthrough для передачи исходной темы. Отдельный ChatPromptTemplate определён для финального синтеза. Полная цепочка `full_parallel_chain` создаётся путём связывания параллельного блока с промптом синтеза, моделью и парсером. Асинхронная функция `run_parallel_example` демонстрирует вызов с использованием `ainvoke`.

По сути, код создаёт рабочий процесс, где несколько вызовов LLM (обобщение, вопросы, термины) выполняются одновременно, а их результаты затем объединяются финальным вызовом LLM.

## Практический пример кода (Google ADK)

```python
from google.adk.agents import LlmAgent, ParallelAgent, SequentialAgent
from google.adk.tools import google_search

GEMINI_MODEL = "gemini-2.0-flash"


# --- 1. Определение подагентов-исследователей (параллельное выполнение) ---

# Исследователь 1: Возобновляемая энергия
researcher_agent_1 = LlmAgent(
    name="RenewableEnergyResearcher",
    model=GEMINI_MODEL,
    instruction="""You are an AI Research Assistant specializing in energy. Research the latest advancements in 'renewable energy sources'. Use the Google Search tool provided. Summarize your key findings concisely (1-2 sentences). Output *only* the summary. """,
    description="Researches renewable energy sources.",
    tools=[google_search],
    output_key="renewable_energy_result",
)

# Исследователь 2: Электромобили
researcher_agent_2 = LlmAgent(
    name="EVResearcher",
    model=GEMINI_MODEL,
    instruction="""You are an AI Research Assistant specializing in transportation. Research the latest developments in 'electric vehicle technology'. Use the Google Search tool provided. Summarize your key findings concisely (1-2 sentences). Output *only* the summary. """,
    description="Researches electric vehicle technology.",
    tools=[google_search],
    output_key="ev_technology_result",
)

# Исследователь 3: Углеродный захват
researcher_agent_3 = LlmAgent(
    name="CarbonCaptureResearcher",
    model=GEMINI_MODEL,
    instruction="""You are an AI Research Assistant specializing in climate solutions. Research the current state of 'carbon capture methods'. Use the Google Search tool provided. Summarize your key findings concisely (1-2 sentences). Output *only* the summary. """,
    description="Researches carbon capture methods.",
    tools=[google_search],
    output_key="carbon_capture_result",
)


# --- 2. Создание ParallelAgent (запуск исследователей параллельно) ---
parallel_research_agent = ParallelAgent(
    name="ParallelWebResearchAgent",
    sub_agents=[researcher_agent_1, researcher_agent_2, researcher_agent_3],
    description="Runs multiple research agents in parallel to gather information.",
)


# --- 3. Определение агента-объединителя (запускается после параллельных) ---
merger_agent = LlmAgent(
    name="SynthesisAgent",
    model=GEMINI_MODEL,
    instruction="""You are an AI Assistant responsible for combining research findings into a structured report. Your primary task is to synthesize the following research summaries, clearly attributing findings to their source areas. Structure your response using headings for each topic.

**Crucially:** Your entire response MUST be grounded *exclusively* on the information provided in the 'Input Summaries' below. Do NOT add any external knowledge.

**Input Summaries:**
*   **Renewable Energy:**
    {renewable_energy_result}
*   **Electric Vehicles:**
    {ev_technology_result}
*   **Carbon Capture:**
    {carbon_capture_result}

**Output Format:**
## Summary of Recent Sustainable Technology Advancements

### Renewable Energy Findings
[Synthesize and elaborate *only* on the renewable energy input summary provided above.]

### Electric Vehicle Findings
[Synthesize and elaborate *only* on the EV input summary provided above.]

### Carbon Capture Findings
[Synthesize and elaborate *only* on the carbon capture input summary provided above.]

### Overall Conclusion
[Provide a brief (1-2 sentence) concluding statement that connects *only* the findings presented above.]

Output *only* the structured report following this format. Do not include introductory or concluding phrases outside this structure, and strictly adhere to using only the provided input summary content.
""",
    description="Combines research findings from parallel agents into a structured report.",
)


# --- 4. Создание SequentialAgent (оркестрация общего потока) ---
sequential_pipeline_agent = SequentialAgent(
    name="ResearchAndSynthesisPipeline",
    sub_agents=[parallel_research_agent, merger_agent],
    description="Coordinates parallel research and synthesizes the results.",
)

root_agent = sequential_pipeline_agent
```

Этот код определяет многоагентную систему для исследования и синтеза информации. Определяются три LlmAgent-исследователя: по возобновляемой энергии, электромобилям и углеродному захвату. Каждый настроен на модель GEMINI_MODEL и инструмент `google_search`, обобщает находки в 1–2 предложения и сохраняет в состоянии сессии через `output_key`.

ParallelAgent запускает трёх исследователей одновременно. Завершается, когда все подагенты заполнили состояние. Затем MergerAgent синтезирует результаты в структурированный отчёт с атрибуцией, строго основанный только на предоставленных входных данных.

SequentialAgent orchestrates: сначала ParallelAgent для исследования, затем MergerAgent для синтеза. Это `root_agent` системы.

## Краткий обзор

**Что:** Многие агентные рабочие процессы включают независимые подзадачи. Чисто последовательное выполнение неэффективно и медленно, особенно при обращениях к внешним API.

**Почему:** Паттерн параллелизации позволяет одновременно выполнять независимые задачи. Фреймворки LangChain и Google ADK предоставляют встроенные конструкции для управления параллельными операциями.

**Когда использовать:** Когда рабочий процесс содержит несколько независимых операций: запросы к разным API, обработка разных фрагментов данных, генерация нескольких вариантов контента.

**Визуальное резюме:**

![Паттерн параллелизации](../assets/Parallelization_Design_Pattern.png)

Рис. 2: Паттерн «Параллелизация»

## Ключевые выводы

* Параллелизация — паттерн одновременного выполнения независимых задач для повышения эффективности.
* Особенно полезен, когда задачи связаны с ожиданием внешних ресурсов (API-вызовы).
* Использование параллельной архитектуры вносит существенную сложность: дизайн, отладка, логирование.
* LangChain и Google ADK предоставляют встроенную поддержку параллельного выполнения.
* В LCEL RunnableParallel — ключевая конструкция для параллельного запуска.
* Google ADK реализует параллелизация через LLM-делегирование: координатор идентифицирует независимые подзадачи и запускает параллельное выполнение специализированными подагентами.
* Параллелизация снижает общую латентность и повышает отзывчивость агентных систем.

## Заключение

Паттерн параллелизации оптимизирует вычислительные рабочие процессы за счёт одновременного выполнения независимых подзадач. Интеграция параллельной обработки с последовательной (цепочки) и условной (маршрутизация) логикой позволяет строить сложные высокопроизводительные системы.

## Ссылки

1. Документация LangChain Expression Language (LCEL): [https://python.langchain.com/docs/concepts/lcel/](https://python.langchain.com/docs/concepts/lcel/)
2. Документация Google ADK (многоагентные системы): [https://google.github.io/adk-docs/agents/multi-agents/](https://google.github.io/adk-docs/agents/multi-agents/)
3. Документация Python asyncio: [https://docs.python.org/3/library/asyncio.html](https://docs.python.org/3/library/asyncio.html)
