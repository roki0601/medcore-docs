# Architecture Overview

## High-level

```text
Next.js frontend
      ↓ REST API / Bearer JWT
Django + DRF backend
      ↓
PostgreSQL
```

Django Admin используется как внутренний CMS для наполнения и проверки контента.

## Authentication

Поддерживаются два способа аутентификации:

1. Telegram — основной пользовательский сценарий.
2. Логин + пароль — резервный способ входа.

После успешной аутентификации backend выдает JWT. Frontend обращается к API через:

```text
Authorization: Bearer <access_token>
```

API версионируется с первого дня:

```text
/api/v1/...
```

Telegram-аутентификация должна проверяться на backend. Frontend не является доверенной стороной для подтверждения Telegram identity.

## Principles

- Backend и frontend разделены.
- PostgreSQL — основной источник данных.
- Контент хранится статически после генерации и проверки.
- AI не нужен для большинства пользовательских запросов во время работы сервиса.
- Архитектура должна оставаться простой до появления реальной нагрузки.
- Роли пользователя и коммерческий доступ через подписку — разные подсистемы.
- Публичные URL учебных сущностей используют slug, внутренние связи — обычные primary key/FK.
- Основной формат текстового учебного контента — Markdown, а не HTML.

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

Статусы контента закладываются заранее:

```text
draft → generated → reviewed → published
```

При этом первое время контент может загружаться сразу как `published`, если он уже проверен вручную вне системы.

## Search

Поиск первого MVP — простой поиск средствами Django/PostgreSQL через `icontains`.

Начальный scope:
- Subject.title
- Section.title
- Topic.title

Полнотекстовый поиск, Elasticsearch/OpenSearch и отдельный search-service пока не нужны.

## Files

Текстовые данные и metadata хранятся в PostgreSQL.

Изображения и файлы при необходимости хранятся в object storage, а в БД сохраняются URL и metadata.

## Analytics

Продуктовая event-аналитика полезна в будущем, но не является обязательной частью первого MVP. Техническое логирование backend-ошибок остается обычной эксплуатационной необходимостью.

## Deferred infrastructure

Для первого MVP не нужны:
- Celery;
- Redis;
- Kafka/RabbitMQ;
- Kubernetes;
- микросервисы;
- Elasticsearch/OpenSearch;
- FSRS.

Если генерация/импорт позже станет тяжелым процессом, фоновые задачи добавляются отдельно. FSRS можно добавить позже поверх сохраненной истории повторений карточек.
