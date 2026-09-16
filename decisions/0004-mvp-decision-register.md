# ADR 0004: MVP decision register

## Status

Accepted / partially TBD

Этот файл фиксирует все 200 обсуждённых пунктов. Если решение не принято, оно явно помечено `TBD`, чтобы его не додумывали разработчики или AI-агенты.

## Product and auth

1. MVP включает регистрацию/вход, предметы, разделы, темы, контентные блоки, карточки, тесты и прогресс — **да**.
2. Архитектура и UI должны допускать несколько предметов сразу, даже если сначала наполнен один.
3. Auth: Telegram или логин+пароль.
4. Основной auth flow — Telegram; login/password — резервный.
5. Bearer JWT с access + refresh — **да**.
6. Refresh token хранится в httpOnly cookie.
7. Нужен blacklist/revocation refresh-токенов.
8. Telegram identity и login/password могут принадлежать одному User.
9. Email оставить в модели на будущее.
10. Из Telegram сохранять минимум `telegram_id` и `username`; остальные поля пока не нужны.
11. Роли: `user`, `editor`, `admin`.
12. Premium — не роль, а отдельная подписка/доступ.
13. `Subscription` создать сразу.
14. Сразу заложить поддержку нескольких тарифов.
15. Заложить возможность trial.

## Subject / Section / Topic / URL

16. Поля Subject — **TBD**.
17. Поля Section — **TBD**.
18. Поля Topic — **TBD**.
19. Область уникальности slug — **TBD**.
20. Использовать человекочитаемые URL со slug — **да**.

## Content blocks

21. `ContentBlock` — универсальный блок темы.
22. Типы блоков хранить в БД, чтобы новые типы можно было создавать без изменения enum/кода.
23. Правила custom blocks — **TBD**.
24. Основной текст хранить в Markdown.
25. Политика raw HTML — **TBD**.
26. Порядок блоков и механизм сортировки — **TBD**, нужно отдельно обсудить.
27. Статусы контента: `draft`, `generated`, `reviewed`, `published`.
28. Разрешено создавать/заливать контент сразу как `published`.
29. `reviewed_by` — нужен.
30. `published_at` — нужен.
31. Полноценное version history на MVP не нужно.
32. `created_at` и `updated_at` нужны у ключевых сущностей.
33. Удаление контента — физическое, без soft delete.
34. Удаление Topic каскадно удаляет его ContentBlock/Flashcard/Quiz и зависимые данные.
35. При удалении Source M2M-связи удаляются; связанные сущности не удаляются.

## Sources

36. Sources optional.
37. Source — отдельная переиспользуемая сущность.
38. Нужны типы источников: учебник, презентация, статья, сайт, методичка, другое.
39. Source metadata: title, author, year, URL, note — заложить.
40. Source может быть связан с Topic, ContentBlock, Flashcard, QuizQuestion.
41. Страницы/слайды на relation-level пока не храним.

## Flashcards

42. Названия полей `question` / `answer` вместо front/back — **TBD, ранее предложено, но явно не подтверждено**.
43. Difficulty у Flashcard — **TBD**.
44. Ручной order у Flashcard — **TBD**.
45. Source у Flashcard — optional, это уже принято выше.
46. Собственный status у Flashcard или наследование от Topic — **TBD**.
47. Flashcard может существовать без Topic: связь с Topic сделать optional на всякий случай.

## Quiz model

48. Несколько Quiz на Topic — **TBD**.
49. Обязательность title у Quiz — **TBD**.
50. Description у Quiz — **TBD**.
51. Один правильный ответ на MVP — **принято ранее, подтверждение не отменено**.
52. Не фиксировать ровно 4 варианта ответа на уровне БД — **принято ранее, не отменено**.
53. Минимум 2 ответа validation — **TBD**.
54. Explanation required/optional — **TBD**.
55. Difficulty у QuizQuestion — **TBD**.
56. Source у QuizQuestion optional — **да, принято через source policy**.
57. Порядок вопросов — **TBD**.
58. Перемешивание ответов — **TBD**.
59. Retake теста без ограничений — да.
60. Лучший результат вычислять из attempts, отдельно не хранить.
61. Каждую QuizAttempt сохранять.
62. Не хранить полный `QuizAttemptAnswer` по каждому ответу; хранить ошибки/неправильные ответы пользователя в более компактной форме. Конкретная модель — **TBD**.
63. Незавершённый тест: прогресс сохранять.
64. Timeout теста не нужен.
65. Проходной процент глобально пока не вводить.

