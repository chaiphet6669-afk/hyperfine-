# Hyperfine Architecture & Workflow Guide

## 📊 System Overview Flowchart

```
┌─────────────────────────────────────────────────────────────────┐
│                    HYPERFINE BENCHMARKING FLOW                  │
└─────────────────────────────────────────────────────────────────┘

START
  │
  ├─→ [1. CLI INPUT PARSING]
  │   ├─ Parse command arguments
  │   ├─ Load configuration (warmup, runs, prepare commands)
  │   └─ Validate input parameters
  │
  ├─→ [2. SHELL CALIBRATION]
  │   ├─ Measure shell startup overhead
  │   └─ Store baseline for correction
  │
  ├─→ [3. WARMUP PHASE] (if enabled)
  │   ├─ Execute setup commands
  │   ├─ Run command N times (warmup runs)
  │   └─ Discard results
  │
  ├─→ [4. BENCHMARKING LOOP]
  │   ├─ For each run:
  │   │   ├─ Execute prepare command (if set)
  │   │   ├─ Execute target command
  │   │   ├─ Measure execution time
  │   │   ├─ Subtract shell overhead
  │   │   └─ Store result
  │   │
  │   └─ Continue until min runs OR min time reached
  │
  ├─→ [5. STATISTICAL ANALYSIS]
  │   ├─ Calculate: Mean, Median, Std Dev, Min, Max
  │   ├─ Detect outliers
  │   └─ Generate comparative statistics
  │
  ├─→ [6. OUTPUT FORMATTING]
  │   ├─ Terminal display
  │   ├─ Export formats:
  │   │   ├─ JSON
  │   │   ├─ CSV
  │   │   ├─ Markdown
  │   │   └─ AsciiDoc
  │
  └─→ END (Success)
```

---

## 🎯 Core Methodology Flow

### Trigger → Action → Process → Mission

```
TRIGGER PHASE
├─ User invokes: hyperfine [OPTIONS] [COMMANDS...]
├─ Configuration loaded from CLI args
└─ Target benchmark commands identified

        ⬇️

ACTION PHASE
├─ Initialize execution environment
├─ Setup signal handlers
├─ Create timing structures
└─ Prepare measurement infrastructure

        ⬇️

PROCESS PHASE
├─ CALIBRATION STAGE
│   └─ Measure shell overhead
│
├─ EXECUTION STAGE
│   ├─ Warmup runs (optional)
│   ├─ Main benchmark runs
│   └─ Prepare commands (each iteration)
│
└─ MEASUREMENT STAGE
    ├─ High-precision timing
    ├─ Statistical collection
    └─ Outlier detection

        ⬇️

MISSION (GOAL ACHIEVEMENT)
├─ Accurate performance metrics
├─ Comparative analysis results
├─ Exportable reports
└─ Data-driven insights for optimization
```

---

## 🌳 Execution Tree: Decision Hierarchy

```
HYPERFINE EXECUTION
│
├─ CONFIGURATION VALIDATION
│   ├─ Valid? ✓ → Continue
│   └─ Invalid? ✗ → Error & Exit
│
├─ WARMUP DECISION
│   ├─ --warmup flag set?
│   │   ├─ YES → Execute N warmup iterations
│   │   └─ NO → Skip to benchmarking
│   │
│   └─ Setup commands?
│       ├─ YES → Run before warmup
│       └─ NO → Continue
│
├─ BENCHMARKING LOOP
│   ├─ Runs completed?
│   │   ├─ NO → Next iteration
│   │   └─ YES → Analyze results
│   │
│   ├─ Time threshold met?
│   │   ├─ NO → Continue iterations
│   │   └─ YES → Finalize
│   │
│   └─ Prepare command?
│       ├─ YES → Execute before each run
│       └─ NO → Direct command execution
│
├─ STATISTICAL ANALYSIS
│   ├─ Calculate metrics
│   ├─ Detect outliers
│   └─ Generate comparisons
│
└─ OUTPUT GENERATION
    ├─ Format selection
    ├─ Export options
    └─ Display results
```

---

## 📋 Detailed Stage Flows

