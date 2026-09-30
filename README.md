# ai-config

Single source of truth for AI behavior across tools: `ai-profiles.md` (base layer +
4 situation profiles) plus per-profile subagents for Claude Code and GitHub Copilot.
Each tool's global config links back to this repo, so there is no per-repo setup.

Background: https://tmstff.github.io/knowledge-base/ai/ai-behavior-profiles.html

## Contents

| Path | Purpose |
| --- | --- |
| `ai-profiles.md` | Base layer + profiles (Coding, Reviewing, Local Debugging, Prod/INT Analysis) |
| `agents/*.md` | Claude Code subagents, one per profile |
| `agents/copilot/*.agent.md` | GitHub Copilot subagents, one per profile |

---

## Setup (for an AI agent)

Goal: clone this repo, then link its files into each tool's global config. All link
commands are idempotent (`ln -sf`) — safe to re-run. On update, `git pull` in the
repo; links keep pointing at the fresh files.

### 1. Clone

```sh
# Pick a stable location; do not move it afterwards (links use absolute paths).
git clone <REPO_URL> "$HOME/repos/personal/ai-config"
export AI_CONFIG="$HOME/repos/personal/ai-config"
```

### 2. Claude Code

Reference the profiles from the global instructions, and link the subagents.

```sh
# a) Load profiles every session: add an @import line to ~/.claude/CLAUDE.md
mkdir -p "$HOME/.claude/agents"
LINE="@$AI_CONFIG/ai-profiles.md"
grep -qxF "$LINE" "$HOME/.claude/CLAUDE.md" 2>/dev/null || echo "$LINE" >> "$HOME/.claude/CLAUDE.md"

# b) Link the four profile subagents into the global agents dir
for f in "$AI_CONFIG"/agents/*.md; do
  ln -sf "$f" "$HOME/.claude/agents/$(basename "$f")"
done
```

### 3. GitHub Copilot CLI

Symlink the profiles file and the Copilot agents dir into `~/.copilot/`, and include
the profiles via `@` from the instructions file.

```sh
mkdir -p "$HOME/.copilot"
ln -sfn "$AI_CONFIG/ai-profiles.md"   "$HOME/.copilot/ai-profiles.md"
ln -sfn "$AI_CONFIG/agents/copilot"   "$HOME/.copilot/agents"

INSTR="$HOME/.copilot/copilot-instructions.md"
grep -qxF "@ai-profiles.md" "$INSTR" 2>/dev/null || echo "@ai-profiles.md" >> "$INSTR"
```

### 4. GitHub Copilot (VS Code) — manual

Add to user `settings.json`:

```json
"github.copilot.chat.codeGeneration.instructions": [
  { "file": "<absolute-path>/ai-config/ai-profiles.md" }
]
```

### 5. Verify

```sh
ls -l "$HOME/.claude/agents"/{coder,reviewer,local-debugger,prod-analyst}.md
ls -l "$HOME/.copilot/ai-profiles.md" "$HOME/.copilot/agents"
grep -n ai-profiles.md "$HOME/.claude/CLAUDE.md" "$HOME/.copilot/copilot-instructions.md"
```

Each Claude agent line should show `-> $AI_CONFIG/agents/...`; the Copilot entries
should show symlinks into the repo.

---

## How the profiles are used at runtime

Read `ai-profiles.md`. Infer the active profile from the task, state which one you
are applying, and apply the always-on base layer on top of it. Subagents map 1:1 to
profiles (`coder`→Coding, `reviewer`→Reviewing, `local-debugger`→Local Debugging,
`prod-analyst`→Prod/INT Analysis).
