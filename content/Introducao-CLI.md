---
title: Fundamentação Teórica
description: O que é uma CLI, paradigma REPL e contexto histórico
---

# Fundamentação Teórica — O que é uma CLI e seu Contexto Histórico

## Definição

A **Interface de Linha de Comando (CLI — Command Line Interface)** é um mecanismo de interação humana com computadores baseado estritamente na troca de strings de texto através de um terminal. Diferente das Interfaces Gráficas de Usuário (GUI), que dependem de metáforas visuais (janelas, ícones, ponteiros), a CLI opera sob o paradigma de **REPL (Read-Eval-Print Loop)**:

1. **Read** — O sistema lê a entrada do usuário
2. **Eval** — Avalia e executa o comando
3. **Print** — Imprime o resultado na tela
4. **Loop** — Aguarda a próxima instrução

## Componentes básicos de uma CLI

```
$ docker container run --detach --name meu-servidor nginx:latest
   │        │       │       │          │             │
  cmd    subcmd   ação    flag    valor-da-flag    argumento
```

1. **Comando** — O nome do programa ou ação
2. **Subcomandos** — Comandos aninhados (`git commit`, `docker ps`)
3. **Argumentos** — Valores passados para o comando
4. **Flags/Opções** — Modificadores de comportamento (`--verbose`, `-d`)
5. **Valores de flag** — Parâmetros associados a uma flag

## Por que estudar CLI em Sistemas de Informação?

- **Eficiência**: Tarefas repetitivas podem ser automatizadas via scripts e cronjobs
- **Recursos mínimos**: Funciona em ambientes sem interface gráfica (servidores, containers, IoT)
- **Composição**: Comandos podem ser combinados via pipes (`|`)
- **Reprodutibilidade**: Comandos podem ser versionados e auditados
- **Amplamente utilizado**: DevOps, desenvolvimento, administração de sistemas, CI/CD

Voltar para [[index|página inicial]].
