# Go Implementation Guide

Production-ready patterns for Go 1.21+ projects following Go idioms and best practices.

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
├── cmd/
│   └── api/
│       └── main.go            # Application entry point
├── internal/
│   ├── config/
│   │   └── config.go          # Configuration
│   ├── models/
│   │   ├── domain.go          # Domain models
│   │   └── database.go        # Database models
│   ├── services/              # Business logic
│   │   └── processor.go
│   ├── repository/            # Data access layer
│   │   └── contract.go
│   └── api/                   # API handlers
│       └── handlers.go
├── pkg/                       # Public libraries (optional)
│   └── utils/
│       └── helpers.go
├── tests/
│   ├── integration/
│   └── fixtures/
├── docs/
│   └── decisions/
├── go.mod
├── go.sum
├── .env.example
├── .gitignore
└── README.md
```

### Key Principles
- Use `internal/` for private code
- Use `cmd/` for application entry points
- Use `pkg/` for reusable libraries
- Keep packages focused and small
- Follow standard Go project layout

---

## Configuration Management

### Using envconfig

**internal/config/config.go**
```go
package config

import (
	"fmt"
	"os"

	"github.com/kelseyhightower/envconfig"
	"github.com/joho/godotenv"
)

type Config struct {
	// Database
	DatabaseURL string `envconfig:"DATABASE_URL" required:"true"`

	// API
	APIKey string `envconfig:"API_KEY" required:"true"`
	Port   int    `envconfig:"PORT" default:"8080"`

	// Application
	Environment string `envconfig:"ENVIRONMENT" default:"development"`
	LogLevel    string `envconfig:"LOG_LEVEL" default:"info"`

	// Features
	EnableCaching bool `envconfig:"ENABLE_CACHING" default:"true"`
	MaxWorkers    int  `envconfig:"MAX_WORKERS" default:"10"`
}

var cfg *Config

// Load reads configuration from environment variables
func Load() (*Config, error) {
	// Load .env file if it exists (optional in production)
	_ = godotenv.Load()

	var c Config
	if err := envconfig.Process("", &c); err != nil {
		return nil, fmt.Errorf("failed to load config: %w", err)
	}

	// Validation
	if err := c.Validate(); err != nil {
		return nil, fmt.Errorf("config validation failed: %w", err)
	}

	cfg = &c
	return cfg, nil
}

// Get returns the global config instance
func Get() *Config {
	if cfg == nil {
		panic("config not loaded - call Load() first")
	}
	return cfg
}

// Validate performs custom validation
func (c *Config) Validate() error {
	validEnvs := map[string]bool{
		"development": true,
		"staging":     true,
		"production":  true,
	}

	if !validEnvs[c.Environment] {
		return fmt.Errorf("invalid environment: %s", c.Environment)
	}

	validLogLevels := map[string]bool{
		"debug": true,
		"info":  true,
		"warn":  true,
		"error": true,
	}

	if !validLogLevels[c.LogLevel] {
		return fmt.Errorf("invalid log level: %s", c.LogLevel)
	}

	return nil
}

// IsDevelopment returns true if running in development
func (c *Config) IsDevelopment() bool {
	return c.Environment == "development"
}

// IsProduction returns true if running in production
func (c *Config) IsProduction() bool {
	return c.Environment == "production"
}
```

**.env.example**
```bash
# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb

# API
API_KEY=your-api-key-here
PORT=8080

# Application
ENVIRONMENT=development
LOG_LEVEL=info
MAX_WORKERS=10

# Features
ENABLE_CACHING=true
```

### Best Practices
- Use struct tags for environment variables
- Validate configuration at startup
- Provide sensible defaults
- Fail fast on invalid config
- Use singleton pattern for global config

---

## Type Safety

### Struct Definitions

```go
package models

import (
	"fmt"
	"time"
)

// User represents a user in the system
type User struct {
	ID        string    `json:"id" db:"id"`
	Email     string    `json:"email" db:"email"`
	Name      string    `json:"name" db:"name"`
	Age       *int      `json:"age,omitempty" db:"age"`
	Tags      []string  `json:"tags" db:"tags"`
	CreatedAt time.Time `json:"created_at" db:"created_at"`
}

// Validate validates user data
func (u *User) Validate() error {
	if u.ID == "" {
		return fmt.Errorf("id is required")
	}
	if u.Email == "" {
		return fmt.Errorf("email is required")
	}
	if u.Name == "" {
		return fmt.Errorf("name is required")
	}
	if u.Age != nil && (*u.Age < 0 || *u.Age > 150) {
		return fmt.Errorf("age must be between 0 and 150")
	}
	return nil
}

