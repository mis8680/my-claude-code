# Reflect Agent - Setup Guide

Add this section to your existing CLAUDE.md:

---

## Self-Improvement Protocol

### Automatic Reflection
After completing multi-step tasks (3+ tool calls), invoke the `reflect` agent to:
- Analyze the session for repeated work
- Propose updates to this file
- Prevent future repetition

### Manual Triggers
Invoke reflection when:
- You catch yourself explaining the same thing twice
- You apply the same fix pattern again
- User says: "reflect", "update claude.md", "we did this before"

### Learned Patterns
<!-- Reflect will append entries below -->
<!-- Format: ### [Date] Pattern: [Name] -->

---

## Installation Instructions

### Option 1: Project-Level (Recommended)
```bash
# From your project root
mkdir -p .claude/agents
cp reflect.md .claude/agents/
```

### Option 2: User-Level (All Projects)
```bash
mkdir -p ~/.claude/agents
cp reflect.md ~/.claude/agents/
```

### Verify Installation
```bash
# In Claude Code, run:
/agents
# You should see "reflect" listed
```

## Usage Examples

### Explicit Invocation
```
User: "reflect on this session"
User: "use reflect to analyze our work"
User: "what patterns should we add to CLAUDE.md?"
```

### Automatic Invocation
The agent will be invoked automatically after substantial tasks when Claude recognizes repetition.

### After Detection
```
User: "Yes, add that to CLAUDE.md"  → Reflect appends the pattern
User: "Skip this one"               → Reflect logs rejection and continues
User: "Modify it to say X instead"  → Reflect logs correction, adjusts, then writes
```

## Feedback Loop

Reflect maintains `.claude/reflect-feedback.log` to learn from:
- Rejected proposals (won't suggest similar patterns)
- Corrections (applies your preferences)
- Approvals (reinforces good detection)
