# environment_setup

Phase 1 role that prepares the execution environment before benchmarking.

## Purpose

- Creates required directories for results, reports, and logs
- Clones/updates the gas-benchmarks repository
- Sets up git-lfs and pulls large files
- Installs dotnet SDK (if not present or version too old)
- Installs uv Python package manager (if not present)
- Installs Python dependencies via uv
- Builds Nethermind.Tools.Kute via Makefile
- Cleans up stale overlay mounts (when overlay mode is enabled)

## Variables

### Required

| Variable | Description |
|----------|-------------|
| `benchmark_workspace_root` | Root directory for benchmark execution |

### Optional

| Variable | Default | Description |
|----------|---------|-------------|
| `benchmark_source_dir` | `gas-benchmarks` | Directory containing benchmark scripts |
| `benchmark_source_repo` | `https://github.com/NethermindEth/gas-benchmarks.git` | Repository URL |
| `benchmark_source_branch` | `main` | Branch to checkout |
| `benchmark_results_dir` | `results` | Directory for result artifacts |
| `benchmark_reports_dir` | `reports` | Directory for HTML reports |
| `benchmark_logs_dir` | `logs` | Directory for execution logs |
| `benchmark_warmup_results_dir` | `warmupresults` | Directory for warmup results |
| `benchmark_preparation_results_dir` | `prepresults` | Directory for preparation results |
| `benchmark_overlay_enabled` | `false` | Enable overlay snapshot mode |
| `benchmark_snapshot_root` | `""` | Path to snapshot directory (required when overlay enabled) |
| `benchmark_overlay_tmp_root` | `overlay-runtime` | Temporary directory for overlay mounts |
| `min_dotnet_version` | `9.0` | Minimum required dotnet SDK version |

## Dependencies Installed

| Dependency | Purpose |
|------------|---------|
| **git-lfs** | Large file storage for test payloads |
| **yq** | YAML processor for clean docker-compose modifications |
| **dotnet SDK** | Required for Nethermind.Tools.Kute benchmark tooling |
| **uv** | Fast Python package manager (from astral.sh) |
| **Python packages** | From gas-benchmarks/requirements.txt |
| **Nethermind.Tools.Kute** | Built via `make prepare_tools` |

## Overlay Mode

When `benchmark_overlay_enabled: true`, the role cleans up stale overlay mounts from previous runs. This requires:

- `benchmark_snapshot_root` to be set (validated by benchmark_validator role)
- sudo access for unmounting filesystems

## Directory Structure Created

```
{{ benchmark_workspace_root }}/
├── {{ benchmark_results_dir }}/
├── {{ benchmark_warmup_results_dir }}/
├── {{ benchmark_reports_dir }}/
├── {{ benchmark_logs_dir }}/
├── {{ benchmark_preparation_results_dir }}/
└── {{ benchmark_source_dir }}/  (cloned repository)
```

## Example Usage

```yaml
- name: Setup environment
  ansible.builtin.include_role:
    name: environment_setup
  vars:
    benchmark_workspace_root: "/opt/benchmarks"

# With overlay mode
- name: Setup environment with overlays
  ansible.builtin.include_role:
    name: environment_setup
  vars:
    benchmark_workspace_root: "/opt/benchmarks"
    benchmark_overlay_enabled: true
    benchmark_snapshot_root: "/mnt/snapshots/mainnet"
```
