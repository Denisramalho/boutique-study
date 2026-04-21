# Study Notes — Platform Engineering with Online Boutique

Diário do que eu construo, quebro e aprendo.

---
## Fase 0 — Setup

### T0.3  — Rodar `go build` e `go test` local (antes de qualquer CI)

**O que aprendi:**

| Comando              | O que faz                                                   |
| -------------------- | ----------------------------------------------------------- |
| `go mod download`    | Baixa todas as dependências para `~/go/pkg/mod/`            |
| `go mod vendor`      | Copia as dependências para a pasta `vendor/` no repositório |
| `go build`           | Compila e gera o binário (nome = último segmento do módulo) |
| `go test`            | Executa os testes nos arquivos `*_test.go`                  |
| `git rm -r --cached` | Remove arquivo do tracking sem deletar do disco             |

**Sobre dependências:**
- `go.mod` e `go.sum` listam as dependências e versões necessárias

---
## Fase 1 — Fundamentos GitHub Actions
### T1.2  — Job: `checkout` + `setup-go` + `go build`

 **O que aprendi:**
- **Actions (`uses:`) vs Commands (`run:`)** são diferentes — não podem misturar no mesmo step
- **`working-directory`** no `defaults` funciona APENAS para `run:`, não para `uses:`
- **Actions sempre rodam na raiz** do repositório — se precisar de arquivo em subpasta, use caminho relativo completo
- **Padrão de mercado** é usar `working-directory` centralizado (`defaults:`), não repetir em cada step
---
### T1.4 — Cache de dependências

 **O que aprendi:**

- `actions/setup-go@v6` já vem com **cache integrado** — não precisa action separada
- **`cache-dependency-path`** aponta para `go.sum` — o arquivo de lock
- **Como funciona:** Hash de `go.sum` → se não mudou, usa cache; se mudou, baixa de novo
- **Economia:** 30-60 segundos por workflow (multiplicado por múltiplos serviços = ganho real)
---
### T1.5 — Linting e qualidade

**O que aprendi:**

- **Linter é padrão em TODAS as linguagens** — Go, Python, JS, Java, Rust, etc.
- **`golangci-lint`** valida qualidade do código estaticamente (sem rodar)
- **Pipeline quebra com linter** — é obrigatório em ambiente corporativo
- **6 tipos de issues encontrados:**
    - `errcheck`: erros não checados (responsabilidade do dev)
    - `staticcheck`: imports deprecated, estilo inconsistente
    - `unused`: código morto
---
### T1.6 — Matrix: Go 1.21, 1.22, 1.23

**O que aprendi:**

- **Matrix permite rodar o mesmo job em múltiplas versões** — Go 1.21, 1.22, 1.23 em paralelo
- Benefício: Detecta incompatibilidades com versões mais novas ANTES do usuario atualizar
- Padrão de mercado: Testar com versão mínima suportada + latest + próxima (se beta)
- GitHub Actions: Cada combinação roda em paralelo (não sequencial) — testes mais rápidos
- Usar ${{ matrix.go-version }} no setup-go — passa a versão dinamicamente

---
### T1.7 — Separar triggers `push`, `pull_request` e `workflow_dispatch`

**O que aprendi:**

- **Diferentes triggers = diferentes contextos:**
    - `push` → code chegou na main (produção)
    - `pull_request` → review ainda está aberto (feedback para dev)
    - `workflow_dispatch` → disparado manualmente (debugging, releases)
- **Padrão de mercado:** Usar triggers separados para controlar quando cada workflow roda
- **`push: branches: [main]`** — evita executar em todas as branches (economia de minutos)
- **`pull_request`** — padrão automático sem filtro (todo PR disparando é OK)
- **`workflow_dispatch`** — permite disparar manualmente via GitHub UI
- **Não misturar contextos** — main pode ter comportamento diferente (ex: deploy), PR não
- **Cada trigger pode ter ambiente/secrets diferentes** → segurança

---

### T1.8 — `concurrency:` cancelando runs antigos no mesmo PR

**O que aprendi:**

- **`concurrency:` agrupa runs** — mesma PR = mesmo grupo
- **`cancel-in-progress: true`** — cancela runs anteriores quando nova inicia
- **Economia:** 10+ minutos de CI por push subsequente (desenvolvedor não fica esperando)
- **NUNCA cancele em produção (main):**
    
    ```yaml
    cancel-in-progress: ${{ github.ref != 'refs/heads/main' }}
    ```
    
- **Por que é perigoso em main:** Deploy pode ser interrompido no meio, migrations quebram, rollback falha
- **Padrão de mercado:** Grandes empresas (Google, Meta, AWS) só usam em PRs, protegem main
- **Grupo recomendado:** `${{ github.workflow }}-${{ github.ref }}` (separa por branch)
- **Resultado:** Desenvolvedor sempre vê o teste **do código mais recente**, não de código desatualizado

---
### T1.9 — Pin de actions por SHA (e documentar por quê)

**O que aprendi:**

- **Pinnar = substituir `@v6` por `@sha`** — hash imutável de 40 caracteres

```markdown
  # ❌ Antes (versionado, pode mudar)
  - uses: actions/checkout@v6
  
  # ✅ Depois (imutável, seguro)
  - uses: actions/checkout@1af3b93b6815bc44a9784bd300feb67ff0d1eeb3 # v6
```

