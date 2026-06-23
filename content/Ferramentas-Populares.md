---
title: Ferramentas CLI Populares
description: Análise de CLIs famosas e suas arquiteturas
---

# Ferramentas CLI Populares

## git

- **Framework**: Script shell original → C (linha de comando)
- **Subcomandos**: `git commit`, `git push`, `git log`
- **Destaques**: Paginação automática (`less`), cores condicionais

## docker

- **Framework**: cobra (Go)
- **Arquitetura**: Cliente-servidor (dockerd + docker CLI)
- **Destaques**: Output em múltiplos formatos (JSON, tabela)

## kubectl

- **Framework**: cobra (Go)
- **Plugins**: Sistema extensível via `kubectl plugin`
- **Destaques**: Context switching, output customizável

## cargo (Rust)

- **Framework**: clap (Rust)
- **Destaques**: Progress bars nativas, cores, sugestões
- **Subcomandos**: `cargo build`, `cargo run`, `cargo test`

## gh (GitHub CLI)

- **Framework**: cobra (Go)
- **Destaques**: Autenticação integrada, output em Markdown

## Características comuns

- Help automático (`--help`, `-h`)
- Autocomplete (bash/zsh/fish)
- Múltiplos formatos de output
- Códigos de saída padronizados
- Modo silencioso (`-q`) e verboso (`-v`)

Voltar para [[index|página inicial]].
