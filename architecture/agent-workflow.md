# OMP / Coding Agent Workflow

## Status

Planned / working design. Это описание целевого workflow для разработки MedCore через Oh My Pi (OMP) с корпоративной coding-моделью. Конкретные OMP config/model IDs и команды будут настроены позже по фактической локальной конфигурации.

## Goal

Использовать корпоративный `Qwen3-Coder-Next-NVFP4` как бесплатную основную рабочую модель для большей части реализации проекта.

Ключевые продуктовые и архитектурные решения остаются за человеком. Coding agent получает высокую автономию в реализации утверждённых решений, но низкую автономию в изменении продукта/архитектуры.

## Planned roles

На старте одна и та же модель Qwen используется в разных независимых ролях:

- **Executor / Primary** — пишет код, тесты, migrations, запускает проверки, работает с Git/PR.
- **Reviewer** — независимо проверяет SPEC compliance, correctness, domain boundaries, security, transactions, migrations, tests и complexity. Предпочтительно read-only.
- **Advisor / Watchdog** — наблюдает за работой и сигнализирует о scope/architecture/correctness проблемах. Предпочтительно read-only.
- **Human (owner) + architecture discussion** — принимает business/product/architecture/TBD/security-sensitive решения и выполняет финальный review чувствительных изменений.

Reviewer по возможности должен стартовать с чистым контекстом и получать SPEC/docs/diff/tests, а не reasoning Executor, чтобы уменьшить корреляцию ошибок одной модели.

## Intended OMP mapping

Предполагается использовать нативные возможности OMP вместо написания собственного orchestrator:

```text
our concept           OMP mechanism
Executor              primary/task agent
Reviewer              bundled/custom task agent
Advisor               Advisor/Watchdog runtime
advisor guardrails     WATCHDOG.md / WATCHDOG.yml
engineering rules      root AGENTS.md
local domain rules     nested apps/<domain>/AGENTS.md
role model routing     modelRoles
isolated work          Git worktrees where useful
parallel read/review   task/subagents where useful
hard safety guards     extensions/hooks
Git/PR operations      shell/git/gh
external validation    GitHub Actions
```

Exact OMP configuration remains TBD until the installed version/config and corporate model registration are inspected.

## Minimal initial setup

Не включать максимальную агентскую сложность сразу.

Первая версия:

```text
1 Primary Executor (Qwen)
1 Watchdog/Advisor (Qwen, read-only)
1 Reviewer (Qwen, read-only)
Git safety guard
GitHub Actions CI
Human-controlled merge
```

Не включать сразу:

- swarm/parallel implementation writers;
- несколько advisors без необходимости;
- auto-merge;
- auto-deploy;
- unlimited persistence/retry loops;
- write access для Advisor/Reviewer.

## Task lifecycle

Целевой lifecycle:

```text
SPEC READY
   ↓
READ TARGETED CONTEXT
   ↓
UNDERSTAND
   ↓
PLAN
   ↓
SCOPE / RISK CHECK
   ↓
IMPLEMENT
   ↓
LOCAL VERIFY
   ↓
EXECUTOR SELF-REVIEW
   ↓
INDEPENDENT REVIEWER
   ↓
FIX FINDINGS
   ↓
FINAL VALIDATION
   ↓
COMMIT + PUSH FEATURE BRANCH
   ↓
CREATE / UPDATE PR
   ↓
GITHUB ACTIONS
   ↓
CI failure -> fix/retry (bounded)
   ↓
READY / NEEDS HUMAN REVIEW
   ↓
HUMAN MERGE
```

## Context loading

Agent не должен автоматически загружать весь `medcore-docs` для каждой мелкой задачи. Использовать targeted context:

1. root project/engineering instructions;
2. relevant accepted ADR/decision-register sections;
3. relevant architecture docs;
4. nearest domain `AGENTS.md`;
5. current task SPEC;
6. existing implementation patterns and tests.

