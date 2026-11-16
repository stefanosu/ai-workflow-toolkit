# Rust Implementation Guide

Production-ready patterns for Rust 1.70+ projects following Rust idioms and best practices.

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
│   │   └── mod.rs              # Configuration
│   ├── models/
│   │   ├── mod.rs
│   │   ├── domain.rs           # Domain models
│   │   └── database.rs         # Database models
│   ├── services/               # Business logic
│   │   ├── mod.rs
│   │   └── processor.rs
│   ├── repository/             # Data access
│   │   ├── mod.rs
│   │   └── contract.rs
│   ├── api/                    # API layer
│   │   ├── mod.rs
│   │   └── handlers.rs
│   ├── error.rs                # Error types
│   ├── lib.rs                  # Library root
│   └── main.rs                 # Binary entry point
├── tests/
│   ├── integration.rs
│   └── common/
│       └── mod.rs
├── docs/
│   └── decisions/
├── Cargo.toml
├── Cargo.lock
├── .env.example
├── .gitignore
└── README.md
```

### Key Principles
- Use modules to organize code
- Separate library (lib.rs) from binary (main.rs)
- Keep modules focused and cohesive
- Use standard Rust project structure

---

## Configuration Management

### Using config and serde

**src/config/mod.rs**
```rust
use config::{Config as ConfigLoader, ConfigError, Environment, File};
use serde::Deserialize;
use std::env;

#[derive(Debug, Deserialize, Clone)]
pub struct Config {
    // Database
    pub database_url: String,

    // API
    pub api_key: String,
    #[serde(default = "default_port")]
    pub port: u16,

    // Application
    #[serde(default = "default_environment")]
    pub environment: String,

    #[serde(default = "default_log_level")]
    pub log_level: String,

    // Features
    #[serde(default = "default_enable_caching")]
    pub enable_caching: bool,

    #[serde(default = "default_max_workers")]
    pub max_workers: usize,
}

fn default_port() -> u16 {
    8080
}

fn default_environment() -> String {
    "development".to_string()
}

fn default_log_level() -> String {
    "info".to_string()
}

fn default_enable_caching() -> bool {
    true
}

fn default_max_workers() -> usize {
    10
}

impl Config {
    pub fn load() -> Result<Self, ConfigError> {
        let environment = env::var("ENVIRONMENT").unwrap_or_else(|_| "development".to_string());

        let config = ConfigLoader::builder()
            // Load from .env file (optional)
            .add_source(File::with_name(".env").required(false))
            // Load from environment variables
            .add_source(Environment::default())
            .build()?;

        let cfg: Config = config.try_deserialize()?;
        cfg.validate()?;

        Ok(cfg)
    }

    pub fn validate(&self) -> Result<(), ConfigError> {
        // Validate environment
        if !["development", "staging", "production"].contains(&self.environment.as_str()) {
            return Err(ConfigError::Message(format!(
                "Invalid environment: {}",
                self.environment
            )));
        }

        // Validate log level
        if !["debug", "info", "warn", "error"].contains(&self.log_level.as_str()) {
            return Err(ConfigError::Message(format!(
                "Invalid log level: {}",
                self.log_level
            )));
        }

        Ok(())
    }

    pub fn is_development(&self) -> bool {
        self.environment == "development"
    }

    pub fn is_production(&self) -> bool {
        self.environment == "production"
    }
}

// Global config instance using once_cell
use once_cell::sync::OnceCell;

static CONFIG: OnceCell<Config> = OnceCell::new();

pub fn init() -> Result<(), ConfigError> {
    let config = Config::load()?;
    CONFIG
        .set(config)
        .map_err(|_| ConfigError::Message("Config already initialized".to_string()))?;
    Ok(())
}

pub fn get() -> &'static Config {
    CONFIG.get().expect("Config not initialized - call init() first")
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
- Use serde for deserialization
- Provide sensible defaults
- Validate at load time
- Use OnceCell for global config
- Fail fast on invalid config

---

## Type Safety

### Struct Definitions

