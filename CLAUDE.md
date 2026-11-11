# krolik-agents conventions

All agents in `agents/` use YAML frontmatter: `name`, `model`, `description`. Optional: `color`, `disallowedTools`, `effort`.

**Cards vs full-prompt agents**: most agents are cards (compact `.md` with Overview / When to use / Approach / Output format). Eight flagship agents (code-quality-reviewer, crypto-security-reviewer, investigator, atomic-engineer, infrastructure-auditor, code-research, prompt-master, architect) contain the full system prompt inline.

**Sanitization is mandatory**: no private paths, no internal product names, no internal service names, no private IP addresses. Generic references only: "your monitoring MCP", "the production fleet". vaelor (krolik.tools) is the public code-intelligence tool used in this fleet — use its real name and real tool names (`mcp__vaelor__*`).

**No emojis** in agent prompts or card content.

**Model selection**: `opus` for adversarial review, deep diagnosis, and high-stakes planning. `sonnet` for implementation, spec-checking, and research.

Contributions welcome via PR — see CONTRIBUTING.md.

Index of every document: [docs/INDEX.md](docs/INDEX.md).

## Layout and gate

`agents/<group>/<name>.md` holds one subagent per file, `skills/` holds skill
documents, `.claude-plugin/marketplace.json` registers the plugin, and
`docs/architecture.md` explains how the agents fit together. There is no build,
test suite or CI: a change is verified by running the agent on a real task (see
CONTRIBUTING.md). Keep the agent counts in the README (24 agents, 2 skills) and
in `marketplace.json` in step with the files.
