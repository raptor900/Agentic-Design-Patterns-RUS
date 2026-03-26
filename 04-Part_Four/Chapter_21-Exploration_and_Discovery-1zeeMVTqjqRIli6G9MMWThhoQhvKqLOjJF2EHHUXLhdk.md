# Глава 21: Исследование и открытие

Эта глава рассматривает паттерны, позволяющие агентам активно находить новую информацию и выявлять unknown unknowns в своей среде. Исследование и открытие отличаются от реактивного поведения — они фокусируются на проактивном выходе в незнакомые территории.

## Практические применения и сценарии использования

* **Научные исследования:** Проектирование и проведение экспериментов, генерация гипотез.
* **Игры и стратегии:** Исследование игровых состояний, открытие emergent-стратегий (AlphaGo).
* **Исследование рынка:** Сканирование данных для выявления трендов и возможностей.
* **Поиск уязвимостей:** Пробинг систем для обнаружения слабых мест.
* **Генерация контента:** Исследование комбинаций стилей и тем.
* **Персонализированное обучение:** Адаптивные пути обучения.

**Google Co-Scientist**

AI Co-Scientist — система Google Research, выступающая вычислительным научным сотрудником. Работает на Gemini. Многоагентная архитектура:

* **Generation agent:** Генерирует начальные гипотезы.
* **Reflection agent:** Критически оценивает корректность и новизну.
* **Ranking agent:** Турнирная система ранжирования.
* **Evolution agent:** Непрерывное совершенствование top-гипотез.
* **Proximity agent:** Граф близости для кластеризации.
* **Meta-review agent:** Синтез инсайтов и обратная связь.

![AI Co-Scientist](../assets/AI_Co_Scientist_Ideation_to_Validation.png)

Рис. 1: AI Co-Scientist: от идеации к валидации

Валидация: точность 78.4% на GPQA. Новые кандидаты лекарств для AML (KIRA6), эпигенетические мишени для фиброза печени.

## Практический пример кода: Agent Laboratory

Agent Laboratory — фреймворк для автономного исследования. Включает AgentRxiv. Этапы: обзор литературы, эксперименты, написание отчёта, обмен знаниями.

Механизм оценки: три агента-рецензента с разными перспективами.

```python
class ReviewersAgent:
    def __init__(self, model="gpt-4o-mini", notes=None, openai_api_key=None):
        if notes is None:
            self.notes = []
        else:
            self.notes = notes
        self.model = model
        self.openai_api_key = openai_api_key

    def inference(self, plan, report):
        reviewer_1 = "You are a harsh but fair reviewer and expect good experiments that lead to insights for the research topic."
        review_1 = get_score(
            outlined_plan=plan,
            latex=report,
            reward_model_llm=self.model,
            reviewer_type=reviewer_1,
            openai_api_key=self.openai_api_key
        )

        reviewer_2 = "You are a harsh and critical but fair reviewer who is looking for an idea that would be impactful in the field."
        review_2 = get_score(
            outlined_plan=plan,
            latex=report,
            reward_model_llm=self.model,
            reviewer_type=reviewer_2,
            openai_api_key=self.openai_api_key
        )

        reviewer_3 = "You are a harsh but fair open-minded reviewer that is looking for novel ideas that have not been proposed before."
        review_3 = get_score(
            outlined_plan=plan,
            latex=report,
            reward_model_llm=self.model,
            reviewer_type=reviewer_3,
            openai_api_key=self.openai_api_key
        )

        return f"Reviewer #1:\n{review_1}, \nReviewer #2:\n{review_2}, \nReviewer #3:\n{review_3}"
```

Функция `get_score` генерирует detailed review в JSON (Summary, Strengths, Weaknesses, Originality, Quality, Overall, Decision).

Агенты: Professor (управление), PostDoc (исследование), Reviewers (peer review), ML Engineer (код), SW Engineer (direction).

## Краткий обзор

**Что:** AI-агенты работают в рамках предопределённых знаний. Нужен переход к проактивному исследованию.

**Почему:** Многоагентные системы эмулируют научный метод. Google Co-Scientist, Agent Laboratory.

**Когда использования:** В открытых, сложных, эволюционирующих доменах.

**Визуальное резюме:**

