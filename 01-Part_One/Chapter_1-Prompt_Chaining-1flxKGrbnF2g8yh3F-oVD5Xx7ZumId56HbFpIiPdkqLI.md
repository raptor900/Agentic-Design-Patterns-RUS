# Глава 1: Prompt Chaining

## Обзор паттерна Prompt Chaining

Prompt chaining, иногда называемый Pipeline pattern, представляет мощную парадигму для обработки запутанных задач при использовании больших языковых моделей (LLM). Вместо того чтобы ожидать, что LLM решит сложную проблему за один, монолитный шаг, prompt chaining продвигает стратегию "разделяй и властвуй". Основная идея — разбить исходную, пугающую проблему на последовательность более мелких, более управляемых подзадач. Каждая подзадача решается индивидуально через специально спроектированный промпт, и вывод, сгенерированный из одного промпта, стратегически подаётся как вход в последующий промпт в цепочке.

Эта техника последовательной обработки по своей природе вносит модульность и ясность во взаимодействие с LLM. Декомпозируя сложную задачу, становится легче понимать и отлаживать каждый индивидуальный шаг, делая общий процесс более robust и interpretable. Каждый шаг в цепочке может быть тщательно crafted и оптимизирован, чтобы сосредоточиться на конкретном аспекте более крупной проблемы, приводя к более точным и сфокусированным выводам.

Вывод одного шага, действующий как вход для следующего, crucial. Эта передача информации устанавливает chain зависимости, отсюда и название, где контекст и результаты предыдущих операций направляют последующую обработку. Это позволяет LLM build на своей предыдущей работе, уточнять своё понимание и постепенно двигаться ближе к желаемому решению.

Более того, prompt chaining — это не только о разбиении проблем; он также позволяет интегрировать внешние знания и инструменты. На каждом шаге LLM может быть инструктирован взаимодействовать с внешними системами, API или базами данных, обогащая свои знания и способности за пределы его internal training data. Эта способность dramatically расширяет потенциал LLM, позволяя им функционировать не просто как изолированные модели, а как integral компоненты более широких, более интеллектуальных систем.

Значение prompt chaining выходит за рамки простого problem-solving. Он служит фундаментальной техникой для построения сложных AI-агентов. Эти агенты могут использовать prompt chains для автономного планирования, reasoning и действий в динамических средах. Стратегически структурируя последовательность промптов, агент может engage в задачах, требующих multi-step reasoning, планирования и принятия решений. Такие agent workflows могут более closely imitate человеческие мыслительные процессы, позволяя более естественным и эффективным взаимодействиям со сложными доменами и системами.

**Ограничения single prompts:** Для многогранных задач использование одного сложного промпта для LLM может быть неэффективным, вызывая, что модель борется с ограничениями и инструкциями, потенциально приводя к instruction neglect, где части промпта упускаются, contextual drift, где модель теряет начальный контекст, error propagation, где ранние ошибки усиливаются, промпты, которые требуют более длинного контекстного окна, где модель получает недостаточно информации, чтобы ответить, и hallucination, где cognitive load увеличивает вероятность некорректной информации. Например, запрос, просящий проанализировать отчет маркетингового исследования, суммировать выводы, идентифицировать тренды с data points и составить email, рискует провалом, так как модель может well суммировать, но не извлечь данные или правильно составить email.

**Повышенная надежность через последовательную декомпозицию:** Prompt chaining решает эти проблемы, разбивая сложную задачу на сфокусированный, последовательный workflow, что значительно улучшает надежность и контроль. Учитывая приведённый выше пример, pipeline или chained подход может быть описан так:

1. Initial Prompt (Суммаризация): "Суммируйте ключевые выводы из следующего отчета маркетингового исследования: \[текст\]." Модель focuses solely на суммаризации, увеличивая точность этого initial step.
2. Second Prompt (Идентификация трендов): "Используя суммаризацию, identified top three emerging trends и extract specific data points, которые поддерживают каждый тренд: \[output from step 1\]." Этот промпт теперь более constrained и builds directly upon validated output.
3. Third Prompt (Составление email): "Составьте краткий email команде маркетинга, который излагает следующие тренды и их supporting data: \[output from step 2\]."

