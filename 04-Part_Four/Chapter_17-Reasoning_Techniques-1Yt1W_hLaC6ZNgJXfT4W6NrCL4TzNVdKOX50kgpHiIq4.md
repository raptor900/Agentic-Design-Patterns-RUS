# Глава 17: Техники рассуждения

В этой главе рассматриваются продвинутые методологии рассуждения для интеллектуальных агентов с акцентом на многошаговые логические выводы и решение задач. Эти техники выходят за рамки простых последовательных операций, делая внутреннее рассуждение агента явным. Это позволяет агентам разбивать задачи на части, рассматривать промежуточные шаги и приходить к более надёжным и точным выводам. Основным принципом среди этих продвинутых методов является выделение увеличенных вычислительных ресурсов во время inference. Это означает предоставление агенту или underlying LLM больше времени обработки или шагов для обработки запроса и генерации ответа. Вместо быстрого однопроходного выполнения агент может заниматься итеративной доработкой, исследовать несколько путей решения или использовать внешние инструменты. Расширенное время обработки во время inference часто значительно повышает точность, согласованность и надёжность, особенно для сложных задач, требующих более глубокого анализа и обдумывания.

## Практические применения и сценарии использования

Практические применения включают:

* **Сложные вопросы (Complex Question Answering):** Содействие в разрешении multi-hop запросов, которые требуют интеграции данных из различных источников и выполнения логических выводов, с потенциальным рассмотрением нескольких путей рассуждения и с выгодой от расширенного времени inference для синтеза информации.
* **Решение математических задач (Mathematical Problem Solving):** Позволяет разделять математические задачи на более мелкие решаемые компоненты, демонстрирует пошаговый процесс и использует выполнение кода для точных вычислений, где длительный inference позволяет более сложную генерацию кода и валидацию.
* **Отладка и генерация кода (Code Debugging and Generation):** Поддерживает объяснение агентом своего обоснования для генерации или исправления кода, последовательное выявление потенциальных проблем и итеративную доработку кода на основе результатов тестирования (Self-Correction), используя расширенное время inference для thorough debugging cycles.
* **Стратегическое планирование (Strategic Planning):** Содействие в разработке комплексных планов через рассуждение по various вариантам, последствиям и предусловиям, и корректировка планов на основе обратной связи в реальном времени (ReAct), где расширенное обдумывание может привести к более эффективным и надёжным планам.
* **Медицинская диагностика (Medical Diagnosis):** Помощь агенту в систематической оценке симптомов, результатов анализов и историй болезни для постановки диагноза, артикулирование его рассуждений на каждом этапе и потенциальное использование внешних инструментов для извлечения данных (ReAct). Увеличенное время inference позволяет проводить более полную дифференциальную диагностику.
* **Юридический анализ (Legal Analysis):** Поддержка анализа юридических документов и прецедентов для формулирования аргументов или предоставления рекомендаций, детализация логических шагов и обеспечение логической согласованности через самокоррекцию. Увеличенное время inference позволяет более глубокое юридическое исследование и построение аргументации.

## Техники рассуждения

Для начала рассмотрим ключевые техники рассуждения, используемые для улучшения способностей AI моделей к решению задач.

**Chain-of-Thought (CoT)** prompting значительно улучшает сложные способности рассуждения LLM, имитируя пошаговый мыслительный процесс (см. Рис. 1). Вместо предоставления прямого ответа CoT промпты guiding модель генерировать последовательность промежуточных шагов рассуждения. Эта явная декомпозиция позволяет LLM tackling сложные задачи, разбивая их на более мелкие управляемые подзадачи. Эта техника markedly improves производительность модели на задачах, requiring многошаговое рассуждение, such как арифметика, common sense reasoning и символьная манипуляция. Основным преимуществом CoT является способность превращать сложную одношаговую задачу в серию более простых шагов, thereby повышая прозрачность процесса рассуждения LLM. Этот подход не только повышает точность, но и provides ценные insights в процесс принятия решений модели, aiding в отладке и понимании. CoT может быть implemented с использованием various стратегий, включая предоставление few-shot примеров, которые демонстрируют пошаговое рассуждение, или simply instructing модель «думать шаг за шагом». Его эффективность stems из способности guiding внутреннюю обработку модели toward более deliberate и логическое progression. В результате Chain-of-Thought стал cornerstone техникой для enabling продвинутых возможностей рассуждения в современных LLM. Эта улучшенная прозрачность и декомпозиция сложных задач на управляемые подзадачи particularly важна для автономных агентов, since позволяет им выполнять более надёжные и auditable действия в сложных средах.

