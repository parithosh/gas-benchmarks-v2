# benchmark_runner

Phase 3 role that executes benchmarks for a single Ethereum client.

## Purpose

- Cleans up previous execution state (containers, volumes, data directories)
- Patches client docker-compose for non-standard images (with git restore after)
- Executes `run.sh` with client-specific parameters
- Collects Docker logs after execution
- Validates report generation (optional)
- Synchronizes artifacts to top-level directories

Note: Remote artifact fetching is handled at the playbook level after all clients complete.

## Variables

### Required

| Variable | Description |
|----------|-------------|
| `client_name` | Name of the client to benchmark (e.g., `nethermind`) |
| `benchmark_workspace_root` | Root directory for benchmark execution |
| `benchmark_source_dir` | Directory containing benchmark scripts |
| `benchmark_test_paths` | List of test path configurations |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `benchmark_runs` | `1` | Number of benchmark iterations |
| `benchmark_filter` | Empty | Test name pattern filter |
| `benchmark_warmup_file` | Empty | Warmup file path |
| `benchmark_network` | Empty | Network configuration |
| `benchmark_overlay_enabled` | `false` | Enable overlay snapshot mode |
| `benchmark_snapshot_root` | Empty | Snapshot directory for overlay testing |
| `benchmark_restart_before_testing` | `false` | Restart client before each test |
| `benchmark_skip_forkchoice` | `false` | Skip forkchoice updates |
| `benchmark_opcodes_warmup_count` | `1` | Opcode warmup iterations |
| `benchmark_images` | See defaults | Docker image overrides per client |
| `benchmark_jwt_path` | `/tmp/jwtsecret` | JWT secret file path |
| `benchmark_logs_dir` | `logs` | Directory for Docker logs |
| `benchmark_validate_reports` | `true` | Validate that reports are generated |

## Execution Flow

### 1. Pre-Execution Cleanup

- Stops existing Docker containers for the client
- Removes Docker volumes (named and common patterns)
- Cleans or creates `execution-data` directory
- Removes stale genesis files from `/tmp`

### 2. Image Patching (Nethermind only)

For non-performance Nethermind images:
1. Uses `yq` to cleanly remove unsupported flags from docker-compose.yaml:
   - `--Blocks.CachePrecompilesOnBlockProcessing`
   - `--Init.GenesisHash`
2. After run, restores file using `git checkout` to keep repo clean

### 3. Benchmark Execution

Builds and executes `run.sh` with:

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
  [-R] [-F] \
  [-o warmup_count] \
  [-i '{"nethermind": "image:tag"}']
```

### 4. Post-Execution (always runs)

1. **Restore patched files**: `git checkout` for Nethermind docker-compose
2. **Docker logs**: Collected to `logs/{client}/`
3. **Report validation**: Fails if `reports/` is missing or empty (when `benchmark_validate_reports: true`)
4. **Artifact sync**: Copies from `gas-benchmarks/{results,reports,logs}/` to top-level

## Timeout

Benchmarks have a **2-hour (7200 second)** timeout via Ansible async.

## Error Handling

| Scenario | Behavior |
|----------|----------|
| Benchmark script fails | Logs error with stderr/stdout, fails task |
| Reports directory missing | Fails with troubleshooting guidance (if validation enabled) |
| Reports directory empty | Fails with filter mismatch suggestion (if validation enabled) |
| Artifact sync fails | Logs warning, continues execution |
| Docker log collection fails | Logs warning, continues execution |

## Example Usage

```yaml
- name: Run benchmarks for nethermind
  ansible.builtin.include_role:
    name: benchmark_runner
  vars:
    client_name: "nethermind"
    benchmark_workspace_root: "/opt/benchmarks"
    benchmark_runs: 3
    benchmark_filter: "bn128"
    benchmark_validate_reports: true
    benchmark_images:
      nethermind: "ethpandaops/nethermind:performance"
```

## Debugging

To skip report validation (useful for debugging):

```yaml
benchmark_validate_reports: false
```
