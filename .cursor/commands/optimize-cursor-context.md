Analyze my Cursor base context and suggest optimizations.

I want to understand exactly what is loaded into my Cursor agent context on **every new session before I type anything**. Investigate and report on **all** of the following:

---

## 1. Workspace rules

- Enumerate `.cursor/rules/` and identify rules that apply every session (`alwaysApply: true` **or** `alwaysApply` absent with frontmatter that implies inclusion in system context—note how Cursor resolves this for this workspace).
- For each relevant rule file: **filename**, **approximate character count**, and **approximate token cost** (use approximate chars ÷ 4 as a coarse token estimate unless you tokenize accurately).
- Call out the **heaviest** rules.
- Recommend which could become **agent-requestable** / on-demand (`agentRequestable: true`, `alwaysApply: false`) instead of always-on.

---

## 2. User rules

- List **user-level rules** configured in Cursor (typically from global Cursor settings—not repo files).
- Approximate token cost **per rule** using the same heuristic.
- Flag any rule large enough that a **skill** (loaded on demand) would be lighter for default sessions.

---

## 3. Skills metadata overhead

Count skills registered across these sources **where they exist**:

| Source | Purpose |
|--------|---------|
| `<repo>/.cursor/skills/` | repo-local skills |
| `~/.cursor/skills/` | user skills |
| `~/.cursor/skills-cursor/` | Cursor-bundled / template skills paths |
| `~/.codex/skills/` | Codex-skills conventions (if present) |
| `~/.claude/skills/` | Claude/Code skills dirs (if symlinked/copied here) |

Also include **skills exposed by enabled Cursor plugins/extensions** where discoverable via filesystem manifests or documented plugin paths—state limitations if some plugin metadata is opaque.

For **each source that exists**:

- How many discrete skills (`SKILL.md` roots or declared entries) appear registered.
- **Combined metadata size** (sum of SKILL.md fronts + descriptors the agent reliably sees **before** a skill file is explicitly read—be explicit about what you're measuring).

Identify plugin-related sources contributing the most overhead **relative** to usefulness for **this repo’s domain**.

---

## 4. Claude plugin bridge

- Read **`~/.claude/settings.json`** (if present) for `enabledPlugins` (and related bridge config).
- List enabled plugins and their **approximate skill-count contribution** to the Claude/Cursor skill surface **from this workspace’s perspective**.
- Flag plugins unlikely to matter for **this repository** (explain briefly).

---

## 5. Git status noise

> **Workspace policy reminder:** Before running **any** `git` command (including read-only ones), obtain **explicit user permission** if required by workspace rules—or skip this section if permission is withheld.

After permission:

- Run **`git status --short`** and report total **line count**.
- Identify **untracked large generated dirs** (e.g. `node_modules/`, build outputs, vendor trees) escaping `.gitignore`.
- Explain **why** they escape when possible (examples: anchored pattern vs recursive `**/`, negative rules, submodule, global gitignore interplay, casing, file already tracked historically).
- Check whether **`.cursorignore`** exists at the repo root (and summarize what large paths it hides from indexing/context if skimmed briefly).

---

## 6. Summary table

Produce a markdown table:

| Component | ~Chars | ~Tokens | Reducible? | Recommended Action |
|-----------|--------|---------|------------|---------------------|

(one row per major bucket: aggregated workspace rules, each heavy rule if split out, user rules, skill metadata buckets, MCP server descriptors surfaced to the agent, notable git/index noise implications, etc.)

---

## 7. Top 3 quick wins

Based on findings, rank the **three highest-impact** changes I can make **now**—ordered by **estimated tokens saved per fresh session**.

Each quick win must include:

- Estimated savings (qualitative bracket is fine if exact unknown)
- Implementation hint (toggle, move to skill, split rule, prune plugin, fix ignore, etc.)
- Caveat/trade-off (if any)

---

## Constraints

- **Read-only**: do **not** modify settings, ignore files, rules, `settings.json`, or install/uninstall plugins as part of this run.
- **Transparent methodology**: briefly state assumptions (paths checked, tokenizer vs chars/4, what counts as “metadata”).
- Prefer **measurable** counts from the filesystem; mark estimates clearly when unavoidable.
