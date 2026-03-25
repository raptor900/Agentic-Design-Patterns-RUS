# Часто задаваемые вопросы: Паттерны проектирования агентных систем

## 1. Что такое "агентный паттерн проектирования"?

Агентный паттерн проектирования — это повторно используемое высокоуровневое решение распространённой проблемы, возникающей при создании интеллектуальных автономных систем (агентов). Эти паттерны предоставляют структурированный фреймворк для проектирования поведения агентов, аналогично тому, как паттерны проектирования ПО работают для традиционного программирования. Они помогают разработчикам создавать более надёжные, предсказуемые и эффективные AI-агенты.

## 2. Какова основная цель этого руководства?

Руководство aims предоставить practical, hands-on введение в проектирование и создание agentic систем. Оно выходит за рамки теоретических обсуждений, предлагая конкретные архитектурные blueprints, которые разработчики могут использовать для создания агентов, capable сложного, goal-oriented поведения надёжным способом.

## 3. Для кого предназначено это руководство?

Это руководство написано для AI-разработчиков, software engineers и system architects, которые строят приложения с large language models (LLM) и другими AI компонентами. Оно для тех, кто хочет перейти от простых prompt-response взаимодействий к созданию сложных автономных агентов.

## 4. Какие ключевые агентные паттерны обсуждаются?

Согласно оглавлению, руководство covers several ключевых паттернов, включая:

- **Reflection:** Способность агента критиковать свои собственные действия и выводы для улучшения производительности.
- **Planning:** Процесс декомпозиции сложной цели на smaller, manageable шаги или задачи.
- **Tool Use:** Паттерн использования агентом внешних инструментов (such as code интерпретаторы, поисковые системы или другие API) для получения информации или выполнения действий, которые он не может сделать сам.
- **Multi-Agent Collaboration:** Архитектура для совместной работы multiple специализированных агентов для решения проблемы, often involving "leader" или "orchestrator" агента.
- **Human-in-the-Loop:** Интеграция human oversight и intervention, allowing feedback, коррекцию и одобрение действий агента.

## 5. Почему "планирование" — важный паттерн?

Планирование crucial, because оно allowing агенту tackling сложные, multi-step задачи, которые не могут быть solved с single action. Создавая план, агент может maintaining coherent стратегию, tracking свой progress и handling ошибки или unexpected obstacles structured образом. Это preventing агента от getting "stuck" или deviating от конечной цели пользователя.


## 6. В чём разница между "инструментом" и "навыком" для агента?

Хотя термины often используются interchangeably, "tool" generally refers к external ресурсу, который агент может использовать (например, weather API, calculator). "Skill" — это более integrated capability, которую агент learned, often combining tool use с internal reasoning для performing конкретной функции (например, skill "booking flight" might включать использование calendar и airline API).

## 7. Как паттерн "Reflection" улучшает производительность агента?

Reflection acts как форма self-correction. После генерации ответа или completing задачи, агент может быть prompted к review своей работы, checking на ошибки, assessing качество по certain criteria, или considering alternative подходы. Этот iterative refinement process helping агенту producing более accurate, relevant и high-quality результаты.

## 8. Какова основная идея паттерна Reflection?

Паттерн Reflection giving агенту ability к step back и критиковать свою own работу. Вместо producing final output за один раз, агент генерирует draft и затем "reflects" на нём, identifying flaws, missing information или areas для improvement. Этот self-correction process ключевой к enhancing качеству и accuracy его ответов.

## 9. Почему простого "prompt chaining" недостаточно для high-quality output?

Простой prompt chaining (где output одного промпта becomes input для следующего) often too basic. Модель might просто rephrase свой previous output без genuine улучшения. True Reflection паттерн requires more structured critique, prompting агента анализировать свою работу по specific стандартам, checking на logical errors, или verifying facts.

## 10. Какие два основных типа reflection упоминаются в этой главе?

Глава обсуждает two primary форм reflection:

- **"Check your work" Reflection:** Это базовая форма, где агента simply просят review и исправить свой previous output. Это good starting point для catching простых ошибок.
- **"Internal Critic" Reflection:** Это более advanced форма, где отдельный, "critic" агент (или dedicated промпт) используется для evaluating output "worker" агента. Этот critic может быть given specific criteria для поиска, leading к более rigorous и targeted improvements.

## 11. Как reflection помогает в снижении "галлюцинаций"?

Путём prompting агента к review своей работы, especially путём comparing statements против known source или checking своих own reasoning steps, Reflection паттерн может significantly reduce likelihood hallucinations (измышления фактов). Агент forced быть более grounded в provided контексте и less likely генерировать unsupported информацию.


## 12. Может ли паттерн Reflection применяться более одного раза?

Да, reflection может быть iterative процесс. Агент может быть made к reflect на своей работе multiple раз, с each loop уточняя output further. Это particularly useful для сложных задач, где первая или вторая попытка may still содержать subtle ошибки или could быть substantially improved.