Если task/spec конфликтует с accepted ADR/decision register, конфликт эскалируется, а не трактуется как молчаливое новое решение.

## Planning rule

Перед изменениями Executor формирует короткий implementation plan и проверяет:

- goal;
- affected domain;
- acceptance criteria;
- likely affected files;
- relevant existing patterns;
- unresolved questions.

План не требует ручного подтверждения, если он не вводит новых product/architecture решений.

## Decision autonomy

### GREEN — agent decides independently

Примеры implementation-level решений:

- local variable/function naming;
- internal function decomposition;
- ORM optimization (`select_related`, `prefetch_related`) при сохранении semantics;
- queryset implementation;
- test fixture/mocking details;
- imports/formatting;
- локальный refactoring непосредственно затронутого кода без изменения поведения;
- другие очевидные implementation details в рамках существующих conventions.

### YELLOW — agent may decide, but reports it

Несущественные implementation choices, которые полезно видеть в final report (например локальная enum/constraint/index implementation), если они не меняют product semantics и не конфликтуют с docs.

### RED — STOP / decision required

Agent не принимает самостоятельно:

- новое business rule;
- закрытие любого `TBD`;
- изменение accepted ADR;
- изменение domain ownership/boundaries;
- новую cross-domain dependency;
- изменение public API вне SPEC;
- существенное изменение data model, не следующее из SPEC;
- новый infrastructure component;
- новую внешнюю dependency без явно оправданной необходимости;
- неопределённое security-sensitive решение;
- изменение auth/token semantics;
- изменение paywall/subscription semantics;
- ослабление validation/tests ради прохождения проверки;
- изменение/переписывание migration history;
- расширение scope, необходимое только потому, что текущая реализация так удобнее.

При RED condition agent возвращает структурированный `BLOCKED` report и не делает speculative implementation.

## Scope rule

Основной принцип реализации — **smallest coherent diff**.

Разрешён полный минимальный набор изменений, необходимый для законченного use case (model/migration/service/API/tests и т.д.), но запрещены unrelated refactoring и архитектурные перестройки.

## Verification

Ожидаемый локальный validation pipeline (точные команды будут зафиксированы после skeleton/tooling setup):

```text
ruff format/check
ruff lint
chosen type checker
pytest
python manage.py check
migration consistency checks
```

Проверки желательно запускать инкрементально после существенных этапов, а не только один раз в конце.

## Failure / retry policy

Не допускать бесконечных agent loops.

Рабочее правило: максимум 3 самостоятельные попытки исправить одну и ту же проблему/CI failure.

После повторных неудач agent обязан переоценить root cause. После лимита — Advisor/STOP вместо бесконечного patching.

## Reviewer protocol

Reviewer должен искать реальные дефекты в таком приоритете:

1. SPEC compliance;
2. correctness;
3. business invariants;
4. security;
5. transactionality;
6. domain boundaries;
7. DB constraints/migrations;
8. API compatibility;
9. tests;
10. performance;
11. unnecessary complexity;
12. style only when materially relevant.

Severity:

```text
BLOCKER
HIGH
MEDIUM
LOW
NIT
```

Executor автоматически обязан исправлять BLOCKER/HIGH/MEDIUM либо аргументированно эскалировать. LOW/NIT не должны запускать бесконечный polish-loop.

Reviewer не должен сам переписывать implementation; он выдаёт findings, Executor исправляет.

## Advisor / Watchdog priorities

Планируемый `WATCHDOG.md` должен особенно следить за:

- попытками закрыть TBD;
- scope creep;
- нарушением domain dependency direction;
- `content -> learning` dependency;
- знанием content models о внутренних деталях Subscription;
- использованием Django signals для core business flow;
- неправильными transaction boundaries;
- появлением Repository/UnitOfWork/EventBus/других abstractions без необходимости;
- public API changes вне SPEC;
- ослаблением tests/validation;
- architecture changes без принятого решения.

Advisor/Watchdog на старте read-only.

## Git workflow

