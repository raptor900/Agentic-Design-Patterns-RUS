# Приложение C — Краткий обзор агентных фреймворков

## LangChain

LangChain — фреймворк для разработки приложений на основе LLM. Ядро — LangChain Expression Language (LCEL), позволяющий «соединять» компоненты в цепочку: вывод одного шага → вход следующего. Создан для Directed Acyclic Graphs (DAG) — поток в одном направлении без циклов.

Применение:
* Простой RAG: извлечение документа → промпт → ответ LLM.
* Обобщение: текст → промпт обобщения → вывод.
* Извлечение: структурированные данные (JSON) из текста.

```python
chain = prompt | model | output_parser
```

## LangGraph

LangGraph — библиотека поверх LangChain для продвинутых агентных систем. Определяет workflow как граф с узлами (функции/LCEL-цепочки) и рёбрами (условная логика). Главное преимущество — циклы: приложение может зацикливаться, повторять, вызывать инструменты до завершения задачи. Явное управление состоянием.

Применение:
* Многоагентные системы: супервизор маршрутизирует задачи специализированным агентам.
* Plan-and-Execute: агент создаёт план → выполняет шаг → обновляет план.
* Human-in-the-Loop: граф ждёт ввода человека.

| Возможность | LangChain | LangGraph |
|---|---|---|
| Базовая абстракция | Цепочка (LCEL) | Граф узлов |
| Тип workflow | Линейный (DAG) | Циклический |
| Управление состоянием | Stateless | Явное персистентное состояние |
| Основное применение | Простые последовательности | Сложные динамичные агенты |

### Какой выбрать?

* LangChain — когда процесс чёткий и линейный (A → B → C).
* LangGraph — когда нужны циклы, планирование, рефлексия, повторные попытки.

```python
class State(TypedDict):
    topic: str
    joke: str
    story: str

def call_llm_1(state: State):
    msg = llm.invoke(f"Write a joke about {state['topic']}")
    return {"joke": msg.content}

def call_llm_2(state: State):
    msg = llm.invoke(f"Write a story about {state['topic']}")
    return {"story": msg.content}

def aggregator(state: State):
    combined = f"STORY:\n{state['story']}\n\nJOKE:\n{state['joke']}"
    return {"combined_output": combined}

builder = StateGraph(State)
builder.add_node("call_llm_1", call_llm_1)
builder.add_node("call_llm_2", call_llm_2)
builder.add_node("aggregator", aggregator)
builder.add_edge(START, "call_llm_1")
builder.add_edge(START, "call_llm_2")
builder.add_edge("call_llm_1", "aggregator")
builder.add_edge("call_llm_2", "aggregator")
builder.add_edge("aggregator", END)
workflow = builder.compile()
```

Код определяет параллельный workflow LangGraph: шутка и история генерируются одновременно, затем объединяются.

## Google ADK

Google ADK — высокоуровневый фреймворк для построения и развёртывания многоагентных приложений. В отличие от LangChain/LangGraph, предлагает более opinionated и production-ориентированную систему.

LangGraph даёт тонкий контроль: developer определяет каждый узел и ребро. ADK абстрагирует это, предоставляя готовые паттерны: SequentialAgent, ParallelAgent. State и session management — имплицитно. Если LangGraph — инструмент для проектирования роботов, ADK — фабричный конвейер.

```python
from google.adk.agents import LlmAgent
from google.adk.tools import google_search

agent = LlmAgent(
    model="gemini-2.0-flash-exp",
    name="question_answer_agent",
    instruction="Respond to the query using google search",
    tools=[google_search],
)
```

## CrewAI

CrewAI — фреймворк оркестрации многоагентных систем через collaborative роли и структурированные процессы. Базовые компоненты: Agent (роль, цель, backstory), Task (единица работы), Crew (команда с процессом: sequential или hierarchical).

В отличие от LangGraph (граф с тонким контролем) и ADK (production-платформа), CrewAI фокусируется на логике агентного сотрудничества — симуляции команды специалистов.

```python
@crew
def crew(self) -> Crew:
    return Crew(
        agents=self.agents, tasks=self.tasks,
        process=Process.sequential, verbose=True,
    )
```

## Другие фреймворки

* **Microsoft AutoGen:** Оркестрация через разговор. Гибкий, но менее предсказуемый.
* **LlamaIndex:** Фреймворк данных для подключения LLM к внешним источникам. Силен в RAG, слабее в агентном control flow.
* **Haystack:** Open-source для масштабируемых поисковых систем. Оптимизирован для enterprise-grade retrieval.
* **MetaGPT:** Многоагентная система по SOP. Структурированные выходы, но узкая специализация.
* **SuperAGI:** Lifecycle management для автономных агентов. Production-readiness, но сложность.
* **Semantic Kernel (Microsoft):** SDK для интеграции LLM с кодом через plugins и planners. Seamless с .NET/Python.
* **Strands Agents (AWS):** Лёгкий SDK с model-driven подходом. Model-agnostic, MCP-интеграция, простота и гибкость.

## Заключение

Ландшафт агентных фреймворков предлагает спектр от low-level библиотек до high-level платформ. LangChain — линейные workflow, LangGraph — циклические графы, CrewAI и ADK — команды агентов, LlamaIndex — данные. Ключевой trade-off: тонкий контроль graph-based систем vs. streamlined development opinionated платформ. Выбор зависит от потребностей: простая последовательность, динамический цикл или управляемая команда.

## Ссылки

1. LangChain: [https://www.langchain.com/](https://www.langchain.com/)
2. LangGraph: [https://www.langchain.com/langgraph](https://www.langchain.com/langgraph)
3. Google ADK: [https://google.github.io/adk-docs/](https://google.github.io/adk-docs/)
4. CrewAI: [https://docs.crewai.com/en/introduction](https://docs.crewai.com/en/introduction)
