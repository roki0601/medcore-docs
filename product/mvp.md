# MVP

## Цель

Сделать минимальный рабочий сервис для подготовки студентов-медиков к экзаменам. Архитектура сразу допускает несколько предметов, даже если сначала наполнен только один.

## Основной сценарий

1. Новый пользователь может без авторизации посмотреть ограниченный demo-контент и понять формат продукта.
2. Для полноценного использования пользователь регистрируется или входит.
3. Выбирает предмет.
4. Выбирает раздел и тему.
5. Читает материал.
6. Проходит карточки.
7. Проходит тесты.
8. Отмечает тему изученной и/или получает автоматическое завершение по score.
9. Видит прогресс.

## В MVP

- Ограниченный anonymous demo-контент до авторизации.
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
- Content access на уровне Topic: `demo`, `free`, `paid`; paywall проверяет backend.
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

Модель доступа к контенту:

- `demo` Topic доступен anonymous users;
- `free` Topic требует authentication;
- `paid` Topic требует authentication + active entitlement/subscription;
- каталог и безопасные metadata Subject/Section/Topic могут быть публичными;
- внутри `free`/`paid` Topic отдельные элементы могут быть явно доступны как demo preview;
- anonymous demo не создаёт персональный progress/attempts/reviews.

Главный access boundary — Topic; сложная система ACL/наследования по каждому
уровню в MVP не вводится. Детали реализации preview/API задаёт content spec.

## Source of truth

Базовый реестр обсуждённых решений и TBD: `../decisions/0004-mvp-decision-register.md`.
Более новые ADR в `../decisions/` имеют приоритет над конфликтующим старым
текстом; anonymous demo access зафиксирован в ADR 0006, а модель
`demo/free/paid` — в ADR 0007.
