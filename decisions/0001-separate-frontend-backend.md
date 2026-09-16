# ADR 0001: Separate frontend and backend

## Status
Accepted

## Decision

Используем отдельный frontend и backend:

- Next.js — frontend;
- Django + DRF — backend;
- PostgreSQL — основная БД.

## Why

- доменная логика, пользователи, прогресс и будущая монетизация остаются в Django;
- Django Admin используется как внутренний CMS;
- frontend можно развивать независимо;
- архитектура остается привычной и понятной для проекта с большим количеством контентных и пользовательских сущностей.
