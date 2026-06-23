---
title: Metodologia de Desenvolvimento
description: Roteiro metodológico para o ciclo de desenvolvimento de uma ferramenta CLI
---

# Metodologia de Desenvolvimento — Como Criar uma CLI

Roteiro metodológico padrão para o ciclo de desenvolvimento de uma ferramenta CLI:

## 1. Definição do Escopo e Engenharia de Requisitos

Mapeamento das dores do usuário de terminal e desenho da interface de comandos.

**Atividades:**
- Identificar o problema que a CLI resolve
- Definir comandos, subcomandos e flags (especificação antes de codificar)
- Esboçar a árvore de comandos

```
ferramenta
├── init          — Inicializar projeto
├── build         — Compilar/buildar
│   ├── --release — Modo produção
│   └── --watch   — Modo observação
├── test          — Executar testes
│   ├── --unit    — Testes unitários
│   └── --e2e     — Testes end-to-end
└── deploy        — Publicar
    └── --env     — Ambiente (staging/production)
```

## 2. Escolha da Stack Tecnológica

Alinhamento entre requisitos do sistema e tecnologia:

| Requisito | Stack recomendada |
|-----------|------------------|
| Portabilidade (binário único) | Go ou Rust |
| Integração com ecossistema web | Node.js/TypeScript |
| Prototipagem rápida | Python (Click/Typer) |
| Performance máxima | Rust |
| Facilidade de contratação | Go ou Node.js |

## 3. Modelagem de Dados e Estado Local

Definição de como a CLI armazenará configurações globais na máquina do usuário.

**Especificação XDG Base Directory:**

| Plataforma | Configuração | Dados | Cache |
|------------|-------------|-------|-------|
| Linux | `~/.config/ferramenta/` | `~/.local/share/ferramenta/` | `~/.cache/ferramenta/` |
| macOS | `~/Library/Application Support/` | `~/Library/Application Support/` | `~/Library/Caches/` |
| Windows | `%APPDATA%\ferramenta\` | `%APPDATA%\ferramenta\` | `%LOCALAPPDATA%\ferramenta\` |

Arquivos de configuração em formatos estruturados (JSON, YAML, TOML).

## 4. Implementação

- **Parser primeiro** — Configurar definição de comandos e flags
- **Handlers vazios** — Esboçar estrutura de subcomandos
- **Lógica de negócio** — Implementar funcionalidades
- **Saída formatada** — Adicionar formatação e cores

## 5. Estratégia de Empacotamento e Distribuição

Criação de receitas de instalação para gerenciadores de pacotes:

```bash
# Homebrew (macOS/Linux)
brew install ferramenta

# Windows (Winget)
winget install ferramenta

# Linux (APT/YUM)
apt install ferramenta

# Direto (curl pipe)
curl -sSf https://get.ferramenta.dev | sh
```

## 6. Documentação e Help

- `--help` completo e hierárquico
- Man pages ou `ferramenta help <comando>`
- Exemplos de uso na documentação
- Suporte a `--help` em cada subcomando

> Ver [[Boas-Praticas]] para diretrizes de UX e [[Ferramentas-Populares]] para exemplos reais.

Voltar para [[index|página inicial]].
