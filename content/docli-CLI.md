---
title: docli - CLI Documentadora
description: Implementação prática de uma CLI que documenta alterações em .md via git diff
---

# docli — CLI Documentadora de Código

> **Objetivo do trabalho:** Criar uma CLI básica que execute `git diff`, pegue as diferenças, peça uma descrição ao usuário e gere um arquivo `.md` em um caminho pré-determinado.

O **docli** faz exatamente isso: analisa o `git diff` do projeto, mostra as alterações, pergunta uma descrição e gera um `.md` formatado.

## Instalação

```bash
# No diretório raiz do projeto
pip install -e exemplos/docli/

# Verificar
docli --help
```

## Fluxo de Uso

### 1. `docli init` — Configurar caminho de saída

```bash
$ cd meu-projeto
$ docli init docs/
✔ Configurado! Documentos serão salvos em: /meu-projeto/docs/
```

O caminho é salvo em `.docli/config.json`. Pode ser `docs/`, `documentacao/`, `relatorios/`, etc.

### 2. `docli document` — Gerar .md a partir do git diff

```bash
$ # Faz alterações no código...
$ docli document
────────────────────────────── Alterações Detectadas ──────────────────────────────
Arquivos modificados:
  ✗ src/auth.py
  ✗ src/main.py

2 inserções, 1 deleção

Commits recentes:
abc1234 feat: adiciona login
def5678 fix: corrige timeout

Descreva brevemente o que foi feito:
> Implementa autenticação JWT e corrige timeout da sessão

✔ Documento gerado: /meu-projeto/docs/20250623_143000_auth.md
  Tamanho: 845 caracteres, 28 linhas
```

### 3. `docli log` — Listar documentos gerados

```bash
$ docli log
Documentos em docs/:
┌──────────────────────────────────┬──────────┬──────────────┐
│ Arquivo                          │ Tamanho  │ Modificado   │
├──────────────────────────────────┼──────────┼──────────────┤
│ 20250623_143000_auth.md          │ 845      │ 23/06 14:30  │
│ 20250623_150212_main.md          │ 623      │ 23/06 15:02  │
└──────────────────────────────────┴──────────┴──────────────┘
Total: 2 documento(s)
```

## Exemplo de .md Gerado

```markdown
# Documentação de Alterações

**Data:** 23/06/2025 14:30

## Descrição

Implementa autenticação JWT e corrige timeout da sessão.

## Arquivos Modificados

- `src/auth.py`
- `src/main.py`

## Estatísticas

- Inserções: 2
- Deleções: 1
- Total de arquivos: 2

## Diff

```diff
diff --git a/src/auth.py b/src/auth.py
index abc..def 100644
--- a/src/auth.py
+++ b/src/auth.py
@@ -1,5 +1,6 @@
 def login():
-    return "old"
+    return "new"
+    # JWT token
```

---

*Documento gerado automaticamente por docli em 23/06/2025 14:30*
```

## Arquitetura


[ git diff ] ───────────────────────────────┐
       ↓                                     │
[ main.py ] — Typer (Parser + Dispatcher)     │
       ↓                                     │
[ Pede descrição ao usuário ] ◄───────────────┤
       ↓                                     │
[ Gera .md formatado ] ───────────────────────┘
       ↓
[ Salva em docs/<timestamp>_<arquivo>.md ]
```

### Mapeamento com a [[Arquitetura-Interna|arquitetura CLI]]

| Camada | Componente |
|--------|-----------|
| **Entrada** | `git diff` via subprocess |
| **Parser** | Typer (`@app.command()`) |
| **Dispatcher** | Typer roteia `init`, `document`, `log` |
| **Lógica** | `_diff_stats()`, `_gerar_md()` |
| **Estado** | `.docli/config.json` (saída configurável) |
| **Saída** | Arquivo `.md` + console Rich |
| **Exit codes** | `0` sucesso, `1` erro |

### Tríade POSIX

```bash
# stdout → informações no terminal
$ docli document -d "fix: corrige bug" > saida.txt

# stderr → erros
$ docli document 2> erros.log

# exit codes
$ docli document && echo "ok"  # exit 0 = sucesso
```

## Test Harness

```bash
$ pytest exemplos/docli/tests/ -v

===================== 12 passed =====================
```

Ou via comando embutido:

```bash
$ cd meu-projeto
$ docli init docs/
$ # faz algumas alterações...
$ docli document --desc "teste"  # gera um .md
$ # o harness valida a estrutura do .md gerado
```

## Testes Implementados

| Teste | O que verifica |
|-------|---------------|
| `test_count_insertions` | Estatísticas do diff (inserções) |
| `test_count_deletions` | Estatísticas do diff (deleções) |
| `test_contains_description` | .md gerado contém a descrição do usuário |
| `test_contains_stats` | .md contém inserções/deleções/total |
| `test_generates_valid_markdown` | .md tem formato válido (cabeçalho, code blocks) |
| `test_diff_stats_accuracy` | (integração) git diff real → stats corretas |
| `test_md_contains_diff_content` | (integração) .md contém diff real |
| `test_output_dir_is_respected` | Diretório configurado é usado |

## Código Fonte

```
exemplos/docli/
├── pyproject.toml
├── src/docli/
│   ├── __init__.py
│   └── main.py              # ~100 linhas — CLI completa
└── tests/
    └── test_harness.py       # 12 testes
```

---

> Voltar para [[index|página inicial]].
