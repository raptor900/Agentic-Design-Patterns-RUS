# Глава 17: Техники рассуждения

Эта глава посвящена продвинутым методологиям рассуждения для интеллектуальных агентов с фокусом на многошаговые логические выводы и решение задач. Эти техники делают внутреннее рассуждение агента явным. Базовый принцип — выделение увеличенных вычислительных ресурсов во время inference.

## Практические применения и сценарии использования

* **Сложные вопросы:** Multi-hop запросы с интеграцией данных из разных источников.
* **Математические задачи:** Разбиение на компоненты, пошаговое выполнение.
* **Отладка кода:** Объяснение рационала, итеративная доработка.
* **Стратегическое планирование:** Рассуждение по вариантам и последствиям.
* **Медицинская диагностика:** Систематическая оценка симптомов и данных.
* **Юридический анализ:** Анализ документов и прецедентов.

## Техники рассуждения

**Chain of Thought (CoT):** Промптинг, направляющий модель генерировать промежуточные шаги рассуждения. Улучшает точность в задачах с вычислениями и логикой.

* **Zero-Shot CoT:** «Let's think step by step» без примеров.
* **Few-Shot CoT:** CoT + few-shot с пошаговыми рассуждениями.

Преимущества: low-effort, интерпретируемость, робастность. Недостаток: удлинение вывода, рост стоимости. Best practices: ответ после рассуждений, температура = 0.

**Self-Consistency:** Несколько путей рассуждения, majority vote. Улучшает точность, умножает стоимость.

**Step-Back Prompting:** Сначала общий принцип, затем конкретная задача с контекстом.

**Tree of Thoughts (ToT):** Несколько путей рассуждения одновременно, древовидная структура.

```markdown
You are an Information Retrieval Agent. Your goal is to answer the user's question comprehensively and accurately by thinking step-by-step. 

Here's the process you must follow: 
1.  **Analyze the Query:** Understand the core subject and specific requirements of the user's question. Identify key entities, keywords, and the type of information being sought. 
2.  **Formulate Search Queries (for Knowledge Base):** Based on your analysis, generate a list of precise search queries that you would use to retrieve relevant information from a knowledge base or external tools. 
3.  **Simulate Information Retrieval (Self-Correction/Reasoning):** For each search query, mentally consider what kind of information you expect to find. If you were to retrieve the content, what would be the most relevant snippets? Think about potential ambiguities or missing pieces. 
4.  **Synthesize Information:** Based on the simulated retrieval and your understanding of the user's original query, synthesize the gathered information into a coherent and complete answer. Ensure all aspects of the query are addressed. 
5.  **Review and Refine:** Before finalizing, critically evaluate your answer. Is it accurate? Is it comprehensive? Is it easy to understand? Is it concise? If not, identify what needs to be improved and how. 

**User Query:** "Explain the main differences between classical computers and quantum computers, and briefly describe a potential application of quantum computing." 

**Agent's Thought Process (Internal CoT Output - encouraged by the prompt):** 

**Thought 1 (Analyze Query):** 
The user wants two things: 
    1.  Main differences between classical and quantum computers. 
    2.  One potential application of quantum computing. Keywords: classical computers, quantum computers, differences, potential application. 

**Thought 2 (Formulate Search Queries):** 
    - "differences classical vs quantum computers" 
    - "how quantum computers work" 
    - "how classical computers work" 
    - "applications of quantum computing" 
    
**Thought 3 (Simulate Information Retrieval & Identify Key Concepts):** 
    - From "differences": Expect concepts like bits vs. qubits, superposition, entanglement, probabilistic nature, computational power for specific problems. 
    - From "how they work": Classical uses transistors/binary. Quantum uses quantum mechanics. 
    - From "applications": Expect drug discovery, material science, cryptography, optimization. 

**Thought 4 (Synthesize Information):** 
Classical computers use bits (0 or 1), processing information sequentially. Quantum computers use qubits, which can be 0, 1, or both simultaneously (superposition), and can be entangled, allowing for vastly different computational approaches. This enables them to solve certain problems (like factoring large numbers or simulating complex molecules) far more efficiently than classical computers. A potential application is in drug discovery, where quantum computers could simulate molecular interactions with unprecedented accuracy, accelerating the development of new medicines. 

**Thought 5 (Review and Refine):** 
    - Differences are covered (bits vs qubits, superposition, entanglement, processing). 
    - Application is given (drug discovery). - Is it brief? Yes. - Is it clear? Yes. - Looks good. 

**Agent's Final Answer (Output to User):** 
Classical computers process information using bits, which can be either a 0 or a 1 at any given time, performing operations sequentially. In contrast, quantum computers utilize qubits, which can exist in multiple states simultaneously through superposition and can be linked through entanglement, allowing for exponentially more complex computations. This fundamental difference enables quantum machines to tackle certain problems, such as simulating molecular structures or breaking complex encryption, that are intractable for even the most powerful classical supercomputers. A significant potential application of quantum computing lies in drug discovery, where its ability to precisely model molecular behavior could revolutionize the development of new pharmaceuticals.
```

