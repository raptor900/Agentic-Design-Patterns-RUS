# Глава 6: Планирование

Интеллектуальное поведение часто включает нечто большее, чем просто реакцию на непосредственный ввод. Оно требует предусмотрительности, разбиения сложных задач на более мелкие управляемые шаги и выработки стратегии достижения желаемого результата. Именно здесь в игру вступает паттерн «Планирование». В основе своей планирование — это способность агента или системы агентов формулировать последовательность действий для перехода из начального состояния в целевое.

## Обзор паттерна «Планирование»

В контексте ИИ полезно рассматривать агента-планировщика как специалиста, которому вы поручаете сложную цель. Когда вы просите «организовать корпоративный выезд», вы определяете *что* (цель и ограничения), но не *как*. Агент автономно прокладывает курс: понимает начальное состояние (бюджет, участники, даты) и целевое (успешно забронированный выезд), затем находит оптимальную последовательность действий. План заранее неизвестен — он создаётся в ответ на запрос.

Характерная черта — адаптивность. Начальный план — лишь отправная точка. Если площадка становится недоступной, capable агент адаптируется: фиксирует ограничение, переоценивает варианты, формулирует новый план.

Однако важно признать компромисс между гибкостью и предсказуемостью. Динамическое планирование — специфический инструмент. Когда решение уже хорошо понятно, ограничение агента предопределённым workflow эффективнее. Решение: нужно ли «как» открывать, или оно уже известно?

## Практические применения и сценарии использования

Паттерн «Планирование» необходим для построения автономных надёжных агентов:

* **Автоматизация клиентской поддержки:** Цель — «решить вопрос клиента по биллингу». Агент отслеживает диалог, проверяет БД, корректирует счета. Успех — подтверждение изменения и положительная обратная связь.
* **Персонализированные обучающие системы:** Цель — «улучшить понимание алгебры». Агент отслеживает прогресс, адаптирует материалы, корректирует подход.
* **Ассистенты управления проектами:** Задача — «обеспечить завершение вехи X к дате Y». Агент отслеживает статусы, коммуникации, ресурсы.
* **Автоматизированные торговые боты:** Цель — «максимизировать доходность в пределах риска». Агент отслеживает рынок, портфель, индикаторы.
* **Робототехника и беспилотники:** Цель — «безопасно доставить из A в B». Агент отслеживает окружение, состояние, прогресс.
* **Модерация контента:** Цель — «выявлять и удалять вредоносный контент». Агент отслеживает входящий контент, применяет классификацию.

## Практический пример кода

```python
import os
from dotenv import load_dotenv
from crewai import Agent, Task, Crew, Process
from langchain_openai import ChatOpenAI


# Load environment variables from .env file for security
load_dotenv()


# 1. Explicitly define the language model for clarity
llm = ChatOpenAI(model="gpt-4-turbo")


# 2. Define a clear and focused agent
planner_writer_agent = Agent(
    role='Article Planner and Writer',
    goal='Plan and then write a concise, engaging summary on a specified topic.',
    backstory=(
        'You are an expert technical writer and content strategist. '
        'Your strength lies in creating a clear, actionable plan before writing, '
        'ensuring the final summary is both informative and easy to digest.'
    ),
    verbose=True,
    allow_delegation=False,
    llm=llm,  # Assign the specific LLM to the agent
)


# 3. Define a task with a more structured and specific expected output
topic = "The importance of Reinforcement Learning in AI"

high_level_task = Task(
    description=(
        f"1. Create a bullet-point plan for a summary on the topic: '{topic}'.\n"
        f"2. Write the summary based on your plan, keeping it around 200 words."
    ),
    expected_output=(
        "A final report containing two distinct sections:\n\n"
        "### Plan\n"
        "- A bulleted list outlining the main points of the summary.\n\n"
        "### Summary\n"
        "- A concise and well-structured summary of the topic."
    ),
    agent=planner_writer_agent,
)


# Create the crew with a clear process
crew = Crew(
    agents=[planner_writer_agent],
    tasks=[high_level_task],
    process=Process.sequential,
)


# Execute the task
print("## Running the planning and writing task ##")
result = crew.kickoff()

print("\n\n---\n## Task Result ##\n---")
print(result)
```

Этот код реализует AI-агента-менеджера проектов с использованием LangChain. Агент facilitating создание, приоритизацию и назначение задач членам команды, illustrating применение языковых моделей с кастомными инструментами.

## Пример кода (CrewAI)

