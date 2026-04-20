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

### T1.2  — Job: `checkout` + `setup-go` + `go build`
- **Actions (`uses:`) vs Commands (`run:`)** são diferentes — não podem misturar no mesmo step
- **`working-directory`** no `defaults` funciona APENAS para `run:`, não para `uses:`
- **Actions sempre rodam na raiz** do repositório — se precisar de arquivo em subpasta, use caminho relativo completo
- **Padrão de mercado** é usar `working-directory` centralizado (`defaults:`), não repetir em cada step