# AI Context

## Project

MedCore / PrepMed — web-сервис для подготовки студентов-медиков к экзаменам.

## Source of truth

Базовый подробный реестр согласованных решений находится в:

`../decisions/0004-mvp-decision-register.md`

Более новые ADR в `../decisions/` имеют приоритет над конфликтующим старым
текстом. ADR 0006 разрешает anonymous demo scope, ADR 0007 фиксирует
Topic-level access model `demo / free / paid`, ADR 0008 — Telegram-only auth,
synchronous Content API и one-origin self-hosted production, ADR 0009 —
семантику подписок и paid entitlement.

AI-агент обязан прочитать decision register и более новые ADR перед
существенными изменениями. Пункты, помеченные `TBD`, нельзя самостоятельно
превращать в принятые решения.

## Current implementation snapshot (2026-09-23)

This section is a factual delivery snapshot, not a replacement for accepted
product decisions.

Backend `main` already contains:

- Telegram auth/account lifecycle;
- public catalog and Topic access `demo/free/paid`;
- synchronous content authoring API and media access/storage boundary;
- topic/theory progress, flashcard review and resumable quiz attempts;
- `Plan`, `Subscription` and working paid entitlement policy;
- Robokassa one-time payments with signed ResultURL handling and retained Payment audit records;
- dashboard progress aggregates and profile/study preferences;
- stable public Latin slugs plus public subject/topic lookup and sitemap payload;
- OpenAPI/runtime contract alignment;
- production Docker Compose, exact-SHA post-CI deploy and health checks.

Frontend `main` already contains the MVP product surface, public legal/payment
UX, production Docker image, public canonical subject/topic pages at
`/{subject_slug}` and `/{subject_slug}/{topic_slug}`, dynamic
`/sitemap.xml`, `robots.txt`, canonical metadata, breadcrumbs/JSON-LD,
server-rendered homepage discovery links, reliable post-CI deployment and
affected-test CI optimization.

Production baseline:

- public origin: `https://prepmed.ru`;
- frontend and backend share one origin; API is under `/api/v1/`;
- host reverse proxy: Caddy;
- runtime: Docker Compose on the project VPS;
- PostgreSQL and Redis are private to the compose network;
- production media currently uses a persistent local Docker volume through the
  existing Media/File boundary; migration to object storage remains possible later;
- latest backend and frontend main CI/deploy runs on 2026-09-23 are green.

## Current MVP idea

Новый пользователь может до авторизации посмотреть ограниченный
демонстрационный учебный контент. Для полноценного использования он
регистрируется или входит в систему, выбирает один из доступных предметов,
открывает раздел и тему, изучает материал, проходит карточки и тесты и видит
прогресс.

Архитектура с самого начала допускает несколько предметов.

## Core hierarchy

`Subject → Section → Topic`

У темы могут быть необязательные сущности:

- ContentBlock
- Flashcard
- Quiz
- Source

Контент можно дополнять постепенно; отсутствие отдельных блоков не мешает существованию темы.

## Authentication

В текущем MVP поддерживается только проверенный Telegram Login Widget.
Password/email registration, linking других identity и unlink Telegram отложены.

JWT:
- access + refresh;
- API использует Bearer access token;
- refresh хранится в httpOnly cookie;
- нужен blacklist/revocation refresh tokens.

User:
- unique mutable application username;
- unique telegram_id stored directly on User;
- отдельная Telegram identity table не нужна.

API prefix: `/api/v1/`.

## Roles and subscriptions

Роли: user/editor/admin.

Подписки отделены от ролей. `premium` не является ролью.

В MVP уже реализованы `Plan`, `Subscription` и backend-side entitlement для
`paid` Topic. Любая валидная подписка активна в окне
`starts_at <= now < expires_at`; `CANCELLED` означает cancel-at-period-end.
Несколько пересекающихся Subscription допустимы. Trial в текущем MVP не
реализован и остаётся будущим расширением.

Платёжный провайдер: Robokassa. Покупка — разовая оплата периода тарифа без
recurring auto-debit. Фискализация для НПД делегирована Robokassa/Робочеки СМЗ.

## Content

- ContentBlock — универсальная сущность.
- Типы ContentBlock хранятся в БД, а не в жёстком enum.
- Текст — Markdown.
- Status: draft/generated/reviewed/published.
- Manual content может сразу быть published.
- AI import получает generated.
- reviewed_by и published_at нужны.
- Полное version history в MVP не нужно.
- Удаление физическое.
- Topic deletion каскадно удаляет зависимый контент.

