# Backend Structure Draft

## Status

Working agreement / draft. Зафиксировано по итогам обсуждения перед проектированием agent workflow. Детали могут быть уточнены до начала реализации.

## Architectural style

Backend строится как Django + DRF modular monolith.

Предпочтён вариант крупных бизнес-доменов (вариант A): не создавать Django app на каждую модель и не дробить `content` преждевременно.

Рабочий набор apps:

```text
apps/
├── common/
├── users/
├── content/
├── learning/
├── subscriptions/
└── media/
```

### Domain ownership

- `users` — User, identity, authentication/account flows, Telegram identity, login/password.
- `content` — авторский учебный контент: Subject, Section, Topic, ContentBlock/ContentBlockType, Flashcard, Quiz/QuizQuestion/QuizAnswer, Source, content import/CMS logic.
- `learning` — пользовательское взаимодействие с учебным контентом: UserTopicProgress, FlashcardReview, QuizAttempt и будущая learning/spaced-repetition логика.
- `subscriptions` — Plan/Tariff, Subscription, trial и policy/capability проверки платного доступа. Payments пока не выделяются в отдельный app.
- `media` — Media/File и интеграция с object storage.
- `common` — только действительно общие инфраструктурные primitives; не использовать как свалку helpers/utils.

`content` остаётся единым крупным доменом на MVP. Деление на `catalog/content/practice` сейчас не требуется. Если домен реально станет слишком большим, границы будут пересмотрены на основании существующего кода, а не заранее.

## Repository layout

Рабочая верхнеуровневая структура:

```text
medcore-backend/
├── manage.py
├── pyproject.toml
├── uv.lock
├── Dockerfile
├── .env.example
├── README.md
├── AGENTS.md
├── config/
│   ├── settings/
│   │   ├── base.py
│   │   ├── local.py
│   │   └── production.py
│   ├── urls.py
│   ├── api.py
│   ├── asgi.py
│   └── wsgi.py
└── apps/
    ├── common/
    ├── users/
    ├── content/
    ├── learning/
    ├── subscriptions/
    └── media/
```

Не создавать пустые packages только ради симметрии. Файл превращается в package, когда это оправдано размером/сложностью.

Для крупного `content` ожидается примерно:

```text
apps/content/
├── AGENTS.md
├── apps.py
├── migrations/
├── models/
├── api/
│   ├── urls.py
│   ├── serializers/
│   └── views/
├── services/
├── selectors/
├── importing/
├── admin/
└── tests/
```

Для небольшого app допустимы `models.py`, `services.py`, `admin.py` вместо packages.

## Layer responsibilities

### Models

Models отвечают за данные, связи и локальные инварианты собственной сущности.

Не помещать в model cross-domain use cases. Например, `Topic` не должен реализовывать пользовательское завершение темы, поскольку это принадлежит `learning`.

### Serializers

DRF serializers — transport/input-output boundary:

- parsing/serialization;
- shape/type/format validation;
- простая input validation.

Не размазывать существенную бизнес-логику по serializers.

### Views

Views/ViewSets должны быть тонкими: HTTP/DRF orchestration, serializer validation, permission checks, вызов service/selector, формирование response.

Не помещать основную бизнес-логику во views.

### Services

Services используются для write/business operations и use cases, особенно если операция:

- меняет состояние;
- содержит бизнес-правила;
- используется из нескольких entry points;
- пересекает несколько моделей;
- требует transaction boundary.

Примеры: `publish_topic`, `complete_topic`, `submit_quiz`, `activate_subscription`, `delete_account`.

Не создавать service-обёртки вокруг тривиального `.save()` только ради слоя.

### Selectors

Selectors используются для нетривиальных read/query operations и инкапсулируют сложные ORM-запросы (`select_related`, `prefetch_related`, annotations, сложные filters).

Рабочее правило:

```text
READ  -> selector, когда query достаточно сложный/переиспользуемый
WRITE -> service, когда есть use case/business logic
```

Не создавать selector для каждого простого `Model.objects.get(...)` только ради архитектурной симметрии.

### Repository layer

Отдельный repository layer поверх Django ORM на MVP не вводится. Django ORM является persistence abstraction. Не создавать `TopicRepository`, `UserRepository` и аналогичные классы без конкретной необходимости.

## Domain dependency direction

Рабочие разрешённые зависимости:

```text
users          media
  ↑              ↑
  └── content ───┘
        ↑
        │
     learning

users ← subscriptions
```

То есть:

- `users` не должен зависеть от content/learning/subscriptions;
- `media` является низкоуровневым доменом и не должен зависеть от content/learning;
- `content` может ссылаться на `users` (например `reviewed_by`) и `media`;
- `learning` может зависеть от `users` и `content`;
- `subscriptions` зависит от `users`;
- `content` не должен зависеть от `learning`;
- core content models не должны знать внутреннее устройство subscriptions/paywall.

