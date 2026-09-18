# ADR 0007: Content access model

## Status

Accepted

## Context

MedCore already allows a limited anonymous demo flow before authentication.
Теперь фиксируется простая MVP-модель доступа к учебному контенту, чтобы
content specs, subscriptions policy и frontend использовали один контракт.

## Decision

Основной access boundary — `Topic`.

У Topic есть `access_level` с тремя значениями:

- `demo` — полный контент темы доступен без авторизации;
- `free` — полный контент темы доступен после authentication, подписка не нужна;
- `paid` — полный контент темы доступен после authentication и при наличии
  подходящего активного entitlement/subscription.

Матрица доступа:

| User state | demo | free | paid |
|---|---|---|---|
| Anonymous | allow | deny full content | deny full content |
| Authenticated without entitlement | allow | allow | deny |
| Authenticated with entitlement | allow | allow | allow |

## Catalog and metadata

Каталог и безопасные metadata Subject/Section/Topic могут быть публичными.
Anonymous user может видеть структуру каталога, название/описание темы,
состояние доступа и наличие preview, но публичный metadata endpoint не должен
утекать закрытый body/content.

## Demo preview inside free/paid topics

`free` и `paid` Topic могут иметь ограниченный anonymous preview.

Отдельные дочерние учебные элементы (`ContentBlock`, `Flashcard`, `Quiz` или
эквивалентный presentation unit) могут быть явно помечены как доступные в
demo preview. Конкретное имя поля (`is_demo_preview` или эквивалент) и точный
API shape определяются content feature spec.

Если дочерний элемент не помечен как demo preview, он наследует access Topic.
Не вводится независимая полноценная access policy на каждом уровне
`Subject → Section → Topic → child`: Topic остаётся главным access boundary.

## Learning state

Персональный learning state существует только для authenticated users.
Anonymous demo usage не создаёт backend `UserTopicProgress`, `FlashcardReview`,
`QuizAttempt` или другой персональный прогресс.

## Access policy boundary

Backend является security boundary.

Content и Learning не читают `Subscription` напрямую и не размазывают проверки
по views/models. Решение о доступе проходит через единый access/entitlement
policy interface в `subscriptions` application boundary.

Для MVP достаточно активного entitlement для `paid` контента. Архитектура
должна позволять позже добавить trial, разные планы и другие entitlement types
без изменения базовой модели `demo/free/paid`.

## Not in MVP

Не вводить сейчас:

- произвольные ACL `user → topic`;
- сложное наследование и override access на Subject/Section/Topic/каждом child;
- отдельную access rule table для каждого content object;
- несколько независимых boolean-флагов `is_free`, `is_public`, `is_premium`,
  `requires_auth`, `requires_subscription` вместо одного `access_level`;
- привязку content models напрямую к Subscription internals.

## Overrides

Этот ADR уточняет и частично переопределяет пункты 113, 116 и 117
`0004-mvp-decision-register.md` и дополняет ADR 0006.

- старое правило «весь учебный контент только authenticated» больше не действует;
- вместо неопределённого `is_free` используется трехуровневая модель
  `demo / free / paid` на Topic;
- paywall granularity для MVP фиксируется на Topic, с ограниченным child-level
  demo preview как исключением.
