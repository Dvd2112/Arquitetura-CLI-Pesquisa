# CLIs: Conceitos, Arquitetura, Inteligência Artificial e o Projeto docli

> **Site da pesquisa (GitHub Pages):** https://dvd2112.github.io/Arquitetura-CLI-Pesquisa/  
> **Repositório da pesquisa:** https://github.com/Dvd2112/Arquitetura-CLI-Pesquisa  
> **Repositório do docli:** https://github.com/Dvd2112/docli

---

## 1. O que é uma CLI?

**CLI** (Command Line Interface) é um mecanismo de interação humano-computador baseado estritamente na troca de strings de texto através de um terminal. Diferente das interfaces gráficas (GUI), que dependem de metáforas visuais como janelas e ícones, a CLI opera sob o paradigma **REPL (Read-Eval-Print-Loop)**:

1. **Read** — O sistema lê a entrada do usuário
2. **Eval** — Avalia e executa o comando
3. **Print** — Imprime o resultado na tela
4. **Loop** — Aguarda a próxima instrução

### Componentes básicos de uma CLI

```
$ docker container run --detach --name meu-servidor nginx:latest
   │        │       │       │          │             │
  cmd    subcmd   ação    flag    valor-da-flag    argumento
```

1. **Comando** — O nome do programa ou ação principal
2. **Subcomandos** — Comandos aninhados (`git commit`, `docker ps`)
3. **Argumentos** — Valores passados para o comando
4. **Flags/Opções** — Modificadores de comportamento (`--verbose`, `-d`)
5. **Valores de flag** — Parâmetros associados a uma flag

### Por que estudar CLI?

- **Eficiência**: Tarefas repetitivas são automatizadas via scripts e cronjobs
- **Recursos mínimos**: Funciona em servidores, containers e IoT sem GUI
- **Composição**: Comandos são combinados via pipes (`|`)
- **Reprodutibilidade**: Comandos podem ser versionados e auditados
- **Onipresença**: DevOps, desenvolvimento, administração de sistemas, CI/CD

---

## 2. Arquitetura Interna de uma CLI

Uma CLI bem projetada segue um **pipeline de dados estruturado** padronizado por normas POSIX (IEEE Std 1003.1).

### Pipeline Arquitetural

```
[ Entrada do Usuário ]
         ↓
[ Parser de Argumentos (Lexer/Tokenizer) ]
         ↓
[ Roteador de Comandos (Command Dispatcher) ]
         ↓
[ Núcleo de Lógica de Negócio (Core Logic) ]
         ↓
[ Gerenciamento de Fluxos de Saída ]
         ↓
[ stdout / stderr + Exit Code ]
```

### Camadas

1. **Parser** — Analisa os argumentos, identifica flags curtas (`-d`) e longas (`--detach`), valida tipos e gera mensagens de ajuda em caso de erro sintático.

2. **Dispatcher** — Inspirado no padrão **Command**, atua como controlador. Recebe a árvore de argumentos validados e despacha a execução para a função de negócio correta.

3. **Core Logic** — Camada agnóstica à interface. Contém regras de negócio, integrações com APIs, bancos de dados ou manipulação de arquivos. Pode ser reutilizada por outras interfaces (GUI, API).

### Tríade POSIX

| Descritor | Nome | FD | Finalidade |
|-----------|------|----|------------|
| **stdin** | Standard Input | 0 | Entrada de dados (teclado ou pipe) |
| **stdout** | Standard Output | 1 | Saída para dados de sucesso |
| **stderr** | Standard Error | 2 | Canal exclusivo para erros e logs |

Isolar o stderr garante que erros não interrompam o encadeamento de pipes.

### Códigos de Saída (Exit Codes)

| Código | Significado |
|--------|-------------|
| **0** | Sucesso |
| **1** | Erro genérico |
| **2** | Uso incorreto (misuse) |
| **127** | Comando não encontrado |
| **130** | Interrompido (Ctrl+C) |

```bash
if command; then
    echo "Sucesso!"
else
    echo "Falha com código: $?"
fi
```

---

## 3. CLI e Inteligência Artificial

A evolução dos **LLMs (Large Language Models)** inaugurou as **AUI (Agentic User Interfaces)** — CLIs assistidas por IA que compreendem intenção em linguagem natural.

### Mecanismos de Integração

#### Natural Language to Command (NL2Cmd)
O usuário descreve a ação e a IA gera o comando exato:

```
Usuário: "encontre todos os PDFs modificados nos últimos 3 dias e mova para backup"
IA: find . -name "*.pdf" -mtime -3 -exec mv {} ./backup/ \;
```

#### Agentes Autônomos
Ciclo fechado de execução: a IA roda o comando, intercepta stdout/stderr, diagnostica erros e reescreve o comando automaticamente.

### Ferramentas AI-CLI

| Ferramenta | Abordagem | Offline | Código Aberto |
|-----------|-----------|---------|--------------|
| GitHub Copilot CLI | NL2Cmd + Explain | ❌ | ❌ |
| ShellGPT (sgpt) | NL2Cmd via pipe | ✅ (Ollama) | ✅ |
| Claude CLI | Agente autônomo | ❌ | ❌ |
| OpenCode | Agente autônomo + ferramentas IA | ⚠️ | ✅ |

### Arquitetura Expandida com IA

