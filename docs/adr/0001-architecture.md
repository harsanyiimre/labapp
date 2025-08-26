# 0001 – Architektúra és alap stack

- Status: Accepted
- Date: 2025-08-26

## Context and Problem Statement
Gyors MVP-t és skálázható alapokat szeretnénk, erős biztonsági fókusz mellett (OWASP Top 10), felhő/on‑prem/SaaS kompatibilitással és IdP integrációval (OIDC/SAML).

## Decision Drivers
- Gyors indulás, alacsony komplexitás kezdetben
- Későbbi mikroservice bontás lehetősége
- Sztenderd, bevált stack és toolchain
- Több‑bérlős (multi‑tenant) adatvédelem és elkülönítés

## Decision Outcome
- Architektúra: moduláris monolit indulásnak (Spring Modulith mintával), később bontható microservice-re.
- Stack: JHipster 8 (Spring Boot 3 + Java 21 + React/TS + PostgreSQL + OAuth2/OIDC).
- Auth: OIDC by default (Keycloak), SAML/OpenAM támogatás brokerrel (Keycloak), opcionálisan broker nélküli út.
- Deployment: Docker Compose (lokál/on‑prem kicsi), Kubernetes + Helm (SaaS/felhő), IaC: Terraform (AWS elsődleges).
- Multi‑tenancy: single schema + tenant_id + Postgres RLS (később nagy tenantokra per‑schema).

## Consequences
### Positive
- Gyors MVP és egységes fejlesztői élmény (JHipster ökoszisztéma).
- Komponensek modulárisítása megkönnyíti a jövőbeli szétbontást.
- IdP variációk rugalmasan kezelhetők (Keycloak broker).

### Negative
- Multi‑tenancy és RLS bevezetése plusz komplexitást hoz a migrációkban és tesztelésben.
- Brokermentes SAML útvonal külön karbantartást igényelhet.

## Links
- Bevezetési terv: jhipster-bevezetesi-terv.md