// UserInput represents user input for creation
type UserInput struct {
	Email string   `json:"email"`
	Name  string   `json:"name"`
	Age   *int     `json:"age"`
	Tags  []string `json:"tags"`
}

// ToUser converts UserInput to User
func (i *UserInput) ToUser(id string) *User {
	return &User{
		ID:        id,
		Email:     i.Email,
		Name:      i.Name,
		Age:       i.Age,
		Tags:      i.Tags,
		CreatedAt: time.Now(),
	}
}
```

### Using go-validator

```go
import "github.com/go-playground/validator/v10"

var validate = validator.New()

type CreateUserRequest struct {
	Email string   `json:"email" validate:"required,email"`
	Name  string   `json:"name" validate:"required,min=1,max=100"`
	Age   int      `json:"age" validate:"omitempty,gte=0,lte=150"`
	Tags  []string `json:"tags" validate:"omitempty,dive,min=1"`
}

func (r *CreateUserRequest) Validate() error {
	return validate.Struct(r)
}
```

### Interfaces and Contracts

```go
// Repository defines the contract for data access
type Repository interface {
	Create(ctx context.Context, data interface{}) error
	FindByID(ctx context.Context, id string) (interface{}, error)
	Update(ctx context.Context, id string, data interface{}) error
	Delete(ctx context.Context, id string) error
}

// Processor defines contract for processing logic
type Processor interface {
	Process(ctx context.Context, input string) (*ProcessResult, error)
	Validate(input string) error
}
```

---

## Database Patterns

### Using sqlx

**internal/repository/database.go**
```go
package repository

import (
	"context"
	"fmt"
	"time"

	"github.com/jmoiron/sqlx"
	_ "github.com/lib/pq"
)

type DB struct {
	*sqlx.DB
}

// NewDB creates a new database connection with pooling
func NewDB(databaseURL string) (*DB, error) {
	db, err := sqlx.Connect("postgres", databaseURL)
	if err != nil {
		return nil, fmt.Errorf("failed to connect to database: %w", err)
	}

	// Connection pool settings
	db.SetMaxOpenConns(25)
	db.SetMaxIdleConns(5)
	db.SetConnMaxLifetime(5 * time.Minute)

	// Verify connection
	ctx, cancel := context.WithTimeout(context.Background(), 5*time.Second)
	defer cancel()

	if err := db.PingContext(ctx); err != nil {
		return nil, fmt.Errorf("failed to ping database: %w", err)
	}

	return &DB{db}, nil
}

// Close closes the database connection
func (db *DB) Close() error {
	return db.DB.Close()
}
```

**internal/repository/contract.go**
```go
package repository

import (
	"context"
	"database/sql"
	"fmt"
	"time"

	"github.com/google/uuid"
)

type Contract struct {
	ID        string    `db:"id"`
	Content   string    `db:"content"`
	Status    string    `db:"status"`
	CreatedAt time.Time `db:"created_at"`
}

type ContractRepository struct {
	db *DB
}

func NewContractRepository(db *DB) *ContractRepository {
	return &ContractRepository{db: db}
}

// Create creates a new contract
func (r *ContractRepository) Create(ctx context.Context, content string) (*Contract, error) {
	contract := &Contract{
		ID:        uuid.New().String(),
		Content:   content,
		Status:    "pending",
		CreatedAt: time.Now(),
	}

	query := `
		INSERT INTO contracts (id, content, status, created_at)
		VALUES ($1, $2, $3, $4)
	`

	_, err := r.db.ExecContext(ctx, query,
		contract.ID,
		contract.Content,
		contract.Status,
		contract.CreatedAt,
	)

	if err != nil {
		return nil, fmt.Errorf("failed to create contract: %w", err)
	}

	return contract, nil
}

// FindByID finds a contract by ID
func (r *ContractRepository) FindByID(ctx context.Context, id string) (*Contract, error) {
	var contract Contract

	query := `SELECT id, content, status, created_at FROM contracts WHERE id = $1`

	err := r.db.GetContext(ctx, &contract, query, id)
	if err == sql.ErrNoRows {
		return nil, fmt.Errorf("contract not found: %s", id)
	}
	if err != nil {
		return nil, fmt.Errorf("failed to find contract: %w", err)
	}

	return &contract, nil
}