```rust
use serde::{Deserialize, Serialize};
use chrono::{DateTime, Utc};

#[derive(Debug, Clone, Serialize, Deserialize)]
pub struct User {
    pub id: String,
    pub email: String,
    pub name: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    pub age: Option<u8>,
    #[serde(default)]
    pub tags: Vec<String>,
    pub created_at: DateTime<Utc>,
}

impl User {
    pub fn new(id: String, email: String, name: String) -> Self {
        Self {
            id,
            email,
            name,
            age: None,
            tags: Vec::new(),
            created_at: Utc::now(),
        }
    }

    pub fn validate(&self) -> Result<(), ValidationError> {
        if self.id.is_empty() {
            return Err(ValidationError::new("id cannot be empty"));
        }

        if !self.email.contains('@') {
            return Err(ValidationError::new("invalid email address"));
        }

        if self.name.is_empty() {
            return Err(ValidationError::new("name cannot be empty"));
        }

        if let Some(age) = self.age {
            if age > 150 {
                return Err(ValidationError::new("age must be <= 150"));
            }
        }

        Ok(())
    }
}

#[derive(Debug, Deserialize)]
pub struct CreateUserInput {
    pub email: String,
    pub name: String,
    pub age: Option<u8>,
    #[serde(default)]
    pub tags: Vec<String>,
}

impl CreateUserInput {
    pub fn into_user(self, id: String) -> User {
        User {
            id,
            email: self.email,
            name: self.name,
            age: self.age,
            tags: self.tags,
            created_at: Utc::now(),
        }
    }
}
```

### Using validator Crate

```rust
use validator::{Validate, ValidationError as ValidatorError};

#[derive(Debug, Deserialize, Validate)]
pub struct CreateUserRequest {
    #[validate(email)]
    pub email: String,

    #[validate(length(min = 1, max = 100))]
    pub name: String,

    #[validate(range(min = 0, max = 150))]
    pub age: Option<u8>,

    #[validate(length(min = 1))]
    pub tags: Vec<String>,
}

impl CreateUserRequest {
    pub fn validate_and_convert(self) -> Result<CreateUserInput, ValidatorError> {
        self.validate()?;
        Ok(CreateUserInput {
            email: self.email,
            name: self.name,
            age: self.age,
            tags: self.tags,
        })
    }
}
```

### Traits and Contracts

```rust
use async_trait::async_trait;
use std::error::Error;

#[async_trait]
pub trait Repository<T>: Send + Sync {
    async fn create(&self, data: T) -> Result<T, Box<dyn Error>>;
    async fn find_by_id(&self, id: &str) -> Result<Option<T>, Box<dyn Error>>;
    async fn update(&self, id: &str, data: T) -> Result<T, Box<dyn Error>>;
    async fn delete(&self, id: &str) -> Result<(), Box<dyn Error>>;
}

#[async_trait]
pub trait Processor: Send + Sync {
    async fn process(&self, input: String) -> Result<ProcessResult, ProcessError>;
    fn validate(&self, input: &str) -> Result<(), ValidationError>;
}
```

---

## Database Patterns

### Using sqlx

**src/repository/database.rs**
```rust
use sqlx::{postgres::PgPoolOptions, PgPool, Error as SqlxError};
use std::time::Duration;

pub struct Database {
    pool: PgPool,
}

impl Database {
    pub async fn new(database_url: &str) -> Result<Self, SqlxError> {
        let pool = PgPoolOptions::new()
            .max_connections(25)
            .min_connections(5)
            .acquire_timeout(Duration::from_secs(5))
            .idle_timeout(Duration::from_secs(300))
            .connect(database_url)
            .await?;

        Ok(Self { pool })
    }

    pub fn pool(&self) -> &PgPool {
        &self.pool
    }

    pub async fn close(self) {
        self.pool.close().await;
    }
}
```

**src/repository/contract.rs**
```rust
use sqlx::{PgPool, FromRow};
use chrono::{DateTime, Utc};
use uuid::Uuid;

#[derive(Debug, Clone, FromRow)]
pub struct Contract {
    pub id: String,
    pub content: String,
    pub status: String,
    pub created_at: DateTime<Utc>,
}

pub struct ContractRepository {
    pool: PgPool,
}

impl ContractRepository {
    pub fn new(pool: PgPool) -> Self {
        Self { pool }
    }

    pub async fn create(&self, content: String) -> Result<Contract, sqlx::Error> {
        let id = Uuid::new_v4().to_string();
        let created_at = Utc::now();

        let contract = sqlx::query_as::<_, Contract>(
            r#"
            INSERT INTO contracts (id, content, status, created_at)
            VALUES ($1, $2, $3, $4)
            RETURNING id, content, status, created_at
            "#,
        )
        .bind(&id)
        .bind(&content)
        .bind("pending")
        .bind(created_at)
        .fetch_one(&self.pool)
        .await?;

        Ok(contract)
    }

    pub async fn find_by_id(&self, id: &str) -> Result<Option<Contract>, sqlx::Error> {
        let contract = sqlx::query_as::<_, Contract>(
            "SELECT id, content, status, created_at FROM contracts WHERE id = $1",
        )
        .bind(id)
        .fetch_optional(&self.pool)
        .await?;

        Ok(contract)
    }

    pub async fn find_by_status(&self, status: &str) -> Result<Vec<Contract>, sqlx::Error> {
        let contracts = sqlx::query_as::<_, Contract>(
            r#"
            SELECT id, content, status, created_at
            FROM contracts
            WHERE status = $1
            ORDER BY created_at DESC
            "#,
        )
        .bind(status)
        .fetch_all(&self.pool)
        .await?;

        Ok(contracts)
    }

    pub async fn update_status(&self, id: &str, status: &str) -> Result<(), sqlx::Error> {
        let result = sqlx::query("UPDATE contracts SET status = $1 WHERE id = $2")
            .bind(status)
            .bind(id)
            .execute(&self.pool)
            .await?;

        if result.rows_affected() == 0 {
            return Err(sqlx::Error::RowNotFound);
        }

        Ok(())
    }
}
```

