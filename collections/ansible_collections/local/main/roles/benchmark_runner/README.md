# benchmark_runner

Phase 3 role that executes benchmarks for a single Ethereum client.

## Purpose

- Executes `run.sh` with client-specific parameters
- Collects Docker logs after execution
- Validates report generation
- Synchronizes artifacts to top-level directories
- Handles remote artifact fetching for distributed execution

## Variables

### Required

| Variable | Description |
|----------|-------------|
| `client_name` | Name of the client to benchmark (e.g., `nethermind`) |
| `workspace_root` | Root directory for benchmark execution |
| `benchmark_source_dir` | Directory containing benchmark scripts |
| `benchmark_test_paths` | List of test path configurations |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `benchmark_runs` | `1` | Number of benchmark iterations |
| `benchmark_filter` | Empty | Test name pattern filter |
| `benchmark_warmup_file` | Empty | Warmup file path |
| `benchmark_network` | Empty | Network configuration |
| `benchmark_snapshot_root` | Empty | Snapshot directory for overlay testing |
| `benchmark_restart_before_testing` | `false` | Restart client before each test |
| `benchmark_skip_forkchoice` | `false` | Skip forkchoice updates |
| `benchmark_opcodes_warmup_count` | `1` | Opcode warmup iterations |
| `benchmark_images` | See group_vars | Docker image overrides |
| `benchmark_jwt_path` | `/tmp/jwtsecret` | JWT secret file path |
| `benchmark_logs_dir` | `logs` | Directory for Docker logs |

## Command Construction

The role builds a command like:

```bash
bash run.sh \
  -t "eest_tests" \
  -g "zkevmgenesis.json" \
  -c "nethermind" \
  -r 1 \
  [-w "warmup.json"] \
  [-f "filter_pattern"] \
  [-n "network"] \
  [-B "snapshot_root"] \
  [-R] \
  [-F] \
  [-o warmup_count] \
  [-i '{"nethermind": "image:tag"}']
```

## Timeout

Benchmarks have a 2-hour (7200 second) timeout via Ansible async. The timeout is handled by Ansible's async mechanism, not shell timeout.

## Artifact Collection

After execution:

1. **Docker logs**: Collected to `logs/{client}/`
2. **Results**: Synced from `gas-benchmarks/results/` to `results/`
3. **Reports**: Synced from `gas-benchmarks/reports/` to `reports/`
4. **Logs**: Synced from `gas-benchmarks/logs/` to `logs/`

For remote execution, artifacts are also fetched to `local_artifacts_dir`.

## Error Handling

- Benchmark failures are logged with exit code and stderr
- Docker log collection continues even if benchmark fails
- Missing reports directory vs empty directory have different error messages
- Artifact sync failures are logged but don't fail the playbook

## Example Usage

```yaml
- name: Run benchmarks for nethermind
  ansible.builtin.include_role:
    name: benchmark_runner
  vars:
    client_name: "nethermind"
    benchmark_runs: 3
    benchmark_filter: "bn128"
```