Выбран автономный вариант: agent сам выполняет механическую Git/PR работу до готового PR.

```text
main
  ↓
feat/<TASK-ID>-<slug>
  ↓
implementation
  ↓
tests/review fixes
  ↓
commit
  ↓
push feature branch
  ↓
create/update PR
  ↓
GitHub Actions
  ↓
agent fixes CI if needed
  ↓
ready for human review
  ↓
human merge
```

Agent НЕ получает право merge в `main`.

## Git hard guards

Планируется OMP extension/hook, который технически блокирует опасные действия, а не только просит модель их не выполнять.

Минимально блокировать/защищать:

- direct push to `main`;
- `gh pr merge` / auto merge;
- force push, особенно main;
- destructive `git reset --hard` без явного разрешения;
- destructive `git clean` без явного разрешения;
- запись/commit secrets (`.env`, credentials и т.п.);
- потенциально другие protected paths/commands после настройки.

Agent разрешено создавать branch, commit, push feature branch, создавать/обновлять PR и читать CI status/logs.

## GitHub Actions

CI является независимым детерминированным gate и должен повторять ключевые локальные проверки:

- Ruff;
- type checking;
- pytest;
- Django checks;
- migration checks;
- дополнительные проверки по мере появления проекта.

Agent может читать CI failure, исправлять код и повторно push-ить в feature branch в пределах retry policy.

## Risk levels

Рабочая классификация:

### LOW

Простые read endpoints, serializers, admin display, selectors и аналогичные низкорисковые изменения.

### MEDIUM

Models/migrations, business services, progress logic, import logic и аналогичные изменения состояния/данных.

### HIGH

Auth, permissions, payments, paywall/subscription access, account deletion, destructive migrations, security-sensitive code, сложные cross-domain transactions.

HIGH всегда должен завершаться `NEEDS_HUMAN_REVIEW`, даже если tests/reviewer/CI проходят.

## Task terminal states

Использовать три основных состояния:

- `DONE` — реализация и проверки завершены, нет обязательных unresolved findings.
- `BLOCKED` — требуется решение/невозможно продолжить без нарушения guardrails.
- `NEEDS_HUMAN_REVIEW` — реализация технически завершена, но изменение чувствительное/высокорисковое.

## Final report

Executor должен завершать работу компактным структурированным отчётом:

```text
STATUS: DONE | BLOCKED | NEEDS_HUMAN_REVIEW

Implemented:
- ...

Tests added:
- ...

Validation:
- pytest: PASS/FAIL
- ruff: PASS/FAIL
- typecheck: PASS/FAIL
- django check: PASS/FAIL
- migrations: PASS/FAIL

Implementation decisions:
- ...

Docs changed:
- ... / none

Remaining issues:
- ... / none

PR:
- ...
```

Для BLOCKED вместо speculative implementation указывать decision required, why it blocks, relevant constraints и возможные варианты без самостоятельного выбора продуктового решения.

## Definition of Done direction

Task считается законченной только когда:

- SPEC и acceptance criteria выполнены;
- нет unresolved BLOCKER/HIGH/MEDIUM findings;
- необходимые tests добавлены;
- local validation проходит;
- migrations корректны;
- нет unrelated diff/debug/secrets/случайных TODO;
- docs синхронизированы, если изменение этого требует;
- PR создан/обновлён и CI проходит;
- для HIGH-risk изменений выполнен human review до merge.

## To configure later

Перед реальной настройкой нужно проверить локальную установку OMP:

- `omp --version`;
- фактический user/project OMP config;
- как зарегистрирован корпоративный provider/model;
- точный model ID `Qwen3-Coder-Next-NVFP4`;
- доступные model roles в установленной версии;
- доступность/синтаксис Advisor/Watchdog;
- hooks/extensions API установленной версии;
- GitHub CLI authentication/permissions;
- затем создать реальные `.omp` config/agents, `WATCHDOG.md`, git guard и GitHub Actions workflow.