```python
from openai import OpenAI


# Initialize the client with your API key
client = OpenAI(api_key="YOUR_OPENAI_API_KEY")


# Define the agent's role and the user's research question
system_message = """
You are a professional researcher preparing a structured, data-driven report.
Focus on data-rich insights, use reliable sources, and include inline citations.
"""

user_query = "Research the economic impact of semaglutide on global healthcare systems."


# Create the Deep Research API call
response = client.responses.create(
    model="o3-deep-research-2025-06-26",
    input=[
        {
            "role": "developer",
            "content": [{"type": "input_text", "text": system_message}],
        },
        {
            "role": "user",
            "content": [{"type": "input_text", "text": user_query}],
        },
    ],
    reasoning={"summary": "auto"},
    tools=[{"type": "web_search_preview"}],
)


# Access and print the final report from the response
final_report = response.output[-1].content[0].text
print(final_report)


# --- ACCESS INLINE CITATIONS AND METADATA ---
print("--- CITATIONS ---")
annotations = response.output[-1].content[0].annotations

if not annotations:
    print("No annotations found in the report.")
else:
    for i, citation in enumerate(annotations):
        # The text span the citation refers to
        cited_text = final_report[citation.start_index : citation.end_index]
        print(f"Citation {i + 1}:")
        print(f"  Cited Text: {cited_text}")
        print(f"  Title: {citation.title}")
        print(f"  URL: {citation.url}")
        print(f"  Location: chars {citation.start_index}–{citation.end_index}")

print("\n" + "=" * 50 + "\n")


# --- INSPECT INTERMEDIATE STEPS ---
print("--- INTERMEDIATE STEPS ---")

# 1. Reasoning Steps: Internal plans and summaries generated by the model.
try:
    reasoning_step = next(item for item in response.output if item.type == "reasoning")
    print("\n[Found a Reasoning Step]")
    for summary_part in reasoning_step.summary:
        print(f"  - {summary_part.text}")
except StopIteration:
    print("\nNo reasoning steps found.")

# 2. Web Search Calls: The exact search queries the agent executed.
try:
    search_step = next(item for item in response.output if item.type == "web_search_call")
    print("\n[Found a Web Search Call]")
    print(f"  Query Executed: '{search_step.action['query']}'")
    print(f"  Status: {search_step.status}")
except StopIteration:
    print("\nNo web search steps found.")

# 3. Code Execution: Any code run by the agent using the code interpreter.
try:
    code_step = next(item for item in response.output if item.type == "code_interpreter_call")
    print("\n[Found a Code Execution Step]")
    print("  Code Input:")
    print(f"  ```

В этом фрагменте внутри кода — вывод отладочной информации.

```")
    print("  Code Output:")
    print(f"  {code_step.output}")
except StopIteration:
    print("\nNo code execution steps found.")
```

## Пример кода (OpenAI)

Этот скрипт использует OpenAI API для выполнения «Deep Research» — автоматизации сложных исследовательских задач. Использует модель o3-deep-research-2025-06-26, которая автономно рассуждает, планирует и синтезирует информацию из веб-источников. Скрипт определяет системное сообщение (роль исследователя), пользовательский запрос и инструмент `web_search_preview`. После получения ответа извлекается финальный отчёт, цитаты и промежуточные шаги (рассуждения, поисковые запросы, код).

## Краткий обзор

**Что:** AI-агенты работают в динамичных средах, где предопределённой логики недостаточно. Без способности к планированию они не могут решать сложные многошаговые задачи.

**Почему:** Продвинутые системы (Deep Research) используют LLM и эволюционные алгоритмы для автономного обнаружения и оптимизации алгоритмов.

**Когда использования:** Для задач в динамичных, неопределённых или изменяющихся средах.

**Визуальное резюме:**

![Паттерн планирования](../assets/Planning_Design_Pattern.png)

Рис. 1: Паттерн «Планирование»

## Ключевые выводы

* Планирование позволяет агентам разбивать сложные цели на последовательные действия.
* Необходимо для многошаговых задач, автоматизации, навигации в сложных средах.
* LLM могут генерировать пошаговые подходы на основе описаний задач.
* Deep Research — агент, анализирующий источники с Google Search, с рефлексией, планированием и выполнением.

## Заключение

Паттерн «Планирование» — фундаментальный компонент, переводящий агентные системы от реактивных к проактивным целенаправленным исполнителям. Масштабируется от простого последовательного выполнения до сложных динамических систем. Планирование — мост между намерением пользователя и автоматизированным выполнением.

## Ссылки

1. Google DeepResearch: [gemini.google.com](http://gemini.google.com)
2. OpenAI Deep Research: [https://openai.com/index/introducing-deep-research/](https://openai.com/index/introducing-deep-research/)
3. Perplexity Deep Research: [https://www.perplexity.ai/hub/blog/introducing-perplexity-deep-research](https://www.perplexity.ai/hub/blog/introducing-perplexity-deep-research)
