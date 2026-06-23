---
title: Guia Prático - CLI com Ollama
description: Tutorial passo-a-passo para criar uma CLI inteligente usando Ollama e Python
---

# Guia Prático: Construindo Sua Própria CLI com IA Local (Ollama)

Este guia ensina a construir uma CLI inteligente do zero, usando **Python** com o framework **Typer** e **Ollama** como motor de IA local. A ferramenta será capaz de traduzir linguagem natural em comandos shell, explicar comandos e operar como um agente simples.

## Pré-requisitos

```bash
# Python 3.10+
python --version

# Ollama instalado e rodando
ollama --version
ollama pull llama3

# Pip para dependências
pip install typer rich requests
```

## Passo 1: Estrutura do Projeto

```
meu-cli-ia/
├── cli.py              # Ponto de entrada da CLI
├── engine.py           # Comunicação com Ollama
├── safety.py           # Camada de segurança
├── context.py          # Coletor de contexto do terminal
└── pyproject.toml      # Configuração do projeto
```

## Passo 2: Configuração da CLI com Typer

**`cli.py`** — Definição dos comandos usando Typer (framework moderno baseado em Click com type hints):

```python
import typer
from rich.console import Console
from rich.markdown import Markdown
from engine import query_ollama
from safety import validate_command, contains_sensitive_data
from context import collect_context

app = typer.Typer()
console = Console()

@app.command()
def ask(
    prompt: str = typer.Argument(..., help="O que você quer fazer?"),
    model: str = typer.Option("llama3", "--model", "-m", help="Modelo Ollama"),
    execute: bool = typer.Option(False, "--execute", "-e", help="Executar o comando gerado"),
):
    """Traduz linguagem natural em um comando shell."""
    contexto = collect_context()
    system_prompt = f"""Você é um assistivo de terminal. Gere APENAS o comando shell.
Sistema: {contexto['os']}
Diretório: {contexto['cwd']}
Idioma: português
Regras:
- Retorne SOMENTE o comando, sem explicações
- Use flags padrão POSIX
- Prefira comandos seguros e não-destrutivos
- Se a solicitação for destrutiva, comece com # ATENÇÃO:"""

    resposta = query_ollama(system_prompt, prompt, model)
    comando = resposta.strip().strip("```").strip()

    if comando.startswith("# ATENÇÃO"):
        console.print(f"[bold yellow]{comando}[/]")
        if not typer.confirm("Confirma execução?"):
            raise typer.Exit()

    if contains_sensitive_data(prompt):
        console.print("[bold red]Erro: O prompt contém dados sensíveis[/]")
        raise typer.Exit()

    console.print(Markdown(f"```bash\n{comando}\n```"))

    if execute:
        valid, motivo = validate_command(comando)
        if not valid:
            console.print(f"[bold red]Comando bloqueado: {motivo}[/]")
            raise typer.Exit()
        import subprocess
        result = subprocess.run(comando, shell=True, capture_output=True, text=True)
        console.print(result.stdout)
        if result.returncode != 0:
            console.print(f"[red]Erro (código {result.returncode}): {result.stderr}[/]")

@app.command()
def explain(
    comando: str = typer.Argument(..., help="Comando para explicar"),
    model: str = typer.Option("llama3", "--model", "-m"),
):
    """Explica um comando shell detalhadamente."""
    system_prompt = "Explique o seguinte comando shell em português, detalhando cada flag e argumento:"
    resposta = query_ollama(system_prompt, comando, model)
    console.print(Markdown(resposta))

@app.command()
def chat(
    model: str = typer.Option("llama3", "--model", "-m"),
):
    """Modo interativo (REPL) com a IA."""
    console.print("[bold cyan]Modo chat Ollama. Digite 'sair' para encerrar.[/]")
    historico = []
    while True:
        prompt = typer.prompt("Você")
        if prompt.lower() in ("sair", "exit", "quit"):
            break
        historico.append(f"Usuário: {prompt}")
        contexto = "\n".join(historico[-6:])
        resposta = query_ollama(contexto, prompt, model)
        console.print(f"[green]IA:[/] {resposta}")
        historico.append(f"IA: {resposta}")

if __name__ == "__main__":
    app()
```

## Passo 3: Motor de Inferência (Conexão com Ollama)

**`engine.py`** — Comunicação com a API REST do Ollama:

```python
import requests
from typing import Optional

OLLAMA_URL = "http://localhost:11434/api/generate"

def query_ollama(
    system_prompt: str,
    user_prompt: str,
    model: str = "llama3",
    timeout: int = 30,
) -> str:
    """Envia um prompt para o Ollama e retorna a resposta."""
    payload = {
        "model": model,
        "prompt": f"{system_prompt}\n\n{user_prompt}",
        "stream": False,
        "options": {
            "temperature": 0.3,  # Baixa temperatura = mais preciso
            "max_tokens": 500,
        },
    }
    try:
        response = requests.post(
            OLLAMA_URL,
            json=payload,
            timeout=timeout,
        )
        response.raise_for_status()
        return response.json()["response"].strip()
    except requests.exceptions.ConnectionError:
        return "Erro: Ollama não está rodando. Execute 'ollama serve' primeiro."
    except Exception as e:
        return f"Erro na consulta: {e}"
