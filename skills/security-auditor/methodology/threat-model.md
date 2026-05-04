# Threat Model

## Adversarial Framing
Treat the codebase as if it was written by an opposing agent whose goal was to
introduce vulnerabilities that pass casual review. This means:
- Auth checks that appear present but are bypassable
- Validation that runs but does not block
- Error handlers that silently succeed on failure
- Logic that behaves differently on specific undocumented inputs
- Dependencies that are present but dangerously misconfigured
- Obfuscated or unnecessarily complex code in security-sensitive paths

Do not assume incompetence. Assume intent.

## Attack Surface Classification

### Primary Surfaces — Full Adversarial Pass
These receive the full treatment from checklists/application-code.md.

| Surface | Examples |
|---|---|
| Authentication and authorization | Login, session, token validation, permission checks |
| Data access layer | ORM queries, raw SQL, file reads/writes, cache access |
| API layer | Route handlers, middleware, request parsing, response shaping |
| External service calls | HTTP clients, webhooks, third-party SDK usage |
| Business logic | Anything that makes decisions about money, access, or data |
| Input handling | All points where external data enters the application |

### Secondary Surfaces — Single Dedicated Pass
These receive the lighter treatment from checklists/infra-and-config.md.

| Surface | Examples |
|---|---|
| Infrastructure config | Docker, docker-compose, nginx, reverse proxy config |
| CI/CD pipelines | GitHub Actions, build scripts, deploy scripts |
| Environment and secrets | .env files, secret references, key management |
| Frontend / client-side | JS bundles, client-side auth checks, exposed data |

## Layer Weight Table
When classifying attack surfaces in the scope map, use this table.

| Layer | Weight | Rationale |
|---|---|---|
| Backend auth + data access | Full adversarial | Highest blast radius |
| API layer + external calls | Full adversarial | Primary attack entry points |
| Business logic | Full adversarial | Intent-based attacks concentrate here |
| Input handling | Full adversarial | Injection surface |
| Frontend / client-side | Standard review | Lower blast radius than backend |
| Infra + config | Single pass | Important but not primary |
| CI/CD + deploy scripts | Single pass | Supply chain surface |
