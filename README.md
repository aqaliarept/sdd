# sdd

My personal Claude Code plugin — shared skills, agents, and settings across all projects.

## Installation

In any project, run:

```bash
/plugin install https://github.com/aqaliarept/sdd
```

## Usage

### Skills

Invoke a skill by its name prefixed with the plugin name:

```
/sdd:grill-me
```

### Agents

Agents are invoked automatically based on their `description` field, or explicitly:

```
/sdd:example-agent
```

## Contents

### Skills (`skills/`)

Each skill lives in its own subdirectory with a `SKILL.md` file.

| Skill | Description |
|-------|-------------|
| `grill-me` | <!-- short description --> |

**Adding a skill:**

```
skills/
└── my-skill/
    └── SKILL.md
```

Write your instructions in `SKILL.md`. Describe what Claude should do when the skill is invoked.

---

### Agents (`agents/`)

Each agent is a single markdown file with YAML frontmatter.

| Agent | Description |
|-------|-------------|
| `example-agent` | <!-- short description --> |

**Adding an agent:**

```
agents/
└── my-agent.md
```

Frontmatter fields:

```yaml
---
name: my-agent
description: When Claude should invoke this agent
model: sonnet          # sonnet | opus | haiku
tools:                 # tools the agent can use
  - Read
  - Grep
  - Glob
# maxTurns: 10
# disallowedTools:
#   - Write
---
```

---

### Settings (`settings.json`)

Plugin-level defaults applied when the plugin is enabled. Uncomment `agent` to set a default agent.

---

### Hooks (`hooks/hooks.json`)

Shell commands that run in response to Claude Code events (e.g. `PostToolUse`, `PreToolUse`). See the [hooks docs](https://docs.anthropic.com/en/docs/claude-code/hooks) for available events.

---

## Updating

After pushing changes to this repo:

```bash
/plugin update sdd
```