// FindByStatus finds contracts by status
func (r *ContractRepository) FindByStatus(ctx context.Context, status string) ([]*Contract, error) {
	var contracts []*Contract

	query := `
		SELECT id, content, status, created_at
		FROM contracts
		WHERE status = $1
		ORDER BY created_at DESC
	`

	err := r.db.SelectContext(ctx, &contracts, query, status)
	if err != nil {
		return nil, fmt.Errorf("failed to find contracts: %w", err)
	}

	return contracts, nil
}

// UpdateStatus updates contract status
func (r *ContractRepository) UpdateStatus(ctx context.Context, id, status string) error {
	query := `UPDATE contracts SET status = $1 WHERE id = $2`

	result, err := r.db.ExecContext(ctx, query, status, id)
	if err != nil {
		return fmt.Errorf("failed to update contract: %w", err)
	}

	rows, err := result.RowsAffected()
	if err != nil {
		return fmt.Errorf("failed to get rows affected: %w", err)
	}

	if rows == 0 {
		return fmt.Errorf("contract not found: %s", id)
	}

	return nil
}
```

### pgvector Integration

```go
import (
	"github.com/pgvector/pgvector-go"
)

type Document struct {
	ID        string           `db:"id"`
	Content   string           `db:"content"`
	Embedding pgvector.Vector  `db:"embedding"`
}

func (r *DocumentRepository) FindSimilar(ctx context.Context, embedding []float32, limit int) ([]*Document, error) {
	var docs []*Document

	query := `
		SELECT id, content, embedding
		FROM documents
		ORDER BY embedding <=> $1
		LIMIT $2
	`

	vec := pgvector.NewVector(embedding)

	err := r.db.SelectContext(ctx, &docs, query, vec, limit)
	if err != nil {
		return nil, fmt.Errorf("failed to find similar documents: %w", err)
	}

	return docs, nil
}
```

---

## Testing

### Test Structure

**internal/services/processor_test.go**
```go
package services

import (
	"context"
	"testing"

	"github.com/stretchr/testify/assert"
	"github.com/stretchr/testify/require"
)

func TestProcessContract(t *testing.T) {
	t.Run("happy path", func(t *testing.T) {
		t.Run("processes valid contract", func(t *testing.T) {
			content := "Contract content here"
			result, err := ProcessContract(context.Background(), content)

			require.NoError(t, err)
			assert.Equal(t, "success", result.Status)
			assert.NotEmpty(t, result.Sections)
		})

		t.Run("returns expected structure", func(t *testing.T) {
			content := "Contract content"
			result, err := ProcessContract(context.Background(), content)

			require.NoError(t, err)
			assert.NotNil(t, result.Metadata)
		})
	})

	t.Run("edge cases", func(t *testing.T) {
		t.Run("handles empty content", func(t *testing.T) {
			_, err := ProcessContract(context.Background(), "")

			assert.Error(t, err)
			assert.Contains(t, err.Error(), "empty")
		})

		t.Run("handles unicode", func(t *testing.T) {
			content := "Contract with émojis 🎉"
			result, err := ProcessContract(context.Background(), content)

			require.NoError(t, err)
			assert.Equal(t, "success", result.Status)
		})

		t.Run("handles very long content", func(t *testing.T) {
			content := string(make([]byte, 1_000_000))
			_, err := ProcessContract(context.Background(), content)

			require.NoError(t, err)
		})
	})
}
```

### Table-Driven Tests

```go
func TestCountWords(t *testing.T) {
	tests := []struct {
		name     string
		input    string
		expected int
	}{
		{"empty string", "", 0},
		{"single word", "hello", 1},
		{"multiple words", "hello world", 2},
		{"with punctuation", "hello, world!", 2},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			result := CountWords(tt.input)
			assert.Equal(t, tt.expected, result)
		})
	}
}
```

### Test Fixtures

```go
package testutil

import (
	"context"
	"testing"

	"github.com/stretchr/testify/require"
)

// SetupTestDB creates a test database
func SetupTestDB(t *testing.T) *DB {
	db, err := NewDB("postgresql://localhost:5432/test_db")
	require.NoError(t, err)

	t.Cleanup(func() {
		db.Close()
	})

	return db
}

// LoadFixture loads test data
func LoadFixture(t *testing.T, filename string) string {
	content, err := os.ReadFile(filepath.Join("testdata", filename))
	require.NoError(t, err)
	return string(content)
}
```

### Running Tests

```bash
# All tests
go test ./...