**Reinforcement Learning with Verifiable Rewards (RLVR):** Стандартный CoT генерирует одну линию мыслей. Специализированные «модели рассуждения» выделяют переменное время на «размышление». Расширенное рассуждение позволяет self-correction и backtracking. Ключевая инновация — RLVR: обучение на задачах с известными ответами через trial-and-error.

**ReAct (Reasoning and Acting):** Парадигма, объединяющая CoT со способностью агента взаимодействовать с внешней средой через инструменты. Цикл: Мысль, Действие, Наблюдение, Мысль... Позволяет динамически адаптировать план.

![REACT: Рассуждение и действие](../assets/REACT_Reasoning_and_Act.png)

Рис. 1: Рассуждение и действие

**Chain of Debates (CoD):** Фреймворк Microsoft, где несколько моделей сотрудничают и спорят для решения задач.

**Graph of Debates (GoD):** Динамическая нелинейная сеть вместо линейной цепочки.

**MASS (Multi-Agent System Search):** Автоматизация дизайма многоагентных систем. Трёхэтапная оптимизация: оптимизация промптов отдельных агентов, оптимизация топологии workflow, глобальная оптимизация промптов.

```markdown
You are a highly critical and detail-oriented Self-Correction Agent. Your task is to review a previously generated piece of content against its original requirements and identify areas for improvement. Your goal is to refine the content to be more accurate, comprehensive, engaging, and aligned with the prompt. 

Here's the process you must follow for self-correction: 

1.  **Understand Original Requirements:** Review the initial prompt/requirements that led to the content's creation. What was the *original intent*? What were the key constraints or goals? 
2.  **Analyze Current Content:** Read the provided content carefully. 
3.  **Identify Discrepancies/Weaknesses:** Compare the current content against the original requirements. Look for:    
    * **Accuracy Issues:** Are there any factual errors or misleading statements?    
    * **Completeness Gaps:** Does it fully address all aspects of the original prompt? Is anything missing?    
    * **Clarity & Coherence:** Is the language clear, concise, and easy to understand? Does it flow logically?    
    * **Tone & Style:** Does it match the desired tone and style (e.g., professional, engaging, concise)?    
    * **Engagement:** Is it captivating? Does it hold the reader's attention?    
    * **Redundancy/Verbosity:** Can any parts be condensed or removed without losing meaning? 
4.  **Propose Specific Improvements:** For each identified weakness, suggest concrete and actionable changes. Do not just state the problem; propose a solution. 
5.  **Generate Revised Content:** Based on your proposed improvements, rewrite the original content to incorporate all the necessary changes. Ensure the revised content is polished and ready for final use. 

**Original Prompt/Requirements:** "Write a short, engaging social media post (max 150 characters) announcing a new eco-friendly product line: 'GreenTech Gadgets'." 

**Initial Draft (Content to be corrected):** "We have new products. They are green and techy. Buy GreenTech Gadgets now!" 

**Self-Correction Agent's Thought Process (Internal CoT Output - encouraged by the prompt):** 

**Thought 1 (Understand Original Requirements):** 
    - Short, engaging social media post. 
    - Max 150 characters. 
    - Announce "GreenTech Gadgets" - a new eco-friendly product line. 

**Thought 2 (Analyze Current Content):** 
"We have new products. They are green and techy. Buy GreenTech Gadgets now!" (64 characters) 

**Thought 3 (Identify Discrepancies/Weaknesses):** - **Engaging:** Not very engaging. Sounds generic. - **Eco-friendly:** Implied by "green," but not explicitly highlighted as a benefit. - **Call to Action:** "Buy now!" is okay, but could be more compelling. - **Conciseness:** It's short, but could use stronger verbs/phrasing. - **Impact:** Lacks excitement for a new product launch. 

**Thought 4 (Propose Specific Improvements):** - Make it more dynamic and exciting. - Emphasize "eco-friendly" and "innovation." - Use stronger verbs and more active voice. - Add a clear, engaging call to action. - Ensure it stays under 150 characters. **Thought 5 (Generate Revised Content):** Original: "We have new products. They are green and techy. Buy GreenTech Gadgets now!" Revised idea 1: "🌱 Discover GreenTech Gadgets! Our new eco-friendly line blends innovation with sustainability. Go green, go smart! Shop now! #EcoFriendly #GreenTech" (148 characters) This revision directly addresses eco-friendliness, uses emojis for engagement, adds a stronger call to action, and includes relevant hashtags while staying within the character limit. **Self-Correction Agent's Final Revised Content (Output to User):** 🌱 Discover GreenTech Gadgets! Our new eco-friendly line blends innovation with sustainability. Go green, go smart! Shop now! #EcoFriendly #GreenTech
```

