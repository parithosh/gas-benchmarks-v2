# environment_setup

Phase 1 role that prepares the execution environment before benchmarking.

## Purpose

- Creates required directories for results, reports, and logs
- Manages lock file to prevent concurrent execution
- Initializes the gas-benchmarks submodule if not present
- Sets up git-lfs and pulls large files

## Variables

### Required

| Variable | Description |
|----------|-------------|
| `workspace_root` | Root directory for benchmark execution |
| `benchmark_source_dir` | Directory containing benchmark scripts (default: `gas-benchmarks`) |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `benchmark_lock_file` | `.benchmark.lock` | Lock file name to prevent concurrent runs |
| `benchmark_results_dir` | `results` | Directory for result artifacts |
| `benchmark_reports_dir` | `reports` | Directory for HTML reports |
| `benchmark_logs_dir` | `logs` | Directory for execution logs |

## Lock File Mechanism

The role creates a lock file at `{{ workspace_root }}/{{ benchmark_lock_file }}` containing the PID of the running process. If a lock file exists:

1. Checks if the process is still running
2. If running, fails with "Another benchmark is in progress"
3. If not running (stale lock), removes the old lock and continues

## Error Handling

- If submodule clone fails, the role will fail
- Lock file is always cleaned up in the playbook's cleanup phase
- Git-lfs installation failures are logged but don't fail the role

## Example Usage

```yaml
- name: Setup environment
  ansible.builtin.import_role:
    name: environment_setup
  vars:
    workspace_root: "/opt/benchmarks"
    benchmark_source_dir: "gas-benchmarks"
```