### STAGE 1: CLI Input Parsing
```
User Input
    ⬇️
Argument Parser (clap)
    ├─ Extract commands to benchmark
    ├─ Parse options:
    │   ├─ --runs N (number of executions)
    │   ├─ --warmup N (pre-execution runs)
    │   ├─ --prepare "cmd" (setup before each run)
    │   ├─ --cleanup "cmd" (cleanup after each run)
    │   ├─ --shell "bash|zsh|..." (choose shell)
    │   ├─ --export-json "file.json"
    │   └─ --export-markdown "file.md"
    │
    └─→ Config Struct Created
         └─ Ready for execution
```

### STAGE 2: Shell Calibration
```
Calibration Phase
    ⬇️
Measure baseline shell startup:
  1. Run empty shell N times
  2. Record execution times
  3. Calculate average overhead
  4. Store for later subtraction
    ⬇️
Correction Factor = Overhead
    └─ Applied to all subsequent measurements
```

### STAGE 3: Warmup Execution
```
IF --warmup flag present:
    ⬇️
    For i = 1 to N:
    ├─ Execute prepare command
    ├─ Run target command
    ├─ Measure time
    ├─ Discard result
    └─ Continue
    ⬇️
    Warmup Complete
    └─ Caches warmed, system ready
```

### STAGE 4: Main Benchmarking Loop
```
Initialize:
├─ min_runs = 10 (default)
├─ min_time = 3.0 seconds (default)
├─ runs_completed = 0
└─ total_time = 0.0

    ⬇️

WHILE (runs_completed < min_runs OR total_time < min_time):
    ├─ Show progress bar
    ├─ Execute prepare command (if set)
    ├─ Time target command execution
    ├─ Subtract shell overhead
    ├─ Store result
    ├─ Update totals
    ├─ runs_completed++
    └─ Continue

    ⬇️

Benchmark Complete
└─ Collect all timing samples
```

### STAGE 5: Statistical Analysis
```
Raw Timing Data
    ⬇️
Calculate Metrics:
    ├─ Min: Minimum execution time
    ├─ Max: Maximum execution time
    ├─ Mean: Average execution time
    ├─ Median: Middle value
    ├─ Std Dev: Standard deviation
    └─ IQR: Interquartile range
    
    ⬇️

Outlier Detection:
    ├─ Identify extreme values
    ├─ Potential interference markers
    └─ Flag for user awareness

    ⬇️

Comparative Analysis:
    ├─ Baseline selection
    ├─ Relative performance ratios
    └─ Confidence intervals

    ⬇️

Statistics Ready for Output
```

### STAGE 6: Output Generation
```
Analysis Results
    ⬇️
Format Selection:
    ├─ TERMINAL OUTPUT
    │   └─ Formatted table with colors
    │
    ├─ JSON EXPORT
    │   ├─ Structured data
    │   ├─ Machine readable
    │   └─ For scripting/analysis
    │
    ├─ CSV EXPORT
    │   ├─ Spreadsheet compatible
    │   └─ For data tools
    │
    ├─ MARKDOWN EXPORT
    │   ├─ Documentation ready
    │   └─ GitHub compatible
    │
    └─ ASCIIDOC EXPORT
        ├─ Publishing ready
        └─ Professional docs
    
    ⬇️

Display/Export Results
```

---

## 🔄 Dialog Flow: User Interaction Model

```
START: User Interaction Flow
    │
    ├─ USER PROVIDES INPUT
    │  └─ $ hyperfine [options] [commands]
    │     │
    │     ├─ Trigger Dialog 1: Parse & Validate
    │     │  ├─ Q: Valid syntax?
    │     │  ├─ A: Validate against schema
    │     │  └─ Result: Config loaded or error
    │     │
    │     ├─ Trigger Dialog 2: Calibration
    │     │  ├─ Q: Measure shell overhead?
    │     │  ├─ A: Yes, run N calibration samples
    │     │  └─ Result: Baseline computed
    │     │
    │     ├─ Trigger Dialog 3: Warmup Decision
    │     │  ├─ Q: Warmup enabled?
    │     │  ├─ A: If --warmup flag, execute
    │     │  └─ Result: Caches prepared
    │     │
    │     ├─ Trigger Dialog 4: Main Benchmark
    │     │  ├─ Q: Run iterations until done?
    │     │  ├─ A: Yes, loop with progress
    │     │  └─ Result: Timing data collected
    │     │
    │     ├─ Trigger Dialog 5: Analysis
    │     │  ├─ Q: Compute statistics?
    │     │  ├─ A: Calculate all metrics
    │     │  └─ Result: Statistics ready
    │     │
    │     └─ Trigger Dialog 6: Output
    │        ├─ Q: How to display results?
    │        ├─ A: Terminal + exports
    │        └─ Result: Results shown/saved
    │
    └─ END: User reviews results
```

