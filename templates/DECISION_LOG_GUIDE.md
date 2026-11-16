# Decision Log Guide

## What is a Decision Log?

A decision log (also called Architecture Decision Records or ADRs) is a collection of documents that capture important architectural and technical decisions made during a project. Each decision is documented with its context, rationale, alternatives considered, and consequences.

## Why Use Decision Logs?

**For Current Development:**
- Makes implicit decisions explicit
- Forces you to think through trade-offs
- Creates accountability for choices
- Helps onboard new team members

**For Future Maintenance:**
- Remember why you chose approach X over Y
- Avoid revisiting settled decisions
- Understand historical context when things break
- Learn from past decisions

**For Communication:**
- Share reasoning with stakeholders
- Build consensus on approach
- Document tribal knowledge
- Create material for presentations and writing

## When to Create a Decision Log Entry

Create a decision log entry when:

1. **Choosing between multiple valid approaches**
   - Database: SQL vs NoSQL
   - Architecture: Monolith vs Microservices
   - Library: Framework A vs Framework B

2. **Making a significant architectural choice**
   - API design patterns
   - Authentication strategy
   - Caching layer implementation

3. **Deviating from standard practices**
   - Using a less common tool
   - Implementing a custom solution instead of off-the-shelf
   - Breaking with team conventions

4. **Solving a complex technical problem**
   - Performance optimization approaches
   - Data migration strategies
   - Security implementations

5. **Adding or removing major dependencies**
   - New framework or library
   - Removing a core component

**Don't create decision logs for:**
- Trivial choices (variable names, file locations)
- Obvious best practices (use version control, write tests)
- Temporary debugging changes
- Routine bug fixes

## How to Use This Template

### 1. Create a decisions directory
```bash
mkdir -p docs/decisions
```

### 2. Copy the template
```bash
cp templates/DECISION_LOG_TEMPLATE.md docs/decisions/001-your-decision-name.md
```

### 3. Number sequentially
- `001-use-pydantic-settings.md`
- `002-choose-postgres-over-mongodb.md`
- `003-implement-hybrid-retrieval.md`

### 4. Fill in each section
- **Title**: Clear, concise description
- **Status**: Usually "Accepted" for implemented decisions
- **Context**: Why is this decision needed? What problem are we solving?
- **Decision**: What did we decide? One clear statement.
- **Rationale**: Why this choice? List 3-5 key reasons.
- **Alternatives**: What else did we consider? Why not those?
- **Consequences**: What are the trade-offs? Be honest about downsides.

### 5. Keep it concise
Aim for 1-2 pages. If it's longer, the decision might be too broad.

### 6. Update status as needed
- **Proposed**: Under discussion
- **Accepted**: Decision made and implemented
- **Deprecated**: No longer recommended but still in use
- **Superseded**: Replaced by a newer decision (link to it)

## Decision Log Workflow

### During Planning
```
1. Identify decision point
2. Research alternatives
3. Create decision log with Status: Proposed
4. Share with team/stakeholders
5. Discuss and refine
6. Update Status: Accepted (or Rejected)
```

### During Implementation
```
1. Reference decision log in commit messages
   "feat: implement Pydantic settings (see decision 001)"
2. Update Implementation Notes section with actual code
3. Add any unexpected consequences discovered
```

### During Maintenance
```
1. When someone asks "why did we choose X?"
   → Point them to the decision log
2. When revisiting a decision
   → Create new decision log that supersedes old one
   → Update old log's status to "Superseded by 015"
```

## Example Directory Structure

```
docs/
├── decisions/
│   ├── 001-use-pydantic-settings.md
│   ├── 002-postgres-with-pgvector.md
│   ├── 003-no-langchain.md
│   ├── 004-hybrid-retrieval-rrf.md
│   ├── 005-claude-api-direct.md
│   └── README.md  (index of all decisions)
└── architecture.md
```

## Tips for Writing Good Decision Logs

### Do:
- Write for future you who forgot the context
- Be specific about trade-offs
- Include code examples where helpful
- Link to external resources
- Update if circumstances change

### Don't:
- Rationalize bad decisions - be honest
- Write a novel - keep it focused
- Skip alternatives - show you considered options
- Leave status as "Proposed" forever
- Delete old decisions - mark as deprecated instead

## Decision Log Anti-Patterns

**The Rubber Stamp**: Writing decision log after the fact to justify already-made choice
- Fix: Write it when actually making the decision

**The Novel**: 10-page decision log covering everything
- Fix: Break into multiple focused decisions

**The Ghost**: Decision made but never documented
- Fix: Make decision logs part of your workflow

**The Zombie**: Decision deprecated but still referenced as current
- Fix: Update status and link to superseding decision

**The Bikeshed**: Excessive detail on trivial choices
- Fix: Only log significant architectural decisions

## Integration with Presentations and Writing

Decision logs are excellent source material for:

**Technical Talks:**
- "Here's a problem we faced and how we solved it"
- Show the alternatives considered
- Discuss the trade-offs

**Blog Posts:**
- Turn decision logs into case studies
- Share lessons learned
- Help others facing similar choices

**Team Demos:**
- Walk through recent decisions
- Explain rationale to new team members
- Review consequences after implementation

**Post-Mortems:**
- Reference decisions that led to issues
- Understand what went wrong
- Document lessons learned

## Example: Decision Log to Presentation

**Decision Log**: "003-no-langchain.md"
```markdown
Decision: Use Claude API directly instead of LangChain
Rationale:
- Need full control over prompts
- Reduce dependencies
- Simpler debugging
Consequences:
- Must implement retry logic ourselves
- Miss out on LangChain abstractions
```

**Presentation Slide**:
```
Why We Skipped LangChain

Problem: Need RAG system for legal contracts
Options: LangChain vs Direct API

We chose Direct API because:
✓ Full prompt control
✓ Minimal dependencies
✓ Easier debugging

Trade-off: Implemented our own retry logic
Result: 500 lines of code, zero dependency issues
```

## Related Resources

- [Architecture Decision Records (ADRs)](https://adr.github.io/)
- [Documenting Architecture Decisions by Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
- [ADR Tools](https://github.com/npryce/adr-tools)

## Quick Start

```bash
# 1. Create decisions directory
mkdir -p docs/decisions

# 2. Create your first decision log
cp templates/DECISION_LOG_TEMPLATE.md docs/decisions/001-first-decision.md

# 3. Edit and fill it out
# Focus on: Context, Decision, Rationale, Alternatives

# 4. Reference in commits
git commit -m "feat: implement X (see decision 001)"

# 5. Share with team
# Decision logs are living documents - update as you learn
```

Start small. Even documenting one decision per week builds valuable institutional knowledge over time.
