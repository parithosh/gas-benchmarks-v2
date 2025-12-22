# Inventory

This directory contains Ansible inventory definitions.

## Files

### hosts.yml

Default inventory with localhost configuration:

```yaml
---
all:
  hosts:
    localhost:
      ansible_connection: local
      ansible_python_interpreter: "{{ ansible_playbook_python }}"
```

## Local Execution

By default, benchmarks run locally on the machine executing Ansible. The `ansible_connection: local` setting ensures no SSH is used.

## Remote Execution

To run benchmarks on a remote machine:

1. Create a new inventory file:

```yaml
# inventory/remote.yml
---
all:
  hosts:
    benchmark-server:
      ansible_host: 192.168.1.100
      ansible_user: ubuntu
      ansible_ssh_private_key_file: ~/.ssh/benchmark_key
```

2. Run with the remote inventory:

```bash
ansible-playbook ... -i inventory/remote.yml
```

3. Artifacts will be fetched to `local_artifacts_dir` after completion.

## Multiple Hosts

For running benchmarks on multiple machines (not in parallel):

```yaml
---
all:
  hosts:
    benchmark-1:
      ansible_host: 192.168.1.100
      benchmark_clients: ['nethermind', 'geth']
    benchmark-2:
      ansible_host: 192.168.1.101
      benchmark_clients: ['reth', 'besu']
```

Note: Each host runs all its clients sequentially. Hosts are NOT executed in parallel by default.

## Dynamic Inventory

For cloud environments, place executable scripts that output JSON inventory format. See Ansible documentation for AWS, GCP, or Azure dynamic inventory plugins.
