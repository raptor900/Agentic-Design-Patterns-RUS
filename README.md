# Паттерны проектирования агентных систем (Agentic Design Patterns)

Этот репозиторий содержит полный текст книги «Agentic Design Patterns» авторов Antonio Gulli и Mauro Sauco. Контент был скомпилирован и структурирован Tom Mathews для удобного доступа и использования сообществом.
На русский язык текст был переведён Юрием Богдановым.

![Agentic Design Patterns - Book Cover](assets/Agentic_Design_Patterns_Book_Cover.png)

## Авторство и признание

- **Авторы:** [Antonio Gulli](https://www.linkedin.com/in/searchguy/) и [Mauro Sauco](https://www.linkedin.com/in/maurosauco/)
- **Компиляция:** [Tom Mathews](https://www.linkedin.com/in/mathews-tom/)
- **Перевод на русский** [Юрий Богданов](https://github.com/raptor900)

### Чем эта книга выделяется?

Это 424-страничное руководство решает реальные задачи, с которыми мы сталкиваемся при создании интеллектуальных автономных AI-систем. Оно восполняет разрыв между теорией и реализацией — именно то, что сейчас необходимо нашей области. Это лучший ресурс для всех, кто серьёзно настроен на создание настоящих AI-систем. Если вы инженер, исследователь или менеджер по продукту, готовый выйти за рамки базовых LLM-приложений и построить по-настоящему надёжных AI-агентов — эта книга для вас.

Книга охватывает ключевые агентные паттерны, включая Prompt Chaining, Routing, Planning и Multi-Agent Systems, с практическими примерами на коде. Здесь вы найдёте полное описание Tool Use, Memory Management и RAG, а также продвинутые темы — Reasoning Techniques и межагентное взаимодействие.

Внутри вы найдёте:

- **Примеры рабочего кода:** Не просто теория, а реализации, которые можно запустить.
- **Проверенные паттерны:** Управление памятью, обработка исключений, контроль ресурсов, защитные механизмы.
- **Продвинутые техники:** Мультиагентная оркестрация, межагентное общение, human-in-the-loop.
- **Полноценная глава по MCP (Model Context Protocol):** Ключевой фреймворк для интеграции инструментов с агентами.

Книга охватывает 21 базовый паттерн в 4 разделах:

1. Базовые паттерны (prompt chaining, routing, tool use)
2. Продвинутые системы (память, обучение, мониторинг)
3. Продакшен-аспекты (обработка ошибок, безопасность, оценка)
4. Мультиагентные архитектуры

Большинство AI-контента останавливается на «как вызвать API». Но в реальных системах вам нужно задавать вопросы:

- Что делать, если агент застрял в середине задачи?
- Как сохранять память на протяжении длинных сессий?
- Как предотвратить хаос при запуске 10+ агентов?

Эта книга отвечает на все эти вопросы паттернами, которые можно реально применить. Одно только приложение на 70+ страниц стоит затрат — в нём описаны техники продвинутого промптинга и обзор агентных фреймворков.

## Содержание

### Введение

- [Посвящение](00-Introduction/01-Dedication-1cQ61mNpiWn6eSORmWjEjF44vN2Lpba8kyKmNwIC60ig.md)
- [Благодарность](00-Introduction/02-Acknowledgment-1u2y6tY48bw8nriDUuwWEf9s8g66vyIqBKSKZDOS-n0s.md)
- [Предисловие](00-Introduction/03-Foreword-18Q9kfZuCTL37ztrSjLxwf8Elr5UfAiAavmnj0IqSpbU.md)
- [Взгляд мыслителя: сила и ответственность](00-Introduction/04-A_Thought_Leaders_Perspective_Power_and_Responsibility-1PWhaXD_UNKgJaxYe3JBxRFRt3_B8Wm67CFxtSBQ4LkU.md)
- [Введение](00-Introduction/05-Introduction-1K5jwqB6jh20uHL0TTWxqWOxFk-dzFxRvHzrRRV79hrg.md)
- [Что делает AI-систему агентом?](00-Introduction/06-What_makes_an_AI_system_an_Agent-1Nw6hRa7ItdLr_Tj5hF2q-OH8B_uPKb--RLn8SXZKA94.md)

### Часть первая: Базовые паттерны

- [Глава 1: Prompt Chaining](01-Part_One/Chapter_1-Prompt_Chaining-1flxKGrbnF2g8yh3F-oVD5Xx7ZumId56HbFpIiPdkqLI.md)
- [Глава 2: Routing](01-Part_One/Chapter_2-Routing-1ux_n8n3T4bYndOjs1DKW5ccpC802KISdy2IWnlvYbas.md)
- [Глава 3: Parallelization](01-Part_One/Chapter_3-Parallelization-1XVMp4RcRkoUJTVbrP2foWZX703CUJpWkrhyFU2cfUOA.md)
- [Глава 4: Reflection](01-Part_One/Chapter_4-Reflection-1HXXJOQIMWowtLw4WMiSR360caDAlZPtl5dPPgvq9IT4.md)
- [Глава 5: Tool Use (вызов функций)](01-Part_One/Chapter_5-Tool_Use_(Function_Calling)-1bE4iMljhppqGY1p48gQWtZvk6MfRuJRCiba1yRykGNE.md)
- [Глава 6: Planning](01-Part_One/Chapter_6-Planning-18vvNESEwHnVUREzIipuaDNCnNAREGqEfy9MQYC9wb4o.md)
- [Глава 7: Мультиагентное взаимодействие](01-Part_One/Chapter_7-Multi-Agent_Collaboration-1RZ5-2fykDQKOBx01pwfKkDe0GCs5ydca7xW9Q4wqS_M.md)

### Часть вторая: Продвинутые системы

- [Глава 8: Управление памятью](02-Part_Two/Chapter_8-Memory_Management-1asVTObtzIye0I9ypAztaeeI_sr_Hx2TORE02uUuqH_c.md)
- [Глава 9: Обучение и адаптация](02-Part_Two/Chapter_9-Learning_and_Adaptation-1UHTEDCmSM1nwB-iyMoHuYzVcu_B_4KkJ2ITGGUKqo8s.md)
- [Глава 10: Model Context Protocol (MCP)](02-Part_Two/Chapter_10-Model_Context_Protocol_(MCP)-1e6XimYczKmhX9zpqEyxLFWPQgGuG0brp7Hic2sFl_qw.md)
- [Глава 11: Постановка целей и мониторинг](02-Part_Two/Chapter_11-Goal_Setting_and_Monitoring-10ndlCB39BWjyFRWKpcoKib4vuPD1ojD-x0-ynMaf5uw.md)

### Часть третья: Продакшен-аспекты

- [Глава 12: Обработка исключений и восстановление](03-Part_Three/Chapter_12-Exception_Handling_and_Recovery-1C07AuMur6-infwE0viCp4QtAy_wWI-uceFm6MaYHQGk.md)
- [Глава 13: Human in the Loop](03-Part_Three/Chapter_13-Human_in_the_Loop-1ImOZcw6yeb7a-uRBMNP1VdovYfyip4IdsAcLu9yue-0.md)
- [Глава 14: Извлечение знаний (RAG)](03-Part_Three/Chapter_14-Knowledge_Retrieval_(RAG)-1v96Oobio6xDOqbK8ejsXjmOc4Dp2uoLMo5_gfJgi-NE.md)

### Часть четвёртая: Мультиагентные архитектуры

- [Глава 15: Межагентное взаимодействие (A2A)](04-Part_Four/Chapter_15-Inter_Agent_Communication_(A2A)-1H6HmUYcy5kugt5gt7Kh2Zzb8C62d5pu36RsgMNDCX24.md)
- [Глава 16: Ресурсоосознанная оптимизация](04-Part_Four/Chapter_16-Resource_Aware_Optimization-1nAN58l6JjqEJHk43126uh7xgdEblCpcbsNUHXgtBmJQ.md)
- [Глава 17: Техники рассуждения](04-Part_Four/Chapter_17-Reasoning_Techniques-1Yt1W_hLaC6ZNgJXfT4W6NrCL4TzNVdKOX50kgpHiIq4.md)
- [Глава 18: Защитные механизмы и паттерны безопасности](04-Part_Four/Chapter_18-Guardrails_Safety_Patterns-1Gpc5af_okze1kprRLohP6-81e1KwL6HggjeLvxQyIuk.md)
- [Глава 19: Оценка и мониторинг](04-Part_Four/Chapter_19-Evaluation_and_Monitoring-1G3zOZM2ZOd0gUp5dy66FUjKMOcALh9l-JpvPxgGMm8w.md)
- [Глава 20: Приоритизация](04-Part_Four/Chapter_20-Prioritization-1qyXxGM2hNqW_qjXuBFxrEUeoYVO79BoW1ogKu1bfdCY.md)
- [Глава 21: Исследование и открытие](04-Part_Four/Chapter_21-Exploration_and_Discovery-1zeeMVTqjqRIli6G9MMWThhoQhvKqLOjJF2EHHUXLhdk.md)

### Приложения

- [Приложение A: Продвинутые техники промптинга](05-Appendix/Appendix_A-Advanced_Prompting_Techniques-1V7EKEWibOH6IhHD_PtbFZiml492-2191jDQCcTkhtTI.md)
- [Приложение B: AI Agentic Interactions: от GUI к реальному миру](05-Appendix/Appendix_B-AI_Agentic_Interactions_From_GUI_to_Real_World_Environment-11pma_tCoC7uZ2SFKjcR5KyIq0_ooMGSoadI6f9mxG2I.md)
- [Приложение C: Краткий обзор агентных фреймворков](05-Appendix/Appendix_C-Quick_Overview_of_Agentic_Frameworks-151rGsiEYOkXUcNDRus_N8TxxuvjoyTDViBhzt9z0Mfw.md)
- [Приложение D: Создание агента с AgentSpace (только онлайн)](05-Appendix/Appendix_D-Building_an_Agent_with_AgentSpace_(on_line_only)-1bDRJ8mKtLTeWNC-cGD0Cr8pEJQgJHNcjqz5ekloAjaE.md)
- [Приложение E: AI-агенты в CLI](05-Appendix/Appendix_E-AI_Agents_on_the_CLI-1W4znto0a8Ikajw5a4tEyRAaB2nJPJw_iFc4w4qNnjho.md)
- [Приложение F: Взгляд изнутри: движки рассуждения агента](05-Appendix/Appendix_F-Under_the_Hood_An_Inside_Look_at_the_Agents_Reasoning_Engines-14q3fQ-FZmDgiughno_WLSILMWkURvUgR7mlGiFtvwd4.md)
- [Приложение G: Кодирующие агенты](05-Appendix/Appendix_G-Coding_Agents-1tVyhgwrD4fu_D_pHUrwhNxoguRG3tLc1KObXFxrxE_s.md)

## Лицензия

Этот репозиторий распространяется под [MIT License](LICENSE).

![Agentic Design Patterns](assets/Agentic_Design_Patterns.png)

---

Исходный репозиторий: [Mathews-Tom/Agentic-Design-Patterns](https://github.com/Mathews-Tom/Agentic-Design-Patterns)
