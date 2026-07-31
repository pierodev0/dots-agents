# dots-agents

Shared configuration for the AI coding agents used in this environment: **Command Code** and **Codebuff**.

This repository holds the agent-level rules, skills, and MCP server setup for both tools. Each agent picks up its own configuration folder, so both share the same conventions while staying independent.

## Supported agents

| Agent | Config folder | Docs |
| --- | --- | --- |
| Command Code | `.commandcode/` | [Skills](https://commandcode.ai/docs/skills) |
| Codebuff | `.agents/` | [Skills](https://www.codebuff.com/docs/tips/skills) |

## Repository structure

```
.
├── .agents/                      # Codebuff configuration
│   ├── AGENTS.md                 # Global rules and persona
│   ├── mcp.json                  # MCP servers (engram, codegraph)
│   └── skills/                   # Agent Skills (open standard)
│       ├── agent-browser/        # Browser automation CLI
│       └── codegraph/            # CodeGraph CLI reference
└── .commandcode/                 # Command Code configuration
    ├── AGENTS.md                 # Global rules and persona
    ├── mcp.json                  # MCP servers (chrome-devtools, context7, engram)
    └── skills/                   # Agent Skills (open standard)
        ├── codebase-memory/      # Codebase knowledge graph queries
        └── codegraph/            # CodeGraph CLI reference
```

## Skills

Skills follow the [Agent Skills open standard](https://agentskills.io): each skill lives in its own directory with a `SKILL.md` file containing YAML frontmatter (`name`, `description`) and markdown instructions.

| Skill | Agent | Purpose |
| --- | --- | --- |
| `agent-browser` | Codebuff (`.agents/skills/`) | Browser automation CLI for web interaction, scraping, and testing |
| `codebase-memory` | Command Code (`.commandcode/skills/`) | Structural codebase queries via the knowledge graph |
| `codegraph` | Both | Quick reference for the CodeGraph CLI (symbols, callers, dependencies) |

> **Note:** Command Code also discovers skills from `.agents/skills/`, so both agents can share skills defined there. On name conflicts, `.commandcode/skills/` takes priority.

## MCP servers

| Server | Purpose | Agents |
| --- | --- | --- |
| `engram` | Persistent agent memory | Both |
| `codegraph` | Code graph server | Codebuff |
| `chrome-devtools` | Browser automation and DevTools protocol | Command Code |
| `context7` | Up-to-date library documentation | Command Code |

## Usage

To add or update a skill:

1. Create the skill directory under the target agent: `.commandcode/skills/<skill-name>/` or `.agents/skills/<skill-name>/`
2. Add a `SKILL.md` with `name` and `description` frontmatter (the `name` must match the directory name)
3. Invoke it in a session with `/skill-name` (Command Code) or `/skill:<name>` (Codebuff), or let the agent load it automatically when relevant

Keep skills focused and descriptive — the `description` is what the agent uses to decide when to load a skill.
