# AGENTS.md

Этот репозиторий — источник продуктового и архитектурного контекста MedCore для людей и AI-агентов.

## Before making meaningful changes

AI-агент должен:

1. Прочитать `ai/context.md`.
2. Прочитать релевантные файлы из `product/`, `architecture/`, `content/`.
3. Проверить `decisions/`, чтобы не переизобретать уже принятое решение.

## Architecture guardrails

Не менять молча принятые архитектурные решения.

Если изменение затрагивает архитектуру, модель данных, auth, публичный API, контентную модель или инфраструктуру:

- обновить релевантную документацию;
- при существенном решении создать новый ADR в `decisions/`;
- обновить `ai/context.md`, если изменился текущий источник истины.

## MVP-first rule

Предпочитать минимальное решение, достаточное для текущего MVP.

Не добавлять без конкретной необходимости:

- Redis;
- Celery;
- Kafka/RabbitMQ;
- Kubernetes;
- микросервисы;
- Elasticsearch/OpenSearch;
- отдельный search-service;
- сложную event-аналитику;
- FSRS.

Новая инфраструктура или крупная зависимость должна решать конкретную существующую проблему, а не гипотетическую будущую.

## Current fixed decisions

- Backend: Django + DRF.
- Frontend: Next.js.
- Database: PostgreSQL.
- API prefix: `/api/v1/`.
- API authentication: Bearer JWT.
- Auth methods: Telegram + login/password.
- Telegram является основным пользовательским auth flow, login/password — резервным.
- Text content: Markdown.
- Content hierarchy: `Subject → Section → Topic`.
- Topic content entities are optional.
- Flashcards and quizzes are separate domain entities, not generic ContentBlock.
- Public content URLs use slugs; DB relations use normal IDs/FKs.
- Search MVP: Django/PostgreSQL `icontains`.
- Roles and paid subscriptions are separate concepts. `premium` is not a role.
- FSRS is deferred; do not introduce it into MVP without a new decision.
- Product event analytics is deferred for MVP.
- Content is physically deleted in MVP; FK deletion behavior must be explicit.

## Content safety

AI-generated medical content is not automatically trusted. The system must allow human review and publishing states.

Sources are optional but supported for Topic, ContentBlock, Flashcard and QuizQuestion. Do not make sources mandatory at database level without a new ADR.