### pgvector Integration

```rust
use pgvector::Vector;

#[derive(Debug, Clone, FromRow)]
pub struct Document {
    pub id: String,
    pub content: String,
    pub embedding: Vector,
}

pub struct DocumentRepository {
    pool: PgPool,
}

impl DocumentRepository {
    pub async fn find_similar(
        &self,
        embedding: &[f32],
        limit: i64,
    ) -> Result<Vec<Document>, sqlx::Error> {
        let vec = Vector::from(embedding.to_vec());

        let documents = sqlx::query_as::<_, Document>(
            r#"
            SELECT id, content, embedding
            FROM documents
            ORDER BY embedding <=> $1
            LIMIT $2
            "#,
        )
        .bind(vec)
        .bind(limit)
        .fetch_all(&self.pool)
        .await?;

        Ok(documents)
    }

    pub async fn create(&self, content: String, embedding: Vec<f32>) -> Result<Document, sqlx::Error> {
        let id = Uuid::new_v4().to_string();
        let vec = Vector::from(embedding);

        let document = sqlx::query_as::<_, Document>(
            r#"
            INSERT INTO documents (id, content, embedding)
            VALUES ($1, $2, $3)
            RETURNING id, content, embedding
            "#,
        )
        .bind(&id)
        .bind(&content)
        .bind(vec)
        .fetch_one(&self.pool)
        .await?;

        Ok(document)
    }
}
```

---

## Testing

### Test Structure

**src/services/processor.rs**
```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[tokio::test]
    async fn test_process_contract_success() {
        let content = "Contract content here".to_string();
        let result = process_contract(content).await;

        assert!(result.is_ok());
        let process_result = result.unwrap();
        assert_eq!(process_result.status, "success");
        assert!(!process_result.sections.is_empty());
    }

    #[tokio::test]
    async fn test_process_contract_empty_content() {
        let content = "".to_string();
        let result = process_contract(content).await;

        assert!(result.is_err());
    }

    #[tokio::test]
    async fn test_process_contract_unicode() {
        let content = "Contract with émojis 🎉".to_string();
        let result = process_contract(content).await;

        assert!(result.is_ok());
    }

    mod edge_cases {
        use super::*;

        #[tokio::test]
        async fn test_very_long_content() {
            let content = "a".repeat(1_000_000);
            let result = process_contract(content).await;

            assert!(result.is_ok());
        }

        #[tokio::test]
        async fn test_special_characters() {
            let content = "Special chars: @#$%^&*()".to_string();
            let result = process_contract(content).await;

            assert!(result.is_ok());
        }
    }
}
```

### Parameterized Tests with rstest

```rust
use rstest::rstest;

#[rstest]
#[case("hello world", 2)]
#[case("", 0)]
#[case("one", 1)]
#[case("multiple words here", 3)]
fn test_count_words(#[case] input: &str, #[case] expected: usize) {
    assert_eq!(count_words(input), expected);
}
```

### Test Fixtures

**tests/common/mod.rs**
```rust
use sqlx::{PgPool, postgres::PgPoolOptions};
use std::env;

pub async fn setup_test_db() -> PgPool {
    let database_url = env::var("TEST_DATABASE_URL")
        .unwrap_or_else(|_| "postgresql://localhost:5432/test_db".to_string());

    PgPoolOptions::new()
        .max_connections(1)
        .connect(&database_url)
        .await
        .expect("Failed to connect to test database")
}

pub async fn cleanup_db(pool: &PgPool) {
    sqlx::query("DELETE FROM contracts")
        .execute(pool)
        .await
        .expect("Failed to cleanup database");
}

pub fn load_fixture(filename: &str) -> String {
    std::fs::read_to_string(format!("tests/fixtures/{}", filename))
        .expect("Failed to load fixture")
}
```

