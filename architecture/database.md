# Database Draft

Текущая схема — рабочий черновик. Она должна уточняться по мере реализации MVP.

## Content domain

```text
Subject
  ↓
Section
  ↓
Topic
  ├── ContentBlock
  ├── Flashcard
  ├── Quiz
  │    └── QuizQuestion
  │         └── QuizAnswer
  └── Source (optional M2M / through relation)
```

## User domain

```text
User
  ├── UserTopicProgress
  ├── FlashcardReview
  └── QuizAttempt
       └── QuizAttemptAnswer

Subscription (later)
  └── User
```

## Entities

### User

Пользователь системы.

Поддерживаются два способа входа:
- Telegram;
- логин + пароль.

Telegram является основным пользовательским сценарием, логин/пароль — резервным.

Предполагаемые поля/идентификаторы:
- id;
- username / login;
- password hash (стандартный Django auth);
- telegram_id — optional, unique when present;
- telegram_username — optional;
- first_name / last_name — optional;
- role;
- is_active;
- created_at;
- updated_at.

Роль и подписка не смешиваются. `premium` не является ролью.

### Subscription

Добавляется при реализации монетизации.

Примерные поля:
- user;
- plan;
- status;
- starts_at;
- expires_at;
- created_at;
- updated_at.

### Subject

Предмет: микробиология, фармакология и т.д.

Предполагается уникальный `slug` для URL.

### Section

Раздел внутри предмета.

Предполагается `slug` для URL.

### Topic

Основная учебная тема.

Предполагается `slug` для URL. ID остается primary key, slug используется в пользовательских маршрутах.

### ContentBlock

Универсальный необязательный блок внутри темы.

Примеры типов:
- short;
- theory;
- important;
- mistakes;
- custom.

Предполагаемые поля:
- type / semantic_key;
- title;
- body_markdown;
- order;
- status;
- created_at;
- updated_at;
- reviewed_at — optional;
- reviewed_by — optional;
- published_at — optional.

Статусы:
- draft;
- generated;
- reviewed;
- published.

### Flashcard

Карточка с вопросом и ответом. Хранится отдельно от ContentBlock, потому что по карточке нужен пользовательский прогресс и повторения.

В domain-коде предпочтительны названия полей:
- question;
- answer.

`front/back` — общепринятые термины для лицевой и обратной стороны карточки, но для MedCore `question/answer` читаются понятнее.

Источники — optional.

### Quiz

Набор тестовых вопросов, связанный с темой.

Тема может иметь один или несколько Quiz.

### QuizQuestion

Один вопрос теста.

Для MVP:
- один правильный ответ;
- explanation — optional;
- difficulty — optional;
- sources — optional.

### QuizAnswer

Вариант ответа.

На уровне UI обычно ожидается 4 варианта, но БД не должна жестко требовать ровно 4 ответа. Это позволит позже поддерживать вопросы с другим количеством вариантов без изменения схемы.

### Source

Источник информации.

Связи с источниками должны быть необязательными и расширяемыми.

Source может связываться как минимум с:
- Topic;
- ContentBlock;
- Flashcard;
- QuizQuestion.

Одна сущность может иметь несколько источников, один Source может использоваться многими сущностями.

### UserTopicProgress

Состояние изучения темы пользователем.

Нужно поддержать как минимум два независимых сигнала:
- пользователь вручную отметил тему как изученную;
- система считает тему завершенной по учебной активности.

Автоматическое завершение может учитывать прохождение всех обязательных тестов, но не должно ломать темы без тестов.

Примерные поля:
- user;
- topic;
- is_marked_learned;
- completed_at — optional;
- created_at;
- updated_at.

### FlashcardReview

Состояние карточки для конкретного пользователя и история/данные повторений.

FSRS не реализуется в первом MVP, но модель не должна мешать его добавлению позже.

Полезно предусмотреть данные вроде:
- user;
- flashcard;
- last_rating — optional;
- review_count;
- last_reviewed_at — optional;
- next_review_at — optional.

Конкретный алгоритм интервалов — отдельное будущее решение.

### QuizAttempt

Одна попытка пользователя пройти тест.

### QuizAttemptAnswer

Ответ пользователя на конкретный вопрос внутри попытки.

## Search

Первый MVP использует простой поиск через Django ORM / PostgreSQL `icontains`.

Начальный scope:
- Subject.title;
- Section.title;
- Topic.title.

## Deletion

В MVP контент удаляется физически.

При реализации необходимо сознательно выбрать `CASCADE` / `PROTECT` для каждой связи, чтобы удаление родительской сущности не оставляло неконсистентные данные.

## Main principle

Учебные сущности внутри Topic необязательны. База данных не должна требовать, чтобы каждая тема имела краткий ответ, полный ответ, карточки, тест или источники.

Требования к полноте контента лучше реализовать как бизнес-валидацию перед публикацией, а не жесткие ограничения схемы.