Эта декомпозиция позволяет более granular control над процессом. Каждый шаг проще и менее неоднозначен, что снижает cognitive load на модели и приводит к более точному и надежному final output. Эта модульность аналогична вычислительному pipeline, где каждая функция выполняет конкретную операцию перед передачей своего результата следующей. Чтобы обеспечить точный ответ для каждой конкретной задачи, модели можно назначить distinct role на каждом этапе. Например, в данном сценарии initial промпт может быть designated как "Market Analyst", последующий промпт как "Trade Analyst" и третий промпт как "Expert Documentation Writer", и так далее.

**Роль структурированного вывода:** Надежность prompt chain сильно зависит от целостности данных, передаваемых между шагами. Если вывод одного промпта неоднозначен или плохо отформатирован, последующий промпт может fail из-за faulty input. Чтобы смягчить это, specifying a structured output format, such as JSON or XML, является crucial.

Например, вывод из шага идентификации трендов could be formatted as a JSON object:

```json
{  "trends": [    
        {      
            "trend_name": "AI-Powered Personalization",      
            "supporting_data": "73% of consumers prefer to do business with brands that use personal information to make their shopping experiences more relevant."    
        },    
        {      
            "trend_name": "Sustainable and Ethical Brands",      
            "supporting_data": "Sales of products with ESG-related claims grew 28% over the last five years, compared to 20% for products without."    
        } 
    ] 
}
```

Этот структурированный формат гарантирует, что данные являются machine-readable и могут быть точно разобраны и вставлены в следующий промпт без неоднозначности. Эта практика минимизирует ошибки, которые могут возникнуть при интерпретации естественного языка, и является ключевым компонентом в построении robust, multi-step LLM-based систем.

## Практические приложения и сценарии использования

Prompt chaining — это универсальный паттерн, применимый в широком спектре сценариев при построении agentic-систем. Его core utility заключается в разбиении сложных проблем на последовательные, управляемые шаги. Вот несколько практических приложений и use cases:

### 1. Workflow обработки информации

Многие задачи вовлекают обработку сырой информации через multiple transformations. Например, summary документа, извлечение ключевых сущностей, а затем использование этих сущностей для запроса базы данных или генерации отчета. Prompt chain could look like:

* Prompt 1: Извлечь текстовое содержимое из заданного URL или документа.
* Prompt 2: Суммировать очищенный текст.
* Prompt 3: Извлечь конкретные сущности (например, имена, даты, локации) из summary или исходного текста.
* Prompt 4: Использовать сущности для поиска во внутренней базе знаний.
* Prompt 5: Сгенерировать final report, включающий summary, сущности и результаты поиска.

Эта methodology применяется в domains, таких как автоматизированный анализ контента, разработка AI-driven research assistants и сложная генерация отчетов.

### 2. Сложный ответ на запросы

Ответы на сложные вопросы, требующие multiple steps reasoning или information retrieval, — это prime use case. Например, "Каковы были основные причины краха фондового рынка в 1929 году и как правительственная политика отреагировала?"

* Prompt 1: Идентифицировать core sub-questions в запросе пользователя (причины краха, реакция правительства).
* Prompt 2: Исследовать или retrieve информацию specifically о причинах краха 1929 года.
* Prompt 3: Исследовать или retrieve информацию specifically о политике правительства в ответ на крах фондового рынка 1929 года.
* Prompt 4: Синтезировать информацию из шагов 2 и 3 в coherent answer на исходный запрос.

Эта последовательная методология обработки интегральна для развития AI-систем, способных к multi-step inference и information synthesis. Такие системы требуются, когда запрос не может быть отвечен из single data point, но instead требует series logical steps или интеграции информации из diverse sources.

Например, автоматизированный research agent, разработанный для генерации comprehensive report на конкретную тему, выполняет гибридный computational workflow. Initially, система извлекает numerous relevant articles. Последующая задача извлечения ключевой информации из каждой статьи может быть performed concurrently для каждого источника. Этот этап well-suited для parallel processing, где независимые sub-tasks run simultaneously для максимизации efficiency.

