# ADR 0008: MVP auth, learning content and synchronous integration contract

## Status

Accepted

## Context

После уточнения требований к MVP нужно зафиксировать несколько решений,
которые уточняют старый decision register и ADR 0006/0007:

- первый и пока единственный пользовательский способ регистрации и входа —
  Telegram;
- username приложения должен оставаться изменяемым и не может быть
  идентификатором аккаунта;
- content API должен принимать данные напрямую, без отдельной import job;
- учебная модель должна поддерживать несколько quiz, multiple-choice и
  простое планирование карточек;
- production будет размещаться на собственном сервере с одним общим origin.

## Decision

### User and Telegram auth

`User` хранит напрямую:

- `telegram_id` — уникальный основной идентификатор авторизации;
- `username` — уникальный username приложения, который пользователь может
  менять;
- `role`, active flag и timestamps.

Отдельная таблица Telegram identity не вводится. Отдельный snapshot
`telegram_username` не является обязательной частью нового контракта.

Регистрация и вход выполняются только через проверенный Telegram Login Widget.
Password/email registration, linking других identity и unlink Telegram не входят
в текущий MVP.

Auth использует Bearer access JWT и refresh JWT в `HttpOnly` cookie.
Refresh-токены не ротируются, но имеют серверную revocation/blacklist state.
Logout отзывает текущую сессию; logout-all отзывает все сессии пользователя.

### Content and learning

Минимально обязательным пользовательским полем для
`Subject`, `Section` и `Topic` является `title`; отношения и системные
поля обязательны по модели. `slug` генерируется автоматически и остаётся
вспомогательным URL-представлением.

`Topic` остаётся главным access boundary с `access_level`:
`demo`, `free`, `paid`. Дочерние элементы могут иметь явный demo marker,
но отдельная ACL-модель не вводится.

Теория состоит из упорядоченных `ContentBlock`. В MVP поддерживаются
`markdown`, `image` и `callout`. `Flashcard` и `Quiz` являются
отдельными учебными сущностями.

Карточка принадлежит Topic, orphan-карточки не поддерживаются. В MVP
используется простое планирование (`next_review_at`, `last_reviewed_at`,
`interval_days` и эквивалентные поля); FSRS откладывается.

У Topic может быть несколько Quiz. Вопросы поддерживают один или несколько
правильных ответов, ответы и вопросы могут перемешиваться, попытки можно
повторять, незавершённую попытку можно продолжить. Оценка допускает partial
score и не становится отрицательной.

### Content API integration

Внешний клиент записывает контент синхронно через обычные REST CRUD endpoints,
используя отдельный integration key. Единого жёсткого формата импорт-файла,
`ImportJob`, Celery worker и автоматического удаления отсутствующих объектов
нет.

Внешний клиент сам преобразует свои данные в API-запросы. Backend валидирует
каждую сущность на границе API. Явное удаление Topic удаляет только этот Topic
и его собственных дочерних объектов; соседние темы, разделы и предметы не
затрагиваются.

### Hosting and media

Frontend и backend работают под одним origin; backend доступен под `/api/`.
Production запускается на собственном сервере через Docker Compose и reverse
proxy с HTTPS. PostgreSQL регулярно резервируется, Redis остаётся cache.

Binary media хранится в object storage через `Media/File` boundary. Private и
paid media выдаются только после backend access check через короткоживущие
signed URLs.

## Consequences

- Более старые формулировки о password-first или password+Telegram MVP имеют
  более низкий приоритет для текущего auth slice.
- `User.telegram_id` не дублируется отдельной identity table.
- Content API проще для внешнего отправителя, но не предоставляет произвольное
  хранение неизвестного JSON: неизвестные структуры должны быть преобразованы
  во внешний API-контракт.
- Celery не добавляется ради импорта.
- Subscription/Plan/payment semantics остаются отдельной TBD feature и не
  выбираются этим ADR.

## Not in this ADR

Не закрываются:

- payment provider, trial и точная entitlement policy;
- password auth и password reset;
- slug uniqueness/redirect history;
- будущий account linking;
- production storage provider;
- полноценный FSRS.