![COT: Chain of Thought](../assets/COT_Chain_of_Thought.png)

Рис. 1: CoT промпт alongside детальный пошаговый ответ, сгенерированный агентом.

Рассмотрим пример. Он начинается с набора инструкций, которые telling ИИ как думать, defining его persona и чёткий пятишаговый процесс для following. Это промпт, initiating структурированное мышление.

Далее пример shows CoT процесс в действии. Секция, labeled «Процесс мышления Агента» — это внутренний монолог, where модель выполняет instructed шаги. Это буквальная «цепочка мыслей». Finally, «Финальный ответ Агента» — это polished comprehensive выход, сгенерированный в результате этого careful пошагового процесса рассуждения.

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


**Tree-of-Thought (ToT)** — это техника рассуждения, которая развивает Chain-of-Thought (CoT). Она позволяет large language models исследовать_multiple пути рассуждения, ветвясь на различные промежуточные шаги и формируя древовидную структуру (см. Рис. 2). Этот подход поддерживает сложное решение задач, обеспечивая возврат назад, самокоррекцию и исследование альтернативных решений. Поддержание дерева возможностей позволяет модели оценивать различные траектории рассуждения перед финализацией ответа. Этот итеративный процесс усиливает способность модели справляться со сложными задачами, требующими стратегического планирования и принятия решений.

![TOT: Дерево мыслей](../assets/TOT_Tree_of_Thought.png)

Рис. 2: Пример дерева мыслей (Tree of Thoughts)

**Самокоррекция (Self-Correction)**, также известная как self-refinement, является важным аспектом процесса рассуждения агента, особенно в рамках Chain-of-Thought prompting. Она включает внутреннюю оценку агентом сгенерированного контента и промежуточных мыслительных процессов. Эта критическая проверка позволяет агенту выявлять неоднозначности, пробелы в информации или неточности в понимании или решениях. Этот итеративный цикл проверки и доработки позволяет агенту корректировать свой подход, улучшать качество ответа и обеспечивать точность и полноту перед выдачей финального результата. Внутренняя критика усиливает способность агента producing надёжные и высококачественные результаты, как демонстрируется в примерах в посвящённой этому Главе 4.

Этот пример демонстрирует систематический процесс самокоррекции, критически важный для улучшения AI-генерируемого контента. Он включает итеративный цикл составления, проверки по исходным требованиям и реализации конкретных улучшений. Иллюстрация начинается с описания функции ИИ как «Агента самокоррекции» с определённым пятишаговым аналитическим и ревизионным workflow. Затем представлен неудовлетворительный «Первоначальный черновик» поста для социальных сетей. «Процесс мышления Агента самокоррекции» составляет ядро демонстрации. Здесь Агент критически оценивает черновик согласно своим инструкциям, выявляя слабые стороны, такие как низкая вовлечённость и размытый призыв к действию. Затем он предлагает конкретные улучшения, включая использование более выразительных глаголов и эмодзи. Процесс завершается «Финальным переработанным контентом» — отполированным и значительно улучшенным вариантом, который интегрирует самовыявленные корректировки.

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


По своей сути эта техника интегрирует меру контроля качества непосредственно в генерацию контента Агентом, producing более отточенные, точные и превосходные результаты, которые более эффективно удовлетворяют сложные потребности пользователей.

**Program-Aided Language Models (PALMs)** интегрируют LLM с возможностями символьного рассуждения. Эта интеграция позволяет LLM генерировать и выполнять код, например на Python, в рамках процесса решения задач. PALMs переносят сложные вычисления, логические операции и манипуляции с данными в детерминированную программную среду. Этот подход использует преимущества традиционного программирования для задач, где LLM могут проявлять ограничения в точности или согласованности. При столкновении с символьными задачами модель может producing код, выполнять его и преобразовывать результаты в естественный язык. Эта гибридная методология сочетает способности LLM к пониманию и генерации с точными вычислениями, позволяя модели решать более широкий спектр сложных задач с потенциально повышенной надёжностью и точностью. Это важно для агентов, поскольку позволяет им выполнять более точные и надёжные действия, используя точные вычисления наряду с пониманием и генерацией. Примером является использование внешних инструментов в Google's ADK для генерации кода.

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