---

## 💾 Database Storage Extension (SQLx Integration)

### Overview: Persistent Benchmark Storage

```
HYPERFINE WITH DATABASE PERSISTENCE
│
├─ CONFIGURATION
│   ├─ Database URL: DATABASE_URL environment variable
│   ├─ Supported:
│   │   ├─ PostgreSQL: postgresql://user:pass@localhost/hyperfine_db
│   │   ├─ MySQL: mysql://user:pass@localhost/hyperfine_db
│   │   └─ SQLite: sqlite:hyperfine.db
│   │
│   └─ Connection Pool
│       ├─ Create at startup
│       └─ Reuse throughout execution
│
├─ BENCHMARK RUNS TABLE
│   ├─ id: UUID (primary key)
│   ├─ timestamp: DateTime (when run occurred)
│   ├─ benchmark_name: String
│   ├─ command: String (executed command)
│   ├─ shell: String (shell used)
│   ├─ warmup_runs: i32
│   ├─ total_runs: i32
│   ├─ min_time: f64 (seconds)
│   │
│   └─ MEASUREMENTS (child table)
│       ├─ run_id: UUID (foreign key)
│       ├─ sequence: i32 (iteration number)
│       ├─ execution_time_ms: f64
│       ├─ shell_overhead_ms: f64
│       └─ adjusted_time_ms: f64
│
├─ STATISTICS TABLE
│   ├─ run_id: UUID (foreign key)
│   ├─ mean_ms: f64
│   ├─ median_ms: f64
│   ├─ stddev_ms: f64
│   ├─ min_ms: f64
│   ├─ max_ms: f64
│   ├─ outlier_count: i32
│   │
│   └─ Indexes on: run_id, timestamp, benchmark_name
│
└─ ANALYSIS & QUERYING
    ├─ Historical trends
    ├─ Performance regressions
    ├─ Comparative reports
    └─ Time-series analysis
```

### Database Schema (SQL)

#### PostgreSQL / MySQL

```sql
-- Benchmark runs table
CREATE TABLE benchmark_runs (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    timestamp DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    benchmark_name VARCHAR(255) NOT NULL,
    command TEXT NOT NULL,
    shell VARCHAR(50) NOT NULL,
    warmup_runs INT NOT NULL DEFAULT 0,
    total_runs INT NOT NULL,
    min_time_seconds DECIMAL(10, 3) NOT NULL,
    notes TEXT,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_benchmark_name (benchmark_name),
    INDEX idx_timestamp (timestamp)
);

-- Individual run measurements
CREATE TABLE measurements (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    run_id UUID NOT NULL,
    sequence INT NOT NULL,
    execution_time_ms DECIMAL(10, 4) NOT NULL,
    shell_overhead_ms DECIMAL(10, 4) NOT NULL,
    adjusted_time_ms DECIMAL(10, 4) NOT NULL,
    is_outlier BOOLEAN DEFAULT FALSE,
    FOREIGN KEY (run_id) REFERENCES benchmark_runs(id) ON DELETE CASCADE,
    INDEX idx_run_id (run_id)
);

-- Aggregated statistics
CREATE TABLE statistics (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    run_id UUID NOT NULL UNIQUE,
    mean_ms DECIMAL(10, 4) NOT NULL,
    median_ms DECIMAL(10, 4) NOT NULL,
    stddev_ms DECIMAL(10, 4) NOT NULL,
    min_ms DECIMAL(10, 4) NOT NULL,
    max_ms DECIMAL(10, 4) NOT NULL,
    q1_ms DECIMAL(10, 4),
    q3_ms DECIMAL(10, 4),
    iqr_ms DECIMAL(10, 4),
    outlier_count INT DEFAULT 0,
    FOREIGN KEY (run_id) REFERENCES benchmark_runs(id) ON DELETE CASCADE,
    INDEX idx_run_id (run_id)
);

-- Comparative analysis (benchmark comparisons)
CREATE TABLE comparisons (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    baseline_run_id UUID NOT NULL,
    comparison_run_id UUID NOT NULL,
    relative_performance_percent DECIMAL(10, 2),
    difference_ms DECIMAL(10, 4),
    is_regression BOOLEAN,
    confidence_level VARCHAR(50),
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (baseline_run_id) REFERENCES benchmark_runs(id),
    FOREIGN KEY (comparison_run_id) REFERENCES benchmark_runs(id),
    INDEX idx_baseline (baseline_run_id),
    INDEX idx_comparison (comparison_run_id)
);
```

