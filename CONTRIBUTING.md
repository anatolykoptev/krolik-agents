# Contributing

## How to contribute new agents

1. **Pick a clear role.** The agent must have a narrow, well-defined scope. If you can describe what it does in one sentence, the scope is right. If it takes three sentences, split it.

2. **Write the YAML frontmatter.** Required fields:
   ```yaml
   ---
   name: your-agent-name
   description: "One sentence: what it does and when to use it."
   model: sonnet  # or opus for high-stakes review/debug agents
   ---
   ```

3. **Sanitize before submitting.** No private infrastructure details:
   - No internal product names or service names
   - No private IP addresses or port numbers
   - No internal paths (`/home/username/...`)
   - No proprietary tool names that aren't publicly available
   - No company-internal jargon
   Generic references are fine: "your monitoring MCP", "the production fleet". go-code (krolik.tools) is the public code-intelligence tool for this fleet — use its real name where applicable.

4. **Include anti-sycophancy rules.** If the agent reviews or assesses, it must have explicit rules preventing it from capitulating under operator pushback without new evidence.

5. **Define STOP conditions.** If the agent writes code or modifies files, it must have an explicit STOP at `gh pr create`. It never merges.

## PR format

Use conventional commits:
- `feat: add <agent-name> agent` for new agents
- `fix: <agent-name> - <what was wrong>` for corrections
- `docs: update <file>` for documentation changes

## Testing

Before submitting, verify the agent prompt works by running it against a real task in your Claude Code environment. Include in the PR description:
- What task you tested it on
- What the output looked like
- Any edge cases you found

## Questions

Open an issue before building a complex agent — describe the role you have in mind and get feedback on whether it overlaps with existing agents.
