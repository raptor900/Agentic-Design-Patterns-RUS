# Приложение C — Краткий обзор агентных фреймворков

## LangChain

LangChain — фреймворк для разработки приложений на основе LLM. Его основная сила — LangChain Expression Language (LCEL), позволяющий «соединять» компоненты в цепочку. Это создаёт чёткую линейную последовательность, где выход одного шага становится входом для следующего. Создан для Directed Acyclic Graphs (DAG) — поток в одном направлении без циклов.

Применение:

* Простой RAG: извлечение документа, создание промпта, получение ответа от LLM.
* Обобщение: получение текста пользователя, передача промпту обобщения, возврат результата.
* Извлечение: извлечение структурированных данных (JSON) из текстового блока.

Python

```python
# A simple LCEL chain conceptually # (This is not runnable code, just illustrates the flow) 
chain = prompt | model | output_parse
```

## LangGraph

LangGraph — библиотека поверх LangChain для продвинутых агентных систем. Позволяет определить рабочий процесс как граф с узлами (функции или LCEL-цепочки) и рёбрами (условная логика). Главное преимущество — способность создавать циклы, allowing приложению зацикливаться, повторять или вызывать инструменты в гибком порядке до завершения задачи. Явно управляет состоянием приложения, которое передаётся между узлами.

Применение:

* Многоагентные системы: агент-супервизор маршрутизирует задачи специализированным рабочим агентам.
* Агенты Plan-and-Execute: агент создаёт план, выполняет шаг, затем возвращается для обновления плана.
* Human-in-the-loop: граф может ожидать ввода человека перед выбором следующего узла.

| Возможность | LangChain | LangGraph |
| :---- | :---- | :---- |
| Базовая абстракция | Цепочка (LCEL) | Граф узлов |
| Тип workflow | Линейный (DAG) | Циклический (графы с циклами) |
| Управление состоянием | Обычно stateless за запуск | Явный и persistent объект состояния |
| Основное применение | Простые предсказуемые последовательности | Сложные, динамичные, stateful агенты |

### Какой выбрать?

* Выбирайте LangChain, когда ваше приложение имеет чёткий, предсказуемый и линейный поток шагов. Если процесс идёт от A к B к C без необходимости зацикливания, LangChain с LCEL — идеальный инструмент.
* Выбирайте LangGraph, когда приложению необходимо рассуждать, планировать или работать в цикле. Если агенту нужно использовать инструменты, рефлексировать над результатами и повторять с другим подходом, необходимы циклическая и stateful природа LangGraph.

```python
# Graph state
class State(TypedDict):
    topic: str
    joke: str
    story: str
    poem: str
    combined_output: str


# Nodes
def call_llm_1(state: State):
    """First LLM call to generate initial joke"""
    msg = llm.invoke(f"Write a joke about {state['topic']}")
    return {"joke": msg.content}


def call_llm_2(state: State):
    """Second LLM call to generate story"""
    msg = llm.invoke(f"Write a story about {state['topic']}")
    return {"story": msg.content}


def call_llm_3(state: State):
    """Third LLM call to generate poem"""
    msg = llm.invoke(f"Write a poem about {state['topic']}")
    return {"poem": msg.content}


def aggregator(state: State):
    """Combine the joke and story into a single output"""
    combined = f"Here's a story, joke, and poem about {state['topic']}!\n\n"
    combined += f"STORY:\n{state['story']}\n\n"
    combined += f"JOKE:\n{state['joke']}\n\n"
    combined += f"POEM:\n{state['poem']}"
    return {"combined_output": combined}


# Build workflow
parallel_builder = StateGraph(State)

# Add nodes
parallel_builder.add_node("call_llm_1", call_llm_1)
parallel_builder.add_node("call_llm_2", call_llm_2)
parallel_builder.add_node("call_llm_3", call_llm_3)
parallel_builder.add_node("aggregator", aggregator)

# Add edges to connect nodes
parallel_builder.add_edge(START, "call_llm_1")
parallel_builder.add_edge(START, "call_llm_2")
parallel_builder.add_edge(START, "call_llm_3")
parallel_builder.add_edge("call_llm_1", "aggregator")
parallel_builder.add_edge("call_llm_2", "aggregator")
parallel_builder.add_edge("call_llm_3", "aggregator")
parallel_builder.add_edge("aggregator", END)

parallel_workflow = parallel_builder.compile()

# Show workflow
display(Image(parallel_workflow.get_graph().draw_mermaid_png()))

# Invoke
state = parallel_workflow.invoke({"topic": "cats"})
print(state["combined_output"])
```