## 13. Что такое паттерн Planning в контексте AI-агентов?

Паттерн Planning involves enabling агента к разбиению сложной, высокоуровневой цели на последовательность smaller, actionable шагов. Вместо trying решить big problem за раз, агент сначала создаёт "plan" и затем выполняет каждый шаг в плане, что much more надёжный подход.

## 14. Почему планирование необходимо для сложных задач?

LLM могут struggling с задачами, которые требуют multiple steps или dependencies. Без плана, агент might lose track общей objectives, miss crucial steps, или fail обработать output одного шага как input для следующего. План providing clear roadmap, ensuring что all requirements оригинального запроса met в logical порядке.

## 15. Какой распространённый способ реализации паттерна Planning?

Распространённая реализация — заставить агента сначала сгенерировать список шагов в structured format (such as JSON array или numbered list). Система затем может iterate through этот list, выполняя каждый шаг по одному и feeding результат back к агенту для informing следующего действия.

## 16. Как агент обрабатывает ошибки или изменения во время выполнения?

Robust planning паттерн allows для dynamic adjustments. Если шаг fails или situation changes, агент может быть prompted к "re-plan" от текущего состояния. Он может проанализировать ошибку, modify remaining шаги, или even добавить новые для overcoming препятствия.

## 17. Видит ли пользователь план?

Это design choice. В many cases, showing план пользователю first для approval — great practice. Это aligns с "Human-in-the-Loop" паттерном, giving пользователю transparency и control над proposed действиями агента before их выполнения.

## 18. Что включает паттерн "Tool Use"?

Паттерн Tool Use allowing агенту расширить свои capabilities через взаимодействие с external software или API. Since knowledge LLM static и он не может perform real-world actions сам, tools giving ему access к live информации (например, Google Search), proprietary данным (например, база данных компании) или ability к perform действиям (например, отправить email, забронировать встречу).

## 19. Как агент решает, какой инструмент использовать?

Агент typically given список available tools вместе с описаниями what каждый tool делает и what parameters он требует. При встрече с запросом, который он не может handle с internal knowledge, reasoning ability агента allowing ему выбрать most appropriate tool из списка для accomplishing задачи.

## 20. Что такое фреймворк "ReAct" (Reason and Act), упомянутый в этом контексте?

ReAct — это popular фреймворк, integrating reasoning и acting. Агент follows loop из **Thought** (рассуждение about what нужно сделать), **Action** (решение какой tool использовать и с какими inputs) и **Observation** (видение результата от tool). Этот loop продолжается until он gathered достаточно информации для fulfillment запроса пользователя.

## 21. Какие challenges в реализации использования инструментов?

Ключевые challenges включают:

- **Обработка ошибок:** Tools могут fail, возвращать unexpected данные или time out. Агент needs быть capable распознать эти ошибки и решить whether попробовать снова, использовать другой tool, или попросить пользователя о помощи.
- **Безопасность:** Giving агенту access к tools, especially тем, которые performing actions, has security implications. Критически важно иметь safeguards, permissions и often human approval для sensitive операций.
- **Промптинг:** Агент must быть prompted effectively для генерации correctly formatted tool вызовов (например, right function name и parameters).


## 22. Что такое паттерн Human-in-the-Loop (HITL)?

HITL — это паттерн, integrating human oversight и interaction в workflow агента. Вместо being fully автономным, агент pauses на critical junctures для asking human feedback, одобрения, уточнения или направления.

## 23. Почему HITL важен для агентных систем?

Это crucial по several reasons:

- **Безопасность и контроль:** Для high-stakes задач (например, financial transactions, отправка official communications), HITL ensuring human verifies proposed действия агента before их выполнения.
- **Улучшение качества:** Humans могут providing corrections или nuanced feedback, который агент может использовать для улучшения своей производительности, especially в subjective или ambiguous задачах.
- **Построение доверия:** Пользователи more likely trust и adopt AI систему, которую они могут guide и supervise.

## 24. В каких точках workflow нужно включать human?

Common точки для human intervention включают:

- **Одобрение плана:** Before executing multi-step плана.
- **Подтверждение использования инструментов:** Before using tool, имеющего real-world последствия или costing деньги.
- **Разрешение неоднозначности:** Когда агент unsure как proceed или needs more информацию от пользователя.
- **Финальный review output:** Before delivering final результат end-user или системе.

## 25. Разве постоянное human intervention не неэффективно?

Может быть, поэтому ключ — найти right баланс. HITL должен быть implemented на critical checkpoints, не для каждого single действия. Цель — построить collaborative partnership между human и агентом, где агент handles bulk работы и human providing стратегическое руководство.

## 26. Что такое паттерн Multi-Agent Collaboration?

