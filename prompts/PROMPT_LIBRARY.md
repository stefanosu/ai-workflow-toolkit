# Prompt Library

Reusable prompts organized by development phase and technique.

---

## Techniques Reference

| Technique | When to Use | Key Benefit |
|-----------|-------------|-------------|
| **Specification-First** | Starting new components | Eliminates ambiguity |
| **Chain of Thought** | Complex logic | Forces explainable reasoning |
| **Few-Shot** | Pattern-based code | Examples beat descriptions |
| **Constraint-Based** | Preventing bloat | Keeps code minimal |
| **Rubber Duck** | Debugging/reviewing | Reveals gaps in understanding |
| **Iterative Refinement** | Improving code | Targeted changes vs rewrites |

---

## Project Setup

### Scaffolding (Specification-First)
```
Create a [FRAMEWORK] project structure for [DESCRIPTION].

Requirements:
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

Constraints:
- No [thing to avoid]
- Keep it minimal - no extra [middleware/config/etc]
- [Other constraints]

Output the directory structure first, then each file.
```

### Project Kickoff Questions
```
I'm starting a new project: [DESCRIPTION]

Before I begin, help me think through:
1. What are the core requirements vs nice-to-haves?
2. What's the smallest working version I can build first?
3. What are the riskiest technical unknowns?
4. What constraints should I set to avoid over-engineering?
```

---

## Data Modeling

### Pydantic/Schema Models
```
Create [MODEL TYPE] models for [DOMAIN].

Fields needed:
- field1: type (description)
- field2: type (description)
- [...]

Validation rules:
- [Rule 1]
- [Rule 2]

Include type hints and docstrings.
```

### Database Schema
```
Design a database schema for [DOMAIN].

Entities:
- Entity1: [description]
- Entity2: [description]

Relationships:
- Entity1 has many Entity2
- [...]

Constraints:
- Use [PostgreSQL/MySQL/etc]
- Include indexes for [common queries]
- Keep it normalized / denormalized for [reason]
```

---

## Core Logic

### Function Implementation (Constraint-Based)
```
Implement a [function_name] function.

Input: [describe inputs with types]
Output: [describe output with type]

Logic:
- [Step 1]
- [Step 2]
- [Step 3]

Edge cases to handle:
- [Edge case 1]
- [Edge case 2]

Keep it simple:
- No [thing to avoid]
- Just straightforward [description]
```

### Complex Algorithm (Chain of Thought)
```
Design a [algorithm description].

Context:
- [Relevant background]
- [Constraints or requirements]

Think through this step by step:
1. What makes [X] work well?
2. How should we handle [edge case]?
3. What's the simplest approach that works?
4. What could go wrong?

Then implement with comments explaining each step.
```

### Pagination (Few-Shot)
```
Implement pagination for [data type].

Example behavior:
- 25 total, page=1, pageSize=10 → items 0-9, totalPages=3
- 25 total, page=2, pageSize=10 → items 10-19, totalPages=3
- 25 total, page=3, pageSize=10 → items 20-24, totalPages=3
- 25 total, page=4, pageSize=10 → empty list, totalPages=3
- 0 total, page=1, pageSize=10 → empty list, totalPages=0

Returns: { items, totalResults, page, pageSize, totalPages }
```

---

## Error Handling

### Validation Errors
```
Add input validation to [endpoint/function] with clear errors.

Validation rules:
1. [Rule] → error
2. [Rule] → error

Error response format:
{
  "error": "error_type",
  "message": "Human readable message",
  "field": "field_that_failed"
}

Return HTTP [status code] for validation errors.
```

### Error Recovery
```
This code is failing with:
[ERROR MESSAGE]

Current code:
[CODE]

My analysis: I think the issue is [YOUR HYPOTHESIS].

Please:
1. Confirm or correct my analysis
2. Provide a targeted fix (don't rewrite everything)
3. Explain why the fix works
```

---

## Testing

