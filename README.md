# skills

Agent skills for coding agents ([Agent Skills](https://agentskills.io) format).

## Install a skill

```bash
npx skills add 0xrohan10/skills --skill <skill-name>
```

Examples:

```bash
npx skills add 0xrohan10/skills --skill adversarial-code-review
npx skills add 0xrohan10/skills --skill finalize
npx skills add 0xrohan10/skills --skill simplify
```

Options:

```bash
# Global (all projects)
npx skills add 0xrohan10/skills --skill finalize -g

# Specific agents
npx skills add 0xrohan10/skills --skill finalize -a opencode -a claude-code

# Install everything
npx skills add 0xrohan10/skills --all

# List available skills
npx skills add 0xrohan10/skills --list
```

## Skills

| Skill | What it does |
| --- | --- |
| `adversarial-code-review` | Red-teams a change to disprove it is safe; findings are counterexamples |
| `cr-loop` | Runs the full PR → Greptile 5/5 → merge → deploy verification loop |
| `effect-logging` | Effect v4 structured logging, error boundaries, spans, and tracing |
| `finalize` | Hardens a working tree or PR into an objectively merge-ready state |
| `fly-optimization` | Designs and reviews Fly.io app topology, Postgres, networking, and ops |
| `formatting` | Formats agent responses for ADHD actionability |
| `simplify` | Behavior-preserving cleanup of a working tree or explicit diff |
| `style-doc` | Creates and restyles standalone HTML technical reports |
| `typescript-structure` | Structures TypeScript via narrow domain modules and explicit dependency graphs |
