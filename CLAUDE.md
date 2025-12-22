# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

gas-benchmarks-v2 is an Ansible-orchestrated gas benchmark suite for Ethereum execution clients. It automates benchmarking of clients (Nethermind, Geth, Reth, Besu, Erigon, Nimbus, Ethrex) using EELS test payloads, with multi-client comparative analysis and optional PostgreSQL metrics ingestion.

## Build & Development Commands

```bash
# Initial setup
make install              # Create .venv, install Ansible + linting tools
make init-submodule       # Clone gas-benchmarks submodule
make prepare_tools        # Validate Docker, Python, jq dependencies

# Validation
make validate             # Run ansible-lint and yamllint
make test                 # Run pytest tests
make check-deps           # Verify all dependencies installed

# Running benchmarks (activate venv first)
source .venv/bin/activate
ansible-playbook collections/ansible_collections/local/main/playbooks/run_benchmarks.yml \
  -i inventory/hosts.yml \
  -e "benchmark_clients=['nethermind']"

# With PostgreSQL ingestion
ansible-playbook collections/ansible_collections/local/main/playbooks/run_benchmarks.yml \
  -i inventory/hosts.yml \
  -e "benchmark_clients=['nethermind']" \
  --tags benchmarks,postgres \
  -e "postgres_host=..." -e "postgres_user=..." -e "postgres_password=..."

# Update submodule to specific branch
make update-submodule BRANCH=<branch>

# Scaffolding
make new-role NAME=<name>
make new-playbook NAME=<name>

# Cleanup
make clean                # Remove .venv and caches
```

## Architecture

### 5-Phase Execution Model

The main playbook (`collections/ansible_collections/local/main/playbooks/run_benchmarks.yml`) orchestrates:

1. **Environment Setup** (`roles/environment_setup/`) - Directory creation, lock file, submodule initialization, git-lfs setup
2. **Validation** (`roles/benchmark_validator/`) - Validates Docker, Python, jq, disk space, client configs, installs dependencies
3. **Benchmark Execution** (`roles/benchmark_runner/`) - Sequential client benchmarking via `run.sh`, artifact collection
4. **PostgreSQL Ingestion** (`roles/postgres_publisher/`) - Optional, tag-gated CSV ingestion to database
5. **Cleanup** - Lock file removal, overlay unmounting (always runs)

### Directory Structure

```
gas-benchmarks-v2/
├── collections/ansible_collections/local/main/
│   ├── playbooks/run_benchmarks.yml    # Main orchestration playbook
│   └── roles/
│       ├── environment_setup/          # Phase 1
│       ├── benchmark_validator/        # Phase 2
│       ├── benchmark_runner/           # Phase 3
│       └── postgres_publisher/         # Phase 4
├── group_vars/all.yml                  # Default configuration
├── inventory/hosts.yml                 # Ansible inventory
├── gas-benchmarks/                     # Git submodule (test payloads, scripts)
│   ├── run.sh                          # Main benchmark script
│   ├── images.yaml                     # Default Docker images
│   ├── scripts/{client}/               # Client-specific configs
│   └── eest_tests/                     # Test payloads
├── results/                            # Benchmark CSV results
├── reports/                            # HTML reports
└── logs/                               # Docker and execution logs
```

### Configuration Hierarchy

1. CLI extra vars (`-e "param=value"`) - highest priority
2. `group_vars/all.yml` - site defaults
3. Role `defaults/main.yml` - role defaults
4. Environment variables (`DB_HOST`, `DB_USER`, `DB_PASSWORD`) - for PostgreSQL

## Complete Configuration Reference

### Core Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `benchmark_clients` | `['nethermind', 'geth', 'reth']` | List of clients to benchmark |
| `benchmark_runs` | `1` | Number of iterations per test |
| `benchmark_filter` | Empty | Test name pattern filter |
| `benchmark_source_dir` | `gas-benchmarks` | Submodule directory |
| `benchmark_tests_root` | `gas-benchmarks` | Test artifacts root |

### Test Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `benchmark_test_paths` | `[{path: "gas-benchmarks/eest_tests", genesis: "zkevmgenesis.json"}]` | Test directories with genesis files |
| `benchmark_warmup_file` | Empty | Path to warmup test file |
| `benchmark_opcodes_warmup_count` | `1` | Opcode warmup iterations |

### Docker Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `benchmark_images` | `{client: "default", ...}` | Docker image overrides per client |
| `benchmark_jwt_path` | `/tmp/jwtsecret` | JWT secret file path |

