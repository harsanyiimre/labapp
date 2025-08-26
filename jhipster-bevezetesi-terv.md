# JHipster-alapú keretrendszer – bevezetési terv

Cél: gyors MVP, moduláris alapok, skálázhatóság, OWASP Top 10, felhő/on‑prem/SaaS kompatibilitás, IdP integráció (Keycloak/OpenAM/SAML/OIDC).

Döntési összefoglaló
- Architektúra: moduláris monolit indulásnak (Spring Modulith mintával), később bontható microservice-re.
- Stack: JHipster 8 (Spring Boot 3 + Java 21 + React/TS + PostgreSQL + OAuth2/OIDC).
- Auth: OIDC by default (Keycloak), SAML/OpenAM támogatás brokerrel (Keycloak). Broker nélküli út opcionális.
- Deployment: Docker Compose (lokál/on‑prem kicsi), Kubernetes + Helm (SaaS/felhő), IaC: Terraform (AWS elsődleges).
- Multi‑tenancy: single schema + tenant_id + Postgres RLS (később nagy tenantokra per‑schema).

Fázisok és lépések

Fázis 0 – Előkészítés (1 hét)
- Technikai baseline
  - Verziók: Java 21, Node 20, JHipster 8, Gradle, PostgreSQL 16.
  - Repo: monorepo (backend+frontend), conventional commits, SemVer, ADR-ek.
- Eszközök
  - IntelliJ, Prettier/ESLint, Spotless/Checkstyle, SonarQube CE, Testcontainers.
  - Sec: Dependabot, OWASP Dependency-Check, Trivy (image), Checkov (IaC), ZAP DAST.
- Felhő és on‑prem célok
  - AWS: Terraform skeleton (VPC, EKS, ECR, RDS Postgres, S3, ElastiCache).
  - On‑prem: Docker Compose baseline; K8s (Helm/Kustomize) sablonok.

Fázis 1 – Alap alkalmazás generálása (1–2 hét)
- JHipster generálás
  - Típus: monolith, Auth: OAuth2/OIDC, DB: PostgreSQL, Cache: ehcache (dev) / Redis (prod), React+TS.
  - i18n bekapcsolva, DTO/service layer engedélyezve, OpenAPI export.
- Dev környezet
  - Docker Compose: postgres, redis, minio, keycloak, app.
  - Keycloak dev realm import (JHipster default) + OIDC kliens.
- Alap domain
  - JDL-ből entitások (audit mezők), REST + pagination + search.
- OWASP baseline
  - Security headers, CSRF beállítás, rate limiting az ingress/gateway-ben.

Fázis 2 – Multi‑tenancy v1 (1–2 hét)
- Tenant model
  - tenant táblák + tenant_id minden üzleti entitáson.
  - TenantResolver: subdomain/header/JWT claim alapján.
- Adatvédelem
  - Postgres RLS policy-k tenant_id-re.
  - Flyway migrációk tenant‑tudatosan.
- Backend integráció
  - Hibernate filter/specification tenant_id-re, tranzakciós kontextusban.
  - Claims mapping: IdP csoportok → belső szerepkörök.

Fázis 3 – IdP integrációk és jogosultság (1–2 hét)
- OIDC alapértelmezés (Keycloak)
  - Realm/kliens, szerepkörök, csoportok.
  - Backend: Spring Security OIDC Resource Server; Frontend: react-oidc-context.
- SAML/OpenAM kompatibilitás
  - Keycloak broker SAML↔OIDC híd (az app OIDC-et beszél).
  - Opcionális: közvetlen SAML2 Login a backendben (brokermentes tenantokhoz).
- RBAC/ABAC
  - Belső roles/permissions táblák; PermissionEvaluator/OPA opcionális.
  - Admin UI tenant/role managementhez.

Fázis 4 – Observability és audit (1 hét)
- OpenTelemetry (trace), Prometheus (metrics), Loki/ELK (log), Grafana (dashboard).
- Audit log model és export (tenant- és user‑szint), korrelációs ID végigvezetés.

Fázis 5 – CI/CD és környezetek (1–2 hét)
- GitHub Actions
  - Build+test (backend/frontend), SAST, dep scan, container scan, SBOM.
  - Docker build & push (ECR), Helm deploy dev/staging, PR‑enként preview namespace.
  - ZAP DAST stagingon.
