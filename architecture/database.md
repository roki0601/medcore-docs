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
  └── Source (many-to-many / through relation)
```

## User domain

```text
User
  ├── UserTopicProgress
  ├── FlashcardReview
  └── QuizAttempt
       └── QuizAttemptAnswer
```

## Entities

### Subject
Предмет: микробиология, фармакология и т.д.

### Section
Раздел внутри предмета.

### Topic
Основная учебная тема.

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
- updated_at.

### Flashcard
Карточка с вопросом и ответом. Хранится отдельно от ContentBlock, потому что по карточке нужен пользовательский прогресс и повторения.

### Quiz
Набор тестовых вопросов, связанный с темой.

### QuizQuestion
Один вопрос теста.

### QuizAnswer
Вариант ответа. На старте предполагается один правильный ответ.

### Source
Источник информации. Одна тема может иметь несколько источников, один источник может использоваться в нескольких темах.

### UserTopicProgress
Состояние изучения темы пользователем.

Примерные статусы:
- NOT_STARTED;
- LEARNING;
- LEARNED;
- REPEAT.

### FlashcardReview
Состояние карточки для конкретного пользователя и данные для следующего повторения.

### QuizAttempt
Одна попытка пользователя пройти тест.

### QuizAttemptAnswer
Ответ пользователя на конкретный вопрос внутри попытки.

## Main principle

Учебные сущности внутри Topic необязательны. База данных не должна требовать, чтобы каждая тема имела краткий ответ, полный ответ, карточки, тест или источники.

Требования к полноте контента лучше реализовать как бизнес-валидацию перед публикацией, а не жесткие ограничения схемы.