Однако, после того как индивидуальные извлечения завершены, процесс становится inherently sequential. Система должна сначала collate извлечённые данные, затем синтезировать их в coherent draft, и наконец review и refine этот draft, чтобы produce final report. Каждый из этих последующих этапов логически зависит от успешного завершения предыдущего. Вот где prompt chaining применяется: collated data служит input для synthesis prompt, и resulting synthesized text становится input для final review prompt. Таким образом, сложные операции часто комбинируют parallel processing для independent data gathering с prompt chaining для dependent steps synthesis и refinement.

### 3. Извлечение и трансформация данных

Конвертация неструктурированного текста в структурированный format обычно достигается через iterative process, требующую последовательных модификаций для улучшения точности и полноты вывода.

* Prompt 1: Попытка извлечь specific fields (например, имя, адрес, суммаuality) из invoice документа.
* Processing: Проверить, все ли обязательные поля были извлечены и соответствуют ли они format requirements.
* Prompt 2 (Условный): Если поля отсутствуют или malformed, crafted новый промпт, просящий модель конкретно найти missing/malformed информацию, возможно, предоставляя контекст из неудачной попытки.
* Processing: Снова валидировать результаты. Повторить при необходимости.
* Output: Предоставить извлечённые, валидированные структурированные данные.

Эта последовательная методология обработки особенно применима к извлечению и анализу данных из неструктурированных источников, таких как формы, invoices или emails. Например, решение сложных проблем Optical Character Recognition (OCR), таких как processing PDF формы, более эффективно обрабатывается через декомпозированный, multi-step подход.

Initially, large language model используется для выполнения primary text extraction из изображения документа. Following this, модель обрабатывает raw output для normalization данных, шаг, где она может convert numeric text, такой как "one thousand and fifty", в his числовой эквивалент, 1050. Значительный вызов для LLM — выполнение точных математических calculations. Поэтому, на последующем шаге, система может делегировать любые необходимые арифметические операции внешнему tool калькулятору. LLM идентифицирует необходимое вычисление, передаёт нормализованные числа tool, и затем включает точный результат. Эта chained последовательность text extraction, data normalization и external tool use достигает final, точный результат, который часто difficult to obtain reliably из single LLM query.

### 4. Workflow генерации контента

Композиция сложного контента — это procedural task, который обычно decomposes into distinct phases, включая initial ideation, structural outlining, drafting, и последующий revision.

* Prompt 1: Сгенерировать 5 идей тем на основе общего интереса пользователя.
* Processing: Пользователю разрешено выбрать одну идею или автоматически выбрать лучшую.
* Prompt 2: На основе выбранной темы, сгенерировать детальный outline.
* Prompt 3: Написать draft section на основе первого пункта в outline.
* Prompt 4: Написать draft section на основе второго пункта в outline, предоставляя предыдущий раздел для контекста. Продолжить для всех пунктов outline.
* Prompt 5: Review и refine complete draft для coherence, tone и grammar.

Эта methodology используется для range natural language generation задач, включая автоматизированную композицию creative narratives, технической документации и других форм структурированного текстового контента.

### 5. Разговорные агенты с состоянием (Conversational Agents with State)

Хотя comprehensive state management architectures employ methods более complex чем sequential linking, prompt chaining предоставляет foundational mechanism для preservation conversational continuity. Эта техника сохраняет контекст, конструируя каждый conversational turn как новый промпт, который систематически включает информацию или извлечённые сущности из предыдущих взаимодействий в dialogue sequence.

* Prompt 1: Process User Utterance 1, identified intent и key entities.
* Processing: Update conversation state с intent и entities.
* Prompt 2: На основе текущего состояния, сгенерировать response и/или identified the next required piece of information.
* Repeat для subsequent turns, где каждое новое user utterance инициирует chain, которая leverages накапливающуюся conversational history (state).

Этот принцип фундаментален для разработки conversational agents, enabling them to maintain context и coherence across extended, multi-turn dialogues. Сохраняя conversational history, система может понимать и appropriately отвечать на user inputs, которые зависят от ранее обмененной информации.

### 6. Генерация и улучшение кода (Code Generation and Refinement)

Генерация функционального кода обычно является multi-stage process, требуя, чтобы проблема была декомпозирована в последовательность дискретных логических операций, которые выполняются progressively.