### Test Suite Generation
```
Write [pytest/jest/etc] tests for [module/component].

Test categories needed:

1. Happy path:
   - [Test case]
   - [Test case]

2. Edge cases:
   - [Test case]
   - [Test case]

3. Error cases:
   - [Test case]
   - [Test case]

Use fixtures for [sample data description].
Each test should be independent and clearly named.
```

### Adversarial Test Cases
```
Here's my function:
[CODE]

Generate 10 test cases designed to break it:
- Boundary values
- Invalid inputs
- Unexpected types
- Empty/null cases
- Concurrent access (if applicable)
- Large inputs
```

---

## Frontend

### Component Implementation
```
Create a [React/Vue/etc] component for [DESCRIPTION].

Requirements:
- [Requirement 1]
- [Requirement 2]
- [Requirement 3]

State needed:
- [state1]: description
- [state2]: description

Props:
- [prop1]: type - description
- [prop2]: type - description

Keep styling minimal - focus on functionality.
Do NOT use: [libraries to avoid]
```

### API Integration
```
Implement API client for [endpoint].

Endpoint: [METHOD] [URL]
Request: [describe request format]
Response: [describe response format]

Handle:
- Loading state
- Success state
- Error state
- Empty results

Use only [fetch/axios/etc], no additional libraries.
```

---

## Debugging

### Code Explanation (Rubber Duck)
```
Here's my code:
[CODE]

Explain what it does step by step.
Then identify:
- Any bugs
- Edge cases I missed
- Potential improvements (but don't implement unless asked)
```

### Bug Investigation
```
I'm seeing this behavior:
Expected: [EXPECTED]
Actual: [ACTUAL]

Relevant code:
[CODE]

Help me:
1. Identify possible causes
2. Suggest debugging steps
3. Propose a fix once we identify the issue
```

---

## Refactoring

### Code Improvement
```
This code works but needs improvement:
[CODE]

Issues I see:
- [Issue 1]
- [Issue 2]

Please refactor to address these issues.
Keep changes minimal - don't refactor unrelated code.
Maintain the same external interface.
```

### Extract Function
```
Extract [logic description] from this code into a separate function:
[CODE]

The new function should:
- Be named [suggested_name]
- Take [parameters]
- Return [return type]
- Be testable in isolation
```

---

## Documentation

### README Generation
```
Write a README for this project.

Include:
1. What it does (1-2 sentences)
2. How to install/run
3. How to run tests
4. Key design decisions and why
5. What you'd change for production

Write as if explaining to a new team member.
```

### Code Comments
```
Add comments to this code explaining the non-obvious parts:
[CODE]

Don't comment on obvious things (variable assignments, etc).
Focus on:
- Why this approach was chosen
- Edge cases being handled
- Complex logic that needs explanation
```

---

## Live Session / New Requirements

### Adding Features
```
I need to add [NEW FEATURE] to my existing [SYSTEM].

Current architecture:
- [Component 1]
- [Component 2]
- [Component 3]

The new requirement is: [DESCRIBE IT]

Questions:
1. Where does this fit in the current architecture?
2. What existing code needs to change?
3. What new code is needed?
4. What edge cases should I consider?

Think through this before writing code.
```

### Quick Modifications
```
Modify [FUNCTION/COMPONENT] to also [NEW BEHAVIOR].

Current code:
[CODE]

Change needed: [SPECIFIC CHANGE]

Keep the modification minimal - don't refactor unrelated code.
```

---

## System Design

### Architecture Discussion
```
I need to design [SYSTEM DESCRIPTION].

Requirements:
- [Functional requirement 1]
- [Functional requirement 2]
- [Non-functional: scale/performance/etc]

Constraints:
- [Technology constraints]
- [Resource constraints]

Walk me through:
1. High-level components needed
2. How they communicate
3. Key trade-offs to consider
4. Potential bottlenecks
```

### Trade-off Analysis
```
I'm deciding between:
Option A: [Description]
Option B: [Description]

Context: [Relevant constraints]

Compare:
- Complexity
- Performance
- Maintainability
- Scalability
- Development time

Recommend one and explain why for my situation.
```
