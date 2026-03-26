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

```python
from pydantic import BaseModel, EmailStr, Field, ValidationError
from typing import List, Optional
from datetime import date


# --- Pydantic Model Definition (from above) ---
class User(BaseModel):
    name: str = Field(..., description="The full name of the user.")
    email: EmailStr = Field(..., description="The user's email address.")
    date_of_birth: Optional[date] = Field(None, description="The user's date of birth.")
    interests: List[str] = Field(default_factory=list, description="A list of the user's interests.")


# --- Hypothetical LLM Output ---
llm_output_json = """
{
    "name": "Alice Wonderland",
    "email": "alice.w@example.com",
    "date_of_birth": "1995-07-21",
    "interests": [
        "Natural Language Processing",
        "Python Programming",
        "Gardening"
    ]
}
"""


# --- Parsing and Validation ---
try:
    # Use the model_validate_json class method to parse the JSON string.
    # This single step parses the JSON and validates the data against the User model.
    user_object = User.model_validate_json(llm_output_json)

    # Now you can work with a clean, type-safe Python object.
    print("Successfully created User object!")
    print(f"Name: {user_object.name}")
    print(f"Email: {user_object.email}")
    print(f"Date of Birth: {user_object.date_of_birth}")
    print(f"First Interest: {user_object.interests[0]}")

    # You can access the data like any other Python object attribute.
    # Pydantic has already converted the 'date_of_birth' string to a datetime.date object.
    print(f"Type of date_of_birth: {type(user_object.date_of_birth)}")
except ValidationError as e:
    # If the JSON is malformed or the data doesn't match the model's types,
    # Pydantic will raise a ValidationError.
    print("Failed to validate JSON from LLM.")
    print(e)
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

# Продвинутые техники

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