Этот код определяет и запускает параллельный workflow LangGraph. Его основная цель — одновременно генерировать шутку и историю по заданной теме, а затем объединять их в единый отформатированный текстовый вывод.

## Google ADK

Google Agent Development Kit (ADK) предоставляет высокоуровневый структурированный фреймворк для построения и развёртывания приложений из множества взаимодействующих AI-агентов. В противоположность LangChain и LangGraph, ADK предлагает более opinionated и production-ориентированную систему оркестрации агентного взаимодействия.

LangChain работает на самом базовом уровне, предоставляя компоненты и стандартизированные интерфейсы для создания последовательностей операций. LangGraph расширяет это, вводя более гибкий и мощный control flow; он рассматривает рабочий процесс агента как stateful граф. Google ADK абстрагирует многое из низкоуровневого построения графов, предоставляя готовые архитектурные паттерны: SequentialAgent или ParallelAgent.

По сути, LangGraph даёт инструменты для проектирования детальной проводки робота, а Google ADK — фабричный конвейер для построения и управления парком роботов.

```python
from google.adk.agents import LlmAgent
from google.adk.tools import google_Search

dice_agent = LlmAgent(
    model="gemini-2.0-flash-exp",
    name="question_answer_agent",
    description="A helpful assistant agent that can answer questions.",
    instruction="""Respond to the query using google search""",
    tools=[google_search],
)
```

## CrewAI

CrewAI — фреймворк оркестрации многоагентных систем через collaborative роли и структурированные процессы. Работает на более высоком уровне абстракции, предоставляя концептуальную модель, отражающую человеческую команду.

Базовые компоненты: Agent (роль, цель, backstory), Task (дискретная единица работы), Crew (команда с процессом: sequential или hierarchical).

В отличие от LangGraph (тонкий контроль flow) и Google ADK (production-платформа), CrewAI фокусируется на логике агентного сотрудничества.

```python
@crew
def crew(self) -> Crew:
   """Creates the research crew"""
   return Crew(
     agents=self.agents,
     tasks=self.tasks,
     process=Process.sequential,
     verbose=True,
   )
```

Этот код настраивает последовательный workflow для команды AI-агентов, которые решают список задач в определённом порядке с подробным логированием.

## Другие фреймворки разработки агентов

**Microsoft AutoGen:** Фреймворк, центрированный на оркестрации агентов через диалоги. Гибкий conversation-driven подход, но менее предсказуемые пути выполнения.

**LlamaIndex:** Фреймворк данных для подключения LLM к внешним источникам. Исключительно силён в RAG, но native возможности для агентного control flow менее развиты.

**Haystack:** Open-source для масштабируемых поисковых систем на основе языковых моделей. Оптимизирован для enterprise-grade извлечения информации.

**MetaGPT:** Многоагентная система на основе SOP. Структурированные и связные выходы, но узкая специализация.

**SuperAGI:** Open-source для полного lifecycle management автономных агентов. Production-readiness, но сложность.

**Semantic Kernel:** SDK от Microsoft для интеграции LLM с кодом через plugins и planners. Seamless с .NET/Python.

**Strands Agents:** Лёгкий и гибкий SDK от AWS с model-driven подходом. Model-agnostic, MCP-интеграция.

## Заключение

Ландшафт агентных фреймворков предлагает разнообразный спектр инструментов — от low-level библиотек до high-level платформ. LangChain — линейные workflow, LangGraph — stateful циклические графы, CrewAI и Google ADK — команды агентов, LlamaIndex — data-intensive приложения. Ключевой trade-off: тонкий контроль graph-based систем vs. streamlined development opinionated платформ. Выбор зависит от потребностей: простая последовательность, динамический цикл или управляемая команда специалистов.



1. LangChain: [https://www.langchain.com/](https://www.langchain.com/)
2. LangGraph: [https://www.langchain.com/langgraph](https://www.langchain.com/langgraph)
3. Google ADK: [https://google.github.io/adk-docs/](https://google.github.io/adk-docs/)
4. CrewAI: [https://docs.crewai.com/en/introduction](https://docs.crewai.com/en/introduction)
