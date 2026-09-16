# AI Context

## Project

MedCore — web-сервис для подготовки студентов-медиков к экзаменам.

## Current MVP idea

Пользователь регистрируется или входит в систему, выбирает предмет, открывает тему, изучает материал, проходит карточки и тесты и видит прогресс.

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
- Медицинский AI-контент должен поддерживать статусы `draft`, `generated`, `reviewed`, `published`, даже если первое время контент часто будет загружаться сразу как `published`.
- Источники опциональны: Topic, ContentBlock, Flashcard и QuizQuestion могут ссылаться на один или несколько Source, но наличие источника не обязательно на уровне БД.
- Для темы используется `slug` в публичных URL; внутренний primary key остается отдельным ID.
- Удаление контента в MVP — физическое. При реализации нужно корректно настроить `CASCADE` / `PROTECT` для зависимостей.
- Поиск MVP — простой PostgreSQL/Django `icontains`, прежде всего по названиям Subject, Section и Topic.
- Продуктовая аналитика событий не является обязательной частью первого MVP.
- FSRS не реализуется в MVP, но модель прогресса карточек не должна мешать его добавлению позже.

## Authentication

Поддерживаются два способа входа:

1. Telegram — основной и предпочтительный пользовательский сценарий.
2. Логин + пароль — резервный способ авторизации.

После успешной аутентификации frontend работает с backend через Bearer JWT.

API versioning: `/api/v1/...`.

## Roles and subscriptions

Роли и платный доступ — разные понятия.

Роли отвечают за права пользователя в системе, например:
- user
- editor
- admin

Подписка хранится отдельно и определяет коммерческий доступ пользователя. `premium` не является ролью.

## Topic completion

Тема может считаться изученной двумя способами:

- пользователь вручную нажал кнопку «Изучено»;
- система считает тему завершенной по результатам учебной активности, например после прохождения всех обязательных тестов.

Автоматические правила не должны мешать темам, у которых тестов пока нет.

## Planned stack

- Backend: Django + Django REST Framework
- Database: PostgreSQL
- Frontend: Next.js
- Admin/CMS: Django Admin
- API auth: Bearer JWT
- Content format: Markdown
- File storage: object storage, в PostgreSQL хранятся URL/metadata
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
- Subscription (позже, когда появится монетизация)

## Current stage

Сейчас формируется документация и структура проекта. Код backend/frontend еще не является источником истины; решения фиксируются здесь и в `decisions/`.

## AI agent rule

Перед существенными изменениями AI-агент должен прочитать этот файл и релевантную документацию. Архитектурные решения нельзя менять молча: существенное изменение должно быть отражено в документации и при необходимости в ADR.

## Read next

- `../product/mvp.md`
- `../architecture/overview.md`
- `../architecture/database.md`
- `../content/structure.md`
- `../decisions/`