**Reinforcement Learning with Verifiable Rewards (RLVR):** Несмотря на эффективность, стандартный Chain-of-Thought (CoT) prompting, используемый многими LLM, является somewhat базовым подходом к рассуждению. Он генерирует одну предопределённую линию мыслей, не адаптируясь к сложности задачи. Для преодоления этих ограничений был разработан новый класс специализированных «моделей рассуждения». Эти модели работают иначе, выделяя переменное количество «думающего» времени перед предоставлением ответа. Этот «думающий» процесс producing более обширный и динамичный Chain-of-Thought, который может достигать тысяч токенов в длину. Расширенное рассуждение позволяет более复杂ые поведения, такие как самокоррекция и возврат назад (backtracking), при этом модель уделяет больше усилий более сложным задачам. Ключевой инновацией, обеспечивающей эти модели, является стратегия обучения под названием Reinforcement Learning from Verifiable Rewards (RLVR). Обучая модель на задачах с известными правильными ответами (такими как математика или код), она учится через trial-and-error генерировать эффективное развёрнутое рассуждение. Это позволяет модели развивать свои способности к решению задач без прямого человеческого надзора. В конечном счёте эти модели рассуждения не просто producing ответ; они генерируют «траекторию рассуждения», демонстрирующую продвинутые навыки, такие как планирование, мониторинг и оценка. Эта улучшенная способность рассуждать и стратегизировать является фундаментальной для развития автономных AI агентов, которые могут разбивать и решать сложные задачи с минимальным вмешательством человека.

**ReAct** (Reasoning and Acting, см. Рис. 3, где KB обозначает Knowledge Base) — это парадигма, объединяющая Chain-of-Thought (CoT) prompting со способностью агента взаимодействовать с внешней средой через инструменты. В отличие от генеративных моделей, которые producing финальный ответ, ReAct-агент рассуждает о том, какие действия предпринять. Фаза рассуждения включает внутренний процесс планирования, аналогичный CoT, где агент определяет свои следующие шаги, рассматривает доступные инструменты и anticipating исходы. После этого агент действует, выполняя вызов инструмента или функции, например запрос к базе данных, выполнение вычисления или взаимодействие с API.

![REACT: Рассуждение и действие](../assets/REACT_Reasoning_and_Act.png)

Рис. 3: Рассуждение и действие (Reasoning and Act)

ReAct работает в чередующемся режиме: агент выполняет действие, наблюдает результат и incorporates это наблюдение в последующее рассуждение. Этот итеративный цикл «Мысль, Действие, Наблюдение, Мысль...» позволяет агенту динамически адаптировать свой план, исправлять ошибки и достигать целей, требующих_multiple взаимодействий со средой. Это обеспечивает более надёжный и гибкий подход к решению задач по сравнению с линейным CoT, поскольку агент реагирует на обратную связь в реальном времени. Сочетая понимание и генерацию языковой модели со способностью использовать инструменты, ReAct позволяет агентам выполнять сложные задачи, требующие как рассуждения, так и практического исполнения. Этот подход критически важен для агентов, поскольку позволяет им не только рассуждать, но и practically выполнять шаги и взаимодействовать с динамическими средами.

**CoD** (Chain of Debates) — это формальный AI фреймворк, предложенный Microsoft, в котором несколько разнообразных моделей сотрудничают и спорят для решения задачи, выходя за рамки «цепочки мыслей» одного ИИ. Эта система работает как совещание AI совета, где различные модели presenting начальные идеи, критикуют рассуждения друг друга и обмениваются контраргументами. Основная цель — повысить точность, уменьшить предвзятость и улучшить общее качество финального ответа, используя коллективный интеллект. Функционируя как AI-версия peer review, этот метод создаёт прозрачную и достоверную запись процесса рассуждения. В конечном счёте это представляет переход от providing ответа solitary Агента к совместной работе команды Агентов для поиска более надёжного и валидированного решения.

