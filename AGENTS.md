# AGENTS.md

Этот репозиторий — источник продуктового и архитектурного контекста MedCore для людей и AI-агентов.

## New feature repository preflight

Перед началом новой фичи, до создания feature-ветки и до изменений в коде,
синхронизируй репозитории MedCore на выбранных базовых ветках (обычно `main`).

Перед pull проверь `git status --short --branch` в каждом репозитории. Если
рабочее дерево не чистое, upstream отсутствует или ветки разошлись, остановись
и сообщи блокер владельцу. Не применяй для продолжения `stash`, `reset`,
`checkout`, merge или rebase автоматически. Начинать feature-работу можно
только после успешного fast-forward pull нужных репозиториев; ветки и commit
HEAD зафиксируй в отчёте задачи.

Основные репозитории:

- medcore-docs
- medcore-backend
- medcore-frontend
- medcore-content

## Before making meaningful changes

AI-агент должен:

1. Прочитать `ai/context.md`.
2. Прочитать `decisions/0004-mvp-decision-register.md`.
3. Прочитать релевантные файлы из `product/`, `architecture/`, `content/`.
4. Проверить остальные `decisions/`, чтобы не переизобретать уже принятое решение.

Если другой документ противоречит более новому ADR/decision register, приоритет имеет более новое явно принятое решение.

`TBD` означает, что решение не принято. AI не должен самостоятельно выбирать вариант и выдавать его за согласованную архитектуру.

## Architecture guardrails

Не менять молча принятые архитектурные решения.

Если изменение затрагивает архитектуру, модель данных, auth, публичный API, контентную модель или инфраструктуру:

- обновить релевантную документацию;
- при существенном решении создать новый ADR в `decisions/`;
- обновить `ai/context.md`, если изменился текущий источник истины.

## MVP-first rule

Предпочитать минимальное решение, достаточное для текущего MVP.

Не добавлять без конкретной необходимости:

- Celery;
- Kafka/RabbitMQ;
- Kubernetes;
- микросервисы;
- Elasticsearch/OpenSearch;
- отдельный search-service;
- сложную event-аналитику;
- FSRS.

Redis является уже согласованной частью инфраструктуры для кэша и не требует нового ADR сам по себе. Расширять его использование на очереди/фоновые задачи без отдельного решения нельзя.

## Current fixed decisions

- Product/public brand: PrepMed; project/repository family: MedCore.
- Public origin: `https://prepmed.ru`.
- Backend: Django + DRF.
- Frontend: Next.js.
- Database: PostgreSQL.
- Cache: Redis.
- Python: 3.12.
- Package manager: uv.
- Lint/format: Ruff / Ruff format.
- Tests: pytest.
- API prefix: `/api/v1/`.
- API authentication: Bearer JWT.
- Access + refresh JWT; refresh хранится в httpOnly cookie.
- Refresh token blacklist/revocation нужен.
- Auth method in the current MVP: verified Telegram Login Widget.
- `users.User` stores unique `telegram_id` directly and a mutable unique
  application `username`; no separate Telegram identity table is required.
- Password/email registration, identity linking and Telegram unlink are deferred.
- Text content: Markdown.
- Content hierarchy: `Subject → Section → Topic`.
- Topic content entities are optional.
- Flashcards and quizzes are отдельными domain entities.
- Canonical public content URLs: `/{subject_slug}` and
  `/{subject_slug}/{topic_slug}`; DB relations use numeric IDs/FKs.
- Stable Latin slugs are generated for public URLs and are not automatically
  rewritten on ordinary title edits.
- Search MVP: Django/PostgreSQL `icontains` по Subject/Section/Topic titles.
- Roles and paid subscriptions are separate concepts. `premium` is not a role.
- `Plan`, `Subscription` and paid entitlement are implemented.
- Trial is deferred; current MVP has no trial runtime.
- Robokassa is the payment provider for one-time manual period purchases.
- No recurring auto-debit in current MVP.
- FSRS deferred.
- Content physically deleted in MVP.
- Media/File storage boundary is implemented; current production storage is a
  persistent local Docker volume, object storage can be introduced later.
- Django Admin — CMS.
- External content writes use synchronous REST CRUD endpoints with a separate
  integration key; no ImportJob or Celery worker is required for this flow.
- Content access model: Topic-level `demo / free / paid`; public catalog metadata разрешён, `free` требует auth, `paid` требует auth + entitlement, а отдельные child elements могут быть demo-preview.
- Backend enforces paywall.
- Dark mode и mobile-first — сразу.
- OpenAPI/Swagger — сразу.
- Healthcheck: `/api/v1/health/`; readiness: `/api/v1/health/ready/`.
- Production: Caddy + Docker Compose on one VPS, frontend/backend same origin.
- Deploy: successful main CI → exact-SHA SSH deploy; backend/frontend deploy
  independently and server-side lock serializes concurrent deploys.

Базовый реестр решений: `decisions/0004-mvp-decision-register.md`. Более новые
ADR имеют приоритет; для anonymous demo access см.
`decisions/0006-anonymous-demo-content.md`, для access model —
`decisions/0007-content-access-model.md`, для уточнённого MVP-контракта —
`decisions/0008-mvp-auth-content-and-integration-contract.md`, для
subscription semantics — `decisions/0009-subscriptions-core-and-paid-entitlement.md`.

## Content safety

AI-generated medical content is not automatically trusted. Система должна поддерживать статусы `draft`, `generated`, `reviewed`, `published`, но на MVP допускается ручная публикация сразу как `published`.

Sources optional и могут связываться с Topic, ContentBlock, Flashcard и QuizQuestion. Не делать sources обязательными без нового решения.