- **Por quê:** Supply chain seguro — atacante não consegue overwrite a tag v6 com malware
- **Como encontrar:** GitHub Release page → "Commit SHA" ou `gh api` (v6.4, v9.2.0 têm SHAs diferentes)
- **Padrão de mercado:** Todas as empresas com compliance (SLSA, supply chain) usam SHA-pinned actions
- **Não é chato:** Adicione comentário `# v6` pra manter legível — GitHub Actions mostra em linters
- **Todas as actions:** checkout, setup-go, golangci-lint, docker/build, deploy, etc. — todas recebem SHA
- **Resultado:** Seu workflow é auditável e imutável — você controla exatamente qual versão roda
---
## Fase 2 — Docker + Registry

### T2.1 — Build local do Dockerfile; medir tamanho

- Utilizei a imagem base `golang:1.26
- Tamanho final -> 1.94GB

---
### T2.2 — Reescrever como multi-stage com `distroless`

**O que aprendi:**

- **Multi-stage build = 2+ `FROM`s no mesmo Dockerfile** — primeiro stage compila, segundo stage copia só o binário
  ```dockerfile
  FROM golang:1.26 as build      # Stage 1: compilação
  RUN go build -o app            # gera binário
  
  FROM distroless/static         # Stage 2: runtime (sem Go, sem tools)
  COPY --from=build /app /       # copia só o binário
  ```

  - **distroless é "almost nothing"** — sem shell, sem gcc, sem libc dinâmica. Apenas binário + mínimo essencial
- **Redução de tamanho:** 1.94 GB (golang base) → 38.7 MB (distroless) = **98% menor**
- **Binário deve ser estático:** usar `CGO_ENABLED=0` para garantir que Go não depende de libc dinâmica
- **Por que distroless:**
    - Imagem menor = pull mais rápido, menos armazenamento
    - Menos código = menos surface de ataque (segurança)
    - Distroless não tem shell, dificulta exploits
- **Padrão de mercado:** todas as empresas com compliance (supply chain) usam multi-stage + distroless
- **Desvantagem:** sem shell, sem ferramentas de debug (trade-off entre segurança e debugabilidade)
---
---
### T2.2 — Reescrever como multi-stage com `distroless`

**O que aprendi:**

- **Multi-stage build = 2+ `FROM`s no mesmo Dockerfile** — primeiro stage compila, segundo stage copia só o binário
- **distroless é "almost nothing"** — sem shell, sem gcc, sem libc dinâmica. Apenas binário + mínimo essencial
- **Redução de tamanho:** 1.94 GB (golang base) → 38.7 MB (distroless) = **98% menor**
- **Binário deve ser estático:** usar `CGO_ENABLED=0` para garantir que Go não depende de libc dinâmica
- **Por que distroless:** menor tamanho, menos surface de ataque (segurança), sem shell = menos exploits
- **Padrão de mercado:** todas as empresas com compliance (supply chain) usam multi-stage + distroless

---
### T2.3 — Workflow roda `docker build` em PRs (sem push)

**O que aprendi:**

- **`docker build` em CI valida reproducibilidade** — garante que Dockerfile está correto antes de merge
- **Em PRs = sem push** — só valida que a imagem constrói, não precisa autenticar
- **Erro comum:** contexto do build — `docker build -f path/Dockerfile context` precisa que arquivos estejam no `context`
- **Solução:** ou mude contexto (`docker build ... src/service/`) ou use `working-directory`
- **Paralelo com Go:** `docker build` roda em paralelo com `go build + test + golangci-lint` — nenhum depende de outro
- **Resultado:** feedback rápido em ~60s (todos os 3 jobs em paralelo)

---
### T2.4 — Push para GHCR em merge na main, tag com SHA curto

**O que aprendi:**

- **GHCR = GitHub Container Registry** — registro nativo do GitHub, autenticação com `GITHUB_TOKEN`
- **Publicar SÓ em main** — use `if: github.ref == 'refs/heads/main'` para rodar job só na main branch
- **Autenticação:** `docker login ghcr.io -u ${{ github.actor }} --password-stdin` com `secrets.GITHUB_TOKEN`
- **Tag com SHA curto:** `${GITHUB_SHA:0:7}` (7 primeiros chars) = mais legível que 40 caracteres
- **Nome da imagem:** `ghcr.io/OWNER/image-name:tag` — precisa do owner/org, não é opcional
- **Padrão de mercado:** SHA em main = dev consegue testar; semver (v1.0.0) em releases = prod
- **Resultado:** imagem disponível em GHCR para testes imediatos após merge

---
### T2.5 — Tag `latest` + semver em git tags

**O que aprendi:**

- **Git tags disparam workflow:** `on: push: tags: ['v*']` roda APENAS quando você faz `git tag v1.0.0`
- **Múltiplas tags no Docker:**
	- v1.2.3 → ghcr.io/app:v1.2.3 (exato)
	- ghcr.io/app:1.2 (minor)
	- ghcr.io/app:1 (major) 
	- ghcr.io/app:latest (última release)
 **Extrair versão da tag:** `TAG=${GITHUB_REF#refs/tags/}` remove prefixo `refs/tags/`
- **Persistir variáveis entre steps:** use `echo "VAR=value" >> $GITHUB_ENV` (shell vars não persistem)
- **Acessar variáveis persistidas:** `${{ env.VAR }}` em steps seguintes
- **Estratégia:** main → SHA (dev), tag → semver (prod)
- **Resultado:** releases oficiais com versionamento semântico, fácil de rastrear

---
