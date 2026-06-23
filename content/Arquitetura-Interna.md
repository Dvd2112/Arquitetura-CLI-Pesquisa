---
title: Arquitetura Interna de CLIs
description: Como uma CLI é estruturada internamente
---

# Arquitetura Interna de CLIs

## Padrão Arquitetural

A maioria das CLIs modernas segue o padrão:

```
Entrada (stdin/args)
       ↓
  [Parser de Argumentos]
       ↓
  [Dispatcher de Comandos]
       ↓
  [Lógica de Negócio]
       ↓
  [Output Formatador]
       ↓
Saída (stdout/stderr)
```

## Camadas

### 1. Camada de Entrada
- Parsing de argumentos via `os.Args` / `sys.argv`
- Leitura de stdin para pipes
- Expansão de globs/wildcards

### 2. Camada de Roteamento
- Mapeamento de subcomandos para handlers
- Validação de argumentos obrigatórios
- Help automático

### 3. Camada de Negócio
- Lógica principal do comando
- Chamadas a APIs, sistema de arquivos, etc.

### 4. Camada de Saída
- Formatação de resultados (JSON, YAML, tabela)
- Cores, progress bars, spinners
- Códigos de saída (exit codes)

## [[Parsers-Argumentos|Bibliotecas de parsing]] →

Voltar para [[index|página inicial]].
