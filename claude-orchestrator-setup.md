# Claude Code Orchestrator Setup Guide

A complete guide for setting up Claude Code CLI as the master orchestrator with Codex CLI (writer), Gemini CLI (auditor), and Nanobanana (image generation).

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     CLAUDE CODE CLI (Orchestrator)              │
│                                                                 │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────────┐ │
│  │ Codex MCP   │  │ Gemini MCP  │  │ Nanobanana MCP          │ │
│  │ (Writer)    │  │ (Auditor)   │  │ (Image Gen)             │ │
│  │             │  │             │  │                         │ │
│  │ - Rewrites  │  │ - Audits    │  │ - gemini_generate_image │ │
│  │   prompts   │  │   writing   │  │ - gemini_edit_image     │ │
│  │ - User      │  │ - Validates │  │ - gemini_chat           │ │
│  │   stories   │  │   quality   │  │                         │ │
│  │ - Copywrite │  │ - Feedback  │  │                         │ │
│  └─────────────┘  └─────────────┘  └─────────────────────────┘ │
│                                                                 │
│  Claude handles: Programming, CLI commands, code agents        │
└─────────────────────────────────────────────────────────────────┘
```

---

## Part 1: Install All Three CLI Tools

### Prerequisites

- **Node.js v22+** and **npm v10+**
- A terminal (bash, zsh, PowerShell)
- Active subscriptions:
  - ChatGPT Plus/Pro ($20/month) for Codex CLI
  - Google AI Pro ($20/month) for Gemini CLI
  - Anthropic account for Claude Code

---

### 1.1 Install Claude Code CLI

**Option A: Native Binary (Recommended)**

```bash
# macOS / Linux
curl -fsSL https://claude.ai/install.sh | bash

# Windows (PowerShell as Admin)
irm https://claude.ai/install.ps1 | iex
```

**Option B: npm**

```bash
npm install -g @anthropic-ai/claude-code
```

**Authenticate:**

```bash
claude
# Follow prompts to sign in with your Anthropic account
```

---

### 1.2 Install Codex CLI

**Option A: npm (Recommended)**

```bash
npm install -g @openai/codex
```

**Option B: Homebrew (macOS)**

```bash
brew install --cask codex
```

**Authenticate with your ChatGPT subscription:**

```bash
codex --login
# Select "Sign in with ChatGPT"
# This uses your $20/month subscription, NOT API credits
```

**Verify:**

```bash
codex --version
# Should show v0.77.0 or later
```

---

### 1.3 Install Gemini CLI

```bash
npm install -g @google/gemini-cli
```

**Authenticate with your Google AI Pro subscription:**

```bash
gemini
# Select "Login with Google"
# Sign in with the Google account tied to your $20/month subscription
```

**Verify subscription is active:**

```bash
gemini
# Type: /auth
# Should show your Google AI Pro/Ultra status
```

---

## Part 2: Configure MCPs in Claude Code

### 2.1 Add Codex as Writer Agent

This uses the **built-in** `codex mcp-server` command, which uses your ChatGPT subscription:

```bash
claude mcp add codex-writer -s user -- codex mcp-server
```

**What this does:**
- Exposes Codex as an MCP tool inside Claude
- Uses your $20/month ChatGPT subscription (not API credits)
- Provides two tools: `codex` (start session) and `codex-reply` (continue)

---

### 2.2 Add Gemini as Auditor Agent

This uses a wrapper that invokes your authenticated Gemini CLI:

```bash
claude mcp add gemini-auditor -s user -- npx -y gemini-mcp-tool
```

**Alternative wrappers (if the above doesn't work):**

```bash
# Option B: @anthropic-ai/gemini-cli-mcp
claude mcp add gemini-auditor -s user -- npx -y @anthropic-ai/gemini-cli-mcp

# Option C: @jacob/gemini-cli-mcp (OAuth support)
claude mcp add gemini-auditor -s user -- npx -y @jacob/gemini-cli-mcp
```

**Note:** `gemini-mcp-tool` is the recommended wrapper as of Dec 2025.

**What this does:**
- Wraps your Gemini CLI as an MCP tool
- Uses your $20/month Google AI Pro subscription
- Provides tools for chat, analysis, and verification

---

### 2.3 Add Nanobanana MCP (Image Generation - Python Version)

The Python version of Nanobanana provides the best image quality with advanced features:

```bash
claude mcp add nanobanana -s user \
  -e GEMINI_API_KEY=your-api-key \
  -e NANOBANANA_MODEL=pro \
  -- uv --directory /path/to/nanobanana-mcp-server run python -m nanobanana_mcp_server.server