## Topic progress

66. Ручная кнопка «Изучено» — да.
67. Автозавершение темы по тестам — да.
68. Для автозавершения учитывать score, а не только факт прохождения. Конкретный threshold — **TBD**.
69. Если тестов нет, ручная кнопка работает.
70. `UserTopicProgress` хранит прогресс/состояние темы.
71. Статусы: `not_started`, `learning`, `learned`.
72. Отдельный `repeat` для темы пока не нужен; возможно позже только для отдельных вопросов/элементов — **TBD later**.
73. `last_opened_at` нужен.
74. `started_at` и `completed_at` нужны.

## Flashcard progress / FSRS

75. Self-rating карточек в MVP не нужен.
76. Кнопки self-rating, соответственно, не вводим сейчас.
77. FSRS не реализуем сейчас, только задел на будущее.
78. Для будущего FSRS заложить `next_review_at`, `last_reviewed_at`, `repetitions` или эквивалентные данные.
79. `FlashcardReview`: одна текущая запись состояния на пару user+card.
80. Историю оценок/событий карточек предусмотреть позже отдельной таблицей.

## Search / API

81. Search MVP — `icontains`.
82. Искать по `Subject.title`, `Section.title`, `Topic.title`.
83. Возможность поиска по ContentBlock заложить на будущее.
84. Специальный ranking пока не нужен.
85. Нужны фильтры по предмету/разделу.
86. Использовать стандартную DRF pagination.
87. API versioning: `/api/v1/`.
88. REST only; GraphQL не нужен.
89. DRF ViewSets использовать там, где удобно.
90. OpenAPI/Swagger добавить сразу.

## Stack / storage

91. Frontend: Next.js.
92. Backend: Django + DRF.
93. DB: PostgreSQL.
94. Django Admin — CMS.
95. Файлы/изображения хранить в object storage, не в PostgreSQL.
96. Конкретного storage provider пока не фиксировать.
97. Images in Markdown напрямую не использовать как основной механизм.
98. Нужна отдельная `Media/File` модель.
99. Redis нужен сразу как инфраструктура для кэша.
100. Celery — **TBD**, решить по реальной необходимости.
101. Импорт контента не хочется строить вокруг management command; нужен удобный upload/import flow через admin. Конкретная реализация — **TBD**.
102. NotebookLM importer — **TBD later**.
103. NotebookLM/Gemini integration не является пользовательским runtime.
104. Импорт нормализовать в JSON.
105. Повторный импорт должен обновлять существующее, а не плодить дубли; механизм identity/upsert — **TBD implementation**.
106. AI-imported content получает status `generated`.
107. Manual content можно сразу создавать `published`.
108. Review flow поддерживается, но технически не блокирует публикацию в MVP.

## Permissions / admin / paywall

109. Editor может создавать/редактировать контент.
110. Editor не управляет пользователями/системой.
111. Admin — полный доступ.
112. Пока domain admin можно совместить с Django admin/staff/superuser моделью; уточнить позже при необходимости.
113. Учебный контент MVP доступен только авторизованным пользователям.
114. SEO учитывать на будущее, несмотря на закрытый MVP.
115. Paywall проверяется backend-ом, не только frontend-ом.
116. Нужна возможность бесплатного контента (`is_free` или эквивалент).
117. Paywall потенциально может применяться к любому уровню/контенту; точную granularity определить позже.
118. Subscription хранит `starts_at`, `expires_at`.
119. Payment provider пока не фиксировать.

## Logging / security / account

120. Product event analytics в MVP не нужна.
121. Обычные технические Django logs нужны.
122. Auth errors логировать.
123. Importer errors логировать.
124. Sentry — позже.
125. На MVP достаточно Django admin history.
126. Default API permission (`IsAuthenticated` глобально или точечно) — **TBD**.
127. DRF throttling/rate limiting нужен сразу.
128. CORS ограничивать frontend-доменами.
129. Auth endpoints и CSRF/cookie security настроить корректно с учётом JWT + httpOnly refresh cookie.
130. Password reset нужен.
131. Change password нужен.
132. Возможность unlink Telegram заложить на будущее.
133. Self-service account delete нужен.
134. При удалении User его progress/attempts удалять каскадно.
135. Username уникальный.
136. `telegram_id` уникальный.
137. Email optional.
138. MVP locale — русский.
139. В БД время хранить timezone-aware/UTC.

