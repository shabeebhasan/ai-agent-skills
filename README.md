# AI Agent Skills

Reusable AI agent skills, workflows, and configuration for IMSOFT development.

## Structure

```
.agent/
├── agent.md                              # Agent persona & behavior rules
├── skills/
│   ├── software-architecture/SKILL.md    # System design & decomposition
│   ├── ai-coding/SKILL.md               # Spec-driven development & Yii2 patterns
│   ├── problem-solving/SKILL.md          # Debugging & root cause analysis
│   └── channelling-station/SKILL.md      # Build sorting stations (PreSort, PostQC, etc.)
└── workflows/
    └── (task-specific workflows)
```

## Skills

| Skill | Triggers | What It Does |
|-------|----------|-------------|
| **Software Architecture** | "design", "plan", "architect" | Component decomposition, dependency analysis, interface contracts |
| **AI Coding** | Implementation tasks, T-specs | Spec parsing, code generation, refactoring, Yii2 idioms |
| **Problem Solving** | Bugs, failures, performance | 5-step debugging, root cause analysis, performance optimization |
| **Channelling Station** | New sorting station mode | 3-phase workflow: Design → Implement → Test |

## Setup

These files live inside the IMSOFT repo at `.agent/`. To use in another repo, copy the `.agent/` directory.

```bash
# To create a separate repo
mkdir ai-agent-skills && cd ai-agent-skills
cp -r /path/to/imsoft_yii/.agent .
git init && git add -A && git commit -m "Initial agent skills"
```
