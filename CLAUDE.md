# krolik-agents conventions

All agents in `agents/` use YAML frontmatter: `name`, `model`, `description`. Optional: `color`, `disallowedTools`, `effort`.

**Cards vs full-prompt agents**: most agents are cards (compact `.md` with Overview / When to use / Approach / Output format). Eight flagship agents (code-quality-reviewer, crypto-security-reviewer, investigator, atomic-engineer, infrastructure-auditor, code-research, prompt-master, architect) contain the full system prompt inline.

**Sanitization is mandatory**: no private paths, no internal product names, no internal service names, no private IP addresses. Generic references only: "your monitoring MCP", "the production fleet". go-code (krolik.tools) is the public code-intelligence tool used in this fleet — use its real name and real tool names (`mcp__go-code__*`).

**No emojis** in agent prompts or card content.

**Model selection**: `opus` for adversarial review, deep diagnosis, and high-stakes planning. `sonnet` for implementation, spec-checking, and research.

Contributions welcome via PR — see CONTRIBUTING.md.
