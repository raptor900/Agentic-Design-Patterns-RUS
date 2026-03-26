# Приложение A: Продвинутые техники промптинга

## Введение в промптинг

Промптинг — основной интерфейс взаимодействия с языковыми моделями. Процесс создания входных данных для направления модели к генерации желаемого результата. Включает структурирование запросов, предоставление контекста, указание формата вывода.

Хорошо спроектированные промпты максимизируют потенциал моделей. Плохо спроектированные — приводят к неоднозначным результатам.

Цель prompt engineering — consistently получать высококачественные ответы. Требуется понимание возможностей и ограничений моделей.

## Базовые принципы промптинга

**Ясность и конкретность:** Однозначные и точные инструкции. Определяйте задачу, формат, ограничения.

**Краткость:** Прямые инструкции без лишних конструкций.

**Использование глаголов:** Analyze, Categorize, Classify, Compare, Create, Describe, Evaluate, Generate, Identify, List, Organize, Summarize, Translate, Write.

**Инструкции вместо ограничений:** Позитивные инструкции эффективнее негативных.

**Экспериментация и итерация:** Черновик → тест → анализ → доработка.

## Базовые техники промптинга

### Zero-Shot Prompting

Самая простая форма: инструкция + данные без примеров. Для знакомых модели задач.

### One-Shot Prompting

Один пример вход/выход. Для нестандартного формата.

### Few-Shot Prompting

3-5 примеров. Для специфических форматов и классификации. Важность качества и разнообразия примеров. Перемешивание классов. Many-Shot для сотен примеров.

## Структурирование промптов

### System Prompting

Общий контекст и поведение. Тон, стиль, правила. Автоматическая оптимизация (Vertex AI Prompt Optimizer).

### Role Prompting

Назначение персонажа. Определяет тон и экспертизу.

### Разделители

Визуальное разделение инструкций, контекста, примеров, входа.

## Инжиниринг контекста

Динамическое предоставление фоновой информации. Качество контекста важнее архитектуры модели. Слои: системные промпты, внешние данные, неявные данные.

## Структурированный вывод

Запрос JSON, XML, CSV для machine-readable формата. Pydantic для валидации через `model_validate_json`.

```), XML tags (<instruction>, <context>), or markers (---), can be utilized to visually and programmatically separate these sections. This practice, widely used in prompt engineering, minimizes misinterpretation by the model, ensuring clarity regarding the role of each part of the prompt.

* **Example:**  
  <instruction>Summarize the following article, focusing on the main arguments presented by the author.</instruction>  
  <article>  
  [Insert the full text of the article here]  
  </article>

## Contextual Engineering

Context engineering, unlike static system prompts, dynamically provides background information crucial for tasks and conversations. This ever-changing information helps models grasp nuances, recall past interactions, and integrate relevant details, leading to grounded responses and smoother exchanges. Examples include previous dialogue, relevant documents (as in Retrieval Augmented Generation), or specific operational parameters. For instance, when discussing a trip to Japan, one might ask for three family-friendly activities in Tokyo, leveraging the existing conversational context. In agentic systems, context engineering is fundamental to core agent behaviors like memory persistence, decision-making, and coordination across sub-tasks. Agents with dynamic contextual pipelines can sustain goals over time, adapt strategies, and collaborate seamlessly with other agents or tools—qualities essential for long-term autonomy. This methodology posits that the quality of a model's output depends more on the richness of the provided context than on the model's architecture. It signifies a significant evolution from traditional prompt engineering, which primarily focused on optimizing the phrasing of immediate user queries. Context engineering expands its scope to include multiple layers of information.

These layers include:

* **System prompts:** Foundational instructions that define the AI's operational parameters (e.g., "You are a technical writer; your tone must be formal and precise").  
* **External data:**  
  * **Retrieved documents:** Information actively fetched from a knowledge base to inform responses (e.g., pulling technical specifications).  
  * **Tool outputs:** Results from the AI using an external API for real-time data (e.g., querying a calendar for availability).  
* **Implicit data:** Critical information such as user identity, interaction history, and environmental state. Incorporating implicit context presents challenges related to privacy and ethical data management. Therefore, robust governance is essential for context engineering, especially in sectors like enterprise, healthcare, and finance.

The core principle is that even advanced models underperform with a limited or poorly constructed view of their operational environment. This practice reframes the task from merely answering a question to building a comprehensive operational picture for the agent. For example, a context-engineered agent would integrate a user's calendar availability (tool output), the professional relationship with an email recipient (implicit data), and notes from previous meetings (retrieved documents) before responding to a query. This enables the model to generate highly relevant, personalized, and pragmatically useful outputs. The "engineering" aspect involves creating robust pipelines to fetch and transform this data at runtime and establishing feedback loops to continually improve context quality.

To implement this, specialized tuning systems, such as Google's Vertex AI prompt optimizer, can automate the improvement process at scale. By systematically evaluating responses against sample inputs and predefined metrics, these tools can enhance model performance and adapt prompts and system instructions across different models without extensive manual rewriting. Providing an optimizer with sample prompts, system instructions, and a template allows it to programmatically refine contextual inputs, offering a structured method for implementing the necessary feedback loops for sophisticated Context Engineering.  
This structured approach differentiates a rudimentary AI tool from a more sophisticated, contextually-aware system. It treats context as a primary component, emphasizing what the agent knows, when it knows it, and how it uses that information. This practice ensures the model has a well-rounded understanding of the user's intent, history, and current environment. Ultimately, Context Engineering is a crucial methodology for transforming stateless chatbots into highly capable, situationally-aware systems.

## Structured Output

Often, the goal of prompting is not just to get a free-form text response, but to extract or generate information in a specific, machine-readable format. Requesting structured output, such as JSON, XML, CSV, or Markdown tables, is a crucial structuring technique. By explicitly asking for the output in a particular format and potentially providing a schema or example of the desired structure, you guide the model to organize its response in a way that can be easily parsed and used by other parts of your agentic system or application. Returning JSON objects for data extraction is beneficial as it forces the model to create a structure and can limit hallucinations. Experimenting with output formats is recommended, especially for non-creative tasks like extracting or categorizing data.

* **Example:**  
  Extract the following information from the text below and return it as a JSON object with keys `name`, `address`, and `phone.number`.

  Text: "Contact John Smith at 123 Main St, Anytown, CA or call (555) 123-4567."

Effectively utilizing system prompts, role assignments, contextual information, delimiters, and structured output significantly enhances the clarity, control, and utility of interactions with language models, providing a strong foundation for developing reliable agentic systems. Requesting structured output is crucial for creating pipelines where the language model's output serves as the input for subsequent system or processing steps.

**Leveraging Pydantic for an Object-Oriented Facade:** A powerful technique for enforcing structured output and enhancing interoperability is to use the LLM's generated data to populate instances of Pydantic objects. Pydantic is a Python library for data validation and settings management using Python type annotations. By defining a Pydantic model, you create a clear and enforceable schema for your desired data structure. This approach effectively provides an object-oriented facade to the prompt's output, transforming raw text or semi-structured data into validated, type-hinted Python objects.

You can directly parse a JSON string from an LLM into a Pydantic object using the `model.validate.json` method. This is particularly useful as it combines parsing and validation in a single step.

```

