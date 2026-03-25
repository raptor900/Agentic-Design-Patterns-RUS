# Глава 17: Техники рассуждения

Эта глава посвящена продвинутым методологиям рассуждения для интеллектуальных агентов с фокусом на многошаговые логические выводы и решение задач. Эти техники выходят за рамки простых последовательных операций, делая внутреннее рассуждение агента явным. Это позволяет агентам разбивать задачи, рассматривать промежуточные шаги и приходить к более надёжным и точным выводам. Базовый принцип среди этих продвинутых методов — выделение увеличенных вычислительных ресурсов во время inference. Это означает предоставление агенту или underlying LLM больше времени обработки для генерации ответа. Вместо быстрого однопроходного ответа агент может выполнять итеративную доработку, исследовать несколько путей решения или использовать внешние инструменты.

## Практические применения и сценарии использования

* **Сложные вопросы:** Разрешение multi-hop запросов, требующих интеграции данных из разных источников.
* **Математические задачи:** Разбиение на компоненты, пошаговый процесс, использование кода для точных вычислений.
* **Отладка и генерация кода:** Объяснение рационала, итеративная доработка на основе тестов.
* **Стратегическое планирование:** Рассуждение по вариантам, последствиям, предусловиям.
* **Медицинская диагностика:** Систематическая оценка симптомов, результатов анализов, историй болезни.
* **Юридический анализ:** Анализ документов и прецедентов для формулирования аргументов.

## Техники рассуждения

**Цепочка мыслей (Chain-of-Thought, CoT)** значительно расширяет способности LLM к сложным рассуждениям, имитируя пошаговый мыслительный процесс (см. Рис. 1). Вместо прямого ответа CoT-промпты направляют модель генерировать последовательность промежуточных шагов рассуждения. Эта явная декомпозиция позволяет LLM решать сложные задачи, разбивая их на более простые подзадачи.

![CoT: Цепочка мыслей](../assets/COT_Chain_of_Thought.png)

Рис. 1: CoT-промпт и детальный пошаговый ответ агента.

Пример: инструкции определяют персону и пятишаговый процесс. «Процесс мышления агента» — внутренний монолог, выполняющий инструкции. Это буквальная «цепочка мыслей». Финальный ответ — отшлифованный результат.

```markdown
You are an Information Retrieval Agent. Your goal is to answer the user's question comprehensively and accurately by thinking step-by-step. 

Here's the process you must follow: 
1.  **Analyze the Query:** Understand the core subject and specific requirements. 
2.  **Formulate Search Queries:** Generate precise search queries for a knowledge base. 
3.  **Simulate Information Retrieval:** Consider what information you expect to find. 
4.  **Synthesize Information:** Combine gathered information into a coherent answer. 
5.  **Review and Refine:** Critically evaluate your answer for accuracy and completeness. 

**User Query:** "Explain the main differences between classical computers and quantum computers, and briefly describe a potential application of quantum computing." 

**Agent's Thought Process:** 

**Thought 1 (Analyze Query):** The user wants differences and one application. 

**Thought 2 (Formulate Search Queries):** "differences classical vs quantum computers", "applications of quantum computing". 

**Thought 3 (Simulate Information Retrieval):** Bits vs qubits, superposition, entanglement; applications: drug discovery, material science. 

**Thought 4 (Synthesize Information):** Classical computers use bits (0 or 1), quantum — qubits with superposition and entanglement. Application: drug discovery through molecular simulation. 

**Thought 5 (Review and Refine):** Differences covered, application given, concise and clear. 

**Agent's Final Answer:** Classical computers process information using bits (0 or 1), performing operations sequentially. Quantum computers use qubits with superposition and entanglement, enabling exponentially more complex computations. A significant application: drug discovery, where precise molecular modeling could revolutionize pharmaceutical development.
```

**Дерево мыслей (Tree-of-Thought, ToT)** — техника рассуждения, развивающая CoT. Позволяет LLM исследовать несколько путей рассуждения, формируя древовидную структуру (см. Рис. 2). Поддерживает возврат, самокоррекцию и исследование альтернативных решений.

![ToT: Дерево мыслей](../assets/TOT_Tree_of_Thought.png)

Рис. 2: Пример дерева мыслей

**Самокоррекция (Self-Correction)** — итеративный цикл: черновик → оценка → доработка. Агент критически оценивает промежуточные результаты, выявляет неточности и улучшает свой подход.

Пример: агент-самокорректор оценивает пост для соцсетей. Находит слабости (низкая вовлечённость, неясный призыв к действию), предлагает конкретные улучшения, генерирует отшлифованную версию.

**PALMs (Program-Aided Language Models)** интегрируют LLM с символическими возможностями рассуждения. LLM генерирует и выполняет код (Python) как часть решения задач. Сложные вычисления и логические операции передаются детерминированной среде программирования.

```python
from google.adk.agents import Agent
from google.adk.tools import google_search, agent_tool
from google.adk.code_executors import BuiltInCodeExecutor

search_agent = Agent(model="gemini-2.0-flash", name="SearchAgent", instruction="You're a specialist in Google Search", tools=[google_search])
coding_agent = Agent(model="gemini-2.0-flash", name="CodeAgent", instruction="You're a specialist in Code Execution", code_executor=BuiltInCodeExecutor())

root_agent = Agent(name="RootAgent", model="gemini-2.0-flash", tools=[agent_tool.AgentTool(agent=search_agent), agent_tool.AgentTool(agent=coding_agent)])
```

**RLVR (Reinforcement Learning with Verifiable Rewards):** Стандартный CoT — базовый подход: генерирует одну линию мыслей. Для преодоления этого разработаны специализированные «модели рассуждения», выделяющие переменное время на «размышление». Расширенное рассуждение позволяет self-correction, backtracking, с бóльшими усилиями на сложные задачи. Ключевая инновация — стратегия обучения RLVR: на задачах с известными ответами модель учится через trial-and-error генерировать эффективное длинное рассуждение.