Этот код показывает пример DeepSearch с LangGraph.

## Закон масштабирования inference

**Scaling Inference Law:** Производительность LLM predictably улучшается с ростом ресурсов во время inference. Меньшая модель с бóльшим «бюджетом размышления» может превзойти большую. Баланс: размер модели, латентность, стоимость.

## Практический пример кода

**Deep Research:** AI-агентные инструменты для автономного исследования (Perplexity, Gemini, ChatGPT). Стандартный поиск даёт ссылки; Deep Research — развёрнутый отчёт. AI работает агентно: начальное исследование → рассуждение → follow-up → синтез.

```python
from google.adk.tools import agent_tool
from google.adk.agents import Agent
from google.adk.tools import google_search
from google.adk.code_executors import BuiltInCodeExecutor


search_agent = Agent(
    model="gemini-2.0-flash",
    name="SearchAgent",
    instruction="""
    You're a specialist in Google Search
    """,
    tools=[google_search],
)

coding_agent = Agent(
    model="gemini-2.0-flash",
    name="CodeAgent",
    instruction="""
    You're a specialist in Code Execution
    """,
    code_executor=BuiltInCodeExecutor(),
)

root_agent = Agent(
    name="RootAgent",
    model="gemini-2.0-flash",
    description="Root Agent",
    tools=[
        agent_tool.AgentTool(agent=search_agent),
        agent_tool.AgentTool(agent=coding_agent),
    ],
)
```

## So, what do agents think?

Агентный процесс рассуждения — структурированный подход:

1. **Мысль:** Текстовая мысль, разбивающая задачу.
2. **Действие:** Выбор из набора (поиск, извлечение, ответ).
3. **Наблюдение:** Обратная связь из среды.

Цикл повторяется. Частота мыслей настраивается.

## Краткий обзор

**Что:** Сложные задачи требуют структурированного подхода с декомпозицией и инструментами.

**Почему:** Набор техник: CoT, ToT, самокоррекция, ReAct. Фреймворки CoD создают команды агентов.

**Когда использования:** Когда задача сложна для однопроходного ответа. Когда нужна прозрачность рассуждения.

**Визуальное резюме:**

![Паттерн рассуждения](../assets/Reasoning_Design_Pattern.png)

Рис. 2: Паттерн «Рассуждение»

## Ключевые выводы

* Явное рассуждение — прозрачные многошаговые планы.
* ReAct — основной операционный цикл агента.
* Scaling Inference Law: «время размышления» влияет на производительность.
* CoT — внутренний монолог для планирования.
* ToT и самокоррекция — оценка стратегий и backtrack.
* CoD — переход к многоагентным системам.
* Deep Research — автономные исследования.
* MASS — автоматизация дизайна многоагентных систем.

## Заключения

Современный ИИ эволюционирует из пассивных инструментов в автономных агентов. Автономное поведение: внутренний монолог (CoT), самокоррекция, ReAct. Scaling Inference Law — ресурсы. Многоагентные системы (CoD) — коллективный интеллект. Цель — надёжные прозрачные автономные агенты.

## Ссылки

1. Wei et al. (2022). Chain-of-Thought Prompting: [https://arxiv.org/abs/2201.11903](https://arxiv.org/abs/2201.11903)
2. Yao et al. (2023). Tree of Thoughts: [https://arxiv.org/pdf/2305.10601](https://arxiv.org/pdf/2305.10601)
3. Gao et al. (2023). Program-Aided Language Models
4. Yao et al. (2023). ReAct: [https://arxiv.org/abs/2210.03629](https://arxiv.org/abs/2210.03629)
5. Inference Scaling Laws (2024)
6. Multi-Agent Design: [https://arxiv.org/abs/2502.02533](https://arxiv.org/abs/2502.02533)