#### SQLite

```sql
-- Benchmark runs table
CREATE TABLE benchmark_runs (
    id TEXT PRIMARY KEY,
    timestamp DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    benchmark_name TEXT NOT NULL,
    command TEXT NOT NULL,
    shell TEXT NOT NULL,
    warmup_runs INTEGER NOT NULL DEFAULT 0,
    total_runs INTEGER NOT NULL,
    min_time_seconds REAL NOT NULL,
    notes TEXT,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_benchmark_name ON benchmark_runs(benchmark_name);
CREATE INDEX idx_timestamp ON benchmark_runs(timestamp);

-- Individual run measurements
CREATE TABLE measurements (
    id TEXT PRIMARY KEY,
    run_id TEXT NOT NULL,
    sequence INTEGER NOT NULL,
    execution_time_ms REAL NOT NULL,
    shell_overhead_ms REAL NOT NULL,
    adjusted_time_ms REAL NOT NULL,
    is_outlier INTEGER DEFAULT 0,
    FOREIGN KEY (run_id) REFERENCES benchmark_runs(id) ON DELETE CASCADE
);

CREATE INDEX idx_run_id ON measurements(run_id);

-- Aggregated statistics
CREATE TABLE statistics (
    id TEXT PRIMARY KEY,
    run_id TEXT NOT NULL UNIQUE,
    mean_ms REAL NOT NULL,
    median_ms REAL NOT NULL,
    stddev_ms REAL NOT NULL,
    min_ms REAL NOT NULL,
    max_ms REAL NOT NULL,
    q1_ms REAL,
    q3_ms REAL,
    iqr_ms REAL,
    outlier_count INTEGER DEFAULT 0,
    FOREIGN KEY (run_id) REFERENCES benchmark_runs(id) ON DELETE CASCADE
);

CREATE INDEX idx_stats_run ON statistics(run_id);

-- Comparative analysis
CREATE TABLE comparisons (
    id TEXT PRIMARY KEY,
    baseline_run_id TEXT NOT NULL,
    comparison_run_id TEXT NOT NULL,
    relative_performance_percent REAL,
    difference_ms REAL,
    is_regression INTEGER,
    confidence_level TEXT,
    created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (baseline_run_id) REFERENCES benchmark_runs(id),
    FOREIGN KEY (comparison_run_id) REFERENCES benchmark_runs(id)
);

CREATE INDEX idx_baseline ON comparisons(baseline_run_id);
CREATE INDEX idx_comparison ON comparisons(comparison_run_id);
```

---

## 🦀 Rust Code Integration with SQLx

