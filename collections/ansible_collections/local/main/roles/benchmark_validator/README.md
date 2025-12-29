# benchmark_validator

Phase 2 role that validates all prerequisites before benchmark execution.

## Purpose

- Manages lock file to prevent concurrent execution
- Validates Docker, Docker Compose, Python, jq, and dotnet installations
- Checks minimum version requirements
- Validates client directories and configuration files exist
- Validates test paths and genesis files exist

Note: All installations (dotnet, uv, Python dependencies, git-lfs, Makefile tools) are handled by the `environment_setup` role.

## Variables

### Required

| Variable | Description |
|----------|-------------|
| `benchmark_workspace_root` | Root directory for benchmark execution |
| `benchmark_source_dir` | Directory containing benchmark scripts |
| `benchmark_clients` | List of clients to validate |
| `benchmark_test_paths` | List of test path configurations |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `benchmark_lock_file` | `/tmp/gas_benchmark.lock` | Lock file path |
| `min_docker_version` | `20.10` | Minimum Docker version |
| `min_docker_compose_version` | `2.0` | Minimum Docker Compose version |
| `min_python_version` | `3.10` | Minimum Python version |
| `min_dotnet_version` | `9.0` | Minimum dotnet SDK version |
| `benchmark_warmup_file` | Empty | Optional warmup file path |
| `benchmark_overlay_enabled` | `false` | Enable overlay snapshot mode |
| `benchmark_snapshot_root` | `""` | Path to snapshot directory (required when overlay enabled) |

## Lock File Mechanism

The role uses timestamp-based staleness detection for the lock file:

1. If lock file exists and is **less than 2 hours old**: fails with "Benchmark already running"
2. If lock file exists and is **older than 2 hours**: removes stale lock and continues
3. Creates new lock file with current timestamp

The 2-hour threshold matches the benchmark async timeout. Lock files are cleaned up in the playbook's cleanup phase on normal completion.

## Validations Performed

1. **Lock file**: Check for concurrent execution
2. **Overlay config**: Validate snapshot_root is set when overlay enabled
3. **Docker**: Version >= 20.10, daemon running, Docker Hub connectivity
4. **Docker Compose**: Version >= 2.0
5. **Ports**: 8545 (JSON-RPC) and 8551 (Engine API) availability
6. **Python**: Version >= 3.10
7. **jq**: Installed (required for JSON processing)
8. **yq**: Installed (required for YAML modifications)
9. **Dotnet SDK**: Version >= 9.0 (fails if not installed)
10. **Disk space**: Minimum 5GB free, warning at < 10GB
11. **Benchmark scripts**: run.sh (executable), setup_node.py, requirements.txt
12. **Client directories**: scripts/{client}/ for each client
13. **Docker Compose files**: scripts/{client}/docker-compose.yaml
14. **Genesis files**: scripts/genesisfiles/{client}/{genesis}
15. **Test paths**: All configured test directories exist

## Error Handling

- Missing prerequisites fail with descriptive error messages
- Version mismatches show required vs found versions
- Disk space warnings don't fail but are logged
- Port conflicts generate warnings but don't fail

## Example Usage

```yaml
- name: Validate prerequisites
  ansible.builtin.include_role:
    name: benchmark_validator
  vars:
    benchmark_workspace_root: "/opt/benchmarks"
    benchmark_clients:
      - nethermind
      - geth
    benchmark_test_paths:
      - path: "gas-benchmarks/eest_tests"
        genesis: "zkevmgenesis.json"
```