```
[ Interface do Usuário ]
    ├── Linguagem natural
    └── Comando tradicional
           ↓
[ Orquestrador de Entrada ]
    ├── Detecta modo (NL vs comando puro)
    └── Decide fluxo (IA ou parsing direto)
           ↓
[ Camada de Inferência ]
    ├── Context Collector
    │   ├── Diretório atual
    │   ├── Variáveis de ambiente
    │   ├── Histórico de comandos
    │   └── Sistema operacional
    ├── Montagem de Prompt
    ├── Provider Router
    │   ├── Ollama (local)
    │   ├── OpenAI API
    │   └── Anthropic API
    └── Safety Rails
           ↓
[ Execução ou Sugestão ]
    ├── Modo seguro: exibe e aguarda confirmação
    └── Modo agente: executa e avalia resultado
```

### Desafios e Riscos

- **Alucinações sintáticas**: IA pode inventar flags inexistentes
- **Privacidade**: Dados enviados a APIs de terceiros podem violar LGPD
- **Perda de controle**: Scripts gerados por IA sem compreensão humana

**Mitigações**: Preferir modelos locais (Ollama), usar `--dry-run`, validar com `--help`, filtrar segredos do contexto.

---

## 4. docli — CLI Documentadora de Código

O **docli** (repositório oficial: https://github.com/Dvd2112/docli) é uma CLI prática que automatiza a documentação de alterações em código usando `git diff`. Foi desenvolvido como parte de um trabalho acadêmico sobre arquitetura de CLIs.

### Funcionamento

1. Executa `git diff` no diretório do projeto
2. Mostra os arquivos modificados e estatísticas
3. Pede uma descrição textual ao usuário
4. Gera um arquivo `.md` formatado no diretório de saída configurado

### Instalação

**Pré-requisitos:** Python >= 3.10, pip

```bash
# Via pip (a partir do código-fonte)
git clone https://github.com/Dvd2112/docli.git
cd docli
pip install .

# Ou em modo editável (desenvolvimento)
pip install -e .

# Verificar instalação
docli --help
```

### Comandos

| Comando | Descrição |
|---------|-----------|
| `docli init <caminho>` | Configura diretório de saída dos .md |
| `docli document` | Gera .md a partir do git diff |
| `docli splash` | Mostra animação decorativa |
| `docli log` | Lista documentos gerados |

### Fluxo de Uso

#### 1. `docli init` — Configurar diretório

```bash
$ cd meu-projeto
$ docli init docs/
✔ Configurado! Documentos serão salvos em: /meu-projeto/docs/
```

O caminho é salvo em `.docli/config.json`.

#### 2. `docli document` — Gerar documentação

```bash
$ # Faça alterações no código primeiro
$ docli document
──────────────── Alterações Detectadas ────────────────
Arquivos modificados:
  > src/auth.py
  > src/main.py

2 inserções, 1 deleção

Commits recentes:
abc1234 feat: adiciona login

Descreva brevemente o que foi feito:
> Implementa autenticação JWT

✔ Documento gerado: docs/20250624_143000_auth.md
  Tamanho: 845 caracteres, 28 linhas
```

Flags opcionais:
- `--name "meu-doc"` — Nome personalizado do arquivo
- `--desc "descrição"` — Descrição inline (pula o prompt)
- `--verbose` — Exibe o diff completo no terminal

#### 3. `docli log` — Listar documentos

```bash
$ docli log
Documentos em docs/:
┌──────────────────────────┬──────────┬──────────────┐
│ Arquivo                  │ Tamanho  │ Modificado   │
├──────────────────────────┼──────────┼──────────────┤
│ 20250624_143000_auth.md  │ 845 bytes│ 24/06 14:30  │
│ 20250624_150212_main.md  │ 623 bytes│ 24/06 15:02  │
└──────────────────────────┴──────────┴──────────────┘
Total: 2 documento(s)
```

### Exemplo de Documento Gerado

```markdown
# Documentação de Alterações

**Data:** 24/06/2025 14:30

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

*Documento gerado automaticamente por docli em 24/06/2025 14:30*
```

### Arquitetura do docli


[ git diff ] ──────────────────────┐
       ↓                           │
[ main.py — Typer (Parser + Disp.)] │
       ↓                           │
[ Pede descrição ao usuário ] ◄─────┤
       ↓                           │
[ Gera .md formatado ] ────────────┘
       ↓
[ Salva em docs/<timestamp>_<nome>.md ]
```

Mapeamento com a arquitetura CLI:

| Camada | Componente docli |
|--------|-----------------|
| **Entrada** | `git diff` via subprocess |
| **Parser** | Typer (`@app.command()`) |
| **Dispatcher** | Typer roteia `init`, `document`, `log` |
| **Lógica** | `_diff_stats()`, `_gerar_md()` |
| **Estado** | `.docli/config.json` |
| **Saída** | Arquivo `.md` + console Rich |
| **Exit codes** | `0` sucesso, `1` erro |

### Código-Fonte

```
docli/
├── pyproject.toml          # Configuração do pacote
├── src/docli/
│   ├── __init__.py
│   ├── main.py             # ~270 linhas — CLI completa
│   ├── analyzer.py         # Análise de diff
│   ├── docstore.py         # Armazenamento de documentos
│   └── harness.py          # Harness de teste
└── tests/
    └── test_harness.py     # 12 testes automatizados
```

### Testes

```bash
# Instalar dependências de teste
pip install pytest

# Executar test suite
pytest tests/ -v
```

---

## 5. Conclusão

A CLI não é uma tecnologia do passado — é uma interface viva que evoluiu do shell UNIX para os agentes de IA modernos. O `docli` exemplifica uma CLI tradicional bem construída (Python + Typer + Rich), contrastando com ferramentas agênticas como OpenCode e Claude CLI que representam a nova geração de terminais inteligentes.
