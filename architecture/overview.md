# Architecture Overview

## High-level

```text
Next.js frontend
      ↓ REST API / Bearer JWT
Django + DRF backend
   ↓          ↓
PostgreSQL   Redis (cache)
   ↓
Object storage for media
```

Django Admin используется как внутренний CMS для наполнения, импорта и проверки контента.

## Authentication

Поддерживаются два способа аутентификации:
1. Telegram — основной пользовательский сценарий.
2. Логин + пароль — резервный способ входа.

Один User может иметь оба способа входа.

JWT:
- access + refresh;
- access передаётся как `Authorization: Bearer <token>`;
- refresh хранится в httpOnly cookie;
- нужен blacklist/revocation refresh tokens.

API prefix: `/api/v1/`.

## Principles

- Backend и frontend разделены.
- PostgreSQL — основной источник данных.
- Redis используется для кэша.
- Контент хранится статически после генерации/проверки.
- AI/NotebookLM не участвует в пользовательском runtime.
- Roles и subscriptions — разные подсистемы.
- User-facing URLs используют slug; внутренние связи — numeric IDs/FKs.
- Основной формат текста — Markdown.
- Файлы/изображения — object storage + отдельная Media/File model.
- Архитектура поддерживает несколько предметов с самого начала.

## Content flow

```text
Источники
   ↓
Gemini Notebook / другой AI-инструмент
   ↓
нормализованный JSON
   ↓
admin import / upload flow
   ↓
manual review при необходимости
   ↓
PostgreSQL
   ↓
Django API
   ↓
Next.js
```

Статусы: `draft`, `generated`, `reviewed`, `published`.
Manual content может сразу публиковаться как `published`.

## Search

MVP: Django/PostgreSQL `icontains` по `Subject.title`, `Section.title`, `Topic.title`, плюс фильтры по предмету/разделу. Поиск по ContentBlock закладывается на будущее. Elasticsearch/OpenSearch не нужен.

## Runtime / infrastructure

Current:
- PostgreSQL;
- Redis cache;
- object storage;
- Docker;
- Render для backend как текущий план;
- Vercel для frontend.

TBD / deferred:
- Celery;
- Kafka/RabbitMQ;
- Kubernetes;
- microservices;
- Elasticsearch/OpenSearch;
- FSRS;
- Sentry.

## Source of truth

Полный список согласованных и TBD решений: `../decisions/0004-mvp-decision-register.md`.
