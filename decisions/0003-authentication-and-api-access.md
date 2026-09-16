# ADR 0003: Authentication and API access

## Status

Accepted

## Context

MedCore использует отдельные frontend и backend репозитории. Нужен простой способ аутентификации пользователей и стабильный контракт доступа к API.

Основная аудитория потенциально активно использует Telegram, поэтому Telegram должен быть удобным основным способом входа. При этом проект не должен полностью зависеть от одного внешнего identity provider.

## Decision

Поддерживать два способа аутентификации:

1. Telegram — основной пользовательский сценарий.
2. Логин + пароль — резервный способ входа.

После успешной аутентификации backend выдает JWT. Frontend отправляет access token в заголовке:

```text
Authorization: Bearer <access_token>
```

Все пользовательские API начинаются с версии:

```text
/api/v1/...
```

Telegram identity обязательно проверяется backend-ом. Данные, пришедшие только от frontend, не считаются доказательством identity.

Django password hashing используется для login/password flow; пароли никогда не хранятся в открытом виде.

## Consequences

- User model должен поддерживать Telegram identity как optional unique identity и обычную Django credential-based авторизацию.
- Один и тот же User в будущем может быть связан с Telegram и одновременно иметь логин/пароль.
- Frontend не зависит от Django session cookies.
- Refresh/access token lifecycle и logout/revocation будут уточнены при реализации backend auth.
- Добавление других identity providers позже не требует менять общий Bearer JWT API contract.
