---
title: Ferramentas CLI Populares
description: Análise de CLIs famosas e suas arquiteturas
---

# Ferramentas CLI Populares

## git

- **Framework**: Script shell original → C (linha de comando)
- **Subcomandos**: `git commit`, `git push`, `git log`, `git merge`
- **Destaques**: Paginação automática (`less`), cores condicionais, autocomplete

## docker

- **Framework**: cobra (Go)
- **Arquitetura**: Cliente-servidor (dockerd + docker CLI via API REST)
- **Destaques**: Output em múltiplos formatos (JSON, tabela), detecção de TTY

## kubectl

- **Framework**: cobra (Go)
- **Plugins**: Sistema extensível via `kubectl plugin`
- **Destaques**: Context switching, output customizável (`-o json`, `-o yaml`)

## cargo (Rust)

- **Framework**: clap (Rust)
- **Destaques**: Progress bars nativas, cores, sugestões de erro, `did you mean?`
- **Subcomandos**: `cargo build`, `cargo run`, `cargo test`, `cargo publish`

## gh (GitHub CLI)

- **Framework**: cobra (Go)
- **Destaques**: Autenticação integrada via OAuth, output em Markdown, integração com API do GitHub

## Características comuns entre CLIs bem projetadas

- Help automático (`--help`, `-h`) em todos os níveis
- Autocomplete para bash/zsh/fish
- Múltiplos formatos de output (JSON, YAML, tabela)
- Códigos de saída padronizados (POSIX)
- Modo silencioso (`-q`) e verboso (`-v`)
- Variáveis de ambiente para configuração (12-factor)
- Detecção de terminal interativo vs pipe

Voltar para [[index|página inicial]].
