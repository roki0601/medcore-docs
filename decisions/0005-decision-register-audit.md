# ADR 0005: Decision register audit

## Status
Accepted

## Correction

Point 97 in `0004-mvp-decision-register.md` was phrased too softly.

Accepted decision:
- images are not embedded directly through Markdown;
- files/images use a separate `Media/File` model;
- binaries live in object storage; PostgreSQL stores metadata/links.

## Verification

- Concrete user answers are recorded as accepted decisions.
- Answers like `later`, `TBD`, `посмотрим`, or skipped questions remain unresolved and must not be invented by AI agents.
- Previously accepted decisions remain valid unless later answers override them.
- WhiteNoise (point 177) is only a current plan, not a locked decision.
- Redis is approved for cache; older statements that Redis is not needed are obsolete.
- Celery remains TBD.

## Priority

Newer explicit ADRs override older conflicting wording.
