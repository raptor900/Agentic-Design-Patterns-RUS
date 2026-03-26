# Глава 4: Рефлексия

## Обзор паттерна «Рефлексия»

В предыдущих главах мы рассмотрели базовые агентные паттерны: цепочку для последовательного выполнения, маршрутизацию для динамического выбора путей и параллелизацию для одновременного выполнения задач. Эти паттерны позволяют агентам выполнять сложные задачи эффективнее и гибче. Однако даже со сложными рабочими процессами первоначальный вывод или план агента может быть неоптимальным, неточным или неполным. Именно здесь в действие вступает паттерн **«Рефлексия»**.

Паттерн рефлексии предполагает, что агент оценивает собственную работу, вывод или внутреннее состояние и использует эту оценку для улучшения производительности или доработки ответа. Это форма самокоррекции или самоусовершенствования, позволяющая агенту итеративно улучшать свой вывод или корректировать подход на основе обратной связи, внутренней критики или сравнения с желаемыми критериями. Рефлексия иногда может обеспечиваться отдельным агентом, чья специфическая роль — анализировать вывод начального агента.

В отличие от простой последовательной цепочки, где вывод передаётся напрямую следующему шагу, или маршрутизации, выбирающей путь, рефлексия вводит петлю обратной связи. Агент не просто производит результат — он затем анализирует этот результат (или процесс его генерации), выявляет потенциальные проблемы или области для улучшения и использует эти инсайты для генерации лучшей версии или корректировки будущих действий.

Процесс обычно включает:

1. **Выполнение:** Агент выполняет задачу или генерирует первоначальный вывод.
2. **Оценка/Критика:** Агент (часто с помощью другого вызова LLM или набора правил) анализирует результат предыдущего шага.
3. **Рефлексия/Доработка:** На основе критики агент определяет, как улучшить результат.
4. **Итерация (необязательно, но типично):** Доработанный вывод затем выполняется, и процесс рефлексии повторяется до удовлетворительного результата.

Ключевая и highly эффективная реализация паттерна рефлексии разделяет процесс на две различные логические роли: Генератор и Критик. Это часто называется моделью «Генератор-Критик» или «Производитель-Ревьюер». Хотя один агент может выполнять саморефлексию, использование двух специализированных агентов (или двух отдельных вызовов LLM с различными системными промптами) часто даёт более надёжные и непредвзятые результаты.

1. **Агент-генератор (Producer):** Основная ответственность — первоначальное выполнение задачи. Фокусируется на генерации контента.
2. **Агент-критик (Critic):** Единственная цель — оценить вывод генератора. Получает другой набор инструкций и часто другую персону.

Разделение ответственности powerful, поскольку предотвращает «когнитивное смещение» агента, рецензирующего собственную работу.

Реализация рефлексии часто требует структурирования рабочего процесса с петлями обратной связи. Может быть реализовано через итеративные циклы или фреймворки с управлением состоянием.

Паттерн рефлексии критически важен для построения агентов, способных производить высококачественные результаты и демонстрировать самосознание и адаптивность.

Пересечение рефлексии с постановкой целей (Глава 11) заслуживает внимания. Цель обеспечивает конечный ориентир для самооценки, мониторинг отслеживает прогресс. Рефлексия acts as корректирующий механизм.

Кроме того, эффективность паттерна рефлексии значительно усиливается, когда LLM хранит память о разговоре (Глава 8). История взаимодействий предоставляет контекст для фазы оценки.

## Практические применения и сценарии использования

Паттерн рефлексии ценен в сценариях, где качество, точность или соответствие сложным ограничениям критически важны:

### 1. Креативное написание и генерация контента

Доработка сгенерированного текста.

* **Сценарий:** Агент пишет пост для блога.
  * **Рефлексия:** Генерация черновика, критика по плавности и ясности, переписывание.
  * **Выгода:** Более отшлифованный контент.

### 2. Генерация и отладка кода

Написание кода, выявление ошибок и их исправление.

* **Сценарий:** Агент пишет функцию на Python.
  * **Рефлексия:** Написание кода, запуск тестов, выявление ошибок, модификация.
  * **Выгода:** Более надёжный код.