![Паттерн исследования и открытия](../assets/Exploration_and_Discovery_Design_Pattern.png)

Рис. 2: Паттерн «Исследование и открытия»

## Ключевые выводы

* Исследование позволяет находить новую информацию в динамичных средах.
* Google Co-Scientist — автономная генерация гипотез и дизайн экспериментов.
* Agent Laboratory автоматизирует обзор, эксперименты и отчёты.
* Агенты расширяют человеческую креативность, ускоряя инновации.

## Заключение

Паттерн «Исследование и открытия» — суть агентной системы. Многоагентные фреймворки создают иерархии, имитирующие исследовательские команды. Оркестрация emergent-поведений позволяет преследовать долгосрочные цели. Развитие агентных возможностей требует приверженности к безопасности.

## Ссылки

1. Exploration–Exploitation Dilemma: [https://en.wikipedia.org/wiki/Exploration%E2%80%93exploitation_dilemma](https://en.wikipedia.org/wiki/Exploration%E2%80%93exploitation_dilemma)
2. Google Co-Scientist: [https://research.google/blog/accelerating-scientific-breakthroughs-with-an-ai-co-scientist/](https://research.google/blog/accelerating-scientific-breakthroughs-with-an-ai-co-scientist/)
3. Agent Laboratory: [https://github.com/SamuelSchmidgall/AgentLaboratory](https://github.com/SamuelSchmidgall/AgentLaboratory)
4. AgentRxiv: [https://agentrxiv.github.io/](https://agentrxiv.github.io/)
````python
def get_score(outlined_plan, latex, reward_model_llm, reviewer_type=None, attempts=3, openai_api_key=None):
   e = str()
   for _attempt in range(attempts):
       try:
          
           template_instructions = """
           Respond in the following format:

           THOUGHT:
           <THOUGHT>

           REVIEW JSON:
           ```

json
           <JSON>
           ```

           In <THOUGHT>, first briefly discuss your intuitions 
           and reasoning for the evaluation.
           Detail your high-level arguments, necessary choices 
           and desired outcomes of the review.
           Do not make generic comments here, but be specific 
           to your current paper.
           Treat this as the note-taking phase of your review.

           In <JSON>, provide the review in JSON format with 
           the following fields in the order:
           - "Summary": A summary of the paper content and 
           its contributions.
           - "Strengths": A list of strengths of the paper.
           - "Weaknesses": A list of weaknesses of the paper.
           - "Originality": A rating from 1 to 4 
             (low, medium, high, very high).
           - "Quality": A rating from 1 to 4 
             (low, medium, high, very high).
           - "Clarity": A rating from 1 to 4 
             (low, medium, high, very high).
           - "Significance": A rating from 1 to 4 
             (low, medium, high, very high).
           - "Questions": A set of clarifying questions to be
              answered by the paper authors.
           - "Limitations": A set of limitations and potential
              negative societal impacts of the work.
           - "Ethical Concerns": A boolean value indicating 
              whether there are ethical concerns.
           - "Soundness": A rating from 1 to 4 
              (poor, fair, good, excellent).
           - "Presentation": A rating from 1 to 4 
              (poor, fair, good, excellent).
           - "Contribution": A rating from 1 to 4 
             (poor, fair, good, excellent).
           - "Overall": A rating from 1 to 10 
             (very strong reject to award quality).
           - "Confidence": A rating from 1 to 5 
             (low, medium, high, very high, absolute).
           - "Decision": A decision that has to be one of the
             following: Accept, Reject.

           For the "Decision" field, don't use Weak Accept,   
           Borderline Accept, Borderline Reject, or Strong Reject.  
           Instead, only use Accept or Reject.
           This JSON will be automatically parsed, so ensure 
           the format is precise.
           """
```

`

In this multi-agent system, the research process is structured around specialized roles, mirroring a typical academic hierarchy to streamline workflow and optimize output.

**Professor Agent:** The Professor Agent functions as the primary research director, responsible for establishing the research agenda, defining research questions, and delegating tasks to other agents. This agent sets the strategic direction and ensures alignment with project objectives.

````python
class ProfessorAgent(BaseAgent):
   def __init__(self, model="gpt4omini", notes=None, max_steps=100, openai_api_key=None):
       super().__init__(model, notes, max_steps, openai_api_key)
       self.phases = ["report writing"]

   def generate_readme(self):
       sys_prompt = f"""You are {self.role_description()} \n Here is the written paper \n{self.report}. Task instructions: Your goal is to integrate all of the knowledge, code, reports, and notes provided to you and generate a readme.md for a github repository."""
       history_str = "\n".join([_[1] for _ in self.history])
       prompt = (
           f"""History: {history_str}\n{'~' * 10}\n"""
           f"Please produce the readme below in markdown:\n")
       model_resp = query_model(model_str=self.model, system_prompt=sys_prompt, prompt=prompt, openai_api_key=self.openai_api_key)
       return model_resp.replace("```

markdown", "")
````

**PostDoc Agent:** The PostDoc Agent's role is to execute the research. This includes conducting literature reviews, designing and implementing experiments, and generating research outputs such as papers. Importantly, the PostDoc Agent has the capability to write and execute code, enabling the practical implementation of experimental protocols and data analysis. This agent is the primary producer of research artifacts.

```

python
class PostdocAgent(BaseAgent):
    def __init__(self, model="gpt4omini", notes=None, max_steps=100, openai_api_key=None):
        super().__init__(model, notes, max_steps, openai_api_key)
        self.phases = ["plan formulation", "results interpretation"]

    def context(self, phase):
        sr_str = str()
        if self.second_round:
            sr_str = (
                f"The following are results from the previous experiments\n",
                f"Previous Experiment code: {self.prev_results_code}\n"
                f"Previous Results: {self.prev_exp_results}\n"
                f"Previous Interpretation of results: {self.prev_interpretation}\n"
                f"Previous Report: {self.prev_report}\n"
                f"{self.reviewer_response}\n\n\n"
            )

        if phase == "plan formulation":
            return (
                sr_str,
                f"Current Literature Review: {self.lit_review_sum}",
            )
        elif phase == "results interpretation":
            return (
                sr_str,
                f"Current Literature Review: {self.lit_review_sum}\n"
                f"Current Plan: {self.plan}\n"
                f"Current Dataset code: {self.dataset_code}\n"
                f"Current Experiment code: {self.results_code}\n"
                f"Current Results: {self.exp_results}"
            )

        return ""
```

**Reviewer Agents:** Reviewer agents perform critical evaluations of research outputs from the PostDoc Agent, assessing the quality, validity, and scientific rigor of papers and experimental results. This evaluation phase emulates the peer-review process in academic settings to ensure a high standard of research output before finalization.

**ML Engineering Agents**:The Machine Learning Engineering Agents serve as machine learning engineers, engaging in dialogic collaboration with a PhD student to develop code. Their central function is to generate uncomplicated code for data preprocessing, integrating insights derived from the provided literature review and experimental protocol. This guarantees that the data is appropriately formatted and prepared for the designated experiment.

```

markdown
"You are a machine learning engineer being directed by a PhD student who will help you write the code, and you can interact with them through dialogue.\n"
"Your goal is to produce code that prepares the data for the provided experiment. You should aim for simple code to prepare the data, not complex code. You should integrate the provided literature review and the plan and come up with code to prepare data for this experiment.\n"
```

**SWEngineerAgents:** Software Engineering Agents guide Machine Learning Engineer Agents. Their main purpose is to assist the Machine Learning Engineer Agent in creating straightforward data preparation code for a specific experiment. The Software Engineer Agent integrates the provided literature review and experimental plan, ensuring the generated code is uncomplicated and directly relevant to the research objectives.

```

markdown
"You are a software engineer directing a machine learning engineer, where the machine learning engineer will be writing the code, and you can interact with them through dialogue.\n"
"Your goal is to help the ML engineer produce code that prepares the data for the provided experiment. You should aim for very simple code to prepare the data, not complex code. You should integrate the provided literature review and the plan and come up with code to prepare data for this experiment.\n"
```

In summary, "Agent Laboratory" represents a sophisticated framework for autonomous scientific research. It is designed to augment human research capabilities by automating key research stages and facilitating collaborative AI-driven knowledge generation. The system aims to increase research efficiency by managing routine tasks while maintaining human oversight.


**What:** AI agents often operate within predefined knowledge, limiting their ability to tackle novel situations or open-ended problems. In complex and dynamic environments, this static, pre-programmed information is insufficient for true innovation or discovery. The fundamental challenge is to enable agents to move beyond simple optimization to actively seek out new information and identify "unknown unknowns." This necessitates a paradigm shift from purely reactive behaviors to proactive, Agentic exploration that expands the system's own understanding and capabilities.

**Why:** The standardized solution is to build Agentic AI systems specifically designed for autonomous exploration and discovery. These systems often utilize a multi-agent framework where specialized LLMs collaborate to emulate processes like the scientific method. For instance, distinct agents can be tasked with generating hypotheses, critically reviewing them, and evolving the most promising concepts. This structured, collaborative methodology allows the system to intelligently navigate vast information landscapes, design and execute experiments, and generate genuinely new knowledge. By automating the labor-intensive aspects of exploration, these systems augment human intellect and significantly accelerate the pace of discovery.

**Rule of Thumb:** Use the Exploration and Discovery pattern when operating in open-ended, complex, or rapidly evolving domains where the solution space is not fully defined. It is ideal for tasks requiring the generation of novel hypotheses, strategies, or insights, such as in scientific research, market analysis, and creative content generation. This pattern is essential when the objective is to uncover "unknown unknowns" rather than merely optimizing a known process.

**Visual Summary:**

![Exploration and Discovery Design Pattern](../assets/Exploration_and_Discovery_Design_Pattern.png)

Fig.2: Exploration and Discovery design pattern


* Exploration and Discovery in AI enable agents to actively pursue new information and possibilities, which is essential for navigating complex and evolving environments.  
* Systems such as Google Co-Scientist demonstrate how Agents can autonomously generate hypotheses and design experiments, supplementing human scientific research.  
* The multi-agent framework, exemplified by Agent Laboratory's specialized roles, improves research through the automation of literature review, experimentation, and report writing.  
* Ultimately, these Agents aim to enhance human creativity and problem-solving by managing computationally intensive tasks, thus accelerating innovation and discovery.


In conclusion, the Exploration and Discovery pattern is the very essence of a truly agentic system, defining its ability to move beyond passive instruction-following to proactively explore its environment. This innate agentic drive is what empowers an AI to operate autonomously in complex domains, not merely executing tasks but independently setting sub-goals to uncover novel information. This advanced agentic behavior is most powerfully realized through multi-agent frameworks where each agent embodies a specific, proactive role in a larger collaborative process. For instance, the highly agentic system of Google's Co-scientist features agents that autonomously generate, debate, and evolve scientific hypotheses.

Frameworks like Agent Laboratory further structure this by creating an agentic hierarchy that mimics human research teams, enabling the system to self-manage the entire discovery lifecycle. The core of this pattern lies in orchestrating emergent agentic behaviors, allowing the system to pursue long-term, open-ended goals with minimal human intervention. This elevates the human-AI partnership, positioning the AI as a genuine agentic collaborator that handles the autonomous execution of exploratory tasks. By delegating this proactive discovery work to an agentic system, human intellect is significantly augmented, accelerating innovation. The development of such powerful agentic capabilities also necessitates a strong commitment to safety and ethical oversight. Ultimately, this pattern provides the blueprint for creating truly agentic AI, transforming computational tools into independent, goal-seeking partners in the pursuit of knowledge.


1. Exploration-Exploitation Dilemma**:** A fundamental problem in reinforcement learning and decision-making under uncertainty. [https://en.wikipedia.org/wiki/Exploration%E2%80%93exploitation_dilemma](https://en.wikipedia.org/wiki/Exploration%E2%80%93exploitation_dilemma)
2. Google Co-Scientist: [https://research.google/blog/accelerating-scientific-breakthroughs-with-an-ai-co-scientist/](https://research.google/blog/accelerating-scientific-breakthroughs-with-an-ai-co-scientist/)
3. Agent Laboratory: Using LLM Agents as Research Assistants [https://github.com/SamuelSchmidgall/AgentLaboratory](https://github.com/SamuelSchmidgall/AgentLaboratory)
4. AgentRxiv: Towards Collaborative Autonomous Research: [https://agentrxiv.github.io/](https://agentrxiv.github.io/)
