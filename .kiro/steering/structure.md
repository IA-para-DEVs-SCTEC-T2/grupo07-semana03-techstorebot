---
inclusion: always
---

# Project Structure

This project follows **Clean Architecture**, organized into three layers with strict dependency rules: outer layers depend on inner ones, never the reverse.

## Layer Overview

```
src/
├── api/            # Entry points: HTTP handlers, CLI, webhooks
├── domain/         # Core business logic and entities (no external dependencies)
└── infrastructure/ # External integrations: DB, APIs, messaging, file I/O
```

## Layer Responsibilities

### `api/`
- Receives and validates external input (HTTP requests, CLI args, etc.)
- Calls domain use cases
- Formats and returns responses
- Must NOT contain business logic

### `domain/`
- Contains entities, value objects, use cases, and interfaces (ports)
- Pure Python — no framework or infrastructure imports
- Defines abstract interfaces that infrastructure implements
- This is the most important layer; keep it clean and dependency-free

### `infrastructure/`
- Implements domain interfaces (adapters)
- Handles all I/O: databases, external APIs, file systems, queues
- May import third-party libraries
- Must NOT contain business logic

## Dependency Rules

- `api` → `domain` ✅
- `infrastructure` → `domain` ✅
- `domain` → `api` ❌
- `domain` → `infrastructure` ❌
- `api` → `infrastructure` ❌ (only via domain interfaces)

## Code Style

- Use Python type hints on all function signatures
- Prefer `dataclasses` or `pydantic` models for entities and DTOs
- Use dependency injection to wire infrastructure into domain use cases
- Keep modules small and single-purpose
- Name files after the concept they represent (e.g., `ticket_repository.py`, `faq_service.py`)

## Testing

- Unit tests target the `domain` layer in isolation (no mocks of domain internals)
- Integration tests cover `infrastructure` adapters
- Use `pytest` as the test runner
