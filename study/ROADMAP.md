# Platform Engineering Roadmap — Progresso

**Início:** 19/04/2026
**Fase atual:** 1

---

## Fase 0 — Setup

- [x] T0.1 — Criar repo `boutique-study`; copiar `src/productcatalogservice/` + `protos/demo.proto`
- [x] T0.2 — Criar `README.md` e `NOTES.md`
- [x] T0.3 — Rodar `go build` e `go test` local (antes de qualquer CI)

## Fase 1 — Fundamentos GitHub Actions

- [x] T1.1 — Workflow `hello.yml` com `echo` em `on: push`
- [x] T1.2 — Job: `checkout` + `setup-go` + `go build`
- [x] T1.3 — Job separado `go test` com `needs:`
- [x] T1.4 — `actions/cache` para módulos Go (medir antes/depois)
- [x] T1.5 — `golangci-lint` como job paralelo
- [x] T1.6 — Matrix: Go 1.21, 1.22, 1.23
- [x] T1.7 — Separar triggers `push` (main), `pull_request` e `workflow_dispatch`
- [x] T1.8 — `concurrency:` cancelando runs antigos no mesmo PR
- [x] T1.9 — Pin de actions por SHA (e documentar por quê)

## Fase 2 — Docker + Registry

- [x] T2.1 — Build local do Dockerfile; medir tamanho
- [x] T2.2 — Reescrever como multi-stage com `distroless`
- [x] T2.3 — Workflow roda `docker build` em PRs (sem push)
- [x] T2.4 — Push para GHCR em merge na main, tag com SHA curto
- [ ] T2.5 — Tag `latest` + semver em git tags
- [ ] T2.6 — Multi-arch (amd64 + arm64) via buildx + QEMU
- [ ] T2.7 — Cache de layers `type=gha`

## Fase 3 — Supply Chain Security

- [ ] T3.1 — Dependabot para Go modules, Docker, Actions
- [ ] T3.2 — `dependency-review-action` em PRs
- [ ] T3.3 — CodeQL para Go
- [ ] T3.4 — Trivy/Grype escaneando imagem; falhar em HIGH/CRITICAL
- [ ] T3.5 — SBOM via `anchore/sbom-action`
- [ ] T3.6 — Assinar imagem com Cosign keyless (OIDC)
- [ ] T3.7 — SLSA provenance attestation
- [ ] T3.8 — Gitleaks/Trufflehog detectando secrets

## Fase 4 — Release Automation

- [ ] T4.1 — Adotar Conventional Commits
- [ ] T4.2 — `release-please` abrindo PR de release
- [ ] T4.3 — Merge do release: tag + release notes + imagem semver
- [ ] T4.4 — Environments `staging` e `production` com required reviewers

## Fase 5 — Padrões avançados de CI

- [ ] T5.1 — Extrair build/test Go em reusable workflow (`workflow_call`)
- [ ] T5.2 — Composite action local em `.github/actions/`
- [ ] T5.3 — Passar outputs entre jobs
- [ ] T5.4 — Matriz dinâmica via `fromJSON()`
- [ ] T5.5 — Mover reusable workflow para repo `platform-ci-templates`

## Fase 6 — Monorepo multi-serviço

- [ ] T6.1 — Adicionar `currencyservice` (Node.js)
- [ ] T6.2 — Path filters (só builda o que mudou)
- [ ] T6.3 — Matriz dinâmica gerada a partir do diff
- [ ] T6.4 — Reusable workflow parametrizado por linguagem
- [ ] T6.5 — Adicionar `emailservice` (Python) e `adservice` (Java)
- [ ] T6.6 — Mudança em `protos/` rebuilda todos
- [ ] T6.7 — Merge queue (`merge_group`)

## Fase 7 — Qualidade e testes

- [ ] T7.1 — Integration tests com `testcontainers`
- [ ] T7.2 — Smoke test via `docker-compose`
- [ ] T7.3 — Coverage report (codecov/coveralls)
- [ ] T7.4 — `buf` lint + breaking-change detection em `protos/`
- [ ] T7.5 — Contract testing básico entre serviços

## Fase 8 — IaC e cluster

- [ ] T8.1 — Subir `kind`/`k3d` num workflow; `kubectl apply`
- [ ] T8.2 — Repo `platform-infra` com Terraform (GKE ou EKS)
- [ ] T8.3 — CI Terraform: fmt, validate, tflint, tfsec/checkov, plan no PR
- [ ] T8.4 — Apply gated em environment `production`
- [ ] T8.5 — OIDC GitHub → cloud (sem chave estática)
- [ ] T8.6 — Remote state com lock

## Fase 9 — GitOps e CD

- [ ] T9.1 — Repo `boutique-manifests` com Kustomize (base + overlays)
- [ ] T9.2 — Workflow abre PR no repo de manifests quando imagem é publicada
- [ ] T9.3 — Instalar ArgoCD no cluster
- [ ] T9.4 — App-of-apps pattern
- [ ] T9.5 — Experimentar Flux em branch/cluster separado; comparar
- [ ] T9.6 — Progressive delivery com Argo Rollouts (canary)

## Fase 10 — Observabilidade

- [ ] T10.1 — Instrumentar serviço com OpenTelemetry (traces + metrics)
- [ ] T10.2 — Stack Prometheus + Grafana + Tempo
- [ ] T10.3 — Dashboard RED por serviço
- [ ] T10.4 — 2-3 SLOs com error budget
- [ ] T10.5 — Alertas (webhook/Discord)
- [ ] T10.6 — Dashboards as code

## Fase 11 — Policy & Compliance

- [ ] T11.1 — Kyverno/OPA: policy "imagem precisa estar assinada"
- [ ] T11.2 — `conftest` validando manifests no CI
- [ ] T11.3 — Policy: `resources.limits` obrigatório
- [ ] T11.4 — Admission controller bloqueando sem SBOM/provenance

## Fase 12 — Golden Path / Developer Platform

- [ ] T12.1 — Template repository para "novo microserviço Go"
- [ ] T12.2 — Instalar Backstage local; cadastrar serviços
- [ ] T12.3 — Software template: formulário → repo + PR em manifests
- [ ] T12.4 — External Secrets Operator + secret store
- [ ] T12.5 — TechDocs no Backstage

## Fase 13 — Capstone

- [ ] T13.1 — PR dispara lint/test/security/build
- [ ] T13.2 — Merge publica imagem assinada + SBOM + provenance
- [ ] T13.3 — Workflow abre PR automático em `boutique-manifests`
- [ ] T13.4 — Argo Rollouts faz canary
- [ ] T13.5 — Promoção decidida por métrica
- [ ] T13.6 — Notificação Slack/Discord em cada estágio
- [ ] T13.7 — Rollback automático em violação de SLO
- [ ] T13.8 — Vídeo/GIF/post documentando a arquitetura

---

## Ritmo sugerido

| Fase | Tasks | Tempo estimado |
|---|---|---|
| 0 | 3 | 1 dia |
| 1 | 9 | 1 semana |
| 2 | 7 | 1 semana |
| 3 | 8 | 1 semana |
| 4 | 4 | 3-4 dias |
| 5 | 5 | 1 semana |
| 6 | 7 | 2 semanas |
| 7 | 5 | 1 semana |
| 8 | 6 | 1-2 semanas |
| 9 | 6 | 2 semanas |
| 10 | 6 | 1 semana |
| 11 | 4 | 3-4 dias |
| 12 | 5 | 1-2 semanas |
| 13 | 8 | 2 semanas |

**Total:** ~83 tasks, 4-5 meses num ritmo de 5-8h/semana.

---
