# Study Notes — Platform Engineering with Online Boutique

Diário do que eu construo, quebro e aprendo.

---
## Entradas

### T0.3  — Rodar `go build` e `go test` local (antes de qualquer CI)

**O que aprendi:**

| Comando | O que faz |
|---|---|
| `go mod download` | Baixa todas as dependências para `~/go/pkg/mod/` |
| `go mod vendor` | Copia as dependências para a pasta `vendor/` no repositório |
| `go build` | Compila e gera o binário (nome = último segmento do módulo) |
| `go test` | Executa os testes nos arquivos `*_test.go` |
| `git rm -r --cached` | Remove arquivo do tracking sem deletar do disco |

**Sobre dependências:**
- `go.mod` e `go.sum` listam as dependências e versões necessárias

---
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

**O que você aprendeu:**

- **Linter é padrão em TODAS as linguagens** — Go, Python, JS, Java, Rust, etc.
- **`golangci-lint`** valida qualidade do código estaticamente (sem rodar)
- **Pipeline quebra com linter** — é obrigatório em ambiente corporativo
- **6 tipos de issues encontrados:**
    - `errcheck`: erros não checados (responsabilidade do dev)
    - `staticcheck`: imports deprecated, estilo inconsistente
    - `unused`: código morto