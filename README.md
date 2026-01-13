# My Claude Code Configuration

Personal repository for custom Claude Code configurations including slash commands, skills, and documentation templates.

## Contents

### Configuration Files

- `CLAUDE.md` - Project-specific instructions for this repository
- `CLAUDE.global.md` - Synced copy of global standards from `~/.claude/CLAUDE.md`
- `claude-orchestrator-setup.md` - Complete setup guide for Claude Code orchestrator with Codex, Gemini, and Nanobanana MCPs

### Custom Slash Commands

Located in `.claude/commands/`:

#### Documentation Commands (`/docs/*`)
- `/adr [decision-title]` - Create an Architecture Decision Record
- `/research [topic]` - Create a research document
- `/task [task-name]` - Create a task planning document

#### Git Commands (`/git/*`)
- `/branch [ticket] [description]` - Create a git branch following naming conventions

#### Development Commands (`/dev/*`)
- `/analyze` - Analyze code for quality, security, and performance issues
- `/test [test-type]` - Generate comprehensive test suite

### Custom Skills

Located in `.claude/skills/`:

#### Code Review Skill
Automatically invoked for code review tasks. Performs comprehensive analysis of:
- Code quality and style
- Security vulnerabilities
- Performance issues
- Test coverage

Includes detailed checklist in `CHECKLIST.md`.

#### Documentation Generator Skill
Automatically invoked when creating documentation. Generates:
- Architecture Decision Records (ADRs)
- Research documents
- Task planning documents
- Session summaries

#### Subagent Creator Skill
Helps create specialized Claude Code custom subagents with system prompts and tool configurations. Use when creating new agents or specialized assistants.

#### Skill Creator Skill
Guides creation of effective skills that extend Claude's capabilities with specialized knowledge, workflows, or tool integrations.

### Custom Agents

Located in `.claude/agents/`:
- Custom agent configurations for specialized tasks

## Usage

1. Clone this repository
2. Commands are available via `/command-name` in Claude Code
3. Skills are automatically invoked when relevant
4. Use `/help` to see all available commands

## Documentation Structure

All documentation follows the structure in `CLAUDE.md`:

```
docs/
├── prd/              # Product requirements
├── decisions/        # Architecture Decision Records (ADR)
├── research/         # Investigations and spikes
├── tasks/            # Planning and progress logs
├── hooks.md          # Reference guide for Claude Code hooks
├── slash-commands.md # Reference guide for slash commands
└── sub-agent.md      # Reference guide for subagents
```

## Adding New Commands

1. Create a `.md` file in `.claude/commands/` (or subdirectory)
2. Add YAML frontmatter with description and options
3. Write the command prompt
4. Test in a real project

Example:
```markdown
---
description: Your command description
argument-hint: [arg1] [arg2]
---

Your command instructions here.
Use $1, $2 for individual args or $ARGUMENTS for all.
```

## Adding New Skills

1. Create directory in `.claude/skills/skill-name/`
2. Create `SKILL.md` with YAML frontmatter
3. Add supporting files as needed
4. Test that Claude invokes it appropriately

Example SKILL.md:
```markdown
---
name: skill-name
description: What it does and when to use it. Include trigger keywords.
allowed-tools: Read, Grep, Glob
---

# Skill Name

Instructions for Claude...
```

## References

- [Claude Code Documentation](https://code.claude.com/docs)
- [Slash Commands Guide](https://code.claude.com/docs/en/slash-commands.md)
- [Agent Skills Guide](https://code.claude.com/docs/en/skills.md)

## License

Personal use configuration - adapt as needed for your projects.