**GoD** (Graph of Debates) — продвинутый Agentic фреймворк, который reimagine дискуссию как динамическую нелинейную сеть, а не простую цепочку. В этой модели аргументы являются отдельными узлами, соединёнными рёбрами, которые означают отношения типа «поддерживает» или «опровергает», отражая многопоточную природу реальной дебаты. Эта структура позволяет новым линиям исследования динамически ответвляться, развиваться независимо и даже сливаться со временем. Заключение достигается не в конце последовательности, а путём выявления самого надёжного и well-supported кластера аргументов внутри всей графа. В этом контексте «well-supported» относится к знаниям, которые являются прочно установленными и verifiable. Это может включать информацию, считающуюся ground truth, то есть inherently correct и широко принятую как факт. Кроме того, это encompass фактические доказательства, полученные через search grounding, где информация валидируется against внешними источниками и real-world данными. Наконец, это также относится к консенсусу, достигнутому несколькими моделями во время дебатов, указывая на высокую степень согласия и уверенности в presented информации. Этот всеобъемлющий подход обеспечивает более надёжную и достоверную основу для обсуждаемой информации. Подход provides более holistic и реалистичную модель для сложного совместного AI рассуждения.

**MASS (optional advanced topic):** Углублённый анализ дизайна многоагентных систем reveals, что их эффективность критически зависит как от качества промптов, используемых для программирования отдельных агентов, так и от топологии, которая определяет их взаимодействия. Сложность проектирования этих систем значительна, поскольку она включает огромное и сложное пространство поиска. Для решения этой проблемы был разработан новый фреймворк под названием Multi-Agent System Search (MASS) для автоматизации и оптимизации дизайна MAS.

MASS employs многоэтапную стратегию оптимизации, которая систематически навигирует сложное пространство дизайна, чередуя оптимизацию промптов и топологии (см. Рис. 4).

**1. Оптимизация промптов на уровне блоков (Block-Level Prompt Optimization):** Процесс начинается с локальной оптимизации промптов для отдельных типов агентов, или «блоков», чтобы обеспечить эффективное выполнение каждым компонентом своей роли перед интеграцией в более крупную систему. Этот начальный шаг критически важен, поскольку он ensures, что последующая оптимизация топологии строится на хорошо работающих агентах, а не suffers от compounding воздействия плохо настроенных. Например, при оптимизации для датасета HotpotQA промпт для агента «Debator» творчески формулируется для instruct его acting как «эксперт-фактчекер для крупного издания». Его оптимизированная задача — тщательно проверять proposed ответы других агентов, cross-reference их с предоставленными контекстными passages и выявлять любые несоответствия или неподтверждённые утверждения. Этот специализированный промпт ролевой игры, discovered during оптимизации на уровне блоков, aims сделать агента-дебатёра highly effective в синтезе информации перед placement в более крупный workflow.

**2. Оптимизация топологии workflow (Workflow Topology Optimization):** После локальной оптимизации MASS optimizes топологию workflow, выбирая и arranging различные взаимодействия агентов из настраиваемого пространства дизайна. Для эффективного поиска MASS employs метод influence-weighted. Этот метод вычисляет «инкрементальное влияние» каждой топологии, измеряя её прирост производительности relative к baseline агенту, и использует эти оценки для направления поиска к более перспективным комбинациям. Например, при оптимизации для coding задачи MBPP поиск топологии discovers, что конкретный гибридный workflow является наиболее effective. Найденная топология — не простая структура, а комбинация итеративного процесса улучшения с использованием внешних инструментов. Конкретно, она состоит из одного агента-predictor, который engages в несколько раундов рефлексии, с его кодом, валидируемым одним агентом-executor, который запускает код against тестовых cases. Эта обнаруженная workflow показывает, что для coding структура, combining итеративную самокоррекцию с внешней валидацией, превосходит более простые MAS designs.

![MASS: Multi-Agent System Search](../assets/MASS_Multi_Agent_System_Search.png)

Рис. 4: (Courtesy of the Authors): Фреймворк Multi-Agent System Search (MASS) представляет собой трёхэтапный процесс оптимизации, который навигирует пространство поиска, encompassing оптимизируемые промпты (инструкции и демонстрации) и конфигурируемые строительные блоки агентов (Aggregate, Reflect, Debate, Summarize и Tool-use). Первый этап, оптимизация промптов на уровне блоков, independently оптимизирует промпты для каждого модуля агента. Второй этап, оптимизация топологии workflow, samples валидные конфигурации системы из influence-weighted пространства дизайна, integrating оптимизированные промпты. Финальный этап, оптимизация промптов на уровне workflow, включает второй раунд оптимизации промптов для всей многоагентной системы после identification оптимальной workflow со второго этапа.

