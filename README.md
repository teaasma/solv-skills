# Solv Health Team Skills

A shared library of Claude Code skills for the Solv sales and partnerships team.

## How to use a skill

1. Copy the skill folder into your project's `.claude/skills/` directory
2. In Claude Code, invoke it with `/skill-name`

Or install globally (available across all your projects):
```
cp -r skills/<skill-name> ~/.claude/skills/
```

## How to add your own skill

1. Clone this repo
2. Create a folder under `skills/` with your skill name (lowercase, hyphens)
3. Add a `SKILL.md` inside it
4. Open a PR — or push directly if you have access

## Skills

| Skill | Description |
|-------|-------------|
| [deal-pulse](skills/deal-pulse/SKILL.md) | Searches Slack, Gong, and Calendar to surface new logo deal momentum, blockers, and next steps |
| [skill-creator](skills/skill-creator/SKILL.md) | Build, test, and iterate on new Claude skills |