**ReAct (Reasoning and Acting)** — парадигма, объединяющая CoT-промптинг со способностью агента взаимодействовать с внешней средой через инструменты (см. Рис. 3, где KB — база знаний). В отличие от генеративных моделей, ReAct-агент рассуждает о действиях, затем выполняет их, наблюдает результат и корректирует план.

![REACT: Рассуждение и действие](../assets/REACT_Reasoning_and_Act.png)

Рис. 3: Рассуждение и действие

ReAct работает чередованием: действие → наблюдение → рассуждение → действие. Этот цикл «Мысль, Действие, Наблюдение, Мысль...» позволяет динамически адаптировать план. Это критически важно для агентов, позволяя не только рассуждать, но и практически выполнять шаги.

**CoD (Chain of Debates)** — фреймворк Microsoft, где несколько разнообразных моделей сотрудничают и спорят для решения задачи. Система как AI-совет: модели представляют идеи, критикуют рассуждения, обмениваются контраргументами. Цель — повышение точности, снижение смещений.

**GoD (Graph of Debates)** — продвинутый фреймворк, представляющий дискуссию как динамическую нелинейную сеть. Аргументы — узлы, связи — «поддерживает» или «опровергает». Вывод определяется по наиболее устойчивому кластеру аргументов в графе.

**MASS (Multi-Agent System Search)** — фреймворк для автоматизации дизайна многоагентных систем. Трёхэтапная оптимизация:

1. **Block-Level Prompt Optimization:** Оптимизация промптов отдельных агентов.
2. **Workflow Topology Optimization:** Выбор и расположение взаимодействий агентов.
3. **Workflow-Level Prompt Optimization:** Глобальная оптимизация промптов всей системы.

Ключевые принципы: оптимизировать отдельных агентов перед композицией, использовать влиятельные топологии, оптимизировать взаимозависимости.

**Deep Research** — категория AI-агентных инструментов как методичных исследовательских ассистентов (Perplexity AI, Gemini, ChatGPT). Стандартный поиск даёт ссылки; Deep Research даёт развёрнутый отчёт. AI работает агентно: начальное исследование → рассуждение и уточнение → follow-up поиск → финальный синтез.

**Scaling Inference Law:** Производительность LLM predictably улучшается с ростом вычислительных ресурсов во время inference. Меньшая модель с бóльшим «бюджетом размышления» может превзойти большую модель с простой генерацией. Ключевые факторы баланса: размер модели, латентность, операционная стоимость.

## Практический пример кода

DeepSearch от Google доступен через репозиторий `gemini-fullstack-langgraph-quickstart`. Реактивный фронтенд + LangGraph бэкенд. Агент генерирует поисковые запросы, проводит веб-исследование, выполняет рефлексию для выявления пробелов знаний, итеративно уточняет запросы и синтезирует ответ с цитатами.

## Итого: как рассуждают агенты?

Агентный процесс рассуждения — структурированный подход, сочетающий рассуждение и действие:

1. **Мысль:** Агент генерирует текстовую мысль, разбивающую задачу.
2. **Действие:** На основе мысли выбирает действие из набора (поиск, извлечение, ответ).
3. **Наблюдение:** Получает обратную связь из среды.

Цикл повторяется до финального решения. Частота мыслей настраивается: для фактологических задач — с каждым действием, для навигационных — реже.

## Краткий обзор

**Что:** Сложные задачи требуют не одного прямого ответа, а структурированного подхода с декомпозицией, многошаговой логикой и взаимодействием с инструментами.

**Почему:** Набор техник рассуждения: CoT, ToT, самокоррекция, ReAct. Фреймворки вроде CoD создают команды агентов.

**Когда использовать:** Когда задача слишком сложна для однопроходного ответа. Когда нужна прозрачность процесса рассуждения.

**Визуальное резюме:**

![Паттерн рассуждения](../assets/Reasoning_Design_Pattern.png)

Рис. 7: Паттерн «Рассуждение»

## Ключевые выводы

* Явное рассуждение позволяет формулировать прозрачные многошаговые планы.
* ReAct — основной операционный цикл агента: мысль → действие → наблюдение.
* Scaling Inference Law: производительность зависит от «времени размышления».
* CoT — внутренний монолог агента для структурированного планирования.
* ToT и самокоррекция дают способность оценивать стратегии и возвращаться назад.
* CoD — переход от одиночных агентов к многоагентным системам.
* Deep Research — автономное выполнение сложных исследований.
* MASS автоматизирует оптимизацию дизайна многоагентных систем.

## Заключение

Современный ИИ эволюционирует из пассивных инструментов в автономных агентов. Автономное поведение начинается с внутреннего монолога (CoT), развивается через самокоррекцию и ToT, совершает решающий скачок через ReAct (действие + инструменты). Scaling Inference Law обеспечивает ресурсы для глубокого размышления. Многоагентные системы (CoD, GoD) создают коллективный интеллект. Цель — надёжные прозрачные автономные агенты.

## Ссылки

1. Wei et al. (2022). "Chain-of-Thought Prompting Elicits Reasoning in Large Language Models"
2. Yao et al. (2023). "Tree of Thoughts: Deliberate Problem Solving with Large Language Models"
3. Gao et al. (2023). "Program-Aided Language Models"
4. Yao et al. (2023). "ReAct: Synergizing Reasoning and Acting in Language Models"
5. Inference Scaling Laws (2024)
6. Multi-Agent Design: [https://arxiv.org/abs/2502.02533](https://arxiv.org/abs/2502.02533)