```

**Features of Python version:**
- **4K Resolution** - Up to 3840x3840 pixels
- **Nano Banana Pro** - Uses `gemini-3-pro-image-preview` (best model)
- **Thinking Levels** - HIGH for complex prompts
- **Search Grounding** - Google Search for real-world references
- **Auto Model Selection** - Detects keywords like "4k", "professional"

**Tools provided:**
- `generate_image` - Create images (up to 4K)
- `edit_image` - Edit existing images
- `get_generation_info` - View generation details

Verify installation:

```bash
claude mcp list
# Should show: nanobanana: uv --directory ... - ✓ Connected
```

---

### 2.4 Verify All MCPs

```bash
claude mcp list
```

Expected output:

```
codex-writer: codex mcp-server - ✓ Connected
gemini-auditor: npx -y @jacob/gemini-cli-mcp - ✓ Connected
nanobanana: uv --directory /path/to/nanobanana-mcp-server run python -m nanobanana_mcp_server.server - ✓ Connected
```

---

## Part 3: Configure Claude as Orchestrator

Create or update your global Claude instructions:

```bash
mkdir -p ~/.claude
nano ~/.claude/CLAUDE.md
```

Paste the following:

```markdown
# Claude Code Orchestrator Instructions

## Role Assignment

You are the **orchestrator**. You coordinate specialized agents:

| Agent | MCP Tool | Role | Model |
|-------|----------|------|-------|
| Codex | `codex-writer` | Prompt rewriting, copywriting, user stories, all writing | Uses $20/month subscription |
| Gemini | `gemini-auditor` | Audits writing quality, validates instructions were followed | Uses $20/month subscription |
| Nanobanana | `nanobanana` | Image generation (4K) and editing | `gemini-3-pro-image-preview` |
| Claude (you) | Native | Programming, CLI commands, code analysis | Claude Opus 4.5 |

---

## Workflow Rules

### Rule 1: Prompt Rewriting (ALWAYS)

Before processing ANY user prompt, delegate to Codex to rewrite it:

1. Call `codex` tool with: "Rewrite this prompt for clarity and completeness: [original prompt]"
2. Use the rewritten prompt for all subsequent work
3. Show user the rewritten prompt

### Rule 2: Writing Tasks

For ANY writing task (copywriting, user stories, documentation prose, marketing, blog posts, creative writing):

1. Spawn a Task agent that uses the `codex` MCP tool
2. Pass the rewritten prompt to Codex
3. Wait for Codex to complete the writing
4. **DO NOT write copy yourself** - always delegate to Codex

Example writing triggers:
- "write a user story"
- "create copy for"
- "draft an email"
- "write documentation"
- "marketing text"
- "blog post"

### Rule 3: Audit All Writing

After Codex completes ANY writing task:

1. Spawn a Task agent that uses the `gemini-auditor` MCP tool
2. Send the original prompt AND Codex's output to Gemini
3. Ask Gemini: "Audit this writing. Did it follow the original instructions? Rate quality 1-10. List any issues."
4. If audit fails (score < 7 or issues found):
   - Send feedback to Codex via `codex-reply`
   - Request revision
   - Re-audit until passing
5. Present final audited output to user

### Rule 4: Programming Tasks

Handle these yourself (Claude):
- Writing code
- Debugging
- Code review
- CLI commands
- Git operations
- File operations

For heavy programming tasks, spawn your own Task agents with `subagent_type='general-purpose'`.

### Rule 5: Image Generation

For ANY image request, use the Nanobanana MCP (Python version with Nano Banana Pro):
- Use `mcp__nanobanana__generate_image` for new images (supports up to 4K resolution)
- Use `mcp__nanobanana__edit_image` for modifications

**Advanced features available:**
- Request "4k" or "high resolution" for maximum quality
- Thinking level HIGH is enabled by default for complex prompts
- Search grounding available for real-world references

Image triggers:
- "generate an image"
- "create a picture"
- "make a photo"
- "design a logo"
- "edit this image"
- "4k image of..."

---

## Example Workflows

### Example: User Story Request

```
User: "Write a user story for login with OAuth"

1. [Codex Rewrite] -> "Create a user story in standard format (As a... I want... So that...) for implementing OAuth-based login functionality"

2. [Codex Write] -> Produces user story

3. [Gemini Audit] -> "Score: 8/10. Follows format. Suggest adding acceptance criteria."

4. [Codex Revise] -> Adds acceptance criteria

5. [Gemini Re-audit] -> "Score: 9/10. Approved."

6. [Return to user]
```

### Example: Code + Docs Request

```
User: "Add a logout button and document it"

1. [Codex Rewrite] -> Clarified prompt

2. [Claude] -> Implements logout button (programming)

3. [Codex] -> Writes documentation (writing)

4. [Gemini] -> Audits documentation

5. [Return to user]
```

### Example: Image Request

```
User: "Generate a logo for my coffee shop"

1. [Codex Rewrite] -> "Generate a professional logo for a coffee shop. Style: modern, minimalist. Include coffee cup imagery."

2. [Nanobanana] -> gemini_generate_image with enhanced prompt

3. [Return image to user]
```

---

## Tool Reference

