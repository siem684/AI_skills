# AI Skills

A collection of Claude Code skills — reusable slash commands and automated behaviors for Claude Code workflows.

## Structure

```
skills/          # Skill definitions (.md files)
examples/        # Example usage and test inputs
```

## Using a skill

Copy or symlink a skill file into your project's `.claude/commands/` directory, or into `~/.claude/commands/` to make it available globally.

## Contributing

Each skill lives in its own `.md` file under `skills/`. The filename becomes the slash command name (e.g. `skills/review.md` → `/review`).
