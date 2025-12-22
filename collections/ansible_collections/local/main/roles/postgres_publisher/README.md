# postgres_publisher

Phase 4 role that optionally publishes benchmark results to PostgreSQL.

## Purpose

- Tests PostgreSQL connection
- Validates `fill_postgres_db.py` script exists
- Ingests CSV results to database table
- Runs only when `--tags postgres` is specified

## Variables

### Required (when using --tags postgres)

| Variable | Description | Environment Variable |
|----------|-------------|---------------------|
| `postgres_host` | Database hostname | `DB_HOST` |
| `postgres_user` | Authentication username | `DB_USER` |
| `postgres_password` | Authentication password | `DB_PASSWORD` |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `postgres_port` | `5432` | Database port |
| `postgres_database` | `monitoring` | Database name |
| `postgres_table` | `gas_limit_experiments` | Target table name |
| `postgres_timeout` | `10` | Connection timeout in seconds |
| `benchmark_source_dir` | `gas-benchmarks` | Submodule directory |
| `benchmark_reports_dir` | `results` | Directory containing CSV files |

## Security

Credentials are passed via environment variables to child processes, not embedded in shell commands. The `no_log: true` directive prevents credential exposure in Ansible output.

## Prerequisites

- PostgreSQL database accessible from the execution host
- `psycopg2` Python package installed (use `pip install psycopg2-binary`)
- `fill_postgres_db.py` script in the submodule

## Workflow

1. Validate credentials are provided
2. Check if reports directory exists with CSV files
3. Test PostgreSQL connection
4. Verify `fill_postgres_db.py` script exists
5. Execute ingestion script
6. Log success or failure

## Error Handling

- Missing credentials: Fails with descriptive message
- Connection failure: Logs warning, skips ingestion
- Missing script: Fails with submodule initialization hint
- Ingestion failure: Logs error, doesn't fail playbook

## Example Usage

```bash
# Run benchmarks with PostgreSQL ingestion
ansible-playbook run_benchmarks.yml \
  -e "benchmark_clients=['nethermind']" \
  --tags benchmarks,postgres \
  -e "postgres_host=db.example.com" \
  -e "postgres_user=benchmark_user" \
  -e "postgres_password=secret123"

# Or using environment variables
export DB_HOST=db.example.com
export DB_USER=benchmark_user
export DB_PASSWORD=secret123
ansible-playbook run_benchmarks.yml --tags benchmarks,postgres
```

## Table Schema

The target table should have columns matching the CSV output from benchmarks. Refer to the `fill_postgres_db.py` script for expected schema.