### Step 1: Add Dependencies to `Cargo.toml`

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
sqlx = { version = "0.7", features = ["runtime-tokio-rustls", "postgres", "mysql", "sqlite", "uuid", "chrono"] }
uuid = { version = "1.0", features = ["v4", "serde"] }
chrono = { version = "0.4", features = ["serde"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
anyhow = "1.0"
```

### Step 2: Database Models

```rust
// src/models/benchmark.rs

use sqlx::FromRow;
use uuid::Uuid;
use chrono::{DateTime, Utc};

#[derive(Debug, Clone, FromRow, serde::Serialize, serde::Deserialize)]
pub struct BenchmarkRun {
    pub id: String, // UUID as string for SQLite compat
    pub timestamp: DateTime<Utc>,
    pub benchmark_name: String,
    pub command: String,
    pub shell: String,
    pub warmup_runs: i32,
    pub total_runs: i32,
    pub min_time_seconds: f64,
    pub notes: Option<String>,
    pub created_at: DateTime<Utc>,
}

#[derive(Debug, Clone, FromRow, serde::Serialize)]
pub struct Measurement {
    pub id: String,
    pub run_id: String,
    pub sequence: i32,
    pub execution_time_ms: f64,
    pub shell_overhead_ms: f64,
    pub adjusted_time_ms: f64,
    pub is_outlier: bool,
}

#[derive(Debug, Clone, FromRow, serde::Serialize)]
pub struct Statistics {
    pub id: String,
    pub run_id: String,
    pub mean_ms: f64,
    pub median_ms: f64,
    pub stddev_ms: f64,
    pub min_ms: f64,
    pub max_ms: f64,
    pub q1_ms: Option<f64>,
    pub q3_ms: Option<f64>,
    pub iqr_ms: Option<f64>,
    pub outlier_count: i32,
}

#[derive(Debug, Clone, FromRow, serde::Serialize)]
pub struct Comparison {
    pub id: String,
    pub baseline_run_id: String,
    pub comparison_run_id: String,
    pub relative_performance_percent: Option<f64>,
    pub difference_ms: Option<f64>,
    pub is_regression: Option<bool>,
    pub confidence_level: Option<String>,
}
```

### Step 3: Database Repository

```rust
// src/db/repository.rs

use sqlx::Pool;
use sqlx::sqlite::SqlitePool;
use uuid::Uuid;
use chrono::Utc;
use crate::models::{BenchmarkRun, Measurement, Statistics};
use anyhow::Result;

pub struct BenchmarkRepository {
    pool: SqlitePool,
}

impl BenchmarkRepository {
    pub async fn new(database_url: &str) -> Result<Self> {
        let pool = SqlitePool::connect(database_url).await?;
        
        // Run migrations
        sqlx::query(
            r#"CREATE TABLE IF NOT EXISTS benchmark_runs (
                id TEXT PRIMARY KEY,
                timestamp DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
                benchmark_name TEXT NOT NULL,
                command TEXT NOT NULL,
                shell TEXT NOT NULL,
                warmup_runs INTEGER NOT NULL DEFAULT 0,
                total_runs INTEGER NOT NULL,
                min_time_seconds REAL NOT NULL,
                notes TEXT,
                created_at DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP
            )"#,
        )
        .execute(&pool)
        .await?;

        sqlx::query(
            r#"CREATE TABLE IF NOT EXISTS measurements (
                id TEXT PRIMARY KEY,
                run_id TEXT NOT NULL,
                sequence INTEGER NOT NULL,
                execution_time_ms REAL NOT NULL,
                shell_overhead_ms REAL NOT NULL,
                adjusted_time_ms REAL NOT NULL,
                is_outlier INTEGER DEFAULT 0,
                FOREIGN KEY (run_id) REFERENCES benchmark_runs(id) ON DELETE CASCADE
            )"#,
        )
        .execute(&pool)
        .await?;

        sqlx::query(
            r#"CREATE TABLE IF NOT EXISTS statistics (
                id TEXT PRIMARY KEY,
                run_id TEXT NOT NULL UNIQUE,
                mean_ms REAL NOT NULL,
                median_ms REAL NOT NULL,
                stddev_ms REAL NOT NULL,
                min_ms REAL NOT NULL,
                max_ms REAL NOT NULL,
                q1_ms REAL,
                q3_ms REAL,
                iqr_ms REAL,
                outlier_count INTEGER DEFAULT 0,
                FOREIGN KEY (run_id) REFERENCES benchmark_runs(id) ON DELETE CASCADE
            )"#,
        )
        .execute(&pool)
        .await?;

        Ok(BenchmarkRepository { pool })
    }

    // Insert benchmark run
    pub async fn save_run(&self, run: &BenchmarkRun) -> Result<()> {
        sqlx::query(
            r#"INSERT INTO benchmark_runs 
               (id, timestamp, benchmark_name, command, shell, warmup_runs, total_runs, min_time_seconds, created_at)
               VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)"#,
        )
        .bind(&run.id)
        .bind(&run.timestamp)
        .bind(&run.benchmark_name)
        .bind(&run.command)
        .bind(&run.shell)
        .bind(run.warmup_runs)
        .bind(run.total_runs)
        .bind(run.min_time_seconds)
        .bind(&run.created_at)
        .execute(&self.pool)
        .await?;

        Ok(())
    }

    // Insert measurements
    pub async fn save_measurement(&self, measurement: &Measurement) -> Result<()> {
        sqlx::query(
            r#"INSERT INTO measurements 
               (id, run_id, sequence, execution_time_ms, shell_overhead_ms, adjusted_time_ms, is_outlier)
               VALUES (?, ?, ?, ?, ?, ?, ?)"#,
        )
        .bind(&measurement.id)
        .bind(&measurement.run_id)
        .bind(measurement.sequence)
        .bind(measurement.execution_time_ms)
        .bind(measurement.shell_overhead_ms)
        .bind(measurement.adjusted_time_ms)
        .bind(measurement.is_outlier as i32)
        .execute(&self.pool)
        .await?;

        Ok(())
    }

    // Save statistics
    pub async fn save_statistics(&self, stats: &Statistics) -> Result<()> {
        sqlx::query(
            r#"INSERT INTO statistics 
               (id, run_id, mean_ms, median_ms, stddev_ms, min_ms, max_ms, q1_ms, q3_ms, iqr_ms, outlier_count)
               VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?, ?, ?)"#,
        )
        .bind(&stats.id)
        .bind(&stats.run_id)
        .bind(stats.mean_ms)
        .bind(stats.median_ms)
        .bind(stats.stddev_ms)
        .bind(stats.min_ms)
        .bind(stats.max_ms)
        .bind(stats.q1_ms)
        .bind(stats.q3_ms)
        .bind(stats.iqr_ms)
        .bind(stats.outlier_count)
        .execute(&self.pool)
        .await?;

        Ok(())
    }

    // Query benchmarks by name
    pub async fn get_runs_by_name(&self, benchmark_name: &str) -> Result<Vec<BenchmarkRun>> {
        let runs = sqlx::query_as::<_, BenchmarkRun>(
            "SELECT * FROM benchmark_runs WHERE benchmark_name = ? ORDER BY timestamp DESC"
        )
        .bind(benchmark_name)
        .fetch_all(&self.pool)
        .await?;

        Ok(runs)
    }

    // Get latest run for a benchmark
    pub async fn get_latest_run(&self, benchmark_name: &str) -> Result<Option<BenchmarkRun>> {
        let run = sqlx::query_as::<_, BenchmarkRun>(
            "SELECT * FROM benchmark_runs WHERE benchmark_name = ? ORDER BY timestamp DESC LIMIT 1"
        )
        .bind(benchmark_name)
        .fetch_optional(&self.pool)
        .await?;

        Ok(run)
    }

    // Get measurements for a run
    pub async fn get_measurements(&self, run_id: &str) -> Result<Vec<Measurement>> {
        let measurements = sqlx::query_as::<_, Measurement>(
            "SELECT * FROM measurements WHERE run_id = ? ORDER BY sequence"
        )
        .bind(run_id)
        .fetch_all(&self.pool)
        .await?;

        Ok(measurements)
    }

    // Get statistics for a run
    pub async fn get_statistics(&self, run_id: &str) -> Result<Option<Statistics>> {
        let stats = sqlx::query_as::<_, Statistics>(
            "SELECT * FROM statistics WHERE run_id = ?"
        )
        .bind(run_id)
        .fetch_optional(&self.pool)
        .await?;

        Ok(stats)
    }
}
```

### Step 4: Integration with Main Benchmark Flow

```rust
// src/main.rs modifications

