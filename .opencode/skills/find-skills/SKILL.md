---
name: find-skills
description: Helps users discover and install agent skills from the open agent skills ecosystem. Use when user asks "how do I do X", "find a skill for X", "is there a skill that can..." or wants to extend capabilities.
---

# Find Skills

This skill helps discover and install skills from the open agent skills ecosystem.

## When to Use

- User asks "how do I do X" where X might have an existing skill
- Says "find a skill for X" or "is there a skill for X"
- Expresses interest in extending agent capabilities

## What is Skills CLI?

The Skills CLI (`npx skills`) is the package manager for the open agent skills ecosystem.

**Key commands:**
- `npx skills find [query]` - Search for skills interactively
- `npx skills add <package>` - Install a skill from GitHub
- `npx skills check` - Check for skill updates
- `npx skills update` - Update all installed skills

**Browse skills at:** https://skills.sh/

## How to Help Users

### Step 1: Search for Skills

```bash
npx skills find [query]
```

Example searches:
- "react performance" → React optimization skills
- "pr review" → Code review skills
- "changelog" → Documentation skills

### Step 2: Present Options

When you find relevant skills:
1. Skill name and what it does
2. Install command
3. Link to learn more

### Step 3: Install

```bash
npx skills add <owner/repo@skill> -g -y
```

The `-g` flag installs globally, `-y` skips confirmation.

## Common Categories

| Category | Queries |
|----------|---------|
| Web Development | react, nextjs, typescript, tailwind |
| Testing | testing, jest, playwright, e2e |
| DevOps | deploy, docker, kubernetes, ci-cd |
| Documentation | docs, readme, changelog |
| Code Quality | review, lint, refactor |
| Design | ui, ux, design-system |

## When No Skills Found

1. Acknowledge no existing skill was found
2. Offer to help directly with general capabilities
3. Suggest creating own skill: `npx skills init my-skill`