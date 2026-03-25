# Приложение D — Построение агента с AgentSpace

## Обзор

AgentSpace — платформа для «агентно-управляемого предприятия», интегрирующая ИИ в рабочие процессы. Ядро — унифицированный поиск по всему цифровому следу организации: документам, письмам, базам данных. Использует продвинутые модели (Google Gemini) для понимания и синтеза.

Платформа позволяет создавать и развёртывать специализированных AI-агентов для сложных задач и автоматизации. Агенты могут рассуждать, планировать и выполнять многошаговые действия автономно: исследовать тему, составлять отчёт с цитатами, генерировать аудиосводку.

AgentSpace строит граф знаний предприятия, отображающий связи между людьми, документами и данными. Это позволяет ИИ понимать контекст и давать персонализированные результаты. Включает no-code интерфейс Agent Designer для создания агентов без глубоких технических знаний.

Поддерживает многоагентные системы через протокол A2A. Безопасность: role-based доступ, шифрование данных.

## Как построить агента с AgentSpace UI

На Рис. 1 показан доступ к AgentSpace через Google Cloud Console.

![GCP: Доступ к AgentSpace](../assets/GCP_Access_AgentSpace.png)

Рис. 1: Доступ к AgentSpace через Google Cloud Console

Агент подключается к сервисам: Calendar, Gmail, Jira, Outlook, ServiceNow (см. Рис. 2).

![GCP: Интеграция с сервисами](../assets/GCP_Integrate_with_diverse_services.png)

Рис. 2: Интеграция с Google и сторонними платформами

Агент использует промпт из галереи Google или пользовательский (Рис. 3-4).

![GCP: Галерея готовых промптов](../assets/GCP_Googles_Gallery_of_Pre_Assembled_Prompts.png)

Рис. 3: Галерея готовых промптов Google

![GCP: Настройка промпта агента](../assets/GCP_Customizing_the_Agents_Prompt.png)

Рис. 4: Настройка промпта агента

AgentSpace предлагает расширенные возможности: интеграция с datastores, Google Knowledge Graph, веб-интерфейс, аналитика (Рис. 5).

![GCP: Расширенные возможности AgentSpace](../assets/GCP_AgentSpace_Advanced_Capabilities.png)

Рис. 5: Расширенные возможности AgentSpace

После завершения доступен чат-интерфейс (Рис. 6).

![GCP: Интерфейс AgentSpace](../assets/GCP_AgentSpace_User_Interface_for_initiating_a_chat_with_your_Agent.png)

Рис. 6: Интерфейс AgentSpace для начала диалога с агентом.

## Заключение

AgentSpace предоставляет фреймворк для разработки и развёртывания AI-агентов в цифровой инфраструктуре организации. Система связывает сложные бэкенд-процессы (автономное рассуждение, граф знаний) с GUI для построения агентов. Пользователи конфигурируют агенты, интегрируя сервисы и определяя операционные параметры через промпты.

## Ссылки

1. Agent Designer: [https://cloud.google.com/agentspace/agentspace-enterprise/docs/agent-designer](https://cloud.google.com/agentspace/agentspace-enterprise/docs/agent-designer)
2. Google Cloud Skills Boost: [https://www.cloudskillsboost.google/](https://www.cloudskillsboost.google/)