- Helm/Kustomize
  - App chart + values (dev/stage/prod), ExternalDNS, cert-manager, HPA.
- Titkok és konfiguráció
  - AWS Secrets Manager/Vault, mTLS/TLS, rotáció.

Fázis 6 – SaaS és on‑prem csomagolás (1–2 hét)
- SaaS (AWS)
  - EKS + RDS + ElastiCache + S3; backup/DR; SLO/SLI + alerting.
  - Billing/usage metering később (feature flag).
- On‑prem
  - Docker Compose csomag (air‑gapped opció), vagy Helm chart telepítési útmutató.
  - IdP integrációs guide: OIDC és SAML (OpenAM/ADFS/Azure AD példákkal).

Fázis 7 – Mobil alap (1 hét)
- React Native skeleton
  - OIDC (AppAuth), secure storage, certificate pinning opció.
  - OpenAPI‑ból generált kliens; env‑profilok (dev/stage/prod).

Mérföldkövek és kimenetek
- M0: Projekt skeleton + eszközlánc (Fázis 0 vége).
- M1: JHipster monolit + OIDC + alap entitások futnak (Fázis 1).
- M2: Multi‑tenancy RLS‑szel + tenant routing (Fázis 2).
- M3: IdP variációk (SAML broker) + RBAC UI (Fázis 3).
- M4: Observability + audit (Fázis 4).
- M5: CI/CD + preview env + staging deploy (Fázis 5).
- M6: SaaS (AWS) és on‑prem csomagok (Fázis 6).
- M7: RN kliens alap (Fázis 7).

Parancsok és sablonok (példa)

- JHipster CLI
  ```bash
  npm i -g generator-jhipster@latest
  jhipster # interaktív generálás (monolith, OAuth2/OIDC, React, PostgreSQL, Gradle)
  ```

- Docker Compose (dev) – szolgáltatások
  - postgres, redis, minio, keycloak, mailhog (ha kell), app
  - JHipster docker‑compose sablonok + saját minio/rabbitmq fájlok

- Helm deploy (rövid)
  ```bash
  helm repo add myapp ./helm
  helm upgrade --install myapp myapp/ --namespace myapp-dev -f values-dev.yaml
  ```

- Terraform modulok (AWS)
  - vpc, eks, ecr, rds-postgres, s3, elasticache, iam-roles-for-service-accounts

Kockázatok és mitigáció
- Multi‑tenancy összetett: kezdj RLS-szel és integration tesztekkel (Testcontainers + több tenant).
- SAML komplexitás: preferált broker (Keycloak). Brokermentes tenantokhoz külön útvonal, jól dokumentálva.
- Realm‑sprawl a brokerben: tenant-per-realm helyett 1 realm + IdP‑nkénti config, ahol lehet; automatizált provisioning.
- Üzemi méretezés: korai k8s HPA és metrikák, load/perf tesztek.

Elfogadási kritériumok (MVP)
- OIDC login + role alapú jogosultság, 2 IdP integrációs példa (Keycloak + SAML broker).
- 2 tenant izolált adatokkal (RLS), subdomain alapú tenant feloldás.
- CI/CD végig (build/test/scan/release), PR‑preview, staging deploy.
- SaaS (AWS EKS/RDS/S3/ElastiCache) és on‑prem (Compose/Helm) telepítés működik.
- Observability (metrics/log/trace), audit log, OWASP ellenőrzések zöldek.

Ajánlott alapbeállítások JHipsterhez
- React TypeScript, Gradle, PostgreSQL, OAuth2/OIDC.
- DTO/service layer, pagination, MapStruct, Liquibase helyett Flyway (ha RLS policy-kezeléshez szokott a csapat).
- Testcontainers integráció, OpenAPI generálás a frontend/SDK-hoz.

Időkeret (irányadó)
- MVP: 6–8 hét (csapatmérettől függően).
- Production hardening + csomagolás: +2–4 hét.

Megjegyzés
- A terv rugalmas: JHipster monolitból zökkenőmentesen tovább lehet lépni modulith/microservice irányba. A broker (Keycloak) választható komponens: ahol tiltott, biztosítsunk közvetlen OIDC/SAML útvonalat.