### 3. Решение сложных задач

Оценка промежуточных шагов в многошаговых рассуждениях.

* **Сценарий:** Агент решает логическую головоломку.
  * **Рефлексия:** Предложение шага, оценка, возврат при необходимости.
  * **Выгода:** Улучшение навигации в сложных пространствах задач.

### 4. Обобщение и синтез информации

Доработка обобщений на точность, полноту и краткость.

* **Сценарий:** Агент обобщает длинный документ.
  * **Рефлексия:** Начальное обобщение, сравнение с ключевыми пунктами, доработка.
  * **Выгода:** Более точные обобщения.

### 5. Планирование и стратегия

Оценка предложенного плана и выявление недостатков.

* **Сценарий:** Агент планирует действия для достижения цели.
  * **Рефлексия:** Генерация плана, моделирование, корректировка.
  * **Выгода:** Более эффективные планы.

### 6. Разговорные агенты

Обзор предыдущих реплик для поддержания контекста и улучшения ответов.

* **Сценарий:** Чатбот техподдержки.
  * **Рефлексия:** Обзор истории после ответа пользователя.
  * **Выгода:** Более естественные диалоги.

Рефлексия добавляет слой метапознания в агентные системы.

## Практический пример кода (LangChain)

```bash
pip install langchain langchain-community langchain-openai
```

Также необходимо настроить среду с ключом API выбранной модели (OpenAI, Google Gemini, Anthropic).

```python
import os
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.messages import SystemMessage, HumanMessage


# --- Configuration ---
# Load environment variables from .env file (for OPENAI_API_KEY)
load_dotenv()

# Check if the API key is set
if not os.getenv("OPENAI_API_KEY"):
    raise ValueError("OPENAI_API_KEY not found in .env file. Please add it.")

# Initialize the Chat LLM. We use gpt-4o for better reasoning.
# A lower temperature is used for more deterministic outputs.
llm = ChatOpenAI(model="gpt-4o", temperature=0.1)


def run_reflection_loop():
    """
    Demonstrates a multi-step AI reflection loop to progressively improve a Python function.
    """
    # --- The Core Task ---
    task_prompt = """
    Your task is to create a Python function named `calculate_factorial`.
    This function should do the following:
    1.  Accept a single integer `n` as input.
    2.  Calculate its factorial (n!).
    3.  Include a clear docstring explaining what the function does.
    4.  Handle edge cases: The factorial of 0 is 1.
    5.  Handle invalid input: Raise a ValueError if the input is a negative number.
    """

    # --- The Reflection Loop ---
    max_iterations = 3
    current_code = ""

    # We will build a conversation history to provide context in each step.
    message_history = [HumanMessage(content=task_prompt)]

    for i in range(max_iterations):
        print("\n" + "=" * 25 + f" REFLECTION LOOP: ITERATION {i + 1} " + "=" * 25)

        # --- 1. GENERATE / REFINE STAGE ---
        # In the first iteration, it generates. In subsequent iterations, it refines.
        if i == 0:
            print("\n>>> STAGE 1: GENERATING initial code...")
            # The first message is just the task prompt.
            response = llm.invoke(message_history)
            current_code = response.content
        else:
            print("\n>>> STAGE 1: REFINING code based on previous critique...")
            # The message history now contains the task,
            # the last code, and the last critique.
            # We instruct the model to apply the critiques.
            message_history.append(HumanMessage(content="Please refine the code using the critiques provided."))
            response = llm.invoke(message_history)
            current_code = response.content

        print("\n--- Generated Code (v" + str(i + 1) + ") ---\n" + current_code)
        message_history.append(response)  # Add the generated code to history

        # --- 2. REFLECT STAGE ---
        print("\n>>> STAGE 2: REFLECTING on the generated code...")
        # Create a specific prompt for the reflector agent.
        # This asks the model to act as a senior code reviewer.
        reflector_prompt = [
            SystemMessage(content="""
                You are a senior software engineer and an expert
                in Python.
                Your role is to perform a meticulous code review.
                Critically evaluate the provided Python code based
                on the original task requirements.
                Look for bugs, style issues, missing edge cases,
                and areas for improvement.
                If the code is perfect and meets all requirements,
                respond with the single phrase 'CODE_IS_PERFECT'.
                Otherwise, provide a bulleted list of your critiques.
            """),
            HumanMessage(content=f"Original Task:\n{task_prompt}\n\nCode to Review:\n{current_code}"),
        ]

        critique_response = llm.invoke(reflector_prompt)
        critique = critique_response.content

        # --- 3. STOPPING CONDITION ---
        if "CODE_IS_PERFECT" in critique:
            print("\n--- Critique ---\nNo further critiques found. The code is satisfactory.")
            break

        print("\n--- Critique ---\n" + critique)
        # Add the critique to the history for the next refinement loop.
        message_history.append(HumanMessage(content=f"Critique of the previous code:\n{critique}"))

    print("\n" + "=" * 30 + " FINAL RESULT " + "=" * 30)
    print("\nFinal refined code after the reflection process:\n")
    print(current_code)


if __name__ == "__main__":
    run_reflection_loop()
```