* Prompt 1: Понять запрос пользователя для code function. Сгенерировать pseudocode или outline.
* Prompt 2: Написать initial code draft на основе outline.
* Prompt 3: Идентифицировать potential errors или области для улучшения в коде (возможно, используя static analysis tool или другую LLM call).
* Prompt 4: Переписать или refined code на основе identified issues.
* Prompt 5: Добавить документацию или test cases.

В приложениях, таких как AI-assisted разработка ПО, полезность prompt chaining stems из его capacity декомпозировать сложные coding tasks в series manageable sub-проблем. Эта модульная структура снижает operational complexity для large language model на каждом шаге. Критически, этот подход также позволяет вставку deterministic logic между вызовами моделей, enabling intermediate data processing, output validation, и conditional branching внутри workflow. Этим методом, single, multifaceted request, который could otherwise lead to unreliable или incomplete results, converted into структурированную последовательность операций, управляемую underlying execution framework.

### 7. Мультимодальный и multi-step reasoning

Анализ datasets с diverse modalities требует декомпозиции проблемы в smaller, prompt-based задачи. Например, интерпретация изображения, которое содержит картинку с embedded текстом, labels, выделяющие конкретные текстовые сегменты, и tabular data, объясняющее каждый label, требует такого подхода.

* Prompt 1: Извлечь и понять текст из user's image request.
* Prompt 2: Связать извлечённый image text с его corresponding labels.
* Prompt 3: Интерпретировать собранную информацию, используя table, чтобы определить необходимый output.

# Практический пример кода (Hands-On Code Example)

Реализация prompt chaining ranges от direct, последовательных вызовов функций внутри script до использования specialized frameworks, разработанных для управления control flow, state, и component integration. Frameworks, такие как LangChain, LangGraph, Crew AI, и Google Agent Development Kit (ADK), offer structured environments для построения и executing этих multi-step процессов, что особенно advantageous для сложных архитектур.

Для цели демонстрации, LangChain и LangGraph подходят, поскольку их core APIs явно разработаны для composing chains и graphs операций. LangChain предоставляет фундаментальные абстракции для линейных последовательностей, в то время как LangGraph расширяет эти capabilities для поддержки stateful и cyclical computations, необходимых для реализации более сложного agentic behaviors. Этот example сфокусируется на фундаментальной linear sequence.

Следующий код реализует двухшаговую prompt chain, которая functioning как data processing pipeline. Initial stage предназначен для parsing неструктурированного текста и extraction specific information. Последующий stage затем получает этот извлечённый output и трансформирует его в structured data format.

Чтобы replicate эту процедуру, необходимые libraries должны first быть installed. Это может быть accomplished с помощью следующей команды:

```bash
pip install langchain langchain-community langchain-openai langgraph
```

Обратите внимание, что langchain-openai может быть substituted с appropriate package для different model provider. Subsequently, execution environment должен быть configured с necessary API credentials для выбранного language model provider, такого как OpenAI, Google Gemini, или Anthropic.

```python
import os 
from langchain_openai import ChatOpenAI 
from langchain_core.prompts import ChatPromptTemplate 
from langchain_core.output_parsers import StrOutputParser 

# Для лучшей безопасности, загружайте переменные окружения из .env файла 
# from dotenv import load_dotenv 
# load_dotenv() 
# Убедитесь, что ваш OPENAI_API_KEY установлен в .env файле 

# Initialize the Language Model (использование ChatOpenAI рекомендуется) 

llm = ChatOpenAI(temperature=0) 

# --- Prompt 1: Извлечь информацию ---

prompt_extract = ChatPromptTemplate.from_template(
    "Извлеките технические спецификации из следующего текста:\n\n{text_input}" 
) 

# --- Prompt 2: Преобразовать в JSON --- 

prompt_transform = ChatPromptTemplate.from_template(
    "Преобразуйте следующие спецификации в JSON объект с ключами 'cpu', 'memory' и 'storage':\n\n{specifications}" 
) 

# --- Построить Chain с использованием LCEL --- 
# StrOutputParser() конвертирует message output модели в простую строку. 
extraction_chain = prompt_extract | llm | StrOutputParser() 

# Полный chain передаёт output extraction chain в переменную 'specifications' 
# для transformation prompt. 
full_chain = (    
    {"specifications": extraction_chain}
        | 
    prompt_transform
        | 
    llm
        | 
    StrOutputParser() 
) 

# --- Запустить Chain --- 

input_text = "Новая модель ноутбука оснащена процессором 3.5 GHz octa-core, 16GB оперативной памяти и 1TB NVMe SSD." 

# Execute chain с словарём input text. 
final_result = full_chain.invoke({"text_input": input_text})
print("\n--- Финальный JSON вывод ---")
print(final_result)
```

