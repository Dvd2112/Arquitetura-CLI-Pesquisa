---
title: Boas Práticas e UX de Terminal
description: Engenharia de usabilidade, idempotência, detecção de TTY e 12-factor
---

# Engenharia de Usabilidade (UX de Terminal) e Boas Práticas

Um erro comum em projetos de Sistemas de Informação é ignorar a experiência do usuário pelo fato de a interface ser em modo texto. O design de CLIs modernas obedece a um conjunto rigoroso de boas práticas de engenharia.

## Princípios de UX para Terminal

### 1. Idempotência

A execução repetida de um mesmo comando com os mesmos parâmetros deve produzir o **mesmo estado final** no sistema, sem causar efeitos colaterais indesejados (como duplicação de dados).

```bash
# Comportamento idempotente — executar múltiplas vezes é seguro
$ mkdir -p projeto/src
```

### 2. Detecção de TTY (TeleTypewriter)

O software deve detectar se sua saída está sendo exibida diretamente em uma tela humana ou se está sendo redirecionada para um arquivo/pipeline.

| Contexto | Comportamento esperado |
|----------|----------------------|
| Humano (TTY) | Exibir cores ANSI, spinners, barras de progresso |
| Máquina (pipe/arquivo) | Desabilitar cores, saída limpa (JSON, CSV) |

```bash
# Exemplo: comando adapta saída automaticamente
$ npm install          # Mostra progress bar animado
$ npm install > log.txt  # Saída limpa (sem cores)
```

### 3. Aderência aos Doze Fatores (12-Factor App)

Toda parametrização sensível (tokens de autenticação, chaves de API) **não** deve ser passada por argumentos visíveis no histórico do terminal. A CLI deve ler prioritariamente de **Variáveis de Ambiente** do sistema operacional.

```bash
# ❌ Ruim: token aparece no histórico
$ minha-cli --token=ghp_abc123

# ✅ Bom: token lido de variável de ambiente
$ export MINHA_CLI_TOKEN=ghp_abc123
$ minha-cli
```

### 4. Design de Comandos

- **Consistência** — Mesmos padrões de flags em todos os subcomandos (`-v` sempre significa verbose)
- **Hierarquia** — `verbo-substantivo` ou `ferramenta ação alvo`
- **Previsibilidade** — `--help` e `--version` funcionam em toda CLI
- **Silêncio por padrão** — Mostrar apenas o relevante; usar `-v` para detalhes

### 5. Tratamento de Erros

```bash
# Códigos de saída padronizados (POSIX)
exit 0    # Sucesso
exit 1    # Erro genérico
exit 2    # Uso incorreto (misuse)
exit 130  # Interrompido (Ctrl+C)
```

Mensagens de erro devem ser informativas e, quando possível, sugerir correções:

```
$ git pul
git: 'pul' is not a git command. See 'git --help'.

The most similar command is:
        pull
```

### 6. Saída Estruturada

- **stdout** — Apenas dados/resultados (pode ser pipeado)
- **stderr** — Logs, erros e diagnóstico (não quebra pipes)
- Suporte a `--output json`, `--output yaml` para automação

## Referências

- [CLI Guidelines](https://clig.dev) — The essential CLI design guide
- [12-Factor App](https://12factor.net) — Metodologia para apps modernos
- [POSIX.1-2017 (IEEE Std 1003.1)](https://pubs.opengroup.org/onlinepubs/9699919799/)

Voltar para [[index|página inicial]].
