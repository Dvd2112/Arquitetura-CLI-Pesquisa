---
title: Boas Práticas em CLI
description: Design e usabilidade de interfaces de linha de comando
---

# Boas Práticas em CLI

## Design de Comandos

1. **Consistência** — Mesmos padrões de flags em todos os subcomandos
2. **Hierarquia** — `verbo-substantivo` ou `ferramenta ação alvo`
3. **Previsibilidade** — `--help` e `--version` em toda CLI
4. **Silêncio por padrão** — Só mostrar o que for relevante

## UX no Terminal

- **Cores** com moderação (respeitar `NO_COLOR`)
- **Progresso** em operações longas (spinner, progress bar)
- **Sugestões** quando comando não encontrado (did you mean?)
- **Confirmação** antes de operações destrutivas

## Tratamento de Erros

```bash
# Códigos de saída padronizados
exit 0  # Sucesso
exit 1  # Erro genérico
exit 2  # Uso incorreto (misuse)
exit 130 # Interrompido (Ctrl+C)
```

## Saída

- **stdout** para dados/resultados
- **stderr** para logs e erros
- Suporte a `--output json`, `--output yaml`
- Piping amigável (desativar cores/progress quando pipe)

## Referências

- [CLI Guidelines](https://clig.dev) — The essential CLI design guide
- [Command Line Interface Guidelines](https://www.commandlinefu.com)

Voltar para [[index|página inicial]].