Этот Python код демонстрирует, как использовать библиотеку LangChain для обработки текста. Он использует два отдельных промпта: один для извлечения технических спецификаций из входной строки и другой для форматирования этих спецификаций в JSON объект. Модель ChatOpenAI используется для взаимодействий с языковой моделью, и StrOutputParser обеспечивает, что output в usable string format. LangChain Expression Language (LCEL) используется для элегантного связывания этих промптов и языковой модели вместе. Первая цепочка, `extraction_chain`, извлекает спецификации. `full_chain` затем принимает output extraction и использует его как input для transformation prompt. Предоставлен sample input text, описывающий ноутбук. `full_chain` invoked с этим текстом, processing его через оба шага. Final результат, JSON string, содержащий извлечённые и отформатированные спецификации, затем печатается.

## Context Engineering и Prompt Engineering

Context Engineering (см. Рис.1) — это систематическая дисциплина проектирования, конструирования и доставки полного информационного окружения для AI-модели prior to token generation. Эта методология утверждает, что качество вывода модели меньше зависит от архитектуры модели самой и больше от богатства предоставленного контекста.

![Context Engineering](../assets/context_engineering.png)

Рис.1: Context Engineering — это дисциплина построения богатого, всестороннего информационного окружения для AI, так как качество этого контекста является primary factor в enabling advanced Agentic performance.

Это представляет значительную эволюцию от традиционного prompt engineering, которое фокусируется primarily на оптимизации phrasing immediate query пользователя. Context Engineering расширяет эту scope, чтобы включить несколько слоёв информации, таких как **system prompt**, который является фундаментальным набором инструкций, определяющих операционные параметры AI — например, *"You are a technical writer; your tone must be formal and precise."* Контекст далее обогащается external data. Это включает retrieved documents, где AI активно fetches информацию из knowledge base, чтобы inform his response, such as pulling technical specifications for a project. Это также включает tool outputs, которые являются результатами от AI, использующего external API, чтобы obtain real-time data, like querying a calendar to determine user availability. Эта explicit data combined с critical implicit data, such as user identity, interaction history, и environmental state. Core принцип — что даже advanced models underperform, когда provided с limited или poorly constructed view operational environment.

Эта практика, поэтому, reframes задачу от merely answering a question до building comprehensive operational picture for the agent. Например, context-engineered agent не просто respond на query, но first integrates user's calendar availability (tool output), профессиональные отношения с получателем email (implicit data), и заметки с предыдущих встреч (retrieved documents). Это позволяет модели генерировать outputs, которые highly relevant, персонализированные и pragmatically useful. "engineering" component включает создание robust pipelines для fetching и transforming этой данные во время выполнения и establishment feedback loops для continual improvement context quality.

Чтобы implement это, specialized tuning systems могут быть использованы для автоматизации процесса улучшения в масштабе. Например, tools like Google's Vertex AI prompt optimizer могут улучшить model performance, систематически evaluating responses против набора sample inputs и predefined evaluation metrics. Этот подход effective для адаптации промптов и system instructions across different models без requiring extensive manual rewriting. Providing такому оптимизатору sample промпты, system instructions, и template, он может programmatically refine contextual inputs, предлагая structured метод для implementing feedback loops, required для sophisticated Context Engineering.

Этот структурированный подход — то, что differentiates rudimentary AI tool от более sophisticated и contextually-aware системы. Он treating контекст сам по себе как primary component, placing critical importance на то, что агент знает, когда он знает, и как он использует эту информацию. Практика обеспечивает, что модель имеет well-rounded понимание намерения пользователя, истории и текущей среды. Ultimately, Context Engineering — crucial methodology для advancing stateless chatbots в highly capable, situationally-aware системы.

