---
title: Estudo de Caso - OpenCode
description: Análise aprofundada do OpenCode como exemplo de CLI com IA agêntica
---

# Estudo de Caso: OpenCode — O Agente de Programação Open Source

## Visão Geral

**OpenCode** (anteriormente conhecido como Claude Code) é uma CLI interativa que funciona como um **agente de programação com IA**. Com mais de **178.000 estrelas** no GitHub, **900 contribuidores** e **7.5 milhões de desenvolvedores mensais**, o OpenCode representa o estado-da-arte na interseção entre CLI e Inteligência Artificial.

```
Repositório: github.com/anomalyco/opencode
Licença: MIT
Stack: TypeScript + Bun
Modelos: 75+ provedores (OpenAI, Anthropic, Google, locais)
```

## 1. Arquitetura de Agentes Múltiplos

O OpenCode implementa um sistema de **múltiplos agentes especializados** que podem ser combinados em uma única sessão:

```
[ Usuário ] ──────────────────────────────────────────────┐
                                                          ↓
┌──────────────────────────────────────────────────────────┐
│               Orquestrador de Sessão                      │
│  (SessionV2 + SessionRunner + SessionExecution)           │
└──────────────────────────────────────────────────────────┘
         │                 │                   │
         ▼                 ▼                   ▼
┌────────────────┐ ┌────────────────┐ ┌────────────────┐
│   Build Agent  │ │   Plan Agent   │ │  Sub-agents    │
│  (Acesso total)│ │  (Somente lei- │ │@general        │
│  Edita arquivos│ │  tura + análise│ │@explore        │
│  Executa bash  │ │  Sem edição)   │ │@scout          │
└────────────────┘ └────────────────┘ └────────────────┘
```

### Agentes Primários

| Agente | Modo | Permissões | Uso |
|--------|------|------------|-----|
| **Build** (padrão) | Primary | `edit: allow`, `bash: allow` | Desenvolvimento geral |
| **Plan** | Primary | `edit: deny`, `bash: deny` | Análise e planejamento |

### Sub-agentes

| Agente | Função |
|--------|--------|
| **@general** | Pesquisas complexas e tarefas multi-passo (pode editar) |
| **@explore** | Exploração rápida do código (somente leitura) |
| **@scout** | Pesquisa em documentação externa e dependências |

### Agentes de Sistema (ocultos)

| Agente | Função |
|--------|--------|
| **compaction** | Compacta contexto longo em resumo |
| **title** | Gera títulos de sessão automaticamente |
| **summary** | Cria resumos de sessão |

## 2. Pipeline Arquitetural com IA

O OpenCode estende o pipeline CLI clássico ([[Arquitetura-Interna|Seção 3]]) com camadas específicas para IA agêntica:

```
[ Entrada do Usuário (linguagem natural) ]
         ↓
[ Parser Híbrido ]
    ├── Comandos (/undo, /init, /share)
    ├── Menções (@general, @explore)
    └── Prompt livre (linguagem natural)
         ↓
[ Coletor de Contexto (System Context) ]
    ├── LSP (Language Server Protocol)
    ├── Estrutura do projeto (árvore de arquivos)
    ├── Git diff e histórico recente
    ├── Configuração do projeto (AGENTS.md)
    └── Sistema operacional e terminal
         ↓
[ Roteador de Agentes ]
    ├── Decide qual agente deve responder
    └── Aplica permissões do agente selecionado
         ↓
[ Motor de Inferência (SessionRunner) ]
    ├── Mantém histórico da sessão
    ├── Gerencia fila de mensagens (Durable Session Input)
    ├── Suporta múltiplos provedores simultâneos
    └── Controle de iterações (max steps)
         ↓
[ Camada de Ferramentas (Tool Registry) ]
    ├── Leitura/Escrita de arquivos
    ├── Bash shell
    ├── Busca (glob, grep)
    ├── Web fetch / Web search
    └── MCP Servers (extensível)
         ↓
[ Sistema de Permissões ]
    ├── ask → Pergunta antes de executar
    ├── allow → Executa automaticamente
    └── deny → Bloqueia a ação
         ↓
[ Saída + Controle de Versão ]
    ├── /undo → Reverte alterações
    ├── /redo → Reaplica alterações
    └── /share → Gera link compartilhável
```

## 3. Mecanismo de Sessão Durável (SessionV2)

Um dos diferenciais arquiteturais do OpenCode é o sistema de **sessão durável**:

```
SessionV2.prompt("faça X")
       ↓
[ 1. Admite session_input (durável no banco) ]
       ↓
[ 2. Agenda SessionExecution.wake(sessionID) ]
       ↓
[ 3. Runner serializa inputs em mensagens visíveis ]
       ↓
[ 4. Executa iterações até o modelo decidir parar ]
       ↓
[ 5. Suporta retry exato (mesmo ID de prompt → reconciliação) ]
```

Isso permite:
- **Resumir sessões**: Continuar de onde parou
- **Retry exato**: Reexecutar com mesmo ID sem duplicar
- **Concorrência**: Múltiplas sessões rodando em paralelo
- **Recuperação de falhas**: Sessão persiste mesmo se o processo cair

