# Architecture Overview

## High-level

```text
Next.js frontend
      ↓ REST API
Django + DRF backend
      ↓
PostgreSQL
```

Django Admin используется как внутренний CMS для наполнения и проверки контента.

## Principles

- Backend и frontend разделены.
- PostgreSQL — основной источник данных.
- Контент хранится статически после генерации и проверки.
- AI не нужен для большинства пользовательских запросов во время работы сервиса.
- Архитектура должна оставаться простой до появления реальной нагрузки.

## Content flow

```text
Источники
   ↓
Gemini Notebook / другой AI-инструмент
   ↓
теория / карточки / тесты
   ↓
импорт
   ↓
ручная проверка
   ↓
PostgreSQL
   ↓
Django API
   ↓
Next.js
```

## Deferred infrastructure

Для первого MVP не нужны:
- Celery;
- Redis;
- Kafka/RabbitMQ;
- Kubernetes;
- микросервисы.

Если генерация/импорт позже станет тяжелым процессом, фоновые задачи добавляются отдельно.
