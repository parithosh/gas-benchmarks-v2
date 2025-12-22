# Example Configurations

This directory contains example configurations for common benchmark scenarios.

## Quick Usage

```bash
# Use an example configuration
ansible-playbook collections/ansible_collections/local/main/playbooks/run_benchmarks.yml \
  -i inventory/hosts.yml \
  -e @examples/single-client-custom-image.yml
```

## Available Examples

| File | Description |
|------|-------------|
| `single-client-custom-image.yml` | Benchmark a single client with a specific Docker image version |
| `multi-client-comparison.yml` | Compare multiple clients with statistical runs |
| `postgres-config.yml` | Configuration for PostgreSQL metric ingestion |
| `remote-execution-inventory.yml` | Inventory file for running benchmarks on a remote server |

## Combining Examples

You can combine multiple example files:

```bash
ansible-playbook collections/ansible_collections/local/main/playbooks/run_benchmarks.yml \
  -i inventory/hosts.yml \
  -e @examples/multi-client-comparison.yml \
  -e @examples/postgres-config.yml \
  --tags benchmarks,postgres
```

## Customizing

1. Copy the example that best matches your use case
2. Modify values as needed
3. Use with `-e @your-config.yml`

Or copy specific values to `group_vars/all.yml` for persistent defaults.
