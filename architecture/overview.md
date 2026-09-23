# Architecture Overview

## High-level

```text
Internet
   ↓ HTTPS
Caddy on VPS
   ↓
Next.js frontend (:3000, localhost only)
   ↓ same-origin /api/v1/*
Django + DRF backend (:8000, localhost only)
   ↓                 ↓
PostgreSQL          Redis
   ↓
Media/File storage boundary
(current prod: persistent local Docker volume)
```

Django Admin используется как внутренний CMS для наполнения и проверки контента.

## Authentication

Текущий MVP поддерживает один пользовательский способ регистрации и входа:
verified Telegram Login Widget.

Password/email auth, identity linking и Telegram unlink отложены.

JWT:
- access + refresh;
- access передаётся как `Authorization: Bearer <token>`;
- refresh хранится в httpOnly cookie;
- refresh sessions имеют server-side revocation/blacklist state.

API prefix: `/api/v1/`.

## Principles

- Backend и frontend разделены кодом, но в production работают под одним origin.
- PostgreSQL — основной источник данных.
- Redis используется для кэша.
- AI не участвует в пользовательском runtime.
- Roles и subscriptions — разные подсистемы.
- User-facing public content URLs используют stable Latin slugs; внутренние связи — numeric IDs/FKs.
- Основной формат текста — Markdown.
- Файлы/изображения проходят через отдельную Media/File boundary.
- Current production media storage — локальный persistent Docker volume; переход
  на object storage можно сделать позже за этой boundary.
- Архитектура поддерживает несколько предметов с самого начала.

## Content flow

```text
Источники
   ↓
medcore-content / AI-assisted generation
   ↓
normalized requests
   ↓
synchronous Content REST API / Django Admin
   ↓
review / publish
   ↓
PostgreSQL + Media storage
   ↓
Django API
   ↓
Next.js
```

Статусы: `draft`, `generated`, `reviewed`, `published`.
Manual content может сразу публиковаться как `published`.

## Access and payments

Topic остаётся главным access boundary:

- `demo` — anonymous allowed;
- `free` — authentication required;
- `paid` — authentication + active entitlement.

Entitlement вычисляется backend через subscriptions policy. `Plan` и
`Subscription` реализованы. Trial в текущем MVP не реализован.

Платёжный провайдер — Robokassa:

- разовые покупки периода тарифа;
- без recurring auto-debit;
- ResultURL проверяется подписью;
- успешный callback создаёт/связывает Subscription идемпотентно;
- финансовый Payment audit сохраняется независимо от удаления user reference.

## Public URLs / SEO

Canonical public pages:

```text
/{subject_slug}
/{subject_slug}/{topic_slug}
```

Frontend генерирует:

- canonical metadata;
- per-page title/description;
- Open Graph metadata;
- `robots.txt`;
- dynamic `sitemap.xml` с published subject/topic URLs;
- BreadcrumbList and page structured data.

Legacy numeric catalog routes не являются canonical public URLs и редиректят на
slug URL where supported.

## Search

MVP: Django/PostgreSQL `icontains` по `Subject.title`, `Section.title`,
`Topic.title`, плюс фильтры по предмету/разделу. Поиск по ContentBlock
закладывается на будущее. Elasticsearch/OpenSearch не нужен.

## Runtime / infrastructure

Current production:

- one VPS;
- host Caddy terminates HTTPS and reverse-proxies the app;
- Docker Compose services: frontend, backend, PostgreSQL, Redis;
- PostgreSQL and Redis do not publish public host ports;
- media stored in named persistent Docker volume;
- frontend/backend healthchecks;
- readiness endpoint: `/api/v1/health/ready/`;
- GitHub Actions runs CI on `main`, then deploys the exact successful commit SHA over SSH;
- backend and frontend can deploy independently;
- server-side `flock` serializes deploys;
- production deploy does not use `docker compose down` and does not restart Caddy unnecessarily.

Deferred / not required for current MVP:

- Celery;
- Kafka/RabbitMQ;
- Kubernetes;
- microservices;
- Elasticsearch/OpenSearch;
- FSRS;
- Sentry;
- managed PostgreSQL migration;
- object-storage migration.

## Source of truth

Полный список согласованных и TBD решений: `../decisions/0004-mvp-decision-register.md`.
Более новые ADR имеют приоритет, особенно ADR 0008 и ADR 0009.
