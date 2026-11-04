# Project Kickoff Checklist

Use this before starting any new project.

---

## Pre-Flight Checklist

### 1. Requirements Clarity
- [ ] Can I explain what I'm building in one sentence?
- [ ] Do I have clear success criteria?
- [ ] Have I identified what's NOT in scope?
- [ ] Are there open questions I need answered first?

### 2. Technical Decisions
- [ ] Tech stack chosen and justified?
- [ ] Development environment set up?
- [ ] Version control initialized?
- [ ] Project structure planned?

### 3. Scope Management
- [ ] Features prioritized (must-have vs nice-to-have)?
- [ ] MVP defined (smallest working version)?
- [ ] Constraints documented?

---

## Project Brief Template

Fill this out before writing code:

```markdown
## Project: [Name]

### One-Liner
[What does this do in one sentence?]

### Problem
[What problem does this solve?]

### Solution
[How does this solve it?]

### Requirements
Must have:
- [ ] Feature 1
- [ ] Feature 2

Nice to have:
- [ ] Feature 3

Out of scope:
- Feature 4 (not building this)

### Success Criteria
How I'll know it's done:
- [ ] Criterion 1
- [ ] Criterion 2

### Tech Stack
- Frontend: [choice + why]
- Backend: [choice + why]
- Database: [choice + why]
- Hosting: [choice + why]

### Constraints
- No external libraries for X
- Must work offline
- [Other constraints]

### Risks
- [Technical unknown 1]
- [Dependency on X]
```

---

## First Session Plan

### Session 1: Scaffolding
Goal: Project runs (even if it does nothing)

- [ ] Create project structure
- [ ] Install dependencies
- [ ] Verify it builds/runs
- [ ] Initial git commit

### Session 2: Core Feature
Goal: One feature works end-to-end

- [ ] Implement smallest useful feature
- [ ] Test it manually
- [ ] Commit working code

### Session 3+: Iterate
Goal: Add features one at a time

- [ ] Pick next feature
- [ ] Implement
- [ ] Test
- [ ] Commit
- [ ] Repeat

---

## Git Setup Checklist

- [ ] `git init`
- [ ] Create `.gitignore` (use template for your stack)
- [ ] Initial commit with project structure
- [ ] Set up remote (GitHub/GitLab)
- [ ] First push

### .gitignore Essentials
```
# Dependencies
node_modules/
venv/
__pycache__/

# Build
dist/
build/
*.pyc

# Environment
.env
.env.local
*.pem

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db
```

---

## Environment Setup

### Node.js Projects
```bash
npm init -y
# or
npx create-vite@latest my-app
```

### Python Projects
```bash
python -m venv venv
source venv/bin/activate  # or venv\Scripts\activate on Windows
pip install [packages]
pip freeze > requirements.txt
```

### Common First Commands
```bash
# Create structure
mkdir -p src/{components,services,utils}

# Git
git init
git add .
git commit -m "Initial project structure"

# Verify it runs
npm start  # or python main.py, etc.
```

---

## Questions to Ask Before Starting

1. **Scope**: What's the smallest version that's useful?
2. **Users**: Who will use this? What do they need?
3. **Data**: What data flows through? How is it stored?
4. **Scale**: How big does this need to get?
5. **Time**: What's the deadline? What can I cut?
6. **Risks**: What's the hardest part? What could fail?

---

## Anti-Patterns to Avoid

### At Start
- Starting to code before requirements are clear
- Over-planning without building anything
- Choosing complex stack for simple problem
- Not setting up version control first

### During Build
- Adding features not in requirements
- Not committing working code
- Building without testing
- Ignoring edge cases until the end

### At End
- No documentation
- No tests
- "It works on my machine"
- Forgetting to capture learnings