# With coverage
go test -cover ./...

# Verbose
go test -v ./...

# Specific package
go test ./internal/services

# Race detection
go test -race ./...

# Generate coverage report
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

---

## Error Handling

### Custom Error Types

```go
package errors

import "fmt"

// Error types
type ErrorCode string

const (
	ErrCodeValidation  ErrorCode = "VALIDATION_ERROR"
	ErrCodeNotFound    ErrorCode = "NOT_FOUND"
	ErrCodeDatabase    ErrorCode = "DATABASE_ERROR"
	ErrCodeInternal    ErrorCode = "INTERNAL_ERROR"
)

// AppError represents an application error
type AppError struct {
	Code    ErrorCode
	Message string
	Err     error
}

func (e *AppError) Error() string {
	if e.Err != nil {
		return fmt.Sprintf("%s: %s: %v", e.Code, e.Message, e.Err)
	}
	return fmt.Sprintf("%s: %s", e.Code, e.Message)
}

func (e *AppError) Unwrap() error {
	return e.Err
}

// Constructor functions
func NewValidationError(message string) *AppError {
	return &AppError{
		Code:    ErrCodeValidation,
		Message: message,
	}
}

func NewNotFoundError(resource, id string) *AppError {
	return &AppError{
		Code:    ErrCodeNotFound,
		Message: fmt.Sprintf("%s with id %s not found", resource, id),
	}
}

func NewDatabaseError(message string, err error) *AppError {
	return &AppError{
		Code:    ErrCodeDatabase,
		Message: message,
		Err:     err,
	}
}
```

### Error Handling Pattern

```go
package services

import (
	"context"
	"fmt"

	"myproject/internal/errors"
	"myproject/internal/logger"
)

type ProcessResult struct {
	Status   string
	Sections []string
	Metadata map[string]interface{}
}

func ProcessContract(ctx context.Context, content string) (*ProcessResult, error) {
	log := logger.FromContext(ctx)

	// Validation
	if content == "" {
		log.Warn("validation failed: empty content")
		return nil, errors.NewValidationError("content cannot be empty")
	}

	// Processing
	sections, err := splitIntoSections(content)
	if err != nil {
		log.Error("processing failed", "error", err)
		return nil, fmt.Errorf("failed to split sections: %w", err)
	}

	if len(sections) == 0 {
		return nil, errors.NewValidationError("no sections found")
	}

	result := &ProcessResult{
		Status:   "success",
		Sections: sections,
		Metadata: map[string]interface{}{
			"section_count": len(sections),
		},
	}

	log.Info("processing complete", "sections", len(sections))
	return result, nil
}
```

### Error Wrapping

```go
import "fmt"

func fetchData(id string) (*Data, error) {
	data, err := repository.Find(id)
	if err != nil {
		// Wrap error with context
		return nil, fmt.Errorf("failed to fetch data for id %s: %w", id, err)
	}
	return data, nil
}

// Error checking with errors.Is and errors.As
import "errors"

if err != nil {
	if errors.Is(err, sql.ErrNoRows) {
		// Handle not found
		return nil, errors.NewNotFoundError("data", id)
	}

	var appErr *errors.AppError
	if errors.As(err, &appErr) {
		// Handle application error
		log.Error("app error", "code", appErr.Code)
	}

	return nil, err
}
```

---

## Logging

### Using slog (Go 1.21+)

**internal/logger/logger.go**
```go
package logger

import (
	"context"
	"log/slog"
	"os"

	"myproject/internal/config"
)

type contextKey string

const loggerKey contextKey = "logger"

// Setup initializes the global logger
func Setup(cfg *config.Config) {
	var level slog.Level
	switch cfg.LogLevel {
	case "debug":
		level = slog.LevelDebug
	case "info":
		level = slog.LevelInfo
	case "warn":
		level = slog.LevelWarn
	case "error":
		level = slog.LevelError
	default:
		level = slog.LevelInfo
	}

	opts := &slog.HandlerOptions{
		Level: level,
	}

	var handler slog.Handler
	if cfg.IsDevelopment() {
		// Text format for development
		handler = slog.NewTextHandler(os.Stdout, opts)
	} else {
		// JSON format for production
		handler = slog.NewJSONHandler(os.Stdout, opts)
	}

	logger := slog.New(handler)
	slog.SetDefault(logger)
}

// WithContext adds logger to context
func WithContext(ctx context.Context, logger *slog.Logger) context.Context {
	return context.WithValue(ctx, loggerKey, logger)
}

// FromContext retrieves logger from context
func FromContext(ctx context.Context) *slog.Logger {
	if logger, ok := ctx.Value(loggerKey).(*slog.Logger); ok {
		return logger
	}
	return slog.Default()
}

// With creates a child logger with additional fields
func With(fields ...any) *slog.Logger {
	return slog.Default().With(fields...)
}
```