# Техники рассуждения и мыслительного процесса

## Chain of Thought (CoT)

Явное направление модели генерировать промежуточные шаги. «Подумай пошагово.» Улучшает точность в задачах с вычислениями и логикой.

* **Zero-Shot CoT:** «Let's think step by step» без примеров.
* **Few-Shot CoT:** Примеры с пошаговыми рассуждениями.

Преимущества: интерпретируемость, робастность. Недостаток: рост стоимости.

## Self-Consistency

Несколько параллельных путей рассуждения для одной задачи, затем majority vote. Улучшает точность, умножает стоимость.

## Step-Back Prompting

Сначала вопрос об общем принципе, затем конкретная задача с контекстом. Позволяет модели активировать фоновые знания.

## Tree of Thoughts (ToT)

Несколько путей рассуждения одновременно, древовидная структура. Для сложных задач с исследованием и backtrack.

# Техники действия и взаимодействия

## Tool Use / Function Calling

Агент использует внешние инструменты. Модель генерирует JSON-вызов. Система выполняет и возвращает результат.

## ReAct (Reason & Act)

Цикл: Мысль, Действие, Наблюдение, Мысль. Позволяет агенту собирать информацию и адаптировать план.

## Продвинутые техники

### Automatic Prompt Engineering (APE)

Использование LLM для генерации и оценки промптов. Meta-model генерирует кандидатов, оценивает, выбирает лучшего.

### Iterative Prompting / Refinement

Человеко-ориентированный цикл: простой промпт → результат → доработка → повтор.

### Providing Negative Examples

Показ нежелательного вывода для clarification границ.

### Using Analogies

Объяснение задачи через знакомое сравнение.

### Factored Cognition / Decomposition

Разбиение сложной задачи на подзадачи с отдельными промптами.

### Retrieval Augmented Generation (RAG)

Доступ к внешней актуальной информации. Поиск в БД, контекст в промпт, grounded ответ.

### Persona Pattern (User Persona)

Описание аудитории вместо роли модели.

## Using Google Gems

Пользовательские экземпляры Gemini со специализированными инструкциями. Постоянный контекст без повторений.

## Using LLMs to Refine Prompts (The Meta Approach)

LLM анализирует и улучшает промпты. Accelerates итерацию, выявляет blind spots.

## Prompting for Specific Tasks

### Code Prompting

Генерация, объяснение, перевод, отладка кода.

### Multimodal Prompting

Комбинация входов: изображение + текст.

## Best Practices and Experimentation

Примеры, простота, конкретность, формат вывода, переменные, эксперименты, документирование, автоматические тесты.

## Conclusion

Prompts — инженерная дисциплина. Базовые принципы — фундамент. Структурирование — контроль. CoT, ToT — декомпозиция. JSON + Pydantic — predictability. ReAct — действие. RAG и контекст — grounding. Мастерство превращает генератор текста в автономного агента.

## References

1. Prompt Engineering: [https://www.kaggle.com/whitepaper-prompt-engineering](https://www.kaggle.com/whitepaper-prompt-engineering)
2. Chain-of-Thought: [https://arxiv.org/abs/2201.11903](https://arxiv.org/abs/2201.11903)
3. Self-Consistency: [https://arxiv.org/pdf/2203.11171](https://arxiv.org/pdf/2203.11171)
4. ReAct: [https://arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)
5. Tree of Thoughts: [https://arxiv.org/pdf/2305.10601](https://arxiv.org/pdf/2305.10601)
6. Take a Step Back: [https://arxiv.org/abs/2310.06117](https://arxiv.org/abs/2310.06117)
7. DSPy: [https://github.com/stanfordnlp/dspy](https://github.com/stanfordnlp/dspy)
