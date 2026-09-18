# AI Context

## Project

MedCore — web-сервис для подготовки студентов-медиков к экзаменам.

## Source of truth

Базовый подробный реестр согласованных решений находится в:

`../decisions/0004-mvp-decision-register.md`

Более новые ADR в `../decisions/` имеют приоритет над конфликтующим старым
текстом. В частности, ADR 0006 переопределяет старое правило о полном запрете
учебного контента без auth и разрешает ограниченный anonymous demo scope.

AI-агент обязан прочитать decision register и более новые ADR перед
существенными изменениями. Пункты, помеченные `TBD`, нельзя самостоятельно
превращать в принятые решения.

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

Поддерживаются:

1. Telegram — основной flow.
2. Login + password — резервный flow.

Один User может иметь оба способа входа.

JWT:
- access + refresh;
- API использует Bearer access token;
- refresh хранится в httpOnly cookie;
- нужен blacklist/revocation refresh tokens.

User:
- unique username;
- optional email на будущее;
- optional unique telegram_id;
- из Telegram сохраняются минимум telegram_id и username.

API prefix: `/api/v1/`.

## Roles and subscriptions

Роли: user/editor/admin.

Подписки отделены от ролей. `premium` не является ролью.

`Subscription` создаётся сразу. Архитектура должна поддерживать:
- несколько тарифов;
- trial;
- starts_at/expires_at;
- backend-side paywall checks;
- возможность free content;
- будущую разную granularity paywall.

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
- self-rating в MVP не нужен;
- FSRS не реализуется;
- заложить данные для будущего FSRS: next_review_at, last_reviewed_at, repetitions или эквивалент;
- одна текущая FlashcardReview на user+card;
- история review events возможна позже.

## Quizzes

- Retake без ограничения.
- Каждая QuizAttempt сохраняется.
- Лучший score вычисляется из attempts.
- Незавершённый тест сохраняет progress.
- Timeout не нужен.
- Глобального pass percentage пока нет.
- Ошибки/неправильные ответы пользователя нужно хранить, но полную QuizAttemptAnswer по каждому ответу решено не хранить; точная модель TBD.

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
- Backend hosting current plan: Render
- Frontend hosting: Vercel
- Prod DB provider: TBD (Supabase PostgreSQL или другой managed Postgres)
- Celery: TBD
- Product analytics: не нужна в MVP
- Sentry: позже

## Security / API

- DRF throttling нужен.
- CORS ограничен frontend domains.
- Technical logs нужны, включая auth/import errors.
- Password reset/change password нужны.
- Telegram unlink предусмотреть на будущее.
- Self account delete нужен.
- User deletion cascades progress/attempts.
- UTC/timezone-aware timestamps.
- API errors должны иметь единый формат, схема TBD.
- DRF standard pagination.
- Healthcheck: `/api/v1/health/`.

## UX

- mobile-first;
- dark mode сразу;
- базовая accessibility;
- ограниченный demo-контент доступен anonymous users до auth;
- полный учебный контент и персональный learning state требуют authentication;
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
2. `../product/mvp.md`
3. `../architecture/overview.md`
4. `../architecture/database.md`
5. `../content/structure.md`
6. `../decisions/`
