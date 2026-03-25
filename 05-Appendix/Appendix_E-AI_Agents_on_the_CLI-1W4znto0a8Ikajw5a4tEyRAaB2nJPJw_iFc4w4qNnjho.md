# Приложение E — AI-агенты в командной строке

## Введение

Командная строка разработчика, долгое время бастон точных императивных команд, переживает глубокую трансформацию. Она эволюционирует из простого шелла в интеллектуальную совместную рабочую среду на основе нового класса инструментов: AI Agent CLI. Эти агенты выходят за рамки выполнения команд — они понимают естественный язык, хранят контекст всего кода и выполняют сложные многошаговые задачи.

## Claude CLI (Claude Code)

Anthropic Claude CLI — высокоуровневый кодирующий агент с глубоким пониманием архитектуры проекта. Создаёт ментальную модель репозитория для сложных задач. Взаимодействие — как pair programming: объясняет планы перед выполнением.

**Примеры:** Рефакторинг авторизации (session cookies → JWT), интеграция API по OpenAPI-спецификации, генерация документации.

Расширяется через MCP: пользовательские инструменты, API, запросы к БД.

## Gemini CLI

Google Gemini CLI — универсальный open-source агент с Gemini 2.5 Pro, огромным контекстным окном и мультимодальными возможностями. Open-source, generous free tier, цикл «Reason and Act».

**Примеры:** Мультимодальная разработка (скриншот → React-компонент), управление Google Cloud (GKE-кластеры), интеграция enterprise-инструментов через MCP, рефакторинг Java.

Встроенные инструменты: файловая система, шелл, веб-поиск, память. Безопасность: sandboxing, MCP-серверы.

## Aider

Open-source AI-помощник, работающий непосредственно с файлами и коммитящий в Git. Применяет правки, запускает тесты, автоматически коммитит. Model-agnostic — полный контроль над стоимостью.

**Примеры:** TDD (создание failing-теста → код для прохождения), исправление багов с валидацией, обновление зависимостей.

## GitHub Copilot CLI

Расширение AI pair programmer в терминал с глубокой интеграцией GitHub. Может быть назначен на issue, работать над фиксом и создавать pull request.

**Примеры:** Автоматическое решение issue (#123 → branch → код → PR), repository-aware Q&A, помощник shell-команд.

## Terminal-Bench

Фреймворк оценки AI-агентов в CLI. 80 задач из научных и аналитических доменов. Terminus — minimalistic агент-тестовая среда для сравнения моделей.

## Заключение

Появление мощных AI CLI-агентов фундаментально меняет разработку, превращая терминал в динамичную совместную среду. Нет единого «лучшего» инструмента: Claude для архитектурных задач, Gemini для мультимодальных, Aider для git-центричной работы, Copilot для GitHub-интеграции.

## Ссылки

1. Claude CLI: [https://docs.anthropic.com/en/docs/claude-code/cli-reference](https://docs.anthropic.com/en/docs/claude-code/cli-reference)
2. Gemini CLI: [https://github.com/google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli)
3. Aider: [https://aider.chat/](https://aider.chat/)
4. GitHub Copilot CLI: [https://docs.github.com/en/copilot/github-copilot-enterprise/copilot-cli](https://docs.github.com/en/copilot/github-copilot-enterprise/copilot-cli)
5. Terminal-Bench: [https://www.tbench.ai/](https://www.tbench.ai/)