Избегать циклических domain dependencies.

Cross-domain операция принадлежит домену, описывающему use case. Например, submission Quiz и обновление пользовательского Topic progress принадлежат `learning`, хотя используют модели `content`.

## Paywall / access boundary

Role/DRF permission и продуктовый paid access — разные понятия.

- Authentication отвечает на вопрос «кто пользователь?».
- Permission — разрешена ли операция его роли/identity.
- Access policy — есть ли продуктовый доступ к ресурсу (free content, subscription, trial, plan и т.п.).
- Business rule — допустима ли сама операция по состоянию домена.

`content` models не должны напрямую проверять поля/статусы Subscription.

На MVP отдельный `access` Django app не создаётся. Capability/policy проверки доступа предполагается держать около `subscriptions` (точная реализация TBD).

## Transactions

Transaction boundary располагается на уровне business service/use case, когда операция должна быть атомарной.

Принцип: одна атомарная бизнес-операция — одна явно понятная transaction boundary.

Не переносить transaction orchestration во view без необходимости.

## Django signals

Не использовать Django signals для core business logic.

Предпочитать явный execution flow через services. Signals допустимы только для действительно подходящих инфраструктурных side effects при наличии конкретной причины.

## API errors

Нужен единый формат domain/API ошибок. Рабочее направление:

```json
{
  "code": "TOPIC_NOT_FOUND",
  "detail": "Topic not found."
}
```

Точная нормализация стандартных DRF validation errors ещё TBD.

Не плодить отдельные exception classes без необходимости; domain exceptions вводить там, где они выражают реальное бизнес-состояние/ошибку.

## API organization

API остаётся domain-local: serializers/views/urls находятся внутри соответствующего Django app, а не в одном глобальном `api` package для всех доменов.

Prefix уже принят: `/api/v1/`.

Глубокую URL-вложенность не вводить автоматически. Точные API contracts определяются отдельными specs.

## `common` rule

`common` не является местом для непонятного кода.

Кандидат переносится в `common`, когда он действительно domain-agnostic и используется несколькими доменами. Не создавать generic `utils.py`, `helpers.py`, `misc.py` без чёткой ответственности.

## Abstraction rule

Не вводить абстракцию для единственной реализации только «на будущее», если extensibility не является уже существующим требованием.

Не создавать без конкретной необходимости Repository/UnitOfWork/CommandBus/EventBus/factory/protocol слои поверх стандартных Django механизмов.

Абстракция допустима, когда решает реальную текущую проблему или уже зафиксированную неопределённость (например сменяемый external provider).

## Tests placement

Предпочтительно хранить tests рядом с domain app:

```text
apps/content/tests/
├── factories.py
├── test_models.py
├── test_selectors.py
├── test_services.py
└── api/
```

Это упрощает локальный контекст для разработчиков и AI-агентов.

## Nested AGENTS.md

Backend должен иметь корневой `AGENTS.md` с общими engineering rules.

Для крупных/значимых domain apps допускаются вложенные `AGENTS.md`. Их задача — не копировать документацию, а кратко описывать:

- purpose/ownership домена;
- что домен НЕ owns;
- dependency boundaries;
- важные локальные invariants/conventions;
- ссылки на релевантный `medcore-docs` контекст.

Рабочий подход — progressive disclosure: не создавать вложенный `AGENTS.md` для каждого каталога заранее. `content/AGENTS.md` имеет смысл сразу как самый крупный домен; остальные добавлять по мере начала реализации/роста сложности.

Разделение источников контекста:

```text
medcore-docs            -> WHAT / WHY, продукт и принятые решения
backend/AGENTS.md       -> HOW, общие engineering rules
apps/<domain>/AGENTS.md -> WHERE, ownership/boundaries/local conventions
spec/task               -> конкретно WHAT TO IMPLEMENT NOW
existing code/tests     -> текущие implementation patterns
```

Если task/spec противоречит принятому ADR/decision register, AI не должен молча считать task новым архитектурным решением; конфликт нужно эскалировать.

## AI-oriented constraints

Особенно важно для agentic development:

- не менять scope задачи;
- не делать unrelated refactoring;
- не менять архитектуру ради упрощения текущей задачи;
- не додумывать `TBD` business decisions;
- не вводить dependency/abstraction без необходимости;
- не использовать signals как скрытый business flow;
- соблюдать domain dependency direction;
- ориентироваться на существующие patterns после появления эталонных модулей.

Подробный протокол работы coding agent будет зафиксирован отдельно.