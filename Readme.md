# 📚 EMH Documentation

> 🏥 Централизованное хранилище документации для отдела АСУ **Больницы Скорой Медицинской Помощи (EMH)**

---

## 🎯 Назначение

Этот репозиторий содержит **всю текстовую документацию** для проектов EMH:
- 📖 Техническая документация
- 📋 Руководства пользователя
- 🏗️ Архитектурные решения
- 📊 Бизнес-требования
- 🔧 Инструкции по развертыванию
- 📝 Протоколы встреч и решения
- 💻 Реестр оборудования

---

## 📑 Структура документации

```text
EMH.Documentation/
├── 📁 Architecture/           # Архитектурные решения и диаграммы
│   ├── system-overview.md
│   ├── database-schema.md
│   └── api-design.md
│
├── 📁 Business/               # Бизнес-требования и спецификации
│   ├── requirements/
│   ├── user-stories/
│   └── glossary.md
│
├── 📁 Development/            # Руководства для разработчиков
│   ├── setup-guide.md
│   ├── coding-standards.md
│   ├── git-workflow.md
│   └── code-review-checklist.md
│
├── 📁 Deployment/             # Инструкции по развертыванию
│   ├── local-setup.md
│   ├── docker-deployment.md
│   ├── ci-cd-pipeline.md
│   └── environment-config.md
│
├── 📁 Operations/             # Эксплуатация и мониторинг
│   ├── monitoring.md
│   ├── backup-recovery.md
│   ├── troubleshooting.md
│   └── incident-response.md
│
├── 📁 UserGuides/             # Руководства для пользователей
│   ├── employee-portal/
│   ├── admin-panel/
│   └── faq.md
│
├── 📁 Meetings/               # Протоколы встреч и решения
│   ├── 2026/
│   │   ├── 01-january/
│   │   └── 02-february/
│   └── decisions/
│
├── 📁 Templates/              # Шаблоны документов
│   ├── meeting-template.md
│   ├── adr-template.md
│   └── bug-report-template.md
│
└── 📄 README.md               # Этот файл
```
---

## 🚀 Быстрый старт

### Для новых сотрудников

1. 📖 Прочитайте [Обзор системы](./Architecture/system-overview.md)
2. 🛠️ Настройте окружение: [Руководство по установке](./Development/setup-guide.md)
3. 📋 Изучите [Бизнес-требования](./Business/requirements/)
4. 🔍 Ознакомьтесь с [Git Workflow](./Development/git-workflow.md)

### Для разработчиков

- 📐 [Стандарты кодирования](./Development/coding-standards.md)
- 🔄 [Процесс Code Review](./Development/code-review-checklist.md)
- 🗄️ [Схема базы данных](./Architecture/database-schema.md)
- 🔌 [API документация](./Architecture/api-design.md)

### Для DevOps/Системных администраторов

- 🐳 [Docker развертывание](./Deployment/docker-deployment.md)
- 🔄 [CI/CD Pipeline](./Deployment/ci-cd-pipeline.md)
- 📊 [Мониторинг](./Operations/monitoring.md)
- 💾 [Backup & Recovery](./Operations/backup-recovery.md)

---

## 📝 Правила ведения документации

### 📌 Именование файлов

- Используйте **kebab-case**: `user-guide.md`, `api-design.md`
- Добавляйте префикс даты для хронологических документов: `2026-01-15-meeting-notes.md`
- Используйте понятные имена: не `doc1.md`, а `database-migration-guide.md`

### 📌 Структура документа

Каждый документ должен содержать:

```markdown
# Название документа

> Краткое описание (1-2 предложения)

**Автор:** Имя Фамилия  
**Дата создания:** YYYY-MM-DD  
**Последнее обновление:** YYYY-MM-DD  
**Статус:** Draft | Active | Archived

---

## Содержание

[Автоматически генерируемое оглавление]

---

## Основной контент

...
```

### 📌 Статусы документов

| Статус | Описание |
|--------|----------|
| 🟡 **Draft** | В разработке, требует доработки |
| 🟢 **Active** | Актуальная версия, используется в работе |
| 🔵 **Review** | На рассмотрении, ожидает утверждения |
| ⚫ **Archived** | Устарел, сохранен для истории |
| 🔴 **Deprecated** | Не рекомендуется к использованию |

---

## 🔄 Процесс обновления документации

### Создание нового документа

1. 📋 Выберите подходящий шаблон из `Templates/`
2. ✍️ Создайте документ в соответствующей папке
3. 🏷️ Укажите метаданные (автор, дата, статус)
4. 🔗 Добавьте ссылку в этот README (если документ важный)
5. 💾 Закоммитьте с понятным сообщением: `docs: add database schema documentation`

### Обновление существующего документа

1. 📖 Проверьте актуальность информации
2. ✏️ Внесите изменения
3. 📅 Обновите дату в метаданных
4. 💾 Закоммитьте: `docs: update API endpoints section`

### Архивация документов

1. 🏷️ Измените статус на **Archived**
2. 📦 Переместите в `Archive/` (если нужно)
3. 💾 Закоммитьте: `docs: archive outdated deployment guide`

---

## 🏗️ Архитектурные решения (ADR)

Для фиксации важных технических решений используйте [ADR Template](./Templates/adr-template.md):

```markdown
# ADR-001: Использование PostgreSQL вместо SQL Server

**Статус:** Active  
**Дата:** 2026-01-15

## Контекст
[Описание проблемы или ситуации]

## Решение
[Какое решение принято]

## Обоснование
[Почему именно это решение]

## Последствия
[Что это означает для проекта]
```

---


## 🔗 Полезные ссылки

- 📦 [Основной репозиторий EMH](https://github.com/your-org/emh)
- 🛠️ [DevTools документация](https://github.com/your-org/emh/tree/main/devtools)
- 🐛 [Баг-трекер](https://github.com/your-org/emh/issues)

---
