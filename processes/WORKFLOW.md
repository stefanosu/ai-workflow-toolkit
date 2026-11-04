# Development Workflow

A regimented process for AI-assisted development from start to finish.

---

## Overview

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│  Planning   │ → │ Scaffolding │ → │Implementation│ → │ Refinement  │ → │Documentation│
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘
```

Each stage has clear inputs, outputs, and checkpoints.

---

## Stage 1: Planning

### Purpose
Define what you're building before touching code.

### Inputs
- Raw requirements (user story, ticket, idea)
- Constraints (tech stack, timeline, resources)

### Activities
1. **Clarify requirements** - What exactly needs to happen?
2. **Identify non-goals** - What are you NOT building?
3. **Define success criteria** - How will you know it works?
4. **Find the smallest increment** - What's the MVP of this feature?
5. **Identify unknowns** - What technical risks exist?

### Outputs
- [ ] Clear problem statement (1-2 sentences)
- [ ] List of requirements
- [ ] List of constraints/non-goals
- [ ] Definition of done
- [ ] Identified technical risks

### Checkpoint
Can you explain what you're building to someone in 30 seconds?

---

## Stage 2: Scaffolding

### Purpose
Set up project structure and verify it runs before adding logic.

### Inputs
- Clear requirements from Stage 1
- Technology decisions

### Activities
1. **Create project structure** - Folders, config files
2. **Set up dependencies** - Only what you need
3. **Create placeholder files** - Empty modules/components
4. **Verify it runs** - Project compiles, server starts
5. **Initial commit** - Clean starting point

### Outputs
- [ ] Project structure created
- [ ] Dependencies installed
- [ ] Project runs (even if it does nothing)
- [ ] Git initialized with first commit

### Checkpoint
Does `npm start` / `python main.py` / etc run without errors?

---

## Stage 3: Implementation

### Purpose
Build features incrementally, testing each before moving on.

### Process Per Feature

```
┌──────────────┐
│ Pick smallest│
│ next feature │
└──────┬───────┘
       ↓
┌──────────────┐
│Write/generate│
│    code      │
└──────┬───────┘
       ↓
┌──────────────┐
│  Run & test  │←──┐
│  manually    │   │
└──────┬───────┘   │
       ↓           │
   Works? ────No───┘
       │
      Yes
       ↓
┌──────────────┐
│   Commit     │
└──────┬───────┘
       ↓
┌──────────────┐
│ Next feature │
└──────────────┘
```

### Activities
1. **One feature at a time** - Never parallel unfinished features
2. **Run after every change** - Catch errors immediately
3. **Ask for explanations** - Understand complex logic before accepting
4. **Commit working states** - Every green state gets committed
5. **Document decisions** - Why this approach?

### Commit Cadence
- After each working feature
- After each bug fix
- After each refactor
- Basically: if it works, commit it

### Outputs
- [ ] Feature implemented
- [ ] Feature tested manually
- [ ] Code committed
- [ ] You can explain what the code does

### Checkpoint
Can you modify this code without AI help?

---

## Stage 4: Refinement

### Purpose
Harden the code with tests, edge case handling, and review.

### Activities
1. **Generate test cases** - Happy path, edge cases, errors
2. **Run adversarial testing** - Try to break your code
3. **Review for edge cases** - What inputs didn't you consider?
4. **Self-review** - Have AI explain code back to you
5. **Simplify** - Remove unnecessary complexity

### Test Categories
- [ ] Happy path works
- [ ] Edge cases handled (empty, null, max values)
- [ ] Invalid inputs rejected gracefully
- [ ] Error messages are helpful

### Outputs
- [ ] Test suite passing
- [ ] Edge cases documented and handled
- [ ] Code reviewed (self or AI-assisted)
- [ ] Unnecessary complexity removed

### Checkpoint
Would you be comfortable debugging this at 2am?

---

## Stage 5: Documentation

### Purpose
Capture knowledge for future you and others.

### Activities
1. **Write README** - Setup, run, test instructions
2. **Document decisions** - Why, not just what
3. **Add code comments** - For non-obvious parts only
4. **Create API docs** - If applicable
5. **Note trade-offs** - What would change at scale?

### Documentation Checklist
- [ ] How to install/run
- [ ] How to test
- [ ] Architecture overview
- [ ] Key decisions and rationale
- [ ] Known limitations
- [ ] Future improvements

### Checkpoint
Could a new team member get this running in 15 minutes?

---

## Session Workflow

### Starting a Session

```markdown
## Session Goal
[One clear objective for this session]

## Context
[What state is the project in?]

## Success Criteria
[How will I know I'm done?]

## Constraints
[What am I NOT doing today?]
```

### During a Session

1. **Focus on one thing** - Resist scope creep
2. **Commit frequently** - Every working state
3. **Take breaks** - Fresh eyes catch bugs
4. **Ask "why"** - For any code you don't understand
5. **Note what works** - Capture good prompts

### Ending a Session

```markdown
## Completed
- [x] Thing 1
- [x] Thing 2

## Remaining
- [ ] Thing 3 (next session)

## Prompts That Worked
- [Paste effective prompts for reuse]

## Notes
- [Decisions made]
- [Things that didn't work]
- [Ideas for later]
```

---

## Git Workflow

### Branch Strategy (Simple)
```
main ─────────────────────────────────
  │
  └── feature/add-search ──┬── commit
                           ├── commit
                           ├── commit
                           └── merge → main
```

### Commit Message Format
```
type: brief description

- Detail 1
- Detail 2

Types: feat, fix, refactor, test, docs, chore
```

### When to Commit
- Feature complete and working
- Bug fixed and tested
- Refactor complete, tests pass
- Before risky changes (safety checkpoint)

### When NOT to Commit
- Code doesn't run
- Tests are failing
- Mid-feature (incomplete state)

---

## Error Recovery

### When Stuck

```
1. Stop and read the error carefully
           ↓
2. Form a hypothesis about the cause
           ↓
3. Test the hypothesis (add logging, try simpler case)
           ↓
4. If confirmed, ask for targeted fix
   If not, return to step 1
```

### When AI Code Fails
1. **Don't regenerate blindly** - Understand the failure
2. **Share the error** - Give AI full context
3. **Share your diagnosis** - What you think went wrong
4. **Ask for targeted fix** - Not a rewrite

### When Completely Lost
1. **Git stash or branch** - Save current state
2. **Start fresh conversation** - Clear polluted context
3. **Break down smaller** - What's the tiniest working piece?
4. **Build up incrementally** - One thing at a time

---

## Quality Gates

### Before Committing
- [ ] Code runs without errors
- [ ] Feature works as intended
- [ ] No obvious edge case failures
- [ ] You understand every line

### Before PR/Merge
- [ ] All tests pass
- [ ] Code reviewed (self or peer)
- [ ] Documentation updated
- [ ] No debug code left behind

### Before Shipping
- [ ] Tested in production-like environment
- [ ] Error handling in place
- [ ] Logging for debugging
- [ ] Rollback plan exists

---

## Time Boundaries

### Signs You Should Stop
- Going in circles on the same error
- Adding complexity to work around problems
- Can't explain what you're trying to do
- Frustration building

### What to Do Instead
- Take a break (seriously)
- Write down the problem clearly
- Sleep on it
- Ask a human for help

### The 15-Minute Rule
If you're stuck for 15 minutes on the same issue:
1. Stop coding
2. Write down exactly what's wrong
3. Form a hypothesis
4. If still stuck, take a break or ask for help