## В summaria

**Что:** Сложные задачи часто overwhelm LLMs, когда handled within single prompt, leading to significant performance issues. Cognitive load на модели увеличивает likelihood ошибок, таких как overlooking instructions, losing context, и generating incorrect information. Monolithic prompt struggles manage multiple constraints и sequential reasoning steps effectively. Это results in unreliable и inaccurate outputs, так как LLM fails address all facets multifaceted request.

**Почему:** Prompt chaining provides standardized solution by breaking down complex problem в последовательность smaller, interconnected sub-задач. Каждый шаг в chain использует focused промпт для выполнения конкретной операции, значительно улучшая reliability и control. Output от одного промпта передаётся как input для следующего, creating logical workflow, который progressively building towards final solution. Эта модульная, divide-and-conquer стратегия делает процесс более manageable, easier debug, и allows для интеграции external tools или structured data formats между шагами. Этот pattern является foundational для developing sophisticated, multi-step Agentic систем, которые могут plan, reason, и execute complex workflows.

**Правило большого пальца:** Используйте этот pattern, когда задача слишком сложна для single промпта, включает multiple distinct processing stages, требует interaction with external tools между шагами, или когда building Agentic систем, которые нужно perform multi-step reasoning и maintain state.

**Визуальное резюме:**

![Prompt Chaining Pattern](../assets/Prompt_Chaining_Pattern.png)

Рис. 2: Prompt Chaining Pattern: Агенты получают серию промптов от пользователя, с output каждого агента, служащим input для следующего в chain.

## Ключевые выводы

Вот некоторые ключевые выводы:

* Prompt Chaining разбивает сложные задачи на последовательность smaller, focused шагов. Это Occasionally known как Pipeline pattern.
* Каждый шаг в chain включает LLM call или processing logic, используя output предыдущего шага как input.
* Этот pattern улучшает reliability и manageability сложных interactions с языковыми моделями.
* Frameworks, такие как LangChain/LangGraph, и Google ADK提供 robust tools для defining, management, и executing этих multi-step последовательностей.

## Заключение

Декомпозируя сложные проблемы в последовательность более простых, управляемых подзадач, prompt chaining предоставляет robust framework для направления больших языковых моделей. Эта "разделяй и властвуй" стратегия значительно усиливает надежность и контроль вывода, фокусируя модель на одной конкретной операции за раз. Как фундаментальный паттерн, он позволяет разрабатывать сложные AI-агенты, способные к multi-step reasoning, интеграции инструментов и управлению состоянием. В конечном счёте, овладение prompt chaining критически важно для построения robust, context-aware систем, которые могут выполнять сложные workflows далеко за пределами возможностей single промпта.

## Ссылки

1. LangChain Documentation on LCEL: [https://python.langchain.com/v0.2/docs/core_modules/expression_language/](https://python.langchain.com/v0.2/docs/core_modules/expression_language/)
2. LangGraph Documentation: [https://langchain-ai.github.io/langgraph/](https://langchain-ai.github.io/langgraph/)
3. Prompt Engineering Guide \- Chaining Prompts: [https://www.promptingguide.ai/techniques/chaining](https://www.promptingguide.ai/techniques/chaining)
4. OpenAI API Documentation (General Prompting Concepts): [https://platform.openai.com/docs/guides/gpt/prompting](https://platform.openai.com/docs/guides/gpt/prompting)
5. Crew AI Documentation (Tasks and Processes): [https://docs.crewai.com/](https://docs.crewai.com/)
6. Google AI for Developers (Prompting Guides): [https://cloud.google.com/discover/what-is-prompt-engineering?hl=en](https://cloud.google.com/discover/what-is-prompt-engineering?hl=en)
7. Vertex Prompt Optimizer [https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/prompt-optimizer](https://cloud.google.com/vertex-ai/generative-ai/docs/learn/prompts/prompt-optimizer)

---

Оригинальный репозиторий: [https://github.com/Mathews-Tom/Agentic-Design-Patterns](https://github.com/Mathews-Tom/Agentic-Design-Patterns)