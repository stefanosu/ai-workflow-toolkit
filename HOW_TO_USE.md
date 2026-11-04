# How to Use This Toolkit

Best practices for referencing this repo in your workflow.

---

## The Golden Rule

**Reference, don't copy.**

This repo is your external brain - keep it separate, pull from it as needed.

---

## Setup (One Time)

### 1. Clone Locally
```bash
git clone https://github.com/stefanosu/ai-workflow-toolkit.git ~/toolkit
```

Now you have instant local access at `~/toolkit`.

### 2. Create Shell Aliases (Optional)
Add to your `~/.zshrc` or `~/.bashrc`:

```bash
# Quick access to toolkit docs
alias tk="cd ~/toolkit"
alias tkprompts="cat ~/toolkit/prompts/PROMPT_LIBRARY.md"
alias tkworkflow="cat ~/toolkit/processes/WORKFLOW.md"
alias tkalgos="cat ~/toolkit/algorithms/PATTERNS.md"
alias tkdesign="cat ~/toolkit/system-design/FRAMEWORK.md"
alias tkkickoff="cat ~/toolkit/templates/PROJECT_KICKOFF.md"

# Open in VS Code
alias tkedit="code ~/toolkit"
```

Reload: `source ~/.zshrc`

### 3. Bookmark in Browser
- Star the GitHub repo
- Bookmark specific files you use often

---

## When Starting a New Project

### Step 1: Review Kickoff Checklist
```bash
cat ~/toolkit/templates/PROJECT_KICKOFF.md
```

Or open: https://github.com/stefanosu/ai-workflow-toolkit/blob/main/templates/PROJECT_KICKOFF.md

### Step 2: Add Reference to Project README
In your new project's README:

```markdown
## Development

This project follows my [AI-assisted workflow](https://github.com/stefanosu/ai-workflow-toolkit).

Key references:
- [Workflow Process](https://github.com/stefanosu/ai-workflow-toolkit/blob/main/processes/WORKFLOW.md)
- [Prompt Library](https://github.com/stefanosu/ai-workflow-toolkit/blob/main/prompts/PROMPT_LIBRARY.md)
```

This:
- Documents your process for others (and future you)
- Shows interviewers your systematic approach
- Links to living docs that stay updated

### Step 3: Follow the Workflow
Reference `processes/WORKFLOW.md` as you work through stages:
1. Planning
2. Scaffolding
3. Implementation
4. Refinement
5. Documentation

---

## During Development

### When You Need a Prompt
```bash
# View full library
cat ~/toolkit/prompts/PROMPT_LIBRARY.md

# Search for specific type
grep -A 20 "### Debugging" ~/toolkit/prompts/PROMPT_LIBRARY.md
```

### When You're Stuck on an Algorithm
```bash
# Check patterns
cat ~/toolkit/algorithms/PATTERNS.md

# Check mental models
cat ~/toolkit/algorithms/MENTAL_MODELS.md
```

### When Designing a System
```bash
cat ~/toolkit/system-design/FRAMEWORK.md
```

---

## Keeping Toolkit Updated

### Pull Latest Changes
```bash
cd ~/toolkit && git pull
```

### Add Your Own Improvements
```bash
cd ~/toolkit

# Edit files
code .

# Commit and push
git add .
git commit -m "Add new prompt pattern for X"
git push
```

### After Each Project
Ask yourself:
- Did I discover a new prompt that worked well?
- Did I learn a new pattern?
- Should I update any docs?

Add learnings back to the toolkit.

---

## Quick Reference Card

| I Need... | Command |
|-----------|---------|
| Start new project | `cat ~/toolkit/templates/PROJECT_KICKOFF.md` |
| Workflow stages | `cat ~/toolkit/processes/WORKFLOW.md` |
| Prompt templates | `cat ~/toolkit/prompts/PROMPT_LIBRARY.md` |
| Best practices | `cat ~/toolkit/prompts/BEST_PRACTICES.md` |
| Algorithm pattern | `cat ~/toolkit/algorithms/PATTERNS.md` |
| Problem solving | `cat ~/toolkit/algorithms/MENTAL_MODELS.md` |
| System design | `cat ~/toolkit/system-design/FRAMEWORK.md` |
| Practice log | `cat ~/toolkit/templates/PRACTICE_LOG.md` |

---

## Integration with Claude / AI Tools

When starting a session, you can paste relevant sections:

```
I'm following this workflow process:
[Paste from WORKFLOW.md]

For this session, I'm in Stage 3: Implementation.

Current task: [describe]
```

Or reference specific prompts:

```
Using this prompt pattern:
[Paste from PROMPT_LIBRARY.md]

Applied to my situation:
[Your specific context]
```

---

## What NOT to Do

| Don't | Why | Instead |
|-------|-----|---------|
| Copy files into projects | Creates duplicates that diverge | Reference via links |
| Add as git submodule | Overkill for docs | Clone separately |
| Memorize everything | That's what the docs are for | Reference as needed |
| Skip the kickoff checklist | You'll miss requirements | 5 min upfront saves hours |
| Forget to log practice | Lose insights and progress | Log daily, review weekly |

---

## File Structure Reminder

```
~/toolkit/
├── README.md              # Start here
├── HOW_TO_USE.md          # This file
├── prompts/
│   ├── BEST_PRACTICES.md  # Core principles
│   └── PROMPT_LIBRARY.md  # Copy-paste prompts
├── processes/
│   └── WORKFLOW.md        # Step-by-step process
├── system-design/
│   ├── FRAMEWORK.md       # Design approach
│   └── templates/         # Architecture patterns
├── algorithms/
│   ├── PATTERNS.md        # Code patterns
│   └── MENTAL_MODELS.md   # Thinking frameworks
└── templates/
    ├── PROJECT_KICKOFF.md # New project checklist
    └── PRACTICE_LOG.md    # Daily practice tracker
```
