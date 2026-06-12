# CLI-агент Research Agent: архитектура и создание

> **Дата:** 2026-06-12  
> **Версия:** 1.0  
> **Платформа:** OpenCode CLI

---

## 1. Цель документа

Описать процесс создания CLI-агента `researcher`, его базовую структуру, файлы конфигурации, систему разрешений и подключение MCP-сервера Playwright для веб-скрапинга.

---

## 2. Общая архитектура

```
research-agent/
├── opencode.json                 # Проектные настройки и конфигурация
├── agents.md                     # Инструкции верхнего уровня
├── research/                     # Папка для результатов исследований
│   └── cli-agent-creation.md     # ← данный файл
└── .opencode/                    # Системная директория OpenCode
    ├── .gitignore
    ├── package.json
    ├── package-lock.json
    ├── agents/
    │   └── researcher.md         # Описание агента, роль, правила
    ├── skills/                   # Навыки (расширения возможностей)
    └── node_modules/             # Зависимости
```

### 2.1. Компоненты системы

| Компонент | Назначение |
|-----------|------------|
| **opencode.json** | Центральный файл конфигурации проекта: агенты, MCP, разрешения |
| **agents.md** | Инструкции верхнего уровня, общие для всех агентов |
| **.opencode/agents/researcher.md** | Описание агента: роль, правила безопасности, нейминг |
| **.opencode/skills/** | Директория для дополнительных навыков агента |
| **research/** | Выходная директория для отчётов, данных и артефактов |
| **MCP Playwright** | Сервер для веб-скрапинга и браузерной автоматизации |

---

## 3. Файл проектных настроек: `opencode.json`

### 3.1. Структура

```json
{
  "$schema": "https://opencode.ai/config.json",
  "default_agent": "researcher",
  "instructions": ["agents.md"],
  "agent": {
    "researcher": {
      "description": "Агент для исследований и анализа информации",
      "mode": "primary",
      "permission": {
        "edit": "allow",
        "bash": "ask",
        "read": "allow",
        "websearch": "allow",
        "webfetch": "allow"
      }
    }
  },
  "mcp": {
    "playwright": {
      "type": "local",
      "command": ["npx", "-y", "@playwright/mcp"],
      "enabled": true
    }
  },
  "skills": {
    "paths": [".opencode/skills"]
  }
}
```

### 3.2. Описание полей

| Поле | Значение | Описание |
|------|----------|----------|
| `$schema` | URL JSON Schema | Ссылка на схему валидации конфига OpenCode |
| `default_agent` | `"researcher"` | Агент, используемый по умолчанию |
| `instructions` | `["agents.md"]` | Файлы с инструкциями, загружаемые в систему |
| `agent.researcher` | объект | Конфигурация агента-исследователя |
| `agent.researcher.mode` | `"primary"` | Режим: primary (основной) или secondary |
| `agent.researcher.permission` | объект | **Базовая структура разрешений** (см. раздел 5) |
| `mcp.playwright` | объект | Подключение MCP-сервера Playwright |
| `mcp.playwright.type` | `"local"` | Тип запуска: local (локальный процесс) |
| `mcp.playwright.command` | массив | Команда для запуска MCP-сервера |
| `mcp.playwright.enabled` | `true` | Флаг включения MCP |
| `skills.paths` | `[".opencode/skills"]` | Пути к директориям с навыками |

---

## 4. Файл агента: `.opencode/agents/researcher.md`

### 4.1. Frontmatter (YAML)

```yaml
---
description: Проводит исследования, анализирует информацию и готовит отчёты.
mode: primary
permission:
  edit: allow
  bash: ask
  read: allow
  websearch: allow
  webfetch: allow
---
```

### 4.2. Содержание

Файл включает следующие обязательные разделы:

1. **Роль агента** — описание обязанностей, ключевых принципов и зоны ответственности.
2. **Правила безопасности** — политики исполнения команд, ограничения, работа с внешними источниками.
3. **Правило нейминга файлов** — шаблоны именования для отчётов, данных, конфигурации и временных файлов.
4. **Правила работы** — конкретные инструкции по поиску, структурированию, сохранению и валидации результатов.

---

## 5. Базовая структура разрешений

Права доступа определяются на двух уровнях:

### 5.1. В `opencode.json` (глобальные для агента)

| Инструмент | Значение | Пояснение |
|------------|----------|-----------|
| `edit` | `allow` | Разрешено редактирование файлов |
| `bash` | `ask` | Shell-команды требуют подтверждения |
| `read` | `allow` | Чтение файлов разрешено |
| `websearch` | `allow` | Веб-поиск разрешён |
| `webfetch` | `allow` | Запросы к URL разрешены |

### 5.2. В `researcher.md` (детализация политик)

```
permission:
  edit: allow    # Редактирование в рабочей директории
  bash: ask      # Shell — только с подтверждения пользователя
  read: allow    # Чтение любых файлов проекта
  websearch: allow   # Поиск в интернете
  webfetch: allow    # Извлечение контента сайтов
```

### 5.3. Матрица разрешений

| Инструмент | Без подтверждения | С подтверждением | Запрещено |
|------------|:-:|:-:|:-:|
| `read` | ✅ | — | — |
| `edit` | ✅ | — | — |
| `websearch` | ✅ | — | — |
| `webfetch` | ✅ | — | — |
| `bash` | — | ⚠️ | — |
| Запись вне проекта | — | — | 🚫 |
| Раскрытие секретов | — | — | 🚫 |

---

## 6. MCP-подключение (Playwright)

### 6.1. Что такое MCP

**Model Context Protocol (MCP)** — протокол, позволяющий агентам взаимодействовать с внешними инструментами и сервисами. В проекте используется MCP-сервер **Playwright** для браузерной автоматизации.

### 6.2. Конфигурация в `opencode.json`

```json
"mcp": {
  "playwright": {
    "type": "local",
    "command": ["npx", "-y", "@playwright/mcp"],
    "enabled": true
  }
}
```

### 6.3. Доступные инструменты Playwright MCP

После подключения сервера агенту становятся доступны следующие инструменты (все через `playwright_browser_*`):

| Инструмент | Описание |
|------------|----------|
| `playwright_browser_navigate` | Переход по URL |
| `playwright_browser_snapshot` | Снимок доступности страницы (accessibility tree) |
| `playwright_browser_click` | Клик по элементу |
| `playwright_browser_type` | Ввод текста в поле |
| `playwright_browser_fill_form` | Заполнение форм |
| `playwright_browser_select_option` | Выбор опции из выпадающего списка |
| `playwright_browser_hover` | Наведение на элемент |
| `playwright_browser_take_screenshot` | Скриншот страницы или элемента |
| `playwright_browser_evaluate` | Выполнение JavaScript на странице |
| `playwright_browser_network_requests` | Просмотр сетевых запросов |
| `playwright_browser_network_request` | Детали конкретного запроса |
| `playwright_browser_console_messages` | Консольные сообщения браузера |
| `playwright_browser_press_key` | Нажатие клавиши |
| `playwright_browser_drag` | Drag-and-drop между элементами |
| `playwright_browser_drop` | Сброс файлов на элемент |
| `playwright_browser_file_upload` | Загрузка файлов |
| `playwright_browser_handle_dialog` | Работа с диалогами (alert, confirm, prompt) |
| `playwright_browser_resize` | Изменение размера окна |
| `playwright_browser_tabs` | Управление вкладками |
| `playwright_browser_wait_for` | Ожидание текста или времени |
| `playwright_browser_navigate_back` | Назад в истории |
| `playwright_browser_close` | Закрытие страницы |
| `playwright_browser_run_code_unsafe` | Выполнение произвольного Playwright-кода |

### 6.4. Пример использования MCP

```
1. Открыть страницу:  playwright_browser_navigate("https://example.com")
2. Получить содержимое: playwright_browser_snapshot()
3. Найти данные: playwright_browser_evaluate("() => document.title")
4. Сохранить результат: записать в файл research/report.md
```

---

## 7. Созданные файлы (сводка)

| № | Файл | Назначение |
|:-:|------|------------|
| 1 | `opencode.json` | Проектные настройки: агент, MCP, разрешения, навыки |
| 2 | `agents.md` | Инструкции верхнего уровня для агентов |
| 3 | `.opencode/agents/researcher.md` | Описание агента: роль, безопасность, нейминг, правила работы |
| 4 | `.opencode/package.json` | Зависимости OpenCode (`@opencode-ai/plugin`) |
| 5 | `.opencode/.gitignore` | Исключение node_modules из репозитория |
| 6 | `.opencode/skills/` | Директория для дополнительных навыков (пустая, готова к использованию) |
| 7 | `research/` | Директория для результатов исследований и отчётов |

---

## 8. Полученная структура агента

```
research-agent/
├── opencode.json ◄──── файл проектных настроек
│   ├── agent.researcher.permission ◄── базовая структура разрешений
│   ├── mcp.playwright ◄─────────────── подключение MCP-сервера
│   └── skills.paths ◄───────────────── путь к навыкам
│
├── agents.md ◄─────── общие инструкции
│
├── .opencode/agents/
│   └── researcher.md ◄── полное описание агента
│       ├── 1. Роль агента
│       ├── 2. Правила безопасности
│       ├── 3. Правило нейминга файлов
│       └── 4. Правила работы
│
├── research/ ◄────────── директория артефактов
│   └── cli-agent-creation.md ◄── данный файл
│
└── .opencode/skills/ ◄── расширения (навыки)
```

---

## 9. Жизненный цикл запроса

```
Пользователь
    │
    ▼
agents.md (инструкции верхнего уровня)
    │
    ▼
opencode.json (выбор агента, проверка разрешений)
    │
    ▼
.opencode/agents/researcher.md (роль, безопасность, правила)
    │
    ▼
MCP Playwright (браузер, веб-скрапинг)
    │
    ▼
research/ (сохранение результатов)
```

---

## 10. Заключение

Созданная архитектура CLI-агента `researcher` обеспечивает:

- **Модульность** — разделение конфигурации, инструкций и артефактов.
- **Безопасность** — гибкая система разрешений (allow / ask) на каждый инструмент.
- **Расширяемость** — поддержка MCP-серверов и навыков (skills).
- **Документированность** — чёткие правила нейминга и структурирования результатов.
- **Автономность** — агент может самостоятельно выполнять веб-поиск, извлекать и анализировать данные.

Все файлы находятся в директории `research-agent/` и готовы к использованию.
