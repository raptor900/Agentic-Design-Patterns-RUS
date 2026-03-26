# Глава 3: Параллелизация

## Обзор паттерна «Параллелизация»

В предыдущих главах мы рассмотрели цепочку промптов для последовательных рабочих процессов и маршрутизацию для динамического принятия решений. Хотя эти паттерны важны, многие сложные агентные задачи включают несколько подзадач, которые могут выполняться *одновременно*. Именно здесь паттерн **«Параллелизация»** становится ключевым.

Параллелизация подразумевает одновременное выполнение нескольких компонентов — вызовов LLM, использования инструментов или целых подагентов (см. Рис. 1). Вместо ожидания завершения одного шага перед запуском следующего параллельное выполнение позволяет независимым задачам работать одновременно, существенно сокращая общее время.

Рассмотрим агента, исследующего тему и обобщающего результаты. Последовательный подход: поиск A → обобщение A → поиск B → обобщение B → синтез. Параллельный: поиск A и B одновременно → обобщение A и B одновременно → синтез.

Основная идея — выявить части рабочего процесса, не зависящие от вывода других, и выполнить их параллельно. Это особенно эффективно при работе с внешними сервисами (API, БД).

Реализация параллелизации часто требует фреймворков с поддержкой асинхронного выполнения или многопоточности.

![Параллелизация с подагентами](../assets/Parallelization_with_Sub_Agents.png)

Рис. 1. Пример параллелизации с подагентами

Фреймворки LangChain, LangGraph и Google ADK предоставляют механизмы параллельного выполнения. В LCEL параллелизация достигается через объединение исполняемых объектов и структурирование цепочек с параллельными ветвями. LangGraph позволяет определять несколько узлов, запускаемых из одного перехода состояния.

Паттерн параллелизации критически важен для повышения эффективности и отзывчивости агентных систем.

## Практические применения и сценарии использования

Параллелизация — мощный паттерн для оптимизации производительности агента в различных областях:

### 1. Сбор информации и исследования

Сбор информации из нескольких источников одновременно — классический сценарий.

* **Сценарий:** Агент исследует компанию.
  * **Параллельные задачи:** Поиск новостных статей, загрузка биржевых данных, проверка упоминаний в соцсетях, запрос к базе данных компаний — всё одновременно.
  * **Выгода:** Полная картина формируется значительно быстрее.

### 2. Обработка и анализ данных

Применение различных техник анализа или обработка разных сегментов параллельно.

* **Сценарий:** Агент анализирует отзывы клиентов.
  * **Параллельные задачи:** Анализ тональности, извлечение ключевых слов, категоризация, выявление срочных проблем — одновременно.
  * **Выгода:** Многоаспектный анализ за короткое время.

### 3. Взаимодействие с несколькими API или инструментами

Вызов нескольких независимых API для сбора разных типов информации.

* **Сценарий:** Агент планирования путешествий.
  * **Параллельные задачи:** Проверка цен на авиабилеты, поиск отелей, мероприятий, ресторанов — одновременно.
  * **Выгода:** Полный план путешествия формируется быстрее.

### 4. Генерация контента с несколькими компонентами

Создание разных частей сложного контента параллельно.

* **Сценарий:** Агент создаёт маркетинговое письмо.
  * **Параллельные задачи:** Генерация темы, черновик письма, поиск изображения, текст кнопки CTA — одновременно.
  * **Выгода:** Письмо собирается эффективнее.

### 5. Валидация и проверка

Проведение нескольких независимых проверок параллельно.

* **Сценарий:** Агент проверяет пользовательский ввод.
  * **Параллельные задачи:** Проверка формата email, валидация телефона, проверка адреса, фильтрация лексики — одновременно.
  * **Выгода:** Быстрая обратная связь по корректности.

### 6. Мультимодальная обработка

Параллельная обработка различных модальностей одного входа.

* **Сценарий:** Агент анализирует пост в соцсети с текстом и изображением.
  * **Параллельные задачи:** Анализ текста на тональность *и* анализ изображения на объекты — одновременно.
  * **Выгода:** Быстрая интеграция инсайтов из разных модальностей.

### 7. A/B-тестирование или генерация вариантов

Генерация нескольких вариантов параллельно для выбора лучшего.

* **Сценарий:** Генерация разных заголовков для статьи.
  * **Параллельные задачи:** Генерация трёх вариантов с разными промптами — одновременно.
  * **Выгода:** Быстрое сравнение и выбор лучшего.

Параллелизация — фундаментальная техника оптимизации в агентном дизайне.

## Практический пример кода (LangChain)

Параллельное выполнение в LCEL достигается через структурирование нескольких исполняемых компонентов в словарь или список. В LangGraph — через архитектуру с параллельными ветвями из одного узла.

