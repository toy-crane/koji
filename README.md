# Koji Workflow

A modular AI-assisted development workflow.

## Mental Model

```
Idea / Issue
     │
     ▼
  /clarify
  (Socratic questioning → Structured output)
     │
     ▼
  User decides size
     │
     ├─ Small ──→ Claude Plan Tool → Implement
     │
     ├─ Medium ─→ /plan → Implement
     │
     └─ Big ────→ /prd → /plan (×N) → Implement
```

## Size Guidelines

| Size | When to use | Next step |
|------|-------------|-----------|
| **Small** | Clear direction, few touch points (1-2 files) | Use Claude's built-in Plan mode |
| **Medium** | Needs design, several touch points (3-5 files) | Run `/plan` |
| **Big** | Needs phasing, many touch points (6+ files) | Run `/prd` first, then `/plan` per phase |

## Commands

| Command | Purpose |
|---------|---------|
| `/clarify` | Transform vague ideas into structured understanding, then save to Linear/file |
| `/plan` | Create detailed implementation plan |
| `/prd` | Create product requirements document with phases |

## File Organization

The `context/` folder groups files by project:

```
context/
├── {project-name}/
│   ├── brief.md      # Clarified idea from /clarify
│   ├── prd.md        # Product requirements doc
│   └── plan.md       # Implementation plan
├── auth-flow/
│   └── brief.md
└── payment-integration/
    └── brief.md
```

This keeps all related files together and makes archiving easy.

---

*This workflow is iteratively improved. See `commands/` for command details.*
