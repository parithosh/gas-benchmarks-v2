# Group Variables

This directory contains group-specific variable definitions for Ansible.

## Files

### all.yml

Default configuration for all hosts. Contains:

- **Path Configuration**: `benchmark_source_dir`, `benchmark_tests_root`
- **Benchmark Settings**: `benchmark_clients`, `benchmark_runs`, `benchmark_filter`
- **Test Configuration**: `benchmark_test_paths`, `benchmark_warmup_file`
- **Docker Images**: `benchmark_images` (map client to "default" or custom image)
- **Output Directories**: `benchmark_results_dir`, `benchmark_reports_dir`, `benchmark_logs_dir`
- **PostgreSQL Settings**: `postgres_host`, `postgres_user`, `postgres_password`, etc.

## Variable Precedence

Variables are evaluated in this order (later overrides earlier):

1. Role defaults (`roles/*/defaults/main.yml`)
2. Group vars (`group_vars/all.yml`)
3. Host vars (`host_vars/localhost.yml`)
4. Playbook vars
5. Extra vars (`-e "var=value"`) - **highest priority**

## Usage

To override a default value:

```bash
# Via extra vars (recommended for temporary overrides)
ansible-playbook ... -e "benchmark_runs=3"

# Via secrets file (recommended for credentials)
ansible-playbook ... -e "@group_vars/secrets.yml"
```

## Secrets

For PostgreSQL credentials, prefer environment variables or a separate secrets file:

```yaml
# group_vars/secrets.yml (DO NOT COMMIT)
DB_HOST: "db.example.com"
DB_USER: "benchmark_user"
DB_PASSWORD: "secret123"
```

Add `secrets.yml` to `.gitignore` to prevent accidental commits.