"default" in `benchmark_images` means use image from `gas-benchmarks/images.yaml`.

### Advanced Features

| Parameter | Default | Description |
|-----------|---------|-------------|
| `benchmark_network` | Empty | Network configuration for testing |
| `benchmark_snapshot_root` | Empty | Snapshot directory for overlay testing |
| `benchmark_restart_before_testing` | `false` | Restart client before each test |
| `benchmark_skip_forkchoice` | `false` | Skip forkchoice updates |

### Output Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| `benchmark_results_dir` | `results` | Results output directory |
| `benchmark_reports_dir` | `reports` | Reports output directory |
| `benchmark_logs_dir` | `logs` | Logs output directory |
| `local_artifacts_dir` | Project root | Where to fetch remote artifacts |

### PostgreSQL Configuration

| Parameter | Default | Env Var | Description |
|-----------|---------|---------|-------------|
| `postgres_host` | Empty | `DB_HOST` | Database hostname |
| `postgres_port` | `5432` | - | Database port |
| `postgres_user` | Empty | `DB_USER` | Authentication username |
| `postgres_password` | Empty | `DB_PASSWORD` | Authentication password |
| `postgres_database` | `monitoring` | - | Database name |
| `postgres_table` | `gas_limit_experiments` | - | Target table name |
| `postgres_timeout` | `10` | - | Connection timeout (seconds) |

## Critical Patterns

### Submodule Dependency

The `gas-benchmarks/` submodule must be initialized before running benchmarks:
- Auto-clones if missing during environment_setup phase
- Manual init recommended: `make init-submodule`
- Contains: run.sh, images.yaml, test payloads, genesis files, client scripts

### Lock File Mechanism

Prevents concurrent benchmark execution:
- Created at `{{ workspace_root }}/.benchmark.lock` with PID
- Checks if existing lock process is still running
- Stale locks (dead process) are automatically removed
- Always cleaned up in playbook cleanup phase

### Sequential Client Execution

Clients are benchmarked one at a time to avoid resource contention:
- Loop in playbook iterates through `benchmark_clients`
- Each client failure is tracked independently
- Failures don't stop execution of subsequent clients
- Final summary shows all client statuses

### Artifact Synchronization

Results flow: submodule -> top-level directories:
1. Benchmarks run in `gas-benchmarks/` and generate output there
2. `synchronize` module copies to top-level `results/`, `reports/`, `logs/`
3. For remote execution, artifacts are also fetched to local machine

### Async Timeout

Benchmark execution has a 2-hour timeout:
- Implemented via Ansible `async: 7200` and `poll: 30`
- Handles long-running tests without blocking
- No shell-level timeout (removed for clarity)

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| "Another benchmark is in progress" | Stale lock file | Remove `.benchmark.lock` manually |
| "Submodule not initialized" | Missing gas-benchmarks | Run `make init-submodule` |
| "Docker daemon not running" | Docker not started | Start Docker service |
| "reports directory is empty" | Benchmark script failed | Check `logs/{client}/` for errors |
| "fill_postgres_db.py not found" | Submodule issue | Re-initialize submodule |

### Debug Commands

```bash
# Verbose output
ansible-playbook ... -vvv

# Check only (don't run)
ansible-playbook ... --check

# List tasks without running
ansible-playbook ... --list-tasks

# Run specific tags
ansible-playbook ... --tags benchmarks
ansible-playbook ... --tags postgres
```

### Log Locations

- `logs/{client}/{container}.log` - Docker container logs
- Ansible output - Benchmark stdout/stderr
- `gas-benchmarks/logs/` - Original logs before sync

## Technologies & Requirements

| Tool | Minimum Version | Purpose |
|------|-----------------|---------|
| Ansible Core | 2.15.0 | Orchestration |
| Python | 3.10 | Scripting |
| Docker | 20.10 | Container runtime |
| Docker Compose | 2.0 | Container orchestration |
| jq | 1.6 | JSON processing |
| dotnet SDK | 9.0 | Nethermind.Tools.Kute |
| git-lfs | Any | Large file storage |
| uv | Any | Python package manager |

## Key Files

| File | Purpose |
|------|---------|
| `collections/.../playbooks/run_benchmarks.yml` | Main orchestration |
| `group_vars/all.yml` | Default configuration |
| `gas-benchmarks/run.sh` | Core benchmark script |
| `gas-benchmarks/images.yaml` | Default Docker images |
| `requirements.yml` | Ansible collection dependencies |
| `pyproject.toml` | Python project configuration |
