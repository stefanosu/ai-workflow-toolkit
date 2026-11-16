# TypeScript/Node Implementation Guide

Production-ready patterns for TypeScript/Node.js projects following modern best practices.

## Table of Contents
1. [Project Structure](#project-structure)
2. [Configuration Management](#configuration-management)
3. [Type Safety](#type-safety)
4. [Database Patterns](#database-patterns)
5. [Testing](#testing)
6. [Error Handling](#error-handling)
7. [Logging](#logging)
8. [Dependencies](#dependencies)

---

## Project Structure

### Standard Layout
```
my-project/
├── src/
│   ├── config/
│   │   └── index.ts           # Configuration
│   ├── models/
│   │   ├── domain.ts          # Zod schemas
│   │   └── database.ts        # Prisma/TypeORM models
│   ├── services/              # Business logic
│   │   └── processor.ts
│   ├── utils/                 # Utilities
│   │   └── helpers.ts
│   ├── api/                   # API layer
│   │   └── server.ts
│   └── index.ts               # Entry point
├── tests/
│   ├── setup.ts               # Test setup
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── docs/
│   └── decisions/             # Architecture decisions
├── package.json
├── tsconfig.json
├── .env.example
├── .gitignore
└── README.md
```

### Key Principles
- Use TypeScript strict mode
- Separate concerns: models, services, API
- Avoid default exports (use named exports)
- One feature per file/module

---

## Configuration Management

### Using Zod for Validation

**src/config/index.ts**
```typescript
import { z } from 'zod';
import dotenv from 'dotenv';

dotenv.config();

const configSchema = z.object({
  // Required settings
  databaseUrl: z.string().url(),
  apiKey: z.string().min(1),

  // Optional with defaults
  environment: z.enum(['development', 'staging', 'production']).default('development'),
  logLevel: z.enum(['debug', 'info', 'warn', 'error']).default('info'),
  maxConnections: z.coerce.number().int().positive().default(10),
  port: z.coerce.number().int().positive().default(3000),

  // Feature flags
  enableCaching: z.coerce.boolean().default(true),
});

export type Config = z.infer<typeof configSchema>;

function loadConfig(): Config {
  const rawConfig = {
    databaseUrl: process.env.DATABASE_URL,
    apiKey: process.env.API_KEY,
    environment: process.env.NODE_ENV,
    logLevel: process.env.LOG_LEVEL,
    maxConnections: process.env.MAX_CONNECTIONS,
    port: process.env.PORT,
    enableCaching: process.env.ENABLE_CACHING,
  };

  try {
    return configSchema.parse(rawConfig);
  } catch (error) {
    if (error instanceof z.ZodError) {
      console.error('Configuration validation failed:');
      console.error(error.errors);
      process.exit(1);
    }
    throw error;
  }
}

// Singleton instance
let config: Config | null = null;

export function getConfig(): Config {
  if (!config) {
    config = loadConfig();
  }
  return config;
}
```

**.env.example**
```bash
# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb

# API Keys
API_KEY=your-api-key-here

# Application
NODE_ENV=development
LOG_LEVEL=info
PORT=3000
MAX_CONNECTIONS=10

# Features
ENABLE_CACHING=true
```

### Best Practices
- Use Zod for runtime validation
- Fail fast on invalid config
- Singleton pattern for config access
- Provide `.env.example` without secrets
- Use `z.coerce` for type conversion

---

## Type Safety

### TypeScript Configuration

**tsconfig.json**
```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "lib": ["ES2022"],
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "allowJs": false,
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "noImplicitOverride": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist",
    "rootDir": "./src",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

### Zod Schemas for Validation

```typescript
import { z } from 'zod';

// Schema definition
export const userSchema = z.object({
  id: z.string().uuid(),
  email: z.string().email(),
  name: z.string().min(1).max(100),
  age: z.number().int().min(0).max(150).optional(),
  tags: z.array(z.string()).default([]),
  createdAt: z.date().default(() => new Date()),
});

// Infer TypeScript type from schema
export type User = z.infer<typeof userSchema>;

// Partial schema for updates
export const userUpdateSchema = userSchema.partial().omit({ id: true, createdAt: true });
export type UserUpdate = z.infer<typeof userUpdateSchema>;

// Usage
function createUser(data: unknown): User {
  // Parse and validate
  const user = userSchema.parse(data);
  return user;
}

// Safe parsing (no throw)
function tryCreateUser(data: unknown): User | null {
  const result = userSchema.safeParse(data);
  if (result.success) {
    return result.data;
  } else {
    console.error('Validation errors:', result.error.errors);
    return null;
  }
}
```

### Utility Types

```typescript
// Readonly utility
export type Immutable<T> = {
  readonly [K in keyof T]: T[K];
};

// Require specific fields
export type RequireFields<T, K extends keyof T> = T & Required<Pick<T, K>>;

// Result type for error handling
export type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

// Usage
function processData(input: string): Result<number> {
  const parsed = parseInt(input, 10);
  if (isNaN(parsed)) {
    return { success: false, error: new Error('Invalid number') };
  }
  return { success: true, data: parsed };
}
```

---

## Database Patterns

### Prisma (Recommended)

**prisma/schema.prisma**
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model Contract {
  id        String   @id @default(uuid())
  content   String
  status    String   @default("pending")
  createdAt DateTime @default(now())

  @@index([status, createdAt])
  @@map("contracts")
}
```

**src/models/database.ts**
```typescript
import { PrismaClient } from '@prisma/client';
import { getConfig } from '../config';

const config = getConfig();

// Singleton pattern
const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined;
};

export const prisma =
  globalForPrisma.prisma ??
  new PrismaClient({
    log: config.logLevel === 'debug' ? ['query', 'error', 'warn'] : ['error'],
  });

if (config.environment !== 'production') {
  globalForPrisma.prisma = prisma;
}

// Repository pattern
export class ContractRepository {
  async findById(id: string) {
    return prisma.contract.findUnique({
      where: { id },
    });
  }

  async create(data: { content: string }) {
    return prisma.contract.create({
      data: {
        content: data.content,
        status: 'pending',
      },
    });
  }

  async findByStatus(status: string) {
    return prisma.contract.findMany({
      where: { status },
      orderBy: { createdAt: 'desc' },
    });
  }

  async updateStatus(id: string, status: string) {
    return prisma.contract.update({
      where: { id },
      data: { status },
    });
  }
}

export const contractRepository = new ContractRepository();
```

### pgvector Integration

```typescript
import { Prisma } from '@prisma/client';

// Add to schema.prisma
// model Document {
//   id        String                    @id @default(uuid())
//   content   String
//   embedding Unsupported("vector(1536)")?
// }

export class DocumentRepository {
  async findSimilar(embedding: number[], limit: number = 5) {
    // Using raw SQL for pgvector
    return prisma.$queryRaw<Array<{ id: string; content: string; distance: number }>>`
      SELECT id, content, embedding <=> ${Prisma.sql`${embedding}::vector`} as distance
      FROM documents
      ORDER BY embedding <=> ${Prisma.sql`${embedding}::vector`}
      LIMIT ${limit}
    `;
  }

  async create(content: string, embedding: number[]) {
    return prisma.$executeRaw`
      INSERT INTO documents (id, content, embedding)
      VALUES (gen_random_uuid(), ${content}, ${Prisma.sql`${embedding}::vector`})
    `;
  }
}
```

---

## Testing

### Vitest Configuration

**vitest.config.ts**
```typescript
import { defineConfig } from 'vitest/config';

export default defineConfig({
  test: {
    globals: true,
    environment: 'node',
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: ['node_modules/', 'dist/', 'tests/'],
    },
    setupFiles: ['./tests/setup.ts'],
  },
});
```

### Test Setup

**tests/setup.ts**
```typescript
import { beforeAll, afterAll, afterEach } from 'vitest';
import { prisma } from '../src/models/database';

beforeAll(async () => {
  // Setup test database
  process.env.DATABASE_URL = 'postgresql://localhost:5432/test_db';
});

afterEach(async () => {
  // Clean up after each test
  await prisma.contract.deleteMany();
});

afterAll(async () => {
  // Disconnect
  await prisma.$disconnect();
});
```

### Test Examples

**tests/unit/services/processor.test.ts**
```typescript
import { describe, it, expect, beforeEach } from 'vitest';
import { processContract } from '../../../src/services/processor';

describe('processContract', () => {
  describe('happy path', () => {
    it('processes valid contract', () => {
      const content = 'Contract content here';
      const result = processContract(content);

      expect(result.status).toBe('success');
      expect(result.sections).toHaveLength(1);
    });

    it('returns expected structure', () => {
      const content = 'Contract content';
      const result = processContract(content);

      expect(result).toHaveProperty('status');
      expect(result).toHaveProperty('sections');
      expect(result).toHaveProperty('metadata');
    });
  });

  describe('edge cases', () => {
    it('handles empty content', () => {
      const result = processContract('');
      expect(result.status).toBe('error');
    });

    it('handles unicode characters', () => {
      const content = 'Contract with émojis 🎉';
      const result = processContract(content);
      expect(result.status).toBe('success');
    });

    it('handles very long content', () => {
      const content = 'a'.repeat(1_000_000);
      const result = processContract(content);
      expect(result.status).toBe('success');
    });
  });
});

describe('processContract integration', () => {
  it('saves to database', async () => {
    const content = 'Contract content';
    const result = await processContract(content, { saveToDb: true });

    const saved = await prisma.contract.findFirst();
    expect(saved).toBeDefined();
    expect(saved?.content).toBe(content);
  });
});
```

### Parametrized Tests

```typescript
import { describe, it, expect } from 'vitest';

describe('countWords', () => {
  it.each([
    ['hello world', 2],
    ['', 0],
    ['one', 1],
    ['multiple words here', 3],
  ])('counts words in "%s" as %i', (input, expected) => {
    expect(countWords(input)).toBe(expected);
  });
});
```

### Running Tests

```bash
# All tests
npm test

# Watch mode
npm test -- --watch

# Coverage
npm test -- --coverage

# Specific file
npm test processor.test.ts

# UI mode
npm test -- --ui
```

---

## Error Handling

### Custom Error Classes

```typescript
export class ApplicationError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly statusCode: number = 500
  ) {
    super(message);
    this.name = this.constructor.name;
    Error.captureStackTrace(this, this.constructor);
  }
}

export class ValidationError extends ApplicationError {
  constructor(message: string) {
    super(message, 'VALIDATION_ERROR', 400);
  }
}

export class NotFoundError extends ApplicationError {
  constructor(resource: string, id: string) {
    super(`${resource} with id ${id} not found`, 'NOT_FOUND', 404);
  }
}

export class DatabaseError extends ApplicationError {
  constructor(message: string) {
    super(message, 'DATABASE_ERROR', 500);
  }
}
```

### Error Handling Pattern

```typescript
import { logger } from '../utils/logger';

export interface ProcessResult {
  status: 'success' | 'error';
  data?: unknown;
  error?: string;
}

export function processContract(content: string): ProcessResult {
  try {
    // Validation
    if (!content.trim()) {
      throw new ValidationError('Contract content cannot be empty');
    }

    // Processing
    const sections = splitIntoSections(content);

    if (sections.length === 0) {
      throw new ApplicationError('Failed to extract sections', 'PROCESSING_ERROR');
    }

    return {
      status: 'success',
      data: { sections },
    };
  } catch (error) {
    if (error instanceof ValidationError) {
      logger.warn('Validation failed', { error: error.message });
      return {
        status: 'error',
        error: error.message,
      };
    }

    if (error instanceof ApplicationError) {
      logger.error('Processing failed', {
        code: error.code,
        message: error.message,
      });
      return {
        status: 'error',
        error: error.message,
      };
    }

    // Unexpected error
    logger.error('Unexpected error', { error });
    return {
      status: 'error',
      error: 'Internal error occurred',
    };
  }
}
```

### Result Type Pattern

```typescript
export type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

export async function fetchUser(id: string): Promise<Result<User>> {
  try {
    const user = await prisma.user.findUnique({ where: { id } });

    if (!user) {
      return {
        success: false,
        error: new NotFoundError('User', id),
      };
    }

    return { success: true, data: user };
  } catch (error) {
    return {
      success: false,
      error: error instanceof Error ? error : new Error('Unknown error'),
    };
  }
}

// Usage
const result = await fetchUser('123');
if (result.success) {
  console.log(result.data.name);
} else {
  console.error(result.error.message);
}
```

---

## Logging

### Pino Logger Setup

**src/utils/logger.ts**
```typescript
import pino from 'pino';
import { getConfig } from '../config';

const config = getConfig();

export const logger = pino({
  level: config.logLevel,
  transport:
    config.environment === 'development'
      ? {
          target: 'pino-pretty',
          options: {
            colorize: true,
            translateTime: 'SYS:standard',
            ignore: 'pid,hostname',
          },
        }
      : undefined,
  formatters: {
    level: (label) => {
      return { level: label };
    },
  },
  timestamp: pino.stdTimeFunctions.isoTime,
});

// Child logger for specific modules
export function createLogger(module: string) {
  return logger.child({ module });
}
```

**Usage**
```typescript
import { createLogger } from '../utils/logger';

const logger = createLogger('processor');

export function processData(data: Record<string, unknown>) {
  logger.info(
    {
      dataSize: Object.keys(data).length,
      dataType: typeof data,
    },
    'Processing data'
  );

  try {
    const result = doWork(data);

    logger.info(
      {
        resultCount: result.length,
      },
      'Processing complete'
    );

    return result;
  } catch (error) {
    logger.error(
      {
        error,
        dataKeys: Object.keys(data),
      },
      'Processing failed'
    );
    throw error;
  }
}
```

---

## Dependencies

### Package.json

```json
{
  "name": "my-project",
  "version": "0.1.0",
  "type": "module",
  "engines": {
    "node": ">=20.0.0"
  },
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js",
    "test": "vitest",
    "test:ui": "vitest --ui",
    "lint": "eslint src/",
    "format": "prettier --write src/",
    "typecheck": "tsc --noEmit"
  },
  "dependencies": {
    "@prisma/client": "^5.0.0",
    "zod": "^3.22.0",
    "dotenv": "^16.3.0",
    "pino": "^8.15.0"
  },
  "devDependencies": {
    "@types/node": "^20.0.0",
    "typescript": "^5.2.0",
    "tsx": "^4.0.0",
    "vitest": "^1.0.0",
    "prisma": "^5.0.0",
    "eslint": "^8.50.0",
    "prettier": "^3.0.0",
    "@vitest/ui": "^1.0.0"
  }
}
```

### Commands

```bash
# Install dependencies
npm install

# Development
npm run dev

# Build
npm run build

# Start production
npm start

# Tests
npm test

# Type check
npm run typecheck

# Lint
npm run lint

# Format
npm run format
```

---

## Complete Example

**src/index.ts**
```typescript
import { getConfig } from './config';
import { logger } from './utils/logger';
import { contractRepository } from './models/database';
import { processContract } from './services/processor';

async function main() {
  const config = getConfig();

  logger.info(
    {
      environment: config.environment,
      port: config.port,
    },
    'Starting application'
  );

  try {
    // Process contract
    const content = 'Sample contract content';
    const result = processContract(content);

    if (result.status === 'success' && result.data) {
      // Save to database
      const contract = await contractRepository.create({
        content,
      });

      logger.info(
        {
          contractId: contract.id,
          sections: (result.data as any).sections.length,
        },
        'Contract processed successfully'
      );
    } else {
      logger.error(
        {
          error: result.error,
        },
        'Processing failed'
      );
    }
  } catch (error) {
    logger.error({ error }, 'Application error');
    process.exit(1);
  }
}

main();
```

---

## Quick Reference

### Common Patterns

```typescript
// Configuration
import { getConfig } from './config';
const config = getConfig();

// Database
import { prisma } from './models/database';
const users = await prisma.user.findMany();

// Logging
import { createLogger } from './utils/logger';
const logger = createLogger('module-name');
logger.info({ key: 'value' }, 'message');

// Validation
import { z } from 'zod';
const schema = z.object({ name: z.string() });
const data = schema.parse(input);

// Error handling
try {
  await riskyOperation();
} catch (error) {
  if (error instanceof ValidationError) {
    logger.warn({ error: error.message });
  }
  throw error;
}
```

### Best Practices Checklist

- [ ] TypeScript strict mode enabled
- [ ] Zod schemas for validation
- [ ] Environment variables validated with Zod
- [ ] Structured logging with Pino
- [ ] Custom error classes
- [ ] Prisma for database access
- [ ] Comprehensive test coverage
- [ ] Named exports (avoid default exports)
- [ ] Result type for error handling
- [ ] Repository pattern for data access

---

## Related Resources

- [TypeScript Documentation](https://www.typescriptlang.org/docs/)
- [Zod Documentation](https://zod.dev/)
- [Prisma Documentation](https://www.prisma.io/docs/)
- [Vitest Documentation](https://vitest.dev/)
- [Pino Documentation](https://getpino.io/)