use sqlx::sqlite::SqlitePool;
use crate::db::repository::BenchmarkRepository;
use crate::models::{BenchmarkRun, Measurement, Statistics};
use uuid::Uuid;
use chrono::Utc;

#[tokio::main]
async fn main() -> anyhow::Result<()> {
    // Initialize database
    let db_url = std::env::var("DATABASE_URL")
        .unwrap_or_else(|_| "sqlite:hyperfine.db".to_string());
    
    let repo = BenchmarkRepository::new(&db_url).await?;

    // ... existing hyperfine code ...

    // After benchmarking completes:
    let run_id = Uuid::new_v4().to_string();
    let benchmark_run = BenchmarkRun {
        id: run_id.clone(),
        timestamp: Utc::now(),
        benchmark_name: "my_benchmark".to_string(),
        command: "sleep 0.1".to_string(),
        shell: "bash".to_string(),
        warmup_runs: 3,
        total_runs: 10,
        min_time_seconds: 3.0,
        notes: Some("Initial benchmark".to_string()),
        created_at: Utc::now(),
    };

    repo.save_run(&benchmark_run).await?;

    // Save measurements
    for (idx, timing) in timings.iter().enumerate() {
        let measurement = Measurement {
            id: Uuid::new_v4().to_string(),
            run_id: run_id.clone(),
            sequence: idx as i32,
            execution_time_ms: timing.execution_ms,
            shell_overhead_ms: shell_overhead_ms,
            adjusted_time_ms: timing.adjusted_ms,
            is_outlier: outliers.contains(&idx),
        };
        repo.save_measurement(&measurement).await?;
    }

    // Save statistics
    let stats = Statistics {
        id: Uuid::new_v4().to_string(),
        run_id: run_id.clone(),
        mean_ms: computed_mean,
        median_ms: computed_median,
        stddev_ms: computed_stddev,
        min_ms: computed_min,
        max_ms: computed_max,
        q1_ms: Some(q1),
        q3_ms: Some(q3),
        iqr_ms: Some(iqr),
        outlier_count: outliers.len() as i32,
    };
    repo.save_statistics(&stats).await?;

    Ok(())
}
```

---

## 📊 Blog Topics for Deep Dives

### Blog 1: Understanding Benchmarking Accuracy
- **Topic**: Why shell overhead correction matters
- **Deep Dive**: Calibration methodology
- **Significance**: Ensures accurate sub-millisecond measurements

### Blog 2: Statistical Outlier Detection
- **Topic**: How hyperfine identifies interference
- **Deep Dive**: IQR-based outlier detection
- **Significance**: Reliable metrics despite system noise

### Blog 3: Parameterized Benchmarks
- **Topic**: Varying parameters across runs
- **Deep Dive**: `-P` and `-L` options
- **Significance**: Comprehensive performance analysis

### Blog 4: Database-Driven Performance Tracking
- **Topic**: Storing and querying historical benchmarks
- **Deep Dive**: SQLx integration and trend analysis
- **Significance**: Long-term performance monitoring

### Blog 5: Real-World Benchmarking with Persistence
- **Topic**: Best practices with database storage
- **Deep Dive**: Warmup, prepare, cleanup, and retention
- **Significance**: Reliable performance regressions detection

---

## 🎯 Key Success Criteria (Missions)

| Mission | Goal | Measurement |
|---------|------|-------------|
| **Accuracy** | Measure sub-millisecond performance | ±0.1% precision |
| **Statistical Reliability** | Detect true performance differences | Outlier detection, confidence intervals |
| **Usability** | Simple CLI for common cases | Single command execution |
| **Flexibility** | Support advanced benchmarking scenarios | Warmup, prepare, parameter variations |
| **Persistence** | Store historical benchmark data | Full database integration |
| **Trending** | Track performance over time | Regression detection, historical comparisons |

---

## 🚀 Implementation Checklist

- [x] Architecture documentation
- [x] Workflow diagrams and flows
- [x] Database schema (PostgreSQL, MySQL, SQLite)
- [x] SQLx Rust models and repository
- [x] Integration code snippets
- [ ] Full database implementation (next step)
- [ ] Migration tools
- [ ] Reporting dashboard
- [ ] CLI flags for database storage
- [ ] Performance trend analysis

