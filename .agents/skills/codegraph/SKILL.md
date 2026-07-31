---
name: codegraph
description: "Quick reference for CodeGraph CLI — traverse symbols, source, callers, and dependencies via basher."
---

# CodeGraph

## ⚡ tl;dr — 5 golden rules

1. **ALWAYS use `basher`** — The MCP tool `codegraph_explore` does NOT work (empty schema)
2. **`codegraph explore` is your main command** — Multiple symbols + relations in 1 call
3. **Use `--max-files N` on explore** — Start with `--max-files 8` to prevent truncation
4. **NEVER use `what_to_summarize`** on `node` or `explore` — it destroys the verbatim source
5. **If you see the banner "edited since last index sync"** — use `read_files` for those files

---

## 🏃 Quick Reference

```bash
# ⭐ Understand a flow (source + relations across files) — PREFERRED
basher: command: codegraph explore "useExerciseForm useRoutineStore" --max-files 8

# 📖 Understand a specific symbol (when you already know the name)
basher: command: codegraph node useExerciseForm

# 📞 Who calls X
basher: command: codegraph callers useRoutineStore -l 50

# 💥 Blast radius (what breaks if I change X)
basher: command: codegraph impact useRoutineStore

# 📁 Project structure
basher: command: codegraph files --filter src/views --format flat --no-metadata

# 🩺 Index health
basher: command: codegraph status
```

---

## 🔄 Before trusting the index, sync first

`codegraph status` may report the index as up to date, but the file watcher can lag by ~1 second. After editing files, always force a sync before relying on CodeGraph data:

```bash
basher: command: codegraph sync
what_to_summarize: Confirm the sync completed successfully
```

Then run `codegraph status` to verify the index is truly fresh.

---

## 📖 Real example: what `codegraph explore` returns

When you run:
```
basher: command: codegraph explore "useExerciseForm" --max-files 4
```

You get **all of this in a single call**:
- ✅ **Real source** of `useExerciseForm`, `useRoutineStore`, and related files — with line numbers
- ✅ **Called by**: `DashboardView.vue:6`
- ✅ **Calls**: `useRoutineStore (src/stores/useRoutineStore.js:83)`
- ✅ **Blast radius**: tests and callers that depend on these symbols
- ✅ **Dynamic-dispatch links**: Vue `@click` handlers, component renders, router pushes

The output is the exact function bodies, not summaries.

---

## 📋 All commands

| Goal | Command | Example |
|---|---|---|
| **Source + relations across files** ⭐ | `codegraph explore <query> --max-files N` | `codegraph explore "timer session" --max-files 6` |
| Source of a single symbol | `codegraph node <symbol>` | `codegraph node useTimer` |
| Who calls X | `codegraph callers <symbol>` | `codegraph callers saveToStorage -l 50` |
| What X calls | `codegraph callees <symbol>` | `codegraph callees useExerciseForm` |
| Blast radius | `codegraph impact <symbol>` | `codegraph impact useRoutineStore` |
| Search symbols (fuzzy) | `codegraph query <pattern>` | `codegraph query "saveTo.*"` |
| File tree | `codegraph files [flags]` | `codegraph files --filter src --format flat` |
| Index status | `codegraph status` | — |
| Force re-sync | `codegraph sync` | Run this before trusting index after edits |

### Useful flags for `codegraph files`

| Flag | Example |
|---|---|
| `--filter src/composables` | Only that directory |
| `--pattern "*.vue"` | Only .vue files |
| `--format flat` | Flat list (vs tree) |
| `--format grouped` | Grouped by language |
| `--max-depth 2` | Max depth in tree view |
| `--no-metadata` | Hide symbol counts |
| `--json` | Parseable JSON output |

---

## 🧠 Recommended workflows

### Understand a flow — START HERE
```
1. basher: codegraph explore "usePracticeSession addSession" --max-files 8  # Big picture
2. basher: codegraph node usePracticeSession                                # Detail on 1 symbol
3. read_files src/stores/useSessionStore.js                                 # Fallback: read full file if explore showed stale-banner files for it
```

### Trace a value from UI to database
```
1. basher: codegraph explore "addNewExercise useExerciseForm" --max-files 4     # UI → composable
2. basher: codegraph explore "saveToStorage saveToDb _doSave" --max-files 5     # Store → DB
3. basher: codegraph node routines.addExercise                                  # DB entity
```

### Before editing — check blast radius
```
1. basher: codegraph sync                           # Ensure index is fresh
2. basher: codegraph impact myFunction              # What would break?
3. basher: codegraph explore "myFunction" --max-files 6   # See full context
4. basher: codegraph node myFunction                # Read current source
```

### Stale index
If you see the banner:
> "Some files referenced below were edited since the last index sync…"

→ Only those files are stale. Use `read_files` for them.
→ The rest of the index is still trustworthy.

If you see:
> "CodeGraph auto-sync is DISABLED…"

→ The **entire** index is frozen. Use `read_files` + `code_searcher` directly.

**Concrete example**: You edited `DashboardView.vue`. Now you run `codegraph node useExerciseForm` and see the banner "1 file edited since last index sync: src/views/DashboardView.vue". The `useExerciseForm` source is still fresh (wasn't edited). Only the calling code in DashboardView.vue is stale — use `read_files` for that file.

---

## ❌ Anti-patterns

| Don't do this | Why |
|---|---|
| Use the MCP tool `codegraph_explore` directly | Doesn't accept parameters. Validation error. |
| Use `what_to_summarize` on `node`/`explore` | The basher summarizes away the source code lines |
| Forget `--max-files` on `explore` | Output truncates and you miss key symbols. Always start with `--max-files 8`. |
| Re-verify node with grep | CodeGraph uses a full AST parse — more accurate |
| Reconstruct flows manually | `codegraph explore "A B"` gives you the full path in 1 call |
| Use codegraph for literal strings | Doesn't index strings. Use `code_searcher` (grep) |
| Use codegraph for JSON/YAML/markdown | Only indexes source code (js, vue, ts, etc.) |
| Keep using codegraph if `.codegraph/` doesn't exist | The project is not indexed. Switch to native tools. |
| Trust `codegraph status` without syncing first | The file watcher can lag ~1 second. Always run `codegraph sync` first. |