**3. Оптимизация промптов на уровне workflow (Workflow-Level Prompt Optimization):** Финальный этап включает глобальную оптимизацию промптов всей системы. После identification наилучшей топологии промпты fine-tune как единое интегрированное entity, чтобы ensure они tailored для orchestration и что взаимозависимости агентов оптимизированы. Например, после finding наилучшей топологии для датасета DROP финальный этап оптимизации refines промпт агента «Predictor». Финальный оптимизированный промпт highly detailed, beginning с providing агенту summary самого датасета, noting его focus на «extractive question answering» и «numerical information». Затем он includes few-shot examples корректного поведения question-answering и framing core instruction как high-stakes scenario: «Вы — highly specialized AI tasked с extracting критической числовой информации для urgent news report. Live broadcast relies на вашей accuracy and speed». Этот multi-faceted промпт, combining мета-знание, examples и ролевую игру, tuned specifically для финальной workflow для maximization точности.

Ключевые выводы и принципы: Эксперименты demonstrating, что MAS, оптимизированные с помощью MASS, significantly превосходят существующие manually designed системы и другие automated design методы across ряд задач. Ключевые принципы дизайна для effective MAS, derived из этого исследования, являются троякими:

* Оптимизируйте отдельных агентов с помощью высококачественных промптов перед их компоновкой.
* Конструируйте MAS, компонуя influential топологии, rather чем exploring неограниченное пространство поиска.
* Моделируйте и оптимизируйте взаимозависимости между агентами через финальную совместную оптимизацию на уровне workflow.

Продолжая обсуждение ключевых техник рассуждения, рассмотрим сначала базовый принцип производительности: Scaling Inference Law для LLM. Этот закон гласит, что производительность модели predictably улучшается по мере увеличения вычислительных ресурсов, выделяемых ей. Мы можем видеть этот принцип в действии в сложных системах, таких как Deep Research, где AI агент leveraging эти ресурсы для автономного исследования темы, разбивая её на подвопросы, используя веб-поиск как инструмент и синтезируя результаты.

**Deep Research.** Термин «Deep Research» описывает категорию AI Agentic инструментов, designed acting как неутомимые методичные исследовательские assistants. Крупные платформы в этой области включают Perplexity AI, исследовательские возможности Google's Gemini и продвинутые функции OpenAI в ChatGPT (см. Рис. 5).

![Google Deep Research для сбора информации](../assets/Google_Deep_Research_for_Information_Gathering.png)

Рис. 5: Google Deep Research для сбора информации

Фундаментальное изменение, introduced этими инструментами, — это изменение в самом процессе поиска. Стандартный поиск provides немедленные ссылки, оставляя работу по синтезу вам. Deep Research operates на другой модели. Здесь вы assigns AI сложный запрос и предоставляет ему «бюджет времени» — обычно несколько минут. Взамен за это терпение вы получаете детальный отчёт.

Во время этого времени AI работает от вашего имени агентным способом. Он автономно выполняет серию сложных шагов, которые были бы incredibly time-consuming для человека:

1. Начальное исследование: Выполняет_multiple целенаправленных поисков на основе вашего первоначального промпта.
2. Рассуждение и уточнение: Читает и анализирует первую волну результатов, синтезирует находки и критически выявляет пробелы, противоречия или области, требующие более детального рассмотрения.
3. Последующий запрос: На основе внутреннего рассуждения проводит новые более нюансированные поиски для заполнения пробелов и углубления понимания.
4. Финальный синтез: После нескольких раундов итеративного поиска и рассуждения компилирует всю валидированную информацию в единый связный и структурированный summary.

Этот систематический подход обеспечивает comprehensive и well-reasoned ответ, significantly повышая эффективность и глубину сбора информации, thereby facilitating более agentic принятие решений.

## Закон масштабирования inference

Этот критический принцип определяет зависимость между производительностью LLM и вычислительными ресурсами, выделяемыми during её операционной фазы, known как inference. Inference Scaling Law отличается от более знакомых scaling laws для training, которые фокусируются на том, как качество модели improves с увеличением объёма данных и вычислительной мощности during создания модели. Вместо этого этот закон specifically examines динамические trade-offs, occurring когда LLM активно generating выход или ответ.