## Практический пример кода (Google ADK)

Рассмотрим пример с Google ADK.

```python
from google.adk.agents import SequentialAgent, LlmAgent


# The first agent generates the initial draft.
generator = LlmAgent(
    name="DraftWriter",
    description="Generates initial draft content on a given subject.",
    instruction="Write a short, informative paragraph about the user's subject.",
    output_key="draft_text",  # The output is saved to this state key.
)

# The second agent critiques the draft from the first agent.
reviewer = LlmAgent(
    name="FactChecker",
    description="Reviews a given text for factual accuracy and provides a structured critique.",
    instruction="""
    You are a meticulous fact-checker.
    1. Read the text provided in the state key 'draft_text'.
    2. Carefully verify the factual accuracy of all claims.
    3. Your final output must be a dictionary containing two keys:
       - "status": A string, either "ACCURATE" or "INACCURATE".
       - "reasoning": A string providing a clear explanation for your status, citing specific issues if any are found.
    """,
    output_key="review_output",  # The structured dictionary is saved here.
)

# The SequentialAgent ensures the generator runs before the reviewer.
review_pipeline = SequentialAgent(
    name="WriteAndReview_Pipeline",
    sub_agents=[generator, reviewer],
)

# Execution Flow:
# 1. generator runs -> saves its paragraph to state['draft_text'].
# 2. reviewer runs -> reads state['draft_text'] and saves its dictionary output to state['review_output'].
```

## Краткий обзор

**Что:** AI-агенты часто struggle с задачами, требующими нюансированных суждений, этического рассуждения или глубокого понимания. Полная автономия в критических средах несёт значительные риски.

**Почему:** Паттерн рефлексии стратегически интегрирует человеческий надзор в AI-рабочие процессы. Симбиотическое партнёрство: ИИ — вычисления, человек — критическая валидация и вмешательство.

**Когда использовать:** При внедрении ИИ в домены с серьёзными последствиями ошибок. Для задач с неоднозначностью. Для непрерывного улучшения моделей.

**Визуальное резюме:**

![Паттерн «Человек в контуре»](../assets/Human_in_the_Loop_Design_Pattern.png)

Рис. 1: Паттерн «Человек в контуре»

## Ключевые выводы

* HITL интегрирует человеческий интеллект и суждение в AI-рабочие процессы.
* Критически важен для безопасности, этики и эффективности.
* Ключевые аспекты: надзор, вмешательство, обратная связь, аугментация решений.
* Политики эскалации необходимы для передачи задач человеку.
* Главные ограничения: масштабируемость, зависимость от экспертов, приватность.

## Заключение

Эта глава рассмотрела паттерн «Человек в контуре». Интеграция человеческого надзора и обратной связи повышает надёжность и доверие. По мере развития ИИ HITL остаётся краеугольным камнем ответственной разработки.

## Ссылки

1. A Survey of Human-in-the-loop for Machine Learning: [https://arxiv.org/abs/2108.00941](https://arxiv.org/abs/2108.00941)
