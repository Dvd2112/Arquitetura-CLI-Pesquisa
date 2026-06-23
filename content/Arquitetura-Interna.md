---
title: Arquitetura Interna
description: Pipeline arquitetural, POSIX triad, exit codes e engenharia de CLIs
---

# Engenharia de Software e a Arquitetura Interna de uma CLI

Uma CLI bem projetada não é um emaranhado de funções imperativas — ela segue um **pipeline de dados estruturado** e padronizado por normas internacionais como as especificações **POSIX (IEEE Std 1003.1)**.

## Pipeline Arquitetural Interno

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

## Camadas em Detalhe

### 1. Camada de Ingestão e Tokenização (Lexer/Parser)

Quando a CLI é invocada, o sistema operacional passa uma estrutura contendo os argumentos digitados. O componente **Parser** analisa essa string, valida os tipos de dados, identifica flags curtas (`-d`) ou longas (`--detach`) e mapeia as variáveis no código. Se houver erro de sintaxe, o parser aborta a execução e gera uma mensagem de ajuda dinâmica.

```bash
$ docker container run --detach --name meu-servidor nginx:latest
```

### 2. Roteador de Comandos (Command Dispatcher)

Inspirado no **Design Pattern Command**, esse componente atua como um controlador. Ele recebe a árvore de argumentos validados e despacha a execução para a função de negócio específica responsável por aquele subcomando.

```go
// Exemplo conceitual em Go com Cobra
var rootCmd = &cobra.Command{
    Use:   "ferramenta",
    Run: func(cmd *cobra.Command, args []string) {
        // Dispatcher decide qual handler chamar
    },
}
```

### 3. Núcleo de Lógica de Negócio (Core Logic)

Esta camada é **agnóstica** ao fato de ser uma CLI. Ela contém as regras de negócio do sistema, integrações com APIs REST, acesso a bancos de dados ou manipulação de arquivos. Uma CLI bem projetada permite que a mesma lógica de negócio seja reutilizada por outras interfaces (GUI, API, etc.).

### 4. Gerenciamento de Fluxos Padrão (A Tríade POSIX)

O coração da comunicação de uma CLI reside nos **três descritores de arquivo padrão** abertos pelo Sistema Operacional para cada processo:

| Descritor | Nome | FD | Finalidade |
|-----------|------|----|------------|
| **stdin** | Standard Input | 0 | Entrada de dados (teclado ou pipe) |
| **stdout** | Standard Output | 1 | Saída para dados de sucesso |
| **stderr** | Standard Error | 2 | Canal exclusivo para erros e logs |

> Isolar o **stderr** garante que dados corrompidos ou erros não quebrem o encadeamento de pipes entre programas.

### 5. Códigos de Saída (Exit Codes)

Ao encerrar, toda CLI retorna um número inteiro ao sistema operacional. Por convenção universal:

| Código | Significado |
|--------|-------------|
| **0** | Sucesso absoluto |
| **1** | Erro genérico |
| **2** | Uso incorreto (misuse) |
| **127** | Comando não encontrado |
| **130** | Interrompido (Ctrl+C) |
| **137** | SIGKILL (out of memory) |

Isso permite que scripts tomem decisões lógicas baseadas no resultado:

```bash
if command; then
    echo "Sucesso!"
else
    echo "Falha com código: $?"
fi
```

## [[Parsers-Argumentos|Ecossistema de Linguagens e Frameworks]] →

Voltar para [[index|página inicial]].
