# Language-Agnostic Implementation Guides

This directory contains implementation guides for common software engineering patterns across different programming languages. Each guide shows how to implement the same core concepts idiomatically in each language.

## Purpose

The patterns and best practices in this toolkit are language-agnostic. These guides show you how to implement them in your language of choice, following that language's idioms and conventions.

## Available Languages

- [Python](./PYTHON.md) - Python 3.12+, type hints, Pydantic, pytest
- [TypeScript/Node](./TYPESCRIPT.md) - TypeScript, Zod, Vitest/Jest
- [Go](./GO.md) - Go 1.21+, standard library, testify
- [Rust](./RUST.md) - Rust 1.70+, serde, tokio

## Core Patterns Covered

Each language guide covers these essential patterns:

1. **Project Structure** - Standard directory layout
2. **Configuration Management** - Environment variables, type-safe config
3. **Type Safety** - Type systems and validation
4. **Database Patterns** - Connection pooling, query patterns
5. **Testing** - Unit tests, integration tests, fixtures
6. **Error Handling** - Idiomatic error patterns
7. **Logging** - Structured logging
8. **Dependencies** - Package management

## How to Use These Guides

1. **Starting a new project**: Read your language's guide for recommended structure and patterns
2. **Learning a new language**: Compare your familiar language with the new one to see equivalent patterns
3. **Code review**: Reference these guides for best practices
4. **Refactoring**: Use as a checklist for modernizing legacy code

## Philosophy

These guides prioritize:
- **Idiomatic code**: Follow language conventions, not force patterns from other languages
- **Production readiness**: Patterns you'd use in real systems, not toy examples
- **Simplicity**: Standard library and well-established tools, avoid exotic dependencies
- **Type safety**: Leverage type systems to catch errors early
- **Testability**: Code that's easy to test

## Contributing

When adding a new language guide:
1. Follow the structure of existing guides
2. Show idiomatic code for that language
3. Include working code examples
4. Explain trade-offs and alternatives
5. Keep it focused on core patterns

## Pattern Comparison Table

| Pattern | Python | TypeScript | Go | Rust |
|---------|--------|------------|-----|------|
| Config | Pydantic Settings | Zod schemas | viper/envconfig | config crate |
| Testing | pytest | Vitest/Jest | testing package | cargo test |
| DB ORM | SQLAlchemy | Prisma/TypeORM | GORM/sqlx | diesel/sqlx |
| Validation | Pydantic | Zod | validator | serde + custom |
| HTTP Server | FastAPI | Express/Fastify | net/http, gin | axum/actix |
| Async | asyncio | async/await | goroutines | tokio/async-std |
| Package Manager | uv/pip | npm/pnpm/bun | go modules | cargo |

## Quick Reference

### Configuration
```python
# Python - Pydantic
class Settings(BaseSettings):
    database_url: str

# TypeScript - Zod
const settings = z.object({
    databaseUrl: z.string()
})

# Go - envconfig
type Config struct {
    DatabaseURL string `envconfig:"DATABASE_URL"`
}

# Rust - config
#[derive(Deserialize)]
struct Config {
    database_url: String,
}
```

### Testing
```python
# Python
def test_function():
    assert result == expected

# TypeScript
test('function works', () => {
    expect(result).toBe(expected)
})

# Go
func TestFunction(t *testing.T) {
    assert.Equal(t, expected, result)
}

# Rust
#[test]
fn test_function() {
    assert_eq!(result, expected);
}
```

## Related Resources

- [processes/BEST_PRACTICES.md](../processes/BEST_PRACTICES.md) - General best practices
- [templates/DECISION_LOG_TEMPLATE.md](../templates/DECISION_LOG_TEMPLATE.md) - Document decisions
- [system-design/](../system-design/) - Architecture patterns

---

Remember: Good code in any language is readable, testable, and maintainable. Use these guides as a starting point, then adapt to your specific needs.