**Usage**
```go
import (
	"log/slog"
	"myproject/internal/logger"
)

func ProcessData(ctx context.Context, data map[string]interface{}) error {
	log := logger.FromContext(ctx).With(
		"function", "ProcessData",
		"data_size", len(data),
	)

	log.Info("processing data")

	if err := validate(data); err != nil {
		log.Error("validation failed",
			"error", err,
			"data_keys", getKeys(data),
		)
		return err
	}

	log.Info("processing complete")
	return nil
}
```

---

## Dependencies

### go.mod

```go
module github.com/username/my-project

go 1.21

require (
	github.com/google/uuid v1.5.0
	github.com/jmoiron/sqlx v1.3.5
	github.com/kelseyhightower/envconfig v1.4.0
	github.com/lib/pq v1.10.9
	github.com/stretchr/testify v1.8.4
)

require (
	github.com/davecgh/go-spew v1.1.1 // indirect
	github.com/pmezard/go-difflib v1.0.0 // indirect
	gopkg.in/yaml.v3 v3.0.1 // indirect
)
```

### Commands

```bash
# Initialize module
go mod init github.com/username/my-project

# Add dependency
go get github.com/google/uuid

# Update dependencies
go get -u ./...

# Tidy dependencies
go mod tidy

# Download dependencies
go mod download

# Verify dependencies
go mod verify

# Build
go build -o bin/app ./cmd/api

# Run
go run ./cmd/api

# Install tools
go install github.com/golangci/golangci-lint/cmd/golangci-lint@latest

# Lint
golangci-lint run

# Format
go fmt ./...
```

---

## Complete Example

**cmd/api/main.go**
```go
package main

import (
	"context"
	"log"
	"log/slog"

	"myproject/internal/config"
	"myproject/internal/logger"
	"myproject/internal/repository"
	"myproject/internal/services"
)

func main() {
	// Load configuration
	cfg, err := config.Load()
	if err != nil {
		log.Fatalf("failed to load config: %v", err)
	}

	// Setup logging
	logger.Setup(cfg)
	slog.Info("starting application",
		"environment", cfg.Environment,
		"port", cfg.Port,
	)

	// Connect to database
	db, err := repository.NewDB(cfg.DatabaseURL)
	if err != nil {
		slog.Error("failed to connect to database", "error", err)
		return
	}
	defer db.Close()

	// Initialize repositories
	contractRepo := repository.NewContractRepository(db)

	// Process contract
	ctx := context.Background()
	content := "Sample contract content"

	result, err := services.ProcessContract(ctx, content)
	if err != nil {
		slog.Error("processing failed", "error", err)
		return
	}

	// Save to database
	contract, err := contractRepo.Create(ctx, content)
	if err != nil {
		slog.Error("failed to save contract", "error", err)
		return
	}

	slog.Info("contract processed successfully",
		"contract_id", contract.ID,
		"sections", len(result.Sections),
	)
}
```

---

## Quick Reference

### Common Patterns

```go
// Configuration
cfg := config.Get()

// Context
ctx := context.Background()
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()

// Logging
slog.Info("message", "key", "value")
log := logger.FromContext(ctx)

// Error handling
if err != nil {
	return fmt.Errorf("operation failed: %w", err)
}

// Database query
var result Result
err := db.GetContext(ctx, &result, query, args...)

// Struct validation
if err := data.Validate(); err != nil {
	return err
}
```

### Best Practices Checklist

- [ ] Use interfaces for contracts
- [ ] Accept interfaces, return structs
- [ ] Use context.Context for cancellation
- [ ] Wrap errors with context
- [ ] Use struct tags for JSON/DB mapping
- [ ] Connection pooling for database
- [ ] Table-driven tests
- [ ] Structured logging with slog
- [ ] Custom error types
- [ ] Validate configuration at startup

---

## Related Resources

- [Effective Go](https://go.dev/doc/effective_go)
- [Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)
- [Standard Go Project Layout](https://github.com/golang-standards/project-layout)
- [sqlx Documentation](http://jmoiron.github.io/sqlx/)
