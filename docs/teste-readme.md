# Documentação de Alterações

**Data:** 24/06/2026 22:20

## Descrição

Adiciona linha de teste ao README

## Arquivos Modificados

- `README.md`
- `exemplos/docli/pyproject.toml`
- `exemplos/docli/src/docli/main.py`

## Estatísticas

- Inserções: 7
- Deleções: 7
- Total de arquivos: 3

## Diff

```diff
diff --git a/README.md b/README.md
index bcdbbf4..ab391b9 100644
Binary files a/README.md and b/README.md differ
diff --git a/exemplos/docli/pyproject.toml b/exemplos/docli/pyproject.toml
index 3bc5c8a..7b2fe90 100644
--- a/exemplos/docli/pyproject.toml
+++ b/exemplos/docli/pyproject.toml
@@ -13,4 +13,4 @@ docli = "docli.main:app"
 
 [build-system]
 requires = ["setuptools>=68"]
-build-backend = "setuptools.backends._legacy:_Backend"
+build-backend = "setuptools.build_meta"
diff --git a/exemplos/docli/src/docli/main.py b/exemplos/docli/src/docli/main.py
index fa84bb3..70b9f61 100644
--- a/exemplos/docli/src/docli/main.py
+++ b/exemplos/docli/src/docli/main.py
@@ -121,7 +121,7 @@ def init(
     out_path = Path.cwd() / output
     out_path.mkdir(parents=True, exist_ok=True)
 
-    console.print(f"[green]âœ“[/] Configurado! Documentos serÃ£o salvos em: [bold]{out_path}[/]")
+    console.print(f"[green]OK[/] Configurado! Documentos serao salvos em: [bold]{out_path}[/]")
 
 
 @app.command()
@@ -150,7 +150,7 @@ def document(
         recent = _git_log_recent(path)
 
     if not diff.strip():
-        console.print("[yellow]âš  Nenhuma alteraÃ§Ã£o detectada. FaÃ§a alteraÃ§Ãµes no cÃ³digo e tente novamente.[/]")
+        console.print("[yellow]!! Nenhuma alteracao detectada. Faca alteracoes no codigo e tente novamente.[/]")
         raise typer.Exit()
 
     # 3. Mostra resumo
@@ -158,7 +158,7 @@ def document(
     if files:
         console.print("[bold]Arquivos modificados:[/]")
         for f in files:
-            console.print(f"  [cyan]âœ—[/] {f}")
+            console.print(f"  [cyan]>[/] {f}")
 
     stats = _diff_stats(diff)
     console.print(f"\n[dim]{stats['insertions']} inserÃ§Ãµes, {stats['deletions']} deleÃ§Ãµes[/]")
@@ -192,7 +192,7 @@ def document(
 
     # 7. Salva
     filepath.write_text(conteudo, encoding="utf-8")
-    console.print(f"\n[green]âœ“[/] Documento gerado: [bold]{filepath}[/]")
+    console.print(f"\n[green]OK[/] Documento gerado: [bold]{filepath}[/]")
     console.print(f"  Tamanho: {len(conteudo)} caracteres, {len(conteudo.splitlines())} linhas")
 
     return str(filepath)
@@ -205,12 +205,12 @@ def log():
     output_dir = Path.cwd() / cfg["output_dir"]
 
     if not output_dir.exists():
-        console.print("[yellow]âš  Nenhum documento encontrado.[/]")
+        console.print("[yellow]!! Nenhum documento encontrado.[/]")
         return
 
     files = sorted(output_dir.glob("*.md"))
     if not files:
-        console.print("[yellow]âš  Nenhum documento .md encontrado.[/]")
+        console.print("[yellow]!! Nenhum documento .md encontrado.[/]")
         return
 
     table = Table("Arquivo", "Tamanho", "Modificado")

```

---
*Documento gerado automaticamente por docli em 24/06/2026 22:20*