**tests/integration.rs**
```rust
mod common;

use common::*;

#[tokio::test]
async fn test_create_contract() {
    let pool = setup_test_db().await;

    let repo = ContractRepository::new(pool.clone());
    let content = "Test contract".to_string();

    let result = repo.create(content.clone()).await;
    assert!(result.is_ok());

    let contract = result.unwrap();
    assert_eq!(contract.content, content);

    cleanup_db(&pool).await;
}
```

### Running Tests

```bash
# All tests
cargo test

# With output
cargo test -- --nocapture

# Specific test
cargo test test_process_contract

# Integration tests only
cargo test --test integration

# Doc tests
cargo test --doc

# With coverage (requires cargo-tarpaulin)
cargo tarpaulin --out Html
```

---

## Error Handling

### Custom Error Types

**src/error.rs**
```rust
use thiserror::Error;

#[derive(Error, Debug)]
pub enum AppError {
    #[error("Validation error: {0}")]
    Validation(String),

    #[error("Not found: {resource} with id {id}")]
    NotFound { resource: String, id: String },

    #[error("Database error: {0}")]
    Database(#[from] sqlx::Error),

    #[error("Configuration error: {0}")]
    Config(#[from] config::ConfigError),

    #[error("Internal error: {0}")]
    Internal(String),
}

pub type Result<T> = std::result::Result<T, AppError>;

impl AppError {
    pub fn validation(msg: impl Into<String>) -> Self {
        Self::Validation(msg.into())
    }

    pub fn not_found(resource: impl Into<String>, id: impl Into<String>) -> Self {
        Self::NotFound {
            resource: resource.into(),
            id: id.into(),
        }
    }

    pub fn internal(msg: impl Into<String>) -> Self {
        Self::Internal(msg.into())
    }
}
```

### Error Handling Pattern

```rust
use crate::error::{AppError, Result};
use tracing::{error, warn, info};

#[derive(Debug)]
pub struct ProcessResult {
    pub status: String,
    pub sections: Vec<String>,
    pub metadata: std::collections::HashMap<String, String>,
}

pub async fn process_contract(content: String) -> Result<ProcessResult> {
    // Validation
    if content.trim().is_empty() {
        warn!("Validation failed: empty content");
        return Err(AppError::validation("content cannot be empty"));
    }

    // Processing
    let sections = split_into_sections(&content)
        .map_err(|e| {
            error!("Failed to split sections: {}", e);
            AppError::internal(format!("Failed to split sections: {}", e))
        })?;

    if sections.is_empty() {
        return Err(AppError::validation("no sections found"));
    }

    info!("Processing complete, {} sections found", sections.len());

    Ok(ProcessResult {
        status: "success".to_string(),
        sections,
        metadata: std::collections::HashMap::new(),
    })
}
```

### Result Type Usage

```rust
pub async fn fetch_user(id: &str) -> Result<User> {
    let user = repository
        .find_by_id(id)
        .await?
        .ok_or_else(|| AppError::not_found("User", id))?;

    Ok(user)
}

// Usage with match
match fetch_user("123").await {
    Ok(user) => println!("Found user: {}", user.name),
    Err(AppError::NotFound { resource, id }) => {
        println!("{} {} not found", resource, id)
    }
    Err(e) => eprintln!("Error: {}", e),
}

// Usage with ?
let user = fetch_user("123").await?;
```

---

## Logging

### Using tracing

**src/main.rs**
```rust
use tracing::{info, error, warn, debug, Level};
use tracing_subscriber::{fmt, EnvFilter};

fn setup_logging(log_level: &str) {
    let filter = EnvFilter::try_from_default_env()
        .unwrap_or_else(|_| EnvFilter::new(log_level));

    fmt()
        .with_env_filter(filter)
        .with_target(false)
        .with_thread_ids(true)
        .with_line_number(true)
        .json()
        .init();
}

#[tokio::main]
async fn main() -> Result<()> {
    // Initialize config
    crate::config::init()?;
    let config = crate::config::get();

    // Setup logging
    setup_logging(&config.log_level);

    info!(
        environment = %config.environment,
        port = config.port,
        "Starting application"
    );

    // Application logic...

    Ok(())
}
```