```python
import os
import asyncio
from typing import Optional

from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import Runnable, RunnableParallel, RunnablePassthrough


# --- Configuration ---
# Ensure your API key environment variable is set (e.g., OPENAI_API_KEY)
try:
    llm: Optional[ChatOpenAI] = ChatOpenAI(model="gpt-4o-mini", temperature=0.7)
except Exception as e:
    print(f"Error initializing language model: {e}")
    llm = None


# --- Define Independent Chains ---
# These three chains represent distinct tasks that can be executed in parallel.
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


# --- Build the Parallel + Synthesis Chain ---
# 1. Define the block of tasks to run in parallel. The results of these,
#    along with the original topic, will be fed into the next step.
map_chain = RunnableParallel(
    {
        "summary": summarize_chain,
        "questions": questions_chain,
        "key_terms": terms_chain,
        "topic": RunnablePassthrough(),  # Pass the original topic through
    }
)

# 2. Define the final synthesis prompt which will combine the parallel results.
synthesis_prompt = ChatPromptTemplate.from_messages([
    (
        "system",
        """Based on the following information:
        Summary: {summary}
        Related Questions: {questions}
        Key Terms: {key_terms}
        Synthesize a comprehensive answer."""
    ),
    ("user", "Original topic: {topic}"),
])

# 3. Construct the full chain by piping the parallel results directly
#    into the synthesis prompt, followed by the LLM and output parser.
full_parallel_chain = map_chain | synthesis_prompt | llm | StrOutputParser()


# --- Run the Chain ---
async def run_parallel_example(topic: str) -> None:
    """
    Asynchronously invokes the parallel processing chain with a specific topic
    and prints the synthesized result.

    Args:
        topic: The input topic to be processed by the LangChain chains.
    """
    if not llm:
        print("LLM not initialized. Cannot run example.")
        return

    print(f"\n--- Running Parallel LangChain Example for Topic: '{topic}' ---")
    try:
        # The input to `ainvoke` is the single 'topic' string,
        # then passed to each runnable in the `map_chain`.
        response = await full_parallel_chain.ainvoke(topic)
        print("\n--- Final Response ---")
        print(response)
    except Exception as e:
        print(f"\nAn error occurred during chain execution: {e}")


if __name__ == "__main__":
    test_topic = "The history of space exploration"
    # In Python 3.7+, asyncio.run is the standard way to run an async function.
    asyncio.run(run_parallel_example(test_topic))
```

## Практический пример кода (Google ADK)

Код определяет многоагентную систему для исследования и синтеза информации. Определяются три LlmAgent-исследователя: по возобновляемой энергии, электромобилям и углеродному захвату. Каждый настроен на модель Gemini и инструмент `google_search`, обобщает находки в 1-2 предложения и сохраняет в состоянии через `output_key`.

ParallelAgent запускает исследователей одновременно. Затем MergerAgent синтезирует результаты в структурированный отчёт. SequentialAgent оркестрирует: сначала ParallelAgent, затем MergerAgent.