## Frontend UX / admin UX

140. Frontend mobile-first.
141. Dark mode сделать сразу.
142. Базовая accessibility нужна.
143. Механизм сортировки в admin — **TBD**.
144. Bulk import нужен; UX/реализацию обсудить позже.
145. Bulk publish нужен.
146. Preview unpublished Topic — **TBD**.
147. Draft visibility — **TBD**.
148. Slug генерировать автоматически из title.
149. Разрешить manual slug override.
150. Slug можно менять и после публикации; redirect/history пока не решены.
151. Unique constraint для slug — **TBD**.
152. Primary keys: `BigInt`/Django BigAutoField.
153. UUID/public IDs дополнительно пока не нужны.
154. Тип/семантика ordering fields — **TBD**.
155. Timestamps timezone-aware.
156. API: будут и aggregate/detail endpoints, и отдельные endpoints для blocks/cards/quizzes; точные contracts — **TBD**.
157. Lazy loading карточек/тестов — да.
158. Redis заложить сразу для кэша.
159. ETag/Last-Modified — позже.

## Testing / tooling

160. Backend tests: pytest.
161. Coverage target пока не фиксировать.
162. Список обязательных critical tests пока не фиксировать.
163. Frontend tests пока не приоритет.
164. CI, вероятно, GitHub Actions; считать рабочим выбором, но окончательная конфигурация ещё не сделана.
165. Python linting: Ruff.
166. Formatting: Ruff format.
167. Type checking нужен; конкретный tool можно выбрать позже.
168. Package manager: uv.
169. Python 3.12.
170. Django: latest stable на момент старта реализации, не обязательно LTS.
171. Config через `.env`.
172. Docker нужен сразу.
173. `docker-compose` для backend+postgres локально пока не нужен.
174. Prod DB provider не фиксируется: возможен Supabase PostgreSQL или другой managed Postgres.
175. Backend hosting: Render как текущий план.
176. Frontend hosting: Vercel.
177. Admin static: ранее планировался WhiteNoise; пользователь в этом раунде не переопределял — оставить как current plan, не immutable decision.
178. Prod media: object storage — подтверждено пунктом 95.
179. Отдельный staging сейчас не нужен.
180. Django migrations всегда коммитить.
181. Seed command для тестового предмета пока не нужен.
182. Fixtures использовать.
183. Dev admin account provisioning — **TBD**.
184. Secrets management — **TBD implementation**, секреты в репозиторий не коммитить.

## Repository / docs / AI rules

185. Backend README должен содержать setup/env/commands.
186. Frontend README должен содержать setup/env/API URL.
187. `medcore-docs` — источник истины по архитектуре и продуктовым решениям.
188. `AGENTS.md` обязателен для AI-агентов.
189. Крупные решения AI не меняет без ADR/docs update.
190. AI не добавляет зависимости без реальной необходимости.
191. AI не меняет существенную модель данных без обновления docs/ADR.
192. AI не меняет публичный API contract молча.
193. AI не делает optional content fields required без ADR.
194. AI не добавляет новую тяжёлую инфраструктуру без конкретной задачи. Redis является уже одобренным исключением для кэша; Celery/Kafka/Elastic и прочее не добавлять без нового решения.
195. Репозитории остаются раздельными: `medcore-backend`, `medcore-frontend`, `medcore-docs`.
196. Generated/shared API types/client предусмотреть на будущее.
197. Frontend HTTP client (`fetch`/axios/etc.) — **TBD**.
198. Использовать единый формат API errors; точная схема — **TBD**.
199. Pagination response — стандарт DRF.
200. Healthcheck endpoint: `/api/v1/health/`.

## Source of truth rule

Если другой документ противоречит этому decision register, до следующего ADR приоритет имеет более новое явно принятое решение из этого файла. `TBD` не является разрешением AI выбрать вариант самостоятельно: такие пункты требуют отдельного решения пользователя/команды.
