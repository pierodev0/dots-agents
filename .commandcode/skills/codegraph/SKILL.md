---
name: codegraph
description: "Quick reference for CodeGraph CLI — traverse symbols, source, callers, and dependencies."
---

# CodeGraph

## ⚡ tl;dr — 5 golden rules

1. **`codegraph status` es tu primer paso** — Verificá si el proyecto está indexado antes de usar cualquier comando.
2. **`codegraph explore` es tu comando principal** — Varios símbolos + relaciones en 1 solo call.
3. **Usá `--max-files N` en `explore`** — Arrancá con `--max-files 8` para evitar truncamiento.
4. **Si el proyecto no está indexado** → `codegraph init` primero. Sin `.codegraph/` no funciona nada.
5. **Si ves "edited since last index sync"** → solo esos archivos están stale. Usá `read_file` para ellos, el resto del índice está fresco.

---

## 🚀 Setup — un solo paso

Si el proyecto **nunca se indexó**, esto lo crea:

```bash
shell_command: codegraph init
description: "Init CodeGraph index for this project"
```

> ⚠️ Esto puede tardar unos segundos dependiendo del tamaño del proyecto. Se corre **una sola vez**.

---

## 🏃 Quick Reference (Command Code)

```bash
# ⭐ Entender un flujo (source + relaciones cruzando archivos)
shell_command: codegraph explore "query" --max-files 8
description: "Explore symbols with relations"

# 📖 Source de un símbolo específico
shell_command: codegraph node FunctionName
description: "Get source and callers of a symbol"

# 📞 Quién llama a X
shell_command: codegraph callers FunctionName -l 50
description: "Find callers"

# 💥 Blast radius (qué se rompe si cambio X)
shell_command: codegraph impact FunctionName
description: "Analyze blast radius"

# 📁 Estructura del proyecto
shell_command: codegraph files --filter src --format flat --no-metadata
description: "Project file tree"

# 🩺 Estado del índice
shell_command: codegraph status
description: "Check index health"
```

---

## 🔄 Antes de confiar en el índice, sincronizá

El file watcher puede laggear ~1 segundo. Después de editar archivos, forzá un sync:

```bash
shell_command: codegraph sync
description: "Force CodeGraph index sync"
```

Luego `codegraph status` para verificar que está fresco.

---

## 📋 Todos los comandos

| Objetivo | Comando | Ejemplo |
|---|---|---|
| **Source + relaciones** ⭐ | `codegraph explore <query> --max-files N` | `codegraph explore "timer session" --max-files 6` |
| Source de 1 símbolo | `codegraph node <symbol>` | `codegraph node useTimer` |
| Quién llama a X | `codegraph callers <symbol>` | `codegraph callers saveToStorage -l 50` |
| Qué llama X | `codegraph callees <symbol>` | `codegraph callees useExerciseForm` |
| Blast radius | `codegraph impact <symbol>` | `codegraph impact useRoutineStore` |
| Buscar símbolos (fuzzy) | `codegraph query <pattern>` | `codegraph query "saveTo.*"` |
| File tree | `codegraph files [flags]` | `codegraph files --filter src --format flat` |
| Estado del índice | `codegraph status` | — |
| Forzar re-sync | `codegraph sync` | Correr antes de confiar en el índice |

### Flags útiles para `codegraph files`

| Flag | Ejemplo |
|---|---|
| `--filter src/composables` | Solo ese directorio |
| `--pattern "*.vue"` | Solo archivos .vue |
| `--format flat` | Lista plana (vs tree) |
| `--format grouped` | Agrupado por lenguaje |
| `--max-depth 2` | Profundidad máxima en tree |
| `--no-metadata` | Ocultar conteo de símbolos |
| `--json` | Output parseable |

---

## 🧠 Workflows recomendados

### Entender un flujo — ARRANCÁ ACA
```
1. shell_command: codegraph explore "MyFunction RelatedThing" --max-files 8
2. shell_command: codegraph node MyFunction
3. read_file: src/path/to/file.ts   # fallback si el índice tiene archivos stale
```

### Trazar un valor desde UI hasta DB
```
1. shell_command: codegraph explore "handleSubmit saveData" --max-files 4
2. shell_command: codegraph explore "saveToApi saveToDb" --max-files 5
3. shell_command: codegraph node model.save
```

### Antes de editar — check blast radius
```
1. shell_command: codegraph sync
2. shell_command: codegraph impact MyFunction
3. shell_command: codegraph explore "MyFunction" --max-files 6
4. shell_command: codegraph node MyFunction
```

### Stale index
Si ves:
> "Some files referenced below were edited since the last index sync…"

→ Solo esos archivos están stale. Usá `read_file` para ellos.
→ El resto del índice es confiable.

Si ves:
> "CodeGraph auto-sync is DISABLED…"

→ **Todo** el índice está congelado. Usá `read_file` + `grep` directamente.

---

## ⚠️ Pre-flight check (antes de usar cualquier comando)

Siempre arrancá con esto para saber si podés usar codegraph:

```bash
shell_command: codegraph status
description: "Check if project is indexed"
```

**Resultados posibles:**
- `✅ Indexed, X symbols` → todo bien, usá codegraph
- `⚠ Not initialized` → corré `codegraph init` primero
- `✗ CodeGraph isn't available` → no hay `.codegraph/`, no está instalado, o no existe en el sistema. Fallback: `grep` + `read_file`

---

## ❌ Anti-patterns

| No hagas esto | Por qué |
|---|---|
| Usar el MCP tool `codegraph_explore` si devuelve schema vacío | El CLI `codegraph explore` funciona siempre. Preferilo sobre el MCP. |
| Usar `codegraph` sin verificar `status` primero | Si no hay índice, todos los comandos fallan silenciosamente. |
| Olvidar `--max-files` en `explore` | El output se trunca y perdés símbolos clave. Arrancá con `--max-files 8`. |
| Re-verificar un node con grep | CodeGraph usa AST parse completo — es más preciso que grep. |
| Reconstruir flujos manualmente | `codegraph explore "A B"` te da todo el path en 1 call. |
| Usar codegraph para strings literales | No indexa strings. Usá `grep`. |
| Usar codegraph para JSON/YAML/markdown | Solo indexa código fuente (js, vue, ts, etc.). |
| Seguir usando codegraph si `.codegraph/` no existe | El proyecto no está indexado. Fallback a `grep` + `read_file`. |
| Confiar en `codegraph status` sin hacer `sync` primero | El file watcher puede laggear ~1 segundo. Siempre `sync` primero. |
