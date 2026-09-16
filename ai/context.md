# AI Context

## Project

MedCore — web-сервис для подготовки студентов-медиков к экзаменам.

## Current MVP idea

Пользователь выбирает предмет, открывает тему, изучает материал, проходит карточки и тесты и видит прогресс.

На старте проект не привязан к конкретным вузам.

## Core hierarchy

`Subject → Section → Topic`

У темы могут быть необязательные сущности:

- ContentBlock
- Flashcard
- Quiz
- Source

Ничего из этого не обязано существовать у каждой темы сразу.

## Important product decisions

- Контент можно дополнять постепенно.
- Карточки и тесты хранятся отдельно от обычных контентных блоков.
- AI используется как внутренний инструмент производства контента, а не как обязательный runtime-функционал для пользователя.
- Gemini Notebook можно использовать для генерации карточек и тестов; подтверждено, что их данные можно извлекать из ответа и затем импортировать в нашу БД.
- Медицинский AI-контент должен проходить ручную проверку перед публикацией.

## Planned stack

- Backend: Django + Django REST Framework
- Database: PostgreSQL
- Frontend: Next.js
- Admin/CMS: Django Admin
- Redis/Celery: не нужны для первого MVP

## Planned domain entities

Content:
- Subject
- Section
- Topic
- ContentBlock
- Flashcard
- Quiz
- QuizQuestion
- QuizAnswer
- Source

User data:
- User
- UserTopicProgress
- FlashcardReview
- QuizAttempt
- QuizAttemptAnswer

## Current stage

Сейчас формируется документация и структура проекта. Код backend/frontend еще не является источником истины; решения фиксируются здесь и в `decisions/`.

## Read next

- `../product/mvp.md`
- `../architecture/overview.md`
- `../architecture/database.md`
- `../content/structure.md`
