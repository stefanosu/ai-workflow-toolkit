# AI-Assisted Development: Best Practices

A comprehensive guide for effective AI collaboration in software development.

---

## Core Principles

### 1. Incremental Over Monolithic
**Bad**: "Build me a full-stack app with auth, search, filtering, and pagination"
**Good**: "Create the data models first" → "Now add filtering" → "Now add pagination"

Why: Smaller chunks are easier to verify, debug, and understand. You catch errors early.

### 2. Specification Before Implementation
**Bad**: "Make a search function"
**Good**: "Create a search function that takes X, returns Y, handles Z edge cases"

Why: Ambiguity leads to assumptions. Assumptions lead to rewrites.

### 3. Constraints Are Features
**Bad**: "Build a React component for search"
**Good**: "Build a React component for search. Use only useState, no external libraries, minimal styling"

Why: AI tends to over-engineer. Explicit constraints keep code simple and maintainable.

### 4. Read Before Write
**Always** read/understand existing code before asking AI to modify it.

Why: Context matters. AI without context makes incompatible changes.

### 5. Verify, Don't Trust
Run every piece of generated code. Test edge cases yourself.

Why: AI is confident, not correct. It will generate plausible-looking bugs.

---

## Session Hygiene

### Before Each Session
- [ ] Clear goal statement written down
- [ ] List of what this session should accomplish
- [ ] Constraints and non-goals identified
- [ ] Relevant context gathered (existing code, requirements)

### During Development
- [ ] Run code after every generation
- [ ] Commit working states frequently
- [ ] Ask "why" for complex suggestions
- [ ] Note prompts that work well

### After Each Session
- [ ] Capture winning prompts to your library
- [ ] Document decisions made and why
- [ ] Note what didn't work for future reference
- [ ] Clean up any temporary/experimental code

---

## Git Checkpoint Discipline

### The Rule
Commit after every working increment, even if small.

### Why It Matters
- Rollback safety when AI-generated code breaks things
- Clear history of what changed and when
- Easier to isolate which change caused a bug
- Demonstrates professional discipline

### Commit Hygiene
```
Good commits:
- feat: add price filter to search endpoint
- fix: handle empty results in pagination
- refactor: extract scoring logic to separate module

Bad commits:
- updates
- fix
- wip
- changes
```

### Never
- Let AI push directly - always review diffs first
- Commit without running tests
- Bundle unrelated changes in one commit

---

## Context Window Management

### Start Fresh When
- Switching to an unrelated task
- Context has become polluted with failed attempts
- You're going in circles with the same error

### Manage Context By
- Feeding relevant code snippets, not entire files
- Summarizing progress before starting new session
- Clearing conversation when switching domains
- Being explicit about what's changed since last prompt

### Signs of Context Pollution
- AI references code you've since deleted
- Suggestions contradict recent decisions
- Same errors keep recurring
- AI seems confused about project structure

---

## The Teaching Test

### The Standard
If you can't explain the code to someone else, you don't understand it.

### How to Apply
1. Ask AI to explain complex code back to you
2. If the explanation confuses you, simplify the code
3. Before accepting code, verbalize what it does
4. Teach it to a rubber duck (or out loud to yourself)

### Red Flags
- You can't modify the code without AI help
- You don't know what would break if you changed something
- You can't predict what the code does for edge cases
- You'd be lost debugging this at 2am

---

## Error Recovery Pattern

### When AI Code Fails

```
Step 1: Don't regenerate blindly
        ↓
Step 2: Understand WHY it failed
        - Read the error message carefully
        - Identify the root cause
        - Form a hypothesis
        ↓
Step 3: Give AI the error + your diagnosis
        "This failed with [ERROR] because [YOUR ANALYSIS]"
        ↓
Step 4: Ask for targeted fix, not rewrite
        "Fix the X in function Y" not "Regenerate everything"
```

### Anti-Patterns
- Regenerating the same prompt hoping for different results
- Adding more code to work around the bug
- Asking AI to "fix it" without error context
- Abandoning your approach entirely after one failure