Этот паттерн involves creating систему, composed из multiple специализированных агентов, работающих together для достижения общей цели. Вместо одного "generalist" агента, trying делать everything, вы создаёте team из "specialist" агентов, each с specific ролью или expertise.

## 27. Какие преимущества multi-agent системы?

- **Модульность и специализация:** Каждый агент может быть fine-tuned и prompted для своей конкретной задачи (например, "researcher" агент, "writer" агент, "code" агент), leading к higher quality результатам.
- **Сниженная сложность:** Декомпозиция сложного workflow на специализированные роли making общую систему easier design, debug и maintain.
- **Симулированный brainstorming:** Различные агенты могут offering different perspectives на проблему, leading к более creative и robust решениям, similar к тому как работает human team.

## 28. Какова распространённая архитектура для multi-agent систем?

Распространённая архитектура involves **Orchestrator Агента** (sometimes called "manager" или "conductor"). Orchestrator понимает общую цель, разбивает её и делегирует подзадачи appropriate specialist агентам. Затем он собирает результаты от specialists и synthesizes их в final output.

## 29. Как агенты общаются друг с другом?

Коммуникация often managed orchestrator'ом. Например, orchestrator might передать output "researcher" агента "writer" агенту как контекст. Shared "scratchpad" или message bus, где агенты могут posting свои findings, — другой common метод коммуникации.

## 30. Почему оценка агента сложнее, чем оценка традиционной программы?

Традиционный софт has deterministic outputs (same input всегда produces same output). Агенты, especially those использующие LLM, являются non-deterministic, и их производительность может быть subjective. Оценка их requires assessing *quality* и *relevance* их output, не просто whether оно technically "correct".

## 31. Какие распространённые методы оценки производительности агентов?

Руководство предлагает several методы:

- **Оценка на основе результата:** Did агент successfully achieve final goal? Например, если задача была "book flight", был ли flight actually booked correctly? Это most important мера.
- **Оценка на основе процесса:** Был ли *process* агента efficient и logical? Did он использовать right tools? Did он follow sensible plan? Это helps debug почему агент might быть failing.
- **Human оценка:** Having humans score производительность агента по шкале (например, 1-5) based on criteria such as helpfulness, accuracy и coherence. Это crucial для user-facing приложений.

## 32. Что такое "траектория агента" (agent trajectory)?

Траектория агента — это complete log шагов агента while performing задачи. Она includes все его мысли, действия (tool вызовы) и observations. Анализ этих trajectories — ключевая часть debugging и понимания поведения агента.

## 33. Как создать надёжные тесты для non-deterministic системы?

Хотя вы не можете guarantee exact wording output агента, вы можете создать тесты, которые проверяют key элементы. Например, вы можете написать тест, verifying содержит ли финальный ответ агента specific информацию или successfully вызвал ли он certain tool с right parameters. Это often done с использованием mock tools в dedicated testing среде.

## 34. Чем prompting агента отличается от простого промпта ChatGPT?

Prompting агента involves создание detailed "system prompt" или конституции, которая acts как его operating инструкции. Это goes beyond single user query; оно определяет роль агента, его available tools, паттерны, которые он should follow (such as ReAct или Planning), его ограничения и personality.

## 35. Какие ключевые компоненты хорошего system prompt для агента?

Strong system prompt typically включает:

- **Роль и цель:** Clearly определить кто агент и какова его primary purpose.
- **Определения инструментов:** Список available tools, их описания и как использовать (например, в specific function-calling формате).
- **Ограничения и правила:** Явные инструкции about что агент *should not* делать (например, "Не использовать tools без одобрения", "Не предоставлять financial advice").
- **Инструкции по процессу:** Руководство о том, какие паттерны использовать. Например, "Сначала создай план. Затем выполни план step-by-step."
- **Примеры траекторий:** Предоставление нескольких примеров успешных "thought-action-observation" loops может significantly улучшить reliability агента.

## 36. Что такое "prompt leakage"?

Prompt leakage occurs когда parts system prompt (such as tool definitions или internal instructions) inadvertent revealed в финальном ответе агента пользователю. Это может быть confusing для пользователя и expose underlying implementation details. Техники such as использование separate prompts для reasoning и для генерации финального ответа могут помочь prevent это.

## 37. Какие будущие trends в агентных системах?

Руководство указывает к будущему с:

- **More автономными агентами:** Агенты, которые требуют less human intervention и могут learn и adapt самостоятельно.
- **Высоко специализированными агентами:** Экосистема агентов, которые могут быть hired или subscribed для specific задач (например, travel агент, research агент).
- **Better tools и platforms:** Разработка более sophisticated фреймворков и platforms, которые making easier создавать, тестировать и deploy надёжные multi-agent системы.

---

Исходный репозиторий: [Mathews-Tom/Agentic-Design-Patterns](https://github.com/Mathews-Tom/Agentic-Design-Patterns)