Краеугольным камнем этого закона является revelation, что превосходные результаты могут frequently достигаться с помощью comparatively меньшей LLM путём увеличения вычислительных инвестиций во время inference. Это не necessarily означает использование более мощного GPU, а rather применения более сложных или resource-intensive стратегий inference. Примером такой стратегии является instruct модели generating_multiple potential ответов — возможно, through techniques как diverse beam search или self-consistency methods — и затем employing механизм selection для identification самого оптимального выхода. Этот итеративный refinement или multiple-candidate процесс генерации demands больше вычислительных циклов, но может significantly повысить качество финального ответа.

Этот принцип provides crucial framework для informed и economically sound принятия решений при развертывании систем Agents. Он бросает вызов intuitive notion, что larger model всегда даст better performance. Закон posits, что smaller model, given более substantial «thinking budget» during inference, can occasionally surpass performance гораздо larger model, которая relies на simpler, less computationally intensive процессе генерации. «Thinking budget» здесь refers к дополнительным вычислительным шагам или сложным алгоритмам, applied during inference, allowing smaller model исследовать более широкий range possibilities или применять более rigorous внутренние checks перед settling на ответ.

Consequently, Scaling Inference Law становится fundamental для constructing efficient и cost-effective Agentic systems. Он provides methodology для meticulous балансировки several interconnected факторов:

* **Размер модели (Model Size):** Меньшие модели inherently менее требовательны с точки зрения памяти и хранения.
* **Латентность ответа (Response Latency):** While увеличенное inference-time вычисление может add к латентности, закон helps identify точку, в которой прирост производительности outweighs это увеличение, или как стратегически применять вычисления для avoid чрезмерных задержек.
* **Операционная стоимость (Operational Cost):** Развертывание и запуск larger models typically incurs более высокие ongoing operational costs из-за increased потребления энергии и требований к инфраструктуре. Закон demonstrates как оптимизировать производительность without unnecessarily наращивая эти costs.

Понимая и применяя Scaling Inference Law, разработчики и организации могут принимать стратегические решения, которые ведут к оптимальной производительности для конкретных agentic приложений, ensuring что вычислительные ресурсы allocated where они будут иметь наибольшее impact на качество и полезность LLM выхода. Это allows для более nuanced и economically viable подходов к развертыванию ИИ, moving beyond простую «bigger is better» парадигму.

## Практический пример кода

DeepSearch код, open-sourced Google, доступен через репозиторий gemini-fullstack-langgraph-quickstart (Рис. 6). Этот репозиторий provides шаблон для разработчиков для construction full-stack AI агентов, использующих Gemini 2.5 и фреймворк оркестрации LangGraph. Этот open-source стек facilitates экспериментирование с agent-based архитектурами и может быть integrated с local LLM такими как Gemma. Он использует Docker и modular project scaffolding для быстрого прототипирования. Следует отметить, что этот release serves как well-structured demonstration и не intended как production-ready backend.

![Пример DeepSearch с несколькими шагами рефлексии](../assets/Example_of_DeepSearch_with_multiple_Reflection_Steps.png)

Рис. 6: (Courtesy of authors) Пример DeepSearch с несколькими шагами рефлексии

Этот проект provides full-stack приложение featuring React frontend и LangGraph backend, designed для продвинутых исследований и conversational AI. LangGraph агент динамически generates поисковые запросы, используя модели Google Gemini, и integrates веб-исследование через Google Search API. Система employs reflective reasoning для identification пробелов в знаниях, итеративного уточнения поисков и синтеза ответов с цитированием. Frontend и backend поддерживают hot-reloading. Структура проекта includes отдельные директории frontend/ и backend/. Требования для настройки включают Node.js, npm, Python 3.8+ и Google Gemini API key. После настройки API key в backend's .env файле зависимости для backend (using pip install .) и frontend (npm install) могут быть установлены. Серверы разработки могут быть запущены concurrently с make dev или individually. Backend агент, defined в backend/src/agent/graph.py, generates начальные поисковые запросы, проводит веб-исследование, выполняет анализ пробелов в знаниях, итеративно уточняет запросы и синтезирует цитированный ответ, используя модель Gemini. Продакшн deployment involves backend сервер, delivering static frontend build, и requires Redis для streaming real-time выхода и Postgres database для управления данными. Docker image может быть собран и запущен с помощью docker-compose up, что также requires LangSmith API key для docker-compose.yml примера. Приложение utilizes React с Vite, Tailwind CSS, Shadcn UI, LangGraph и Google Gemini. Проект лицензирован under Apache License 2.0.