---

## The Ownership Mindset

### The Philosophy
"AI assisted, human owned" - you're responsible for every line.

### What This Means
- You debug it
- You maintain it
- You explain it in code reviews
- You're on-call when it breaks

### Implications
- Never ship code you don't understand
- Always be able to modify without AI
- Document the "why" not just the "what"
- AI is a tool like an IDE - the work is still yours

---

## Documenting Decision Points

### What to Capture
- Why did you choose approach A over B?
- What trade-offs did you make?
- What constraints influenced the design?
- What would you change for production?

### Why It Matters
- Interview gold - shows engineering judgment
- Future you will thank present you
- Makes code reviews more productive
- Helps onboard others to the codebase

### Format
```markdown
## Decision: [Title]

**Context**: What situation required a decision?

**Options Considered**:
1. Option A - pros/cons
2. Option B - pros/cons

**Decision**: What we chose and why

**Consequences**: What trade-offs this creates
```

---

## Prompting Patterns

### Pattern 1: The Sandwich
```
[CONTEXT]
What the system currently does

[REQUEST]
What you want to add/change

[CONSTRAINTS]
What to avoid, boundaries, non-goals
```

### Pattern 2: Example-Driven (Few-Shot)
```
Here's how X works:
- Input A → Output B
- Input C → Output D

Now implement the function that does this.
```

### Pattern 3: Chain of Thought
```
I need to implement [COMPLEX THING].

Think through:
1. What are the inputs and outputs?
2. What are the edge cases?
3. What's the simplest approach?

Then implement with comments explaining each step.
```

### Pattern 4: Rubber Duck Reverse
```
Here's my code:
[CODE]

Explain what it does step by step.
Then identify any bugs or edge cases I missed.
```

### Pattern 5: Adversarial Testing
```
Here's my function:
[CODE]

Generate 10 test cases designed to break it:
- Boundary values
- Invalid inputs
- Empty/null cases
```

---

## Red Flags to Watch For

| Red Flag | What It Means | What to Do |
|----------|---------------|------------|
| AI adds libraries you didn't ask for | Over-engineering | Re-prompt with explicit constraints |
| Code is longer than expected | Premature abstraction | Ask for simpler version |
| AI says "you might also want..." | Scope creep | Stay focused on current task |
| Generated code doesn't run | Hallucinated APIs | Verify imports/methods exist |
| AI confidently explains wrong code | Confabulation | Test it yourself |
| Same error keeps appearing | Context pollution | Start fresh conversation |

---

## Building Your Prompt Library

### Why
- Saves time on common tasks
- Captures what works through iteration
- Becomes a competitive advantage
- Compounds knowledge over time

### Organization
```
prompts/
├── scaffolding/     # Project setup, boilerplate
├── debugging/       # Error analysis, fix patterns
├── testing/         # Test generation, edge cases
├── refactoring/     # Code improvement patterns
├── documentation/   # README, comments, docs
└── domain/          # Project-specific prompts
```

### Iteration Process
1. Use a prompt
2. Note what worked/didn't
3. Refine for next time
4. Capture winning versions

---

## The Meta-Skill

The real skill isn't prompting—it's **knowing what to build**.

AI accelerates implementation, but you still need to:
- Understand the problem domain
- Make architectural decisions
- Evaluate trade-offs
- Debug production issues
- Communicate with stakeholders

Use AI to move faster through the parts you understand.
Slow down for the parts you don't.

---

## Interview Talking Points

### What to Say
- "I use AI as a force multiplier, not a replacement for understanding"
- "I verify everything AI generates before committing"
- "I prompt incrementally to catch errors early"
- "I can explain and modify any code in my solution"

### What to Demonstrate
- You understand the generated code
- You can modify it on the fly
- You know the trade-offs of your approach
- You can debug without AI help

### What to Avoid
- Copying AI output without understanding it
- Saying "AI wrote this" as a deflection
- Being unable to explain any line of code
- Over-relying on AI for simple changes
