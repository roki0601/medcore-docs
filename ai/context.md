# AI Context

## Project

MedCore — web-сервис для подготовки студентов-медиков к экзаменам.

## Source of truth

Базовый подробный реестр согласованных решений находится в:

`../decisions/0004-mvp-decision-register.md`

Более новые ADR в `../decisions/` имеют приоритет над конфликтующим старым
текстом. ADR 0006 разрешает anonymous demo scope, а ADR 0007 фиксирует
Topic-level access model `demo / free / paid`.

AI-агент обязан прочитать decision register и более новые ADR перед
существенными изменениями. Пункты, помеченные `TBD`, нельзя самостоятельно
превращать в принятые решения.

## Current implementation snapshot (2026-09-20)

This section is a factual delivery snapshot, not a replacement for accepted
product decisions.

Backend `main` already contains:

- Telegram auth/account lifecycle (001);
- catalog and Topic access `demo/free/paid` (002);
- synchronous content authoring API and media access/storage boundary (004);
- topic progress, flashcard review and quiz attempts (003);
- OpenAPI/runtime contract alignment.

The subscriptions app currently exposes only the deny-by-default entitlement
policy seam. Plan/Subscription storage, real paid entitlement and payment
integration are not implemented yet and are the next backend product boundary.

Frontend production already contains the catalog/anonymous demo (FE-001) and
Telegram auth/account lifecycle (FE-002). Protected media rendering (FE-003)
and learning progress (FE-004) have their backend dependencies merged and may
be started after the mandatory fresh contract/preflight check.

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

Plan/Subscription/entitlement входят в MVP отдельной feature. Архитектура должна поддерживать:
- несколько тарифов;
- trial;
- starts_at/expires_at;
- backend-side paywall checks;
- Topic-level access `demo / free / paid`;
- active entitlement для `paid` Topic;
- возможность позже расширить entitlement на разные планы/trial без изменения content ownership.

Payment provider пока не выбран.

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
- Database: PostgreSQL
- Cache: Redis
- Admin/CMS: Django Admin
- Python: 3.12
- Package manager: uv
- Lint/format: Ruff + Ruff format
- Type checking: нужен, tool TBD
- Backend tests: pytest
- API: REST only
- OpenAPI/Swagger: да
- Search: icontains
- Files: object storage + Media/File model
- Docker: да
- Backend и frontend в production работают под одним origin на собственном сервере.
- Prod DB provider: TBD (Supabase PostgreSQL или другой managed Postgres)
- Для синхронного Content API Celery и ImportJob не нужны.
- Product analytics: не нужна в MVP
- Sentry: позже

## Security / API

- DRF throttling нужен.
- CORS ограничен frontend domains.
- Technical logs нужны, включая auth/import errors.
- Password reset/change password не входят в текущий Telegram-only MVP.
- Telegram unlink и account linking предусмотреть на будущее.
- Self account delete нужен.
- User deletion cascades progress/attempts.
- UTC/timezone-aware timestamps.
- API errors используют единый backend envelope с code, detail и optional fields.
- DRF standard pagination.
- Healthcheck: `/api/v1/health/`.

## UX

- mobile-first;
- dark mode сразу;
- базовая accessibility;
- публичный каталог/metadata допустим;
- `demo` Topic доступен anonymous users;
- `free` Topic требует authentication;
- `paid` Topic требует authentication + entitlement;
- `free`/`paid` Topic могут содержать явно отмеченные demo-preview элементы;
- anonymous usage не создаёт персональный learning state;
- slugs используются в человекочитаемых URL;
- slug генерируется из title, может быть вручную переопределён и изменён позже.

## Repositories

- medcore-backend
- medcore-frontend
- medcore-docs

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
   (особенно `0008-mvp-auth-content-and-integration-contract.md`)
2. `../product/mvp.md`
3. `../architecture/overview.md`
4. `../architecture/database.md`
5. `../content/structure.md`
6. `../decisions/`
