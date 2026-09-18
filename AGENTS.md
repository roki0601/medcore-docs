# AGENTS.md

Этот репозиторий — источник продуктового и архитектурного контекста MedCore для людей и AI-агентов.

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
- Public content URLs use slugs; DB relations use numeric IDs/FKs.
- Search MVP: Django/PostgreSQL `icontains` по Subject/Section/Topic titles.
- Roles and paid subscriptions are separate concepts. `premium` is not a role.
- Subscription model создаётся сразу и поддерживает несколько тарифов/trial.
- FSRS deferred; только задел в данных.
- Product event analytics deferred for MVP.
- Content physically deleted in MVP.
- Object storage для файлов; отдельная Media/File model.
- Django Admin — CMS; bulk import/publish нужны, детали позже.
- External content writes use synchronous REST CRUD endpoints with a separate
  integration key; no ImportJob or Celery worker is required for this flow.
- Content access model: Topic-level `demo / free / paid`; public catalog metadata разрешён, `free` требует auth, `paid` требует auth + entitlement, а отдельные child elements могут быть demo-preview.
- Backend enforces paywall.
- Dark mode и mobile-first — сразу.
- OpenAPI/Swagger — сразу.
- Healthcheck: `/api/v1/health/`.

Базовый реестр решений: `decisions/0004-mvp-decision-register.md`. Более новые
ADR имеют приоритет; для anonymous demo access см. `decisions/0006-anonymous-demo-content.md`, для access model — `decisions/0007-content-access-model.md`, для уточнённого MVP-контракта — `decisions/0008-mvp-auth-content-and-integration-contract.md`.

## Content safety

AI-generated medical content is not automatically trusted. Система должна поддерживать статусы `draft`, `generated`, `reviewed`, `published`, но на MVP допускается ручная публикация сразу как `published`.

Sources optional и могут связываться с Topic, ContentBlock, Flashcard и QuizQuestion. Не делать sources обязательными без нового решения.