### Codex Writer Tools
- `codex` - Start new writing session
- `codex-reply` - Continue/revise existing session

### Gemini Auditor Tools
- `gemini_chat` - Send audit request
- Other tools as exposed by the MCP wrapper

### Nanobanana Image Tools (Python - Nano Banana Pro)
- `mcp__nanobanana__generate_image` - Generate new images (up to 4K resolution)
- `mcp__nanobanana__edit_image` - Edit existing images
- `mcp__nanobanana__get_generation_info` - View generation details

**Model:** `gemini-3-pro-image-preview` (best quality)
**Features:** 4K resolution, thinking levels, search grounding, auto model selection

---

## Important Notes

1. **Subscriptions**: Both Codex and Gemini use your $20/month subscriptions, not API credits
2. **No Self-Writing**: Claude must NEVER write copy directly - always use Codex
3. **Mandatory Audit**: ALL Codex output must be audited by Gemini before returning to user
4. **Parallel Agents**: When possible, run independent agents in parallel using multiple Task tool calls
```

---

## Part 4: Test the Setup

### Test 1: Writing Task

```bash
claude
```

Then type:

```
Write a user story for a shopping cart checkout feature
```

Expected behavior:
1. Codex rewrites prompt
2. Codex writes user story
3. Gemini audits
4. Final output returned

### Test 2: Programming Task

```
Create a Python function that validates email addresses
```

Expected: Claude handles directly (no Codex/Gemini involvement)

### Test 3: Image Generation

```
Generate an image of a sunset over mountains
```

Expected: Nanobanana's `gemini_generate_image` is called

### Test 4: Mixed Task

```
Build a contact form component and write the user documentation for it
```

Expected:
1. Claude builds the component
2. Codex writes documentation
3. Gemini audits documentation

---

## Part 5: Troubleshooting

### Codex MCP not connecting

```bash
# Verify Codex is authenticated
codex --login

# Test Codex MCP server directly
codex mcp-server
# Should start without errors

# Re-add to Claude
claude mcp remove codex-writer
claude mcp add codex-writer -s user -- codex mcp-server
```

### Gemini MCP not connecting

```bash
# Verify Gemini is authenticated
gemini
# Type /auth to check status

# Try alternative wrapper
claude mcp remove gemini-auditor
claude mcp add gemini-auditor -s user -- npx -y gemini-mcp-tool
```

### Check all MCP status

```bash
claude mcp list
```

### View MCP logs

```bash
# Start Claude with verbose logging
claude --debug
```

---

## Part 6: Quick Reference Card

| Task Type | Handler | MCP Tool |
|-----------|---------|----------|
| Prompt rewriting | Codex | `codex` |
| User stories | Codex | `codex` |
| Copywriting | Codex | `codex` |
| Documentation prose | Codex | `codex` |
| Blog posts | Codex | `codex` |
| Writing audit | Gemini | `gemini-auditor` |
| Quality check | Gemini | `gemini-auditor` |
| Programming | Claude | Native |
| CLI commands | Claude | Native |
| Git operations | Claude | Native |
| Code review | Claude | Native |
| Image generation | Nanobanana (Python) | `generate_image` (4K, Nano Banana Pro) |
| Image editing | Nanobanana (Python) | `edit_image` (Nano Banana Pro) |

---

## Summary Commands

```bash
# Install CLIs
npm install -g @anthropic-ai/claude-code
npm install -g @openai/codex
npm install -g @google/gemini-cli

# Authenticate
claude          # Sign in with Anthropic
codex --login   # Sign in with ChatGPT (uses $20/month subscription)
gemini          # Login with Google (uses $20/month subscription)

# Add MCPs to Claude
claude mcp add codex-writer -s user -- codex mcp-server
claude mcp add gemini-auditor -s user -- npx -y gemini-mcp-tool
claude mcp add nanobanana -s user \
  -e GEMINI_API_KEY=your-api-key \
  -e NANOBANANA_MODEL=pro \
  -- uv --directory /path/to/nanobanana-mcp-server run python -m nanobanana_mcp_server.server

# Verify
claude mcp list

# Start orchestrator
claude
```

**Models Used:**
- **Codex:** ChatGPT Plus/Pro subscription (no API credits)
- **Gemini Auditor:** Google AI Pro subscription (no API credits)
- **Nanobanana:** `gemini-3-pro-image-preview` (Nano Banana Pro - best quality, 4K)

---

## Sources

- [Claude Code GitHub](https://github.com/anthropics/claude-code)
- [Codex CLI GitHub](https://github.com/openai/codex)
- [Gemini CLI GitHub](https://github.com/google-gemini/gemini-cli)
- [Codex MCP Docs](https://developers.openai.com/codex/mcp/)
- [Gemini CLI MCP Wrapper](https://lobehub.com/mcp/jacob-gemini-cli-mcp)
- [Docker MCP Toolkit](https://docs.docker.com/ai/mcp-catalog-and-toolkit/toolkit/)