```python
from google.adk.agents import LlmAgent, ParallelAgent, SequentialAgent
from google.adk.tools import google_search

GEMINI_MODEL = "gemini-2.0-flash"


# --- 1. Define Researcher Sub-Agents (to run in parallel) ---

# Researcher 1: Renewable Energy
researcher_agent_1 = LlmAgent(
    name="RenewableEnergyResearcher",
    model=GEMINI_MODEL,
    instruction="""You are an AI Research Assistant specializing in energy. Research the latest advancements in 'renewable energy sources'. Use the Google Search tool provided. Summarize your key findings concisely (1-2 sentences). Output *only* the summary. """,
    description="Researches renewable energy sources.",
    tools=[google_search],
    # Store result in state for the merger agent
    output_key="renewable_energy_result",
)

# Researcher 2: Electric Vehicles
researcher_agent_2 = LlmAgent(
    name="EVResearcher",
    model=GEMINI_MODEL,
    instruction="""You are an AI Research Assistant specializing in transportation. Research the latest developments in 'electric vehicle technology'. Use the Google Search tool provided. Summarize your key findings concisely (1-2 sentences). Output *only* the summary. """,
    description="Researches electric vehicle technology.",
    tools=[google_search],
    # Store result in state for the merger agent
    output_key="ev_technology_result",
)

# Researcher 3: Carbon Capture
researcher_agent_3 = LlmAgent(
    name="CarbonCaptureResearcher",
    model=GEMINI_MODEL,
    instruction="""You are an AI Research Assistant specializing in climate solutions. Research the current state of 'carbon capture methods'. Use the Google Search tool provided. Summarize your key findings concisely (1-2 sentences). Output *only* the summary. """,
    description="Researches carbon capture methods.",
    tools=[google_search],
    # Store result in state for the merger agent
    output_key="carbon_capture_result",
)


# --- 2. Create the ParallelAgent (Runs researchers concurrently) ---
# This agent orchestrates the concurrent execution of the researchers.
# It finishes once all researchers have completed and stored their results in state.
parallel_research_agent = ParallelAgent(
    name="ParallelWebResearchAgent",
    sub_agents=[researcher_agent_1, researcher_agent_2, researcher_agent_3],
    description="Runs multiple research agents in parallel to gather information.",
)


# --- 3. Define the Merger Agent (Runs after the parallel agents) ---
# This agent takes the results stored in the session state by the parallel agents
# and synthesizes them into a single, structured response with attributions.
merger_agent = LlmAgent(
    name="SynthesisAgent",
    model=GEMINI_MODEL,  # Or potentially a more powerful model if needed for synthesis
    instruction="""You are an AI Assistant responsible for combining research findings into a structured report. Your primary task is to synthesize the following research summaries, clearly attributing findings to their source areas. Structure your response using headings for each topic. Ensure the report is coherent and integrates the key points smoothly.

**Crucially:** Your entire response MUST be grounded *exclusively* on the information provided in the 'Input Summaries' below. Do NOT add any external knowledge, facts, or details not present in these specific summaries.

**Input Summaries:**
*   **Renewable Energy:**
    {renewable_energy_result}
*   **Electric Vehicles:**
    {ev_technology_result}
*   **Carbon Capture:**
    {carbon_capture_result}

**Output Format:**
## Summary of Recent Sustainable Technology Advancements

### Renewable Energy Findings (Based on RenewableEnergyResearcher's findings)
[Synthesize and elaborate *only* on the renewable energy input summary provided above.]

### Electric Vehicle Findings (Based on EVResearcher's findings)
[Synthesize and elaborate *only* on the EV input summary provided above.]

### Carbon Capture Findings (Based on CarbonCaptureResearcher's findings)
[Synthesize and elaborate *only* on the carbon capture input summary provided above.]

### Overall Conclusion
[Provide a brief (1-2 sentence) concluding statement that connects *only* the findings presented above.]

Output *only* the structured report following this format. Do not include introductory or concluding phrases outside this structure, and strictly adhere to using only the provided input summary content.
""",
    description="Combines research findings from parallel agents into a structured, cited report, strictly grounded on provided inputs.",
    # No tools needed for merging
    # No output_key needed here, as its direct response is the final output of the sequence
)


# --- 4. Create the SequentialAgent (Orchestrates the overall flow) ---
# This is the main agent that will be run. It first executes the ParallelAgent
# to populate the state, and then executes the MergerAgent to produce the final output.
sequential_pipeline_agent = SequentialAgent(
    name="ResearchAndSynthesisPipeline",
    # Run parallel research first, then merge
    sub_agents=[parallel_research_agent, merger_agent],
    description="Coordinates parallel research and synthesizes the results.",
)

root_agent = sequential_pipeline_agent
```

## Краткий обзор

**Что:** Многие агентные рабочие процессы включают независимые подзадачи. Чисто последовательное выполнение неэффективно.

**Почему:** Параллелизация позволяет одновременно выполнять независимые задачи. Фреймворки LangChain и Google ADK предоставляют встроенные конструкции.

**Когда использовать:** Когда рабочий процесс содержит несколько независимых операций: запросы к разным API, обработка фрагментов, генерация вариантов.

**Визуальное резюме:**

![Паттерн параллелизации](../assets/Parallelization_Design_Pattern.png)

Рис. 2: Паттерн «Параллелизация»

## Ключевые выводы

* Параллелизация — паттерн одновременного выполнения независимых задач.
* Особенно полезен при ожидании внешних ресурсов (API-вызовы).
* Вносит сложность: дизайн, отладка, логирование.
* LangChain и Google ADK предоставляют встроенную поддержку.
* RunnableParallel в LCEL — ключевая конструкция.
* Снижает латентность и повышает отзывчивость.

## Заключение

Паттерн параллелизации оптимизирует рабочие процессы за счёт одновременного выполнения подзадач. Интеграция с цепочками и маршрутизацией позволяет строить сложные высокопроизводительные системы.

## Ссылки

1. Документация LangChain Expression Language (LCEL): [https://python.langchain.com/docs/concepts/lcel/](https://python.langchain.com/docs/concepts/lcel/)
2. Документация Google ADK (многоагентные системы): [https://google.github.io/adk-docs/agents/multi-agents/](https://google.github.io/adk-docs/agents/multi-agents/)
3. Документация Python asyncio: [https://docs.python.org/3/library/asyncio.html](https://docs.python.org/3/library/asyncio.html)