| ``# Create our Agent Graph builder = StateGraph(OverallState, config_schema=Configuration) # Define the nodes we will cycle between builder.add_node("generate_query", generate_query) builder.add_node("web_research", web_research) builder.add_node("reflection", reflection) builder.add_node("finalize_answer", finalize_answer) # Set the entrypoint as `generate_query` # This means that this node is the first one called builder.add_edge(START, "generate_query") # Add conditional edge to continue with search queries in a parallel branch builder.add_conditional_edges(    "generate_query", continue_to_web_research, ["web_research"] ) # Reflect on the web research builder.add_edge("web_research", "reflection") # Evaluate the research builder.add_conditional_edges(    "reflection", evaluate_research, ["web_research", "finalize_answer"] ) # Finalize the answer builder.add_edge("finalize_answer", END) graph = builder.compile(name="pro-search-agent")`` |
| :---- |

Рис. 4: Пример DeepSearch с LangGraph (код из backend/src/agent/graph.py)

## So, what do agents think?

В итоге, процесс мышления агента — это структурированный подход, combining рассуждение и действие для решения задач. Этот метод позволяет агенту явно планировать свои шаги, мониторить прогресс и взаимодействовать с внешними инструментами для сбора информации.

В основе «мышления» агента лежит powerful LLM. Эта LLM генерирует серию мыслей, которые guiding последующие действия агента. Процесс typically следует циклу мысль-действие-наблюдение:

1. **Мысль (Thought):** Агент сначала generates текстовую мысль, которая разбивает задачу, формулирует план или анализирует текущую ситуацию. Этот внутренний монолог делает процесс рассуждения агента прозрачным и управляемым.
2. **Действие (Action):** Based на мысли, агент выбирает действие из предопределённого набора дискретных опций. Например, в сценарии вопросов-ответов пространство действий может включать поиск в сети, извлечение информации с конкретной веб-страницы или предоставление финального ответа.
3. **Наблюдение (Observation):** Агент затем receives обратную связь из среды based на performed действии. Это могут быть результаты веб-поиска или содержимое веб-страницы.

Этот цикл repeats, с каждым наблюдением informing следующую мысль, until агент determines, что достиг финального решения, и выполняет действие «finish».

Эффективность этого подхода relies на продвинутых возможностях рассуждения и планирования underlying LLM. Для guiding агента ReAct фреймворк often employs few-shot learning, where LLM provides examples human-like траекторий решения задач. Эти examples демонстрируют, как effectively combine мысли и действия для решения similar задач.

Частота мыслей агента может быть adjusted в зависимости от задачи. Для knowledge-intensive задач рассуждения, such как fact-checking, мысли typically чередуются с каждым действием для ensure логический поток сбора информации и рассуждения. In contrast, для задач принятия решений, requiring many действий, such как навигация simulated среды, мысли могут использоваться more sparingly, allowing агенту decide, когда thinking необходимо.

## At a Glance

**Что:** Решение сложных задач often requires больше, чем одного прямого ответа, posing significant challenge для ИИ. Core problem enabling AI агентов tackling multi-step задачи, demanding логический вывод, декомпозицию и стратегическое планирование. Without structured подход, агенты могут fail handling intricacies, leading к inaccurate или incomplete conclusions. Эти продвинутые методологии рассуждения aim сделать внутренний «мыслительный» процесс агента явным, allowing ему systematically работать through challenges.

**Почему:** Стандартизированное решение — это набор техник рассуждения, providing structured framework для процесса решения задач агентом. Methodologies как Chain-of-Thought (CoT) и Tree-of-Thought (ToT) guiding LLMs разбивать задачи и исследовать_multiple пути решения. Self-Correction позволяет итеративно улучшать ответы, ensuring higher точность. Agentic фреймворки как ReAct integrates рассуждение с действием, enabling агентов взаимодействовать с внешними инструментами и средами для сбора информации и адаптации планов. Это combination явного рассуждения, исследования, уточнения и использования инструментов создаёт более надёжные, прозрачные и capable AI системы.