**Usage in modules**
```rust
use tracing::{info, error, warn, instrument};

#[instrument(skip(data))]
pub async fn process_data(data: Vec<u8>) -> Result<ProcessResult> {
    info!(data_size = data.len(), "Processing data");

    match validate_data(&data) {
        Ok(_) => info!("Validation successful"),
        Err(e) => {
            error!(error = %e, "Validation failed");
            return Err(e);
        }
    }

    let result = do_processing(data).await?;

    info!(
        result_count = result.items.len(),
        "Processing complete"
    );

    Ok(result)
}
```

---

## Dependencies

### Cargo.toml

```toml
[package]
name = "my-project"
version = "0.1.0"
edition = "2021"
rust-version = "1.70"

[dependencies]
# Async runtime
tokio = { version = "1.35", features = ["full"] }

# Database
sqlx = { version = "0.7", features = ["runtime-tokio-native-tls", "postgres", "uuid", "chrono"] }
pgvector = "0.3"

# Serialization
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"

# Configuration
config = "0.13"
once_cell = "1.19"

# Error handling
thiserror = "1.0"
anyhow = "1.0"

# Logging
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["json", "env-filter"] }

# Validation
validator = { version = "0.16", features = ["derive"] }

# UUID
uuid = { version = "1.6", features = ["v4", "serde"] }

# Time
chrono = { version = "0.4", features = ["serde"] }

[dev-dependencies]
rstest = "0.18"
tokio-test = "0.4"

[profile.release]
opt-level = 3
lto = true
codegen-units = 1
```

### Commands

```bash
# Create new project
cargo new my-project

# Add dependency
cargo add tokio --features full

# Build
cargo build

# Build release
cargo build --release

# Run
cargo run

# Run release
cargo run --release

# Test
cargo test

# Check (faster than build)
cargo check

# Format
cargo fmt

# Lint
cargo clippy

# Update dependencies
cargo update

# Generate docs
cargo doc --open
```

---

## Complete Example

**src/main.rs**
```rust
mod config;
mod error;
mod models;
mod repository;
mod services;

use crate::config::Config;
use crate::error::Result;
use crate::repository::{Database, ContractRepository};
use crate::services::process_contract;
use tracing::{info, error};
use tracing_subscriber::{fmt, EnvFilter};

fn setup_logging(config: &Config) {
    let filter = EnvFilter::new(&config.log_level);

    if config.is_development() {
        fmt().with_env_filter(filter).init();
    } else {
        fmt().with_env_filter(filter).json().init();
    }
}

#[tokio::main]
async fn main() -> Result<()> {
    // Load configuration
    config::init()?;
    let config = config::get();

    // Setup logging
    setup_logging(config);

    info!(
        environment = %config.environment,
        port = config.port,
        "Starting application"
    );

    // Connect to database
    let db = Database::new(&config.database_url).await?;
    let repo = ContractRepository::new(db.pool().clone());

    // Process contract
    let content = "Sample contract content".to_string();

    match process_contract(content.clone()).await {
        Ok(result) => {
            // Save to database
            match repo.create(content).await {
                Ok(contract) => {
                    info!(
                        contract_id = %contract.id,
                        sections = result.sections.len(),
                        "Contract processed successfully"
                    );
                }
                Err(e) => {
                    error!(error = %e, "Failed to save contract");
                    return Err(e.into());
                }
            }
        }
        Err(e) => {
            error!(error = %e, "Processing failed");
            return Err(e);
        }
    }

    db.close().await;
    Ok(())
}
```

---

## Quick Reference

### Common Patterns

```rust
// Configuration
let config = config::get();

// Error handling
let result = operation().await?;

// Logging
info!(key = value, "message");

// Database query
let rows = sqlx::query_as::<_, Model>("SELECT * FROM table")
    .fetch_all(&pool)
    .await?;

// Match on Result
match operation().await {
    Ok(value) => { /* success */ },
    Err(e) => { /* error */ },
}

// Option handling
let value = option.ok_or_else(|| AppError::not_found("Resource", "id"))?;
```

### Best Practices Checklist

- [ ] Use `Result<T>` for fallible operations
- [ ] Use `thiserror` for custom errors
- [ ] Use `tracing` for structured logging
- [ ] Use `async/await` with Tokio
- [ ] Use `sqlx` for type-safe database queries
- [ ] Use `serde` for serialization
- [ ] Validate configuration at startup
- [ ] Use traits for contracts
- [ ] Write comprehensive tests
- [ ] Use `#[instrument]` for tracing

---

## Related Resources

- [The Rust Programming Language Book](https://doc.rust-lang.org/book/)
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)
- [Tokio Documentation](https://tokio.rs/)
- [sqlx Documentation](https://github.com/launchbadge/sqlx)
- [tracing Documentation](https://docs.rs/tracing/)
