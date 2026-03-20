# OpenClaw Skills

> Last updated: 2026-03-20

## What They Are

Modular capability folders following the AgentSkills format. Each skill is a directory containing a `SKILL.md` with YAML frontmatter and instructions.

Skills extend what your agent can do without modifying core config.

## ClawHub

The official skill registry. The agent can search for and pull in new skills automatically.

**Critical warning:** At least 230 malicious skills have been uploaded to ClawHub since late January 2026. Vet every skill before installing.

## Skill Structure

```
skills/
  my-skill/
    SKILL.md    # YAML frontmatter + instructions
```

[TODO: Document full SKILL.md format]

## Skill Filtering

Skills can be filtered per-agent based on role and trust level. Not every agent needs every skill.

## Memory (Related)

- Local-first. Stored as Markdown files on your machine.
- Runtime searches memory for context from past conversations.
- Recent update: memory can be "plugged and unplugged" (modular memory).

## TODO: Research Gaps

- [ ] Full SKILL.md specification and format
- [ ] How to write custom skills
- [ ] ClawHub: safe browsing and vetting process
- [ ] Skill trust levels and per-agent filtering
- [ ] Memory architecture and modular memory
- [ ] Community skill recommendations (vetted)