## Sources

Source — переиспользуемая сущность.

Sources optional.

Типы: textbook, presentation, article, website, guideline, other или эквивалентные значения.

Metadata должна поддерживать title, author, year, url, note.

Связи с Topic, ContentBlock, Flashcard, QuizQuestion.

## Progress

Topic progress:
- not_started;
- learning;
- learned.

Поля/данные:
- last_opened_at;
- started_at;
- completed_at.

Тему можно отметить изученной вручную. Также система может автоматически завершить тему по score тестов; threshold пока TBD.

Flashcards:
- принадлежат Topic и не существуют без Topic;
- в MVP используется простое планирование для действий «Знаю»/«Повторить»;
- FSRS не реализуется;
- одна текущая FlashcardReview на user+card;
- история review events возможна позже.

## Quizzes

- Retake без ограничения.
- Каждая QuizAttempt сохраняется.
- Лучший score вычисляется из attempts.
- Незавершённый тест сохраняет progress.
- Timeout не нужен.
- Глобального pass percentage пока нет.
- Незавершённая попытка хранит выбранные answer IDs; submitted result замораживается. Глобальный pass threshold остаётся TBD, как и owner-решение о раскрытии correct answers при unlimited retakes.

## Search

MVP search: Django/PostgreSQL `icontains` по:
- Subject.title;
- Section.title;
- Topic.title.

Фильтры по subject/section нужны.

Поиск по ContentBlock предусмотреть позже.

## Stack

- Backend: Django + DRF
- Frontend: Next.js
- Database: PostgreSQL 16 in production Docker Compose
- Cache: Redis
- Admin/CMS: Django Admin
- Python: 3.12
- Package manager: uv
- Lint/format: Ruff + Ruff format
- Backend tests: pytest
- API: REST only
- OpenAPI/Swagger: да
- Search: icontains
- Files: Media/File abstraction; current production backend is local persistent volume
- Docker: да
- Reverse proxy/TLS: Caddy
- Backend и frontend в production работают под одним origin на собственном VPS.
- Deploy: GitHub Actions after successful main CI, exact commit SHA, server-side lock.
- Для синхронного Content API Celery и ImportJob не нужны.
- Sentry: позже

## Security / API

- DRF throttling нужен.
- CORS ограничен frontend domains.
- Technical logs нужны, включая auth/import errors.
- Password reset/change password не входят в текущий Telegram-only MVP.
- Telegram unlink и account linking предусмотреть на будущее.
- Self account delete нужен.
- User deletion cascades progress/attempts; retained Payment audit nulls the user relation.
- UTC/timezone-aware timestamps.
- API errors используют единый backend envelope с code, detail и optional fields.
- DRF standard pagination.
- Healthcheck: `/api/v1/health/`, readiness: `/api/v1/health/ready/`.

## UX / public SEO

- mobile-first;
- dark mode сразу;
- базовая accessibility;
- публичный каталог/metadata допустим;
- `demo` Topic доступен anonymous users;
- `free` Topic требует authentication;
- `paid` Topic требует authentication + entitlement;
- `free`/`paid` Topic могут содержать явно отмеченные demo-preview элементы;
- anonymous usage не создаёт персональный learning state;
- canonical public subject URL: `/{subject_slug}`;
- canonical public topic URL: `/{subject_slug}/{topic_slug}`;
- numeric catalog routes считаются legacy/internal and redirect to canonical slug pages where applicable;
- slug генерируется в стабильном Latin kebab-case и не меняется автоматически при обычном переименовании title;
- sitemap включает published subject/topic canonical URLs.

## Repositories

- medcore-backend
- medcore-frontend
- medcore-docs
- medcore-content

Docs — источник истины по архитектуре.

## AI rules

AI не должен:
- молча менять архитектуру/API/model contracts;
- превращать TBD в решение;
- делать optional content required;
- добавлять тяжёлую инфраструктуру без конкретного требования;
- добавлять зависимости без необходимости.

Redis уже одобрен для cache. Celery/Kafka/Elastic/Kubernetes/microservices требуют отдельного решения.

## Read next

1. `../decisions/0004-mvp-decision-register.md` и более новые ADR в `../decisions/`
   (особенно `0008-mvp-auth-content-and-integration-contract.md` и
   `0009-subscriptions-core-and-paid-entitlement.md`)
2. `../product/mvp.md`
3. `../architecture/overview.md`
4. `../architecture/database.md`
5. `../content/structure.md`
6. `../decisions/`