**Когда использовать:** Применяйте эти техники рассуждения, когда задача слишком сложна для однопроходного ответа и требует декомпозиции, многошаговой логики, взаимодействия с внешними источниками данных или инструментами, или стратегического планирования и адаптации. Они идеальны для задач, где показ «работы» или процесса рассуждения так же важно, как и финальный ответ.

**Визуальное резюме:**

![Паттерн рассуждения](../assets/Reasoning_Design_Pattern.png)

Рис. 7: Паттерн «Рассуждение»

## Ключевые выводы

* Делая свои рассуждения явными, агенты могут формулировать прозрачные многошаговые планы, что является базовой capability для автономных действий и доверия пользователей.
* Фреймворк ReAct provides агентам их основной операционный цикл, empowering их двигаться beyond merely рассуждения и взаимодействовать с внешними инструментами для динамических действий и адаптации внутри среды.
* Scaling Inference Law implies, что производительность агента — это не только размер underlying модели, но и allocated «thinking time», allowing для более deliberate и более качественных автономных действий.
* Chain-of-Thought (CoT) serves как внутренний монолог агента, providing структурированный способ формулирования плана путём разбиения сложной цели на последовательность управляемых действий.
* Tree-of-Thought и Self-Correction дают агентам crucial способность deliberation, allowing им оценивать_multiple стратегии, backtrack из ошибок и улучшать свои собственные планы перед исполнением.
* Collaborative фреймворки как Chain of Debates (CoD) signal переход от solitary агентов к многоагентным системам, where команды агентов могут рассуждать together для tackling более сложных задач и reducing индивидуальных biases.
* Приложения как Deep Research demonstrating, как эти техники culminate в агентах, которые могут выполнять сложные долгосрочные задачи, such как глубокое исследование, completely автономно от имени пользователя.
* Для build эффективных команд агентов фреймворки как MASS automate оптимизацию того, как отдельные агенты instructed и как они взаимодействуют, ensuring что вся многоагентная система performs оптимально.
* Integrating эти техники рассуждения, мы строим агентов, которые are не просто automated, а truly autonomous, capable быть trusted для планирования, действий и решения сложных задач without direct надзора.

## Заключения

Современный ИИ эволюционирует из пассивных инструментов в автономных агентов, capable tackling сложных целей through структурированное рассуждение. Это agentic поведение begins с внутреннего монолога, powered techniques как Chain-of-Thought (CoT), который allows агенту formulate coherent план перед действием. True autonomy requires deliberation, which agents achieve through Self-Correction и Tree-of-Thought (ToT), enabling их оценивать_multiple стратегии и independently улучшать свою собственную работу. Pivotal leap к fully agentic systems comes из ReAct фреймворка, который empowers агентов move beyond thinking и start acting, используя внешние инструменты. Это establishes core agentic цикл мысли, действия и наблюдения, allowing агенту динамически адаптировать стратегию based на обратной связи среды.

Емкость агента для глубокого deliberation fueled Scaling Inference Law, where больше вычислительного «thinking time» directly translates в более надёжные автономные действия. Next frontier — многоагентная система, where фреймворки как Chain of Debates (CoD) создают collaborative agent общества, которые рассуждают together для достижения общей цели. Это не theoretical; agentic приложения как Deep Research already demonstrating, как автономные агенты могут выполнять сложные многошаговые расследования от имени пользователя. Overarching цель — инжиниринг надёжных и прозрачных автономных агентов, которые могут быть trusted для independent управления и решения complex задач without direct надзора. Ultimately, combining явное рассуждение с power действовать, эти методологии completing трансформацию ИИ в truly agentic problem-solvers.

## Ссылки

1. Wei et al. (2022). Chain-of-Thought Prompting Elicits Reasoning in Large Language Models
2. Yao et al. (2023). Tree of Thoughts: Deliberate Problem Solving with Large Language Models
3. Gao et al. (2023). Program-Aided Language Models
4. Yao et al. (2023). ReAct: Synergizing Reasoning and Acting in Language Models
5. Inference Scaling Laws: An Empirical Analysis of Compute-Optimal Inference for LLM Problem-Solving, 2024
6. Multi-Agent Design: Optimizing Agents with Better Prompts and Topologies, https://arxiv.org/abs/2502.02533
