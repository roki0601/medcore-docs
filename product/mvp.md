# MVP

## Цель

Сделать минимальный рабочий сервис для подготовки студентов-медиков к экзаменам. Архитектура сразу допускает несколько предметов, даже если сначала наполнен только один.

## Основной сценарий

1. Пользователь регистрируется или входит.
2. Выбирает предмет.
3. Выбирает раздел и тему.
4. Читает материал.
5. Проходит карточки.
6. Проходит тесты.
7. Отмечает тему изученной и/или получает автоматическое завершение по score.
8. Видит прогресс.

## В MVP

- Telegram auth + login/password.
- Bearer JWT, access + refresh; refresh в httpOnly cookie; blacklist/revocation.
- `/api/v1/`.
- Несколько предметов поддерживаются моделью и UI.
- `Subject → Section → Topic`.
- Slug URL.
- Optional ContentBlock/Flashcard/Quiz/Source.
- Markdown.
- Content statuses: draft/generated/reviewed/published.
- Manual content можно публиковать сразу.
- Quiz attempts, сохранение незавершённого прогресса, retake без ограничений.
- Topic progress: not_started/learning/learned, ручное и score-based completion.
- Search `icontains` по Subject/Section/Topic + filters.
- Django Admin как CMS, bulk import и bulk publish нужны.
- Subscription сущность создаётся сразу; несколько тарифов и trial должны поддерживаться.
- Free/paywalled content; paywall проверяет backend.
- Redis для кэша.
- Object storage + Media/File model.
- DRF throttling, CORS restrictions, technical logging.
- OpenAPI/Swagger.
- Mobile-first + dark mode.

## Не является частью MVP / TBD

- User-facing AI chat.
- Voice exam.
- FSRS runtime.
- Product event analytics.
- Elasticsearch/OpenSearch.
- Sentry — позже.
- Celery — TBD.
- Payment provider — TBD.
- Конкретный object storage provider — TBD.
- Staging environment сейчас не нужен.

## Roles / subscription

Roles: user/editor/admin.

Subscription не является ролью. `premium` не используется как permission-role. Subscription поддерживает тариф, status, starts_at, expires_at и будущий trial.

## Content

Контент можно добавлять постепенно. Никакой блок темы не обязателен только потому, что существует в модели. Sources optional.

AI-imported content получает `generated`; manual content может сразу быть `published`.

## Source of truth

Детальный реестр всех 200 обсуждённых решений и TBD: `../decisions/0004-mvp-decision-register.md`.
