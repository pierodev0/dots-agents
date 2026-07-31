---
name: codegraph
description: "Explore and understand source code through an indexed knowledge graph: find symbol definitions, trace flows across files, map callers/callees and edit blast radius. Use when you need to locate, navigate, or reason about code before reading or grepping files manually — e.g. 'where is X defined', 'who calls X', 'how does feature Y work', 'what breaks if I change Z'. Runs as a global `codegraph` CLI from any shell. Not for literal-string or doc search (use grep)."
---

# CodeGraph

## ⚡ tl;dr — golden rules

1. **ALWAYS use the CLI (`codegraph ...`)** — one global binary, works from any agent's shell. No MCP tools, no basher wrapper: the same command everywhere.
2. **`codegraph explore` is your main command** — relevant symbols' source + call paths in 1 call.
3. **Use `--max-files N` on explore** — start with `--max-files 8` to prevent truncation.
4. **`codegraph node` is your PRECISE READ** — one symbol (source + caller/callee trail) or a whole file (line numbers + dependents). Treat its output as already-Read. Never request a summary of node/explore output — the verbatim source IS the answer. If output is long, page it (`head`/`tail`, or `--offset/--limit`).
5. **If you see the banner "edited since last index sync"** — that file's indexed copy is stale; use `read_files` for it.

---

## 🏃 Quick Reference

```bash
# ⭐ Understand a flow (source + relations across files) — START HERE
codegraph explore "useExerciseForm useRoutineStore" --max-files 8

# 🎯 Exact symbol: source + signature + caller/callee trail
codegraph node useExerciseForm

# 🎯 Exact file: read WITH line numbers + dependents (smarter read_files)
codegraph node -f src/views/ExerciseFormView.vue

# 📞 Who calls X
codegraph callers useRoutineStore -l 50

# 💥 What breaks if I change X
codegraph impact useRoutineStore

# 🧪 Which tests are affected by my edits?
git diff --name-only | codegraph affected --stdin -q

# 📁 Project structure
codegraph files --filter src/views --format flat --no-metadata

# 🩺 Index health
codegraph status
```

---

## 🔄 Before trusting the index, sync first

`codegraph status` can report "up to date" while the file watcher lags ~1s. After editing files, force a sync before relying on CodeGraph:

```bash
codegraph sync        # -q for git hooks
codegraph status      # verify the index is fresh
```

---

## 🎯 `codegraph node` — the precise read

Two modes, both return **verbatim source with line numbers** (treat as already-Read).

### Symbol mode — one symbol + its trail
```bash
codegraph node <symbol>            # e.g. createExercise
codegraph node <file>.<symbol>     # disambiguate duplicate names, e.g. routines.addExercise
```
Returns the function body, signature, and the **Called by ← trail** — who calls it, without extra queries.

### File mode — whole file with line numbers + dependents
```bash
codegraph node -f <file>                           # full file + "used by N files"
codegraph node -f <file> --symbols-only            # symbol map + dependents, NO source — fast orientation
codegraph node -f <file> --offset 1 --limit 25     # page a range, like a read of lines 1–25
```
This is `read_files` with blast radius: numbered source PLUS who uses the file — keep moving without extra calls.

### Which one when
| Situation | Use |
|---|---|
| Unknown territory | `explore "<concepts>" --max-files 8` |
| I know the symbol/file, need precise detail | `node <symbol>` / `node -f <file>` |
| About to edit a file | `node -f <file>`, then `impact <symbol>` |
| File shows the stale banner | `read_files` — the indexed copy is outdated |

---

## 📋 All commands

| Goal | Command | Example |
|---|---|---|
| **Source + relations across files** ⭐ | `codegraph explore <query> --max-files N` | `codegraph explore "timer session" --max-files 6` |
| Exact symbol (source + trail) | `codegraph node <symbol>` | `codegraph node useTimer` |
| Exact file (lines + dependents) | `codegraph node -f <file> [--symbols-only] [--offset/--limit]` | `codegraph node -f src/lib/seedData.js` |
| Who calls X | `codegraph callers <symbol>` | `codegraph callers saveToStorage -l 50` |
| What X calls | `codegraph callees <symbol>` | `codegraph callees useExerciseForm` |
| Blast radius | `codegraph impact <symbol>` | `codegraph impact useRoutineStore` |
| Test files affected by edits | `codegraph affected <files...> / --stdin -q` | `git diff --name-only \| codegraph affected --stdin -q` |
| Search symbols (fuzzy) | `codegraph query <pattern>` | `codegraph query "saveTo.*"` |
| File tree | `codegraph files [flags]` | `codegraph files --filter src --format flat` |
| Index status | `codegraph status` | — |
| Force re-sync | `codegraph sync` | `codegraph sync -q` (git hooks) |
| Fix stuck index (stale lock) | `codegraph unlock` | Run if `sync`/`index` hang on a lock |

### Useful flags for `codegraph files`

| Flag | Example |
|---|---|
| `--filter src/composables` | Only that directory |
| `--pattern "*.vue"` | Only .vue files |
| `--format flat` / `--format grouped` | Flat list / grouped by language |
| `--max-depth 2` | Max depth in tree view |
| `--no-metadata` | Hide symbol counts |
| `--json` | Parseable JSON output |

---

## 🧠 Recommended workflows

### Understand a flow — START HERE
```
1. codegraph explore "usePracticeSession addSession" --max-files 8   # Big picture
2. codegraph node usePracticeSession                                 # Detail on 1 symbol
3. read_files src/stores/useSessionStore.js                          # Fallback only for stale-banner files
```

### Trace a value from UI to database
```
1. codegraph explore "addNewExercise useExerciseForm" --max-files 4  # UI → composable
2. codegraph explore "saveToStorage saveToDb _doSave" --max-files 5  # Store → DB
3. codegraph node routines.addExercise                               # DB entity
```

### Before editing — check blast radius
```
1. codegraph sync                         # Ensure index is fresh
2. codegraph impact myFunction            # What would break?
3. codegraph node myFunction              # Read current source (no read_files needed)
```

### After editing — find tests to run
```
1. codegraph sync                                          # Re-index changes
2. git diff --name-only | codegraph affected --stdin -q    # Test files touched by my edits
```

### Stale index
If you see the banner:
> "Some files referenced below were edited since the last index sync…"

→ Only those files are stale. Use `read_files` for them. The rest of the index is trustworthy.

If you see:
> "CodeGraph auto-sync is DISABLED…"

→ The **entire** index is frozen. Use `read_files` + grep directly.

---

## ❌ Anti-patterns

| Don't do this | Why |
|---|---|
| Ask for a summary of `node`/`explore` output | The output IS the verbatim source — you lose the code if you summarize it. Summarize only in your final reply. |
| Re-verify `node` with grep | CodeGraph is a full AST parse — more accurate than text search |
| Reconstruct flows manually | `codegraph explore "A B"` gives you the full path in 1 call |
| Forget `--max-files` on explore | Output truncates and you miss key symbols |
| Use codegraph for literal strings | Doesn't index strings — use grep |
| Use codegraph for JSON/YAML/markdown | Only indexes source code (js, vue, ts, etc.) |
| Keep using codegraph if `.codegraph/` doesn't exist | Not indexed → use native tools, or `codegraph init` first |
| Trust `codegraph status` without syncing first | The file watcher can lag ~1s. Always `codegraph sync` first |