```

## Passo 4: Camada de Segurança (Safety Rails)

**`safety.py`** — Filtros heurísticos para prevenir execução de comandos perigosos:

```python
import re

COMANDOS_BLOQUEADOS = [
    r"rm\s+(-rf?\s+)?\/",
    r">\s*/dev/sda",
    r"mkfs\.",
    r"dd\s+if=.*of=/dev/sd",
    r":\(\)\s*\{.*:\)\s*\};",  # Fork bomb
    r"chmod\s+777\s+/",
    r"wget.*\|\s*bash",
    r"curl.*\|\s*bash",
]

PADROES_SENSIVEIS = [
    r"(?i)(senha|password|token|api_key|secret)",
    r"(sk-|pk-)[a-zA-Z0-9]{20,}",  # Chaves OpenAI
    r"ghp_[a-zA-Z0-9]{36}",  # Tokens GitHub
    r"AKIA[0-9A-Z]{16}",  # Chaves AWS
]

def validate_command(comando: str) -> tuple[bool, str]:
    """Valida se um comando é seguro para execução."""
    for padrao in COMANDOS_BLOQUEADOS:
        if re.search(padrao, comando):
            return False, f"Comando destrutivo detectado: {padrao}"
    return True, ""

def contains_sensitive_data(texto: str) -> bool:
    """Verifica se o texto contém dados sensíveis."""
    for padrao in PADROES_SENSIVEIS:
        if re.search(padrao, texto):
            return True
    return False
```

## Passo 5: Coletor de Contexto

**`context.py`** — Inspeciona o ambiente do usuário para enriquecer o prompt da IA:

```python
import os
import platform
import subprocess
from pathlib import Path

def collect_context() -> dict:
    """Coleta informações do ambiente para contexto."""
    contexto = {
        "os": platform.system(),
        "os_version": platform.version(),
        "cwd": os.getcwd(),
        "shell": os.environ.get("SHELL", "unknown"),
        "user": os.environ.get("USER", "unknown"),
        "terminal": os.environ.get("TERM", "unknown"),
    }

    # Arquivos no diretório atual (limitado a 10)
    try:
        files = list(Path.cwd().iterdir())[:10]
        contexto["files"] = [f.name for f in files]
    except:
        contexto["files"] = []

    # Git status se estiver em um repositório
    try:
        branch = subprocess.run(
            ["git", "branch", "--show-current"],
            capture_output=True, text=True, timeout=5
        ).stdout.strip()
        if branch:
            contexto["git_branch"] = branch
    except:
        pass

    return contexto
```

## Passo 6: Executando a CLI

```bash
# Modo consulta (seguro) — mostra o comando sem executar
$ python cli.py ask "listar todos os containers docker parados"
# docker ps --filter "status=exited"

# Modo execução — executa após confirmação
$ python cli.py ask "listar todos os containers docker parados" --execute
# docker ps --filter "status=exited"
# Confirma execução? [y/N]: y

# Explicar um comando
$ python cli.py explain "find . -name '*.py' -exec wc -l {} +"
# Explica cada flag: find, -name, -exec, wc -l

# Modo interativo (REPL)
$ python cli.py chat
# Modo chat Ollama. Digite 'sair' para encerrar.
# Você: como ver o espaço em disco?
# IA: df -h

# Com modelo customizado
$ python cli.py ask "comprimir todos os logs" --model mistral
```

## Passo 7: Empacotamento e Distribuição

**`pyproject.toml`:**

```toml
[build-system]
requires = ["setuptools>=68"]
build-backend = "setuptools.backends._legacy:_Backend"

[project]
name = "meu-cli-ia"
version = "0.1.0"
description = "CLI inteligente com IA local via Ollama"
requires-python = ">=3.10"
dependencies = [
    "typer>=0.9",
    "rich>=13.0",
    "requests>=2.31",
]

[project.scripts]
meu-cli = "cli:app"
```

Instalação:

```bash
# Modo desenvolvimento
pip install -e .

# Agora pode chamar diretamente
meu-cli ask "listar arquivos modificados hoje"
```

## Arquitetura Final

```
[ Usuário ] → cli.py (Typer)
                  ↓
           [ context.py ] — Coleta info do ambiente
                  ↓
           [ engine.py ] — http://localhost:11434/api/generate
                  ↓
           [ Ollama ] → llama3 / mistral / codellama
                  ↓
           [ safety.py ] — Filtra comandos destrutivos
                  ↓
           [ Execução via subprocess ou exibição ]
```

## Pontos de Extensão

1. **Múltiplos provedores**: Adicione suporte a OpenAI e Anthropic no `engine.py`, com fallback automático
2. **Cache local**: Armazene respostas frequentes em SQLite para reduzir latência
3. **Plugins**: Permita que usuários adicionem ferramentas personalizadas ao contexto da IA
4. **Modo agente**: Implemente um loop que executa, avalia saída e re-tenta com correção automática
5. **Histórico por projeto**: Salve o contexto entre sessões usando o diretório `.meu-cli/historico.json`

---

> Voltar para [[IA-CLI|Seção IA + CLI]] ou [[index|página inicial]].