## 4. Sistema de Permissões Granular

O OpenCode implementa um dos sistemas de permissão mais sofisticados entre CLIs com IA:

```json
{
  "agent": {
    "build": {
      "permission": {
        "bash": {
          "*": "ask",           // Perguntar para qualquer comando
          "git status *": "allow",  // Exceto git status
          "npm test": "allow"       // E npm test
        },
        "edit": "ask",         // Perguntar antes de editar
        "webfetch": "deny"     // Bloquear web fetch
      }
    }
  }
}
```

Níveis de permissão:
- **allow** — Executa automaticamente (confiança total)
- **ask** — Pergunta ao usuário antes de executar (padrão)
- **deny** — Bloqueia completamente (segurança)

## 5. Integração com LSP (Language Server Protocol)

Diferente de CLIs tradicionais, o OpenCode se integra com **LSPs** para fornecer contexto semântico do código ao modelo de IA:

```
[ Arquivo ] → [ LSP ] → [ Diagnósticos, símbolos, referências ] → [ Contexto da IA ]
```

Isso permite que a IA:
- Entenda tipos e interfaces do projeto
- Veja erros de compilação em tempo real
- Navegue por definições e referências
- Respeite formatação e estilo do projeto

## 6. Sistema de Compartilhamento

O OpenCode introduz o conceito de **sessões compartilháveis** — um avanço significativo para CLIs:

```
/share → https://opencode.ai/s/4XP1fce5
```

Cada sessão gera um link público que permite:
- Compartilhar prompts e respostas com a equipe
- Debug colaborativo de problemas
- Documentação de decisões arquiteturais

> **Nota:** Sessões não são compartilhadas por padrão — o usuário precisa explicitamente executar `/share`.

## 7. Customização Extensível

### Agentes Customizados

Usuários podem criar agentes especializados:

```bash
$ opencode agent create
# → Agora você tem ~/.config/opencode/agents/review.md
```

```markdown
---
description: Revisa código sem fazer alterações
mode: subagent
permission:
  edit: deny
  bash: deny
---

Você é um revisor de código. Foque em segurança e boas práticas.
```

### MCP Servers

Suporte ao protocolo **Model Context Protocol (MCP)** para integrar ferramentas externas:

```
opencoe → MCP Server (banco de dados) → Consulta SQL
opencoe → MCP Server (API externa) → Requisições HTTP
```

### Skills

Sistema de **skills** que adicionam instruções especializadas ao agente (como o `customize-opencode` usado neste projeto).

## 8. Métricas de Impacto

| Métrica | Valor |
|---------|-------|
| Estrelas GitHub | 178.000+ |
| Contribuidores | 900+ |
| Commits | 14.306+ |
| Desenvolvedores mensais | 7.5M+ |
| Licença | MIT |
| Releases | 826+ (v1.17.9 atual) |
| Issues | 5.000+ |
| Pull Requests | 1.100+ |

## 9. Comparação com o Pipeline CLI Clássico

| Aspecto | CLI Tradicional | OpenCode |
|---------|----------------|----------|
| **Entrada** | Comandos exatos (flags POSIX) | Linguagem natural + comandos |
| **Parser** | Léxico rígido (argparse, clap) | Híbrido (NL + comandos `/`) |
| **Roteamento** | Dispatcher fixo (subcomandos) | Agentes dinâmicos permutáveis |
| **Contexto** | Apenas argumentos | LSP + Git + FS + Env |
| **Execução** | Linear (processo único) | Agêntica (loop de iterações) |
| **Saída** | stdout/stderr + exit code | stdout + undo + share |
| **Extensibilidade** | Plugins (limitado) | MCP Servers + Skills + Custom Agents |
| **Memória** | Stateless | Sessão durável + retry exato |
| **Segurança** | Controle do usuário | Sistema de permissões granular |

## 10. Lições para o Design de CLIs com IA

1. **Agentes especializados > agente monolítico**: Separar build (escrita) de plan (leitura) reduz riscos
2. **Sessão durável é fundamental**: Permite retry, undo e compartilhamento
3. **Permissões granulares dão confiança**: Usuários aceitam mais automação quando podem controlar cada tipo de ação
4. **Contexto rico melhora resultados**: LSP + Git + estrutura do projeto dão à IA o necessário para decisões corretas
5. **Extensibilidade through MCP**: Um protocolo aberto de ferramentas permite que a comunidade expanda a CLI indefinidamente
6. **Compartilhamento como feature**: Transformar sessões em links úteis muda o paradigma de "ferramenta individual" para "ferramenta colaborativa"

---

> Este estudo de caso mostra como os princípios da [[Arquitetura-Interna|arquitetura CLI clássica]] evoluem quando combinados com IA, resultando em ferramentas radicalmente mais capazes. Veja o [[Guia-Pratico-CLI-Ollama|Guia Prático]] para implementar conceitos similares.

Voltar para [[IA-CLI|Seção IA + CLI]] ou [[index|página inicial]].
