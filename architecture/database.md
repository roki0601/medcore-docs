# Database Draft

Текущая схема — рабочий черновик. Детальный реестр решений: `../decisions/0004-mvp-decision-register.md`.

## Content domain

```text
Subject
  ↓
Section
  ↓
Topic
  ├── ContentBlock
  ├── Flashcard (topic optional)
  ├── Quiz
  │    └── QuizQuestion
  │         └── QuizAnswer
  └── Source (optional M2M)

Media/File
```

## User / access domain

```text
User
  ├── UserTopicProgress
  ├── FlashcardReview
  ├── QuizAttempt
  └── Subscription
       └── Plan/Tariff
```

## User

Поддерживает Telegram и login/password в одном аккаунте.

Нужно заложить:
- BigAutoField id;
- unique username;
- password hash через Django auth;
- optional unique telegram_id;
- optional telegram username;
- optional email;
- role: user/editor/admin;
- is_active;
- timestamps.

Из Telegram пока достаточно сохранять telegram_id и username.

## Subscription / tariff

Subscription создаётся сразу, не позже.

Нужно поддержать несколько тарифов и trial.

Минимально:
- user;
- plan/tariff;
- status;
- starts_at;
- expires_at;
- trial-related data при необходимости;
- timestamps.

`premium` не является ролью.

## Subject / Section / Topic

Поддерживают slug для user-facing URL. Slug генерируется автоматически из title, допускается manual override и изменение позже.

Точная область uniqueness slug и полный набор полей — TBD.

Topic deletion физическое и каскадно удаляет зависимый контент.

## ContentBlock

Универсальный optional блок.

Тип блока должен храниться в БД, а не быть только hardcoded enum, чтобы добавлять новые типы без миграции enum-кода.

Основное содержимое — Markdown.

Статусы:
- draft;
- generated;
- reviewed;
- published.

Нужны timestamps, reviewed_by, reviewed_at/published_at по необходимости.

Manual content может сразу быть published.

## Flashcard

Отдельная сущность. Topic relation допускается optional.

Sources optional.

Self-rating в MVP не нужен.

Для будущего FSRS предусмотреть состояние вроде:
- next_review_at;
- last_reviewed_at;
- repetitions.

Одна текущая FlashcardReview на user+flashcard. Историю review events можно добавить позже отдельной таблицей.

Названия question/answer, difficulty/order/status — часть полей ещё TBD согласно decision register.

## Quiz

Retake без ограничений.

На MVP используется single-choice модель с одним правильным ответом (ранее принято, не отменено).

Не фиксировать ровно 4 ответа на уровне БД.

QuizAttempt сохраняется для каждой попытки, в том числе нужен механизм сохранения незавершённого прогресса.

Лучший score не хранить отдельно — вычислять из attempts.

Полную запись ответа на каждый вопрос в виде QuizAttemptAnswer решено не хранить как обязательную модель. Нужно хранить ошибки/неправильные ответы пользователя в компактной форме; точная структура TBD.

Timeout не нужен. Глобального pass threshold пока нет. Для auto-completion Topic нужен score-based criterion, конкретный threshold TBD.

## Source

Переиспользуемая optional сущность.

Типы: textbook, presentation, article, website, guideline/methodical material, other.

Metadata:
- title;
- author;
- year;
- url;
- note.

Связи optional M2M минимум с Topic, ContentBlock, Flashcard, QuizQuestion.

Удаление Source удаляет M2M-связи, но не связанные учебные сущности.

Relation-level page/slide references пока не нужны.

## Media/File

Отдельная модель нужна.

Файл хранится в object storage, в PostgreSQL — metadata, URL/key и связи.

Конкретный storage provider TBD.

## UserTopicProgress

Статусы:
- not_started;
- learning;
- learned.

Отдельный repeat status пока не нужен.

Нужны:
- user;
- topic;
- progress/state;
- last_opened_at;
- started_at;
- completed_at;
- признак/данные ручного завершения при необходимости.

Тема может завершаться вручную или автоматически по score тестов.

## Search

MVP: `icontains` по Subject.title, Section.title, Topic.title + filters по subject/section.

Поиск по ContentBlock предусмотреть позже.

## Deletion

В MVP используется физическое удаление.

User deletion каскадно удаляет progress/attempts.
Topic deletion каскадно удаляет зависимый контент.
Source deletion удаляет связи.

## Infrastructure-related data decisions

- PostgreSQL — primary datastore.
- Redis — cache.
- IDs — BigInt/BigAutoField.
- timestamps timezone-aware, хранение времени в UTC.
- migrations всегда коммитятся.
- fixtures использовать.

## Main principle

Учебные сущности внутри Topic в целом optional. Не делать поля/связи обязательными только для удобства текущего контента без отдельного решения.
