<div align="center">

# 🧠 Hermes Memory for OpenCode

**Layered persistent memory for your OpenCode agent — ported from [Hermes](https://github.com/weaigc/hermes)**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/realchendahuang/opencode-hermes-memory?style=social)](https://github.com/realchendahuang/opencode-hermes-memory)
[![GitHub forks](https://img.shields.io/github/forks/realchendahuang/opencode-hermes-memory?style=social)](https://github.com/realchendahuang/opencode-hermes-memory/network/members)
[![GitHub issues](https://img.shields.io/github/issues/realchendahuang/opencode-hermes-memory)](https://github.com/realchendahuang/opencode-hermes-memory/issues)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](https://github.com/realchendahuang/opencode-hermes-memory/pulls)
[![Follow @realchendahuang](https://img.shields.io/badge/Follow-%40realchendahuang-1DA1F2?logo=x&logoColor=white)](https://x.com/realchendahuang)

Give your OpenCode agent a **real memory** — user preferences, project conventions, past failures, and hard-won lessons survive across sessions. No vector database, no external services. Just Markdown files you can read, edit, and version-control.

</div>

---

## ✨ Features

| Layer | What it does |
|---|---|
| **L0 — Standing instructions** | `STANDING.md` hard rules injected into every session's system prompt |
| **L1 — Markdown truth source** | All memory lives in plain Markdown files — human-readable, hand-editable, git-friendly |
| **L2 — Retrieval** | `memory_search` tool with lightweight token-scored ranking (no vector DB needed) |
| **Learning loop** | Background LLM review on idle summarizes sessions into durable memories; rule-based correction detection saves corrections instantly; flush review before compaction; auto-consolidation at capacity |
| **Auto-injection** | Relevant memories are retrieved and injected into context on every user message (score ≥ 0.4, max 2/turn, deduplicated per session) |
| **Error prefetch** | When a bash command fails, related past-failure lessons are auto-injected into the next turn (Mem0-style) |
| **Bi-temporal evolution** | Replaced entries move to `history.md` — traceable, out of capacity, out of retrieval |

## 🚀 Quick Start

### Install from GitHub (recommended)

```bash
opencode plugin github:realchendahuang/opencode-hermes-memory
```

That's it — OpenCode downloads the plugin from GitHub, installs it, and registers it in your config automatically. Restart OpenCode and the plugin starts learning from your sessions.

> **Note**: add `-g` to install globally (all projects) instead of the current project:
>
> ```bash
> opencode plugin -g github:realchendahuang/opencode-hermes-memory
> ```

### Manual install (local development)

```bash
git clone https://github.com/realchendahuang/opencode-hermes-memory.git
mkdir -p ~/.config/opencode/plugins
cp -R opencode-hermes-memory/hermes-memory.ts opencode-hermes-memory/hermes-memory-lib ~/.config/opencode/plugins/
```

Then add to the `plugin` array in `~/.config/opencode/opencode.json`:

```json
{
  "plugin": [
    "./plugins/hermes-memory.ts"
  ]
}
```

And install the dependency:

```bash
cd ~/.config/opencode
npm install @opencode-ai/plugin
```

The plugin registers 5 tools (`memory_search`, `memory_add`, `memory_replace`, `memory_remove`, `memory_history`) and starts learning from your sessions automatically.

## 🧰 Memory Tools

| Tool | Description |
|---|---|
| `memory_search` | Search memories; filter by `target` (`memory`/`user`/`failure`/`project`) and `category` |
| `memory_add` | Add an entry (≤ 3000 chars, deduplicated) |
| `memory_replace` | Replace an entry; the old version moves to `history.md` |
| `memory_remove` | Remove an entry |
| `memory_history` | Read the evolution history of superseded entries |

## 📁 Data Layout

Memory lives in `~/.config/opencode/memory/`:

```
memory/
├── USER.md               # User profile — who the user is
├── MEMORY.md             # Global notes — environment facts, tool quirks
├── failures.md           # Categorized lessons (failure/correction/insight/preference/convention/tool-quirk)
├── STANDING.md           # Standing hard instructions (injected every session)
├── history.md            # Evolution history of superseded entries
└── projects-memory/<id>/ # Per-project memory, isolated by project
```

All files are plain Markdown — edit them directly whenever you like.

## 🏗️ Architecture

```
hermes-memory.ts          # Plugin entry: event hooks, tool registration, injection logic
hermes-memory-lib/
├── store.ts              # MemoryStore: Markdown I/O, dedup, capacity, consolidation
├── learn.ts              # Learning loop: background review, flush review, correction detection
├── search.ts             # Token-scored retrieval
├── prompts.ts            # Prompts & constants (capacity limits, injection thresholds)
├── llm.ts                # Internal-session LLM channel (no direct completion API in OpenCode)
├── paths.ts              # Path helpers (setMemoryRoot for test isolation)
└── tests/regression.ts   # Isolated regression tests
```

### Event hooks

| Hook | Purpose |
|---|---|
| `experimental.chat.system.transform` | Inject memory policy + STANDING + project memory into system prompt |
| `chat.message` | Correction detection, turn counting, relevant-memory auto-injection |
| `session.idle` | Background learning review (10s debounce, 30-min global rate limit) |
| `experimental.session.compacting` | Flush review before context compaction |
| `tool.execute.after` | Bash error detection → failure-memory prefetch |

## ⚙️ Configuration

| Env var | Default | Description |
|---|---|---|
| `HERMES_NUDGE_INTERVAL` | `10` | Turns between background reviews |

## 🧪 Development

```bash
# Run the isolated regression suite (never touches real memory files)
bun run hermes-memory-lib/tests/regression.ts

# Type-check
bunx tsc --noEmit
```

The test suite uses `setMemoryRoot(临时目录)` to fully isolate from your real memory.

## 🤝 Contributing

Contributions are welcome! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines, and check the [open issues](https://github.com/realchendahuang/opencode-hermes-memory/issues) for ideas.

## 📜 License

[MIT](LICENSE) © [realchendahuang](https://github.com/realchendahuang)

---

## ⭐ Star History

[![Star History Chart](https://api.star-history.com/svg?repos=realchendahuang/opencode-hermes-memory&type=Date)](https://star-history.com/#realchendahuang/opencode-hermes-memory&Date)
