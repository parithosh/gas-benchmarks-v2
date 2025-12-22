# benchmark_validator

Phase 2 role that validates all prerequisites before benchmark execution.

## Purpose

- Validates Docker, Docker Compose, Python, and jq installations
- Checks minimum version requirements
- Installs dotnet SDK 9.0 if needed (for Nethermind.Tools.Kute)
- Validates client directories and configuration files exist
- Installs Python dependencies via uv package manager
- Prepares tools via Makefile

## Variables

### Required

| Variable | Description |
|----------|-------------|
| `workspace_root` | Root directory for benchmark execution |
| `benchmark_source_dir` | Directory containing benchmark scripts |
| `benchmark_clients` | List of clients to validate |
| `benchmark_test_paths` | List of test path configurations |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `min_docker_version` | `20.10` | Minimum Docker version |
| `min_docker_compose_version` | `2.0` | Minimum Docker Compose version |
| `min_python_version` | `3.10` | Minimum Python version |
| `min_dotnet_version` | `9.0` | Minimum dotnet SDK version |
| `benchmark_warmup_file` | Empty | Optional warmup file path |

## Validations Performed

1. **Docker**: Version >= 20.10, daemon running
2. **Docker Compose**: Version >= 2.0
3. **Python**: Version >= 3.10
4. **jq**: Installed (any version)
5. **Disk space**: Minimum 5GB free, warning at < 10GB
6. **Dotnet SDK**: Version >= 9.0 (auto-installed if missing)
7. **Benchmark scripts**: run.sh, setup_node.py, requirements.txt
8. **Client directories**: scripts/{client}/ for each client
9. **Docker Compose files**: scripts/{client}/docker-compose.yaml
10. **Genesis files**: scripts/genesisfiles/{client}/{genesis}
11. **Test paths**: All configured test directories exist

## Dependencies Installed

- **uv**: Python package manager (from astral.sh)
- **dotnet SDK 9.0**: For Nethermind.Tools.Kute (if not present)
- **git-lfs**: For large file storage support
- **Python packages**: From gas-benchmarks/requirements.txt

## Error Handling

- Missing prerequisites fail with descriptive error messages
- Version mismatches show required vs found versions
- Disk space warnings don't fail but are logged

## Example Usage

```yaml
- name: Validate prerequisites
  ansible.builtin.import_role:
    name: benchmark_validator
  vars:
    benchmark_clients:
      - nethermind
      - geth
    benchmark_test_paths:
      - path: "gas-benchmarks/eest_tests"
        genesis: "zkevmgenesis.json"
```
