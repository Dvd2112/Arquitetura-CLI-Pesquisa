---
title: Parsers de Argumentos
description: Bibliotecas e padrões para parsing de argumentos em CLI
---

# Parsers de Argumentos

## Linguagens e Frameworks

### Rust
- [[clap](https://docs.rs/clap)] — Mais popular, derive macros
- [[structopt](https://docs.rs/structopt)] — Agora integrado ao clap

### Go
- [[cobra](https://github.com/spf13/cobra)] — Padrão da indústria (docker, hugo, gh)
- [[flag](https://pkg.go.dev/flag)] — Standard library

### Python
- [[argparse](https://docs.python.org/3/library/argparse.html)] — Standard library
- [[click](https://click.palletsprojects.com)] — Decorators, composição
- [[typer](https://typer.tiangolo.com)] — Type hints automáticos

### Node.js / TypeScript
- [[commander](https://github.com/tj/commander.js)] — Inspirado no Ruby
- [[yargs](https://yargs.js.org)] — Configuração fluente
- [[oclif](https://oclif.io)] — Framework completo (Heroku, Shopify)

## Padrões de Parsing

| Padrão | Exemplo | Descrição |
|--------|---------|-----------|
| GNU/POSIX | `-v`, `--verbose` | Flags curtas e longas |
| Subcomandos | `git commit -m "msg"` | Comandos aninhados |
| Argumentos posicionais | `cp origem destino` | Ordem define significado |
| Ambiente | `DEBUG=true app run` | Config via env vars |

Voltar para [[Arquitetura-Interna|arquitetura interna]] ou [[index|início]].
