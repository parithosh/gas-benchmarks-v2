# Host Variables

This directory contains host-specific variable overrides.

## Files

### localhost.yml

Contains overrides specific to local execution. May include:

- Custom paths for local environment
- Different client list for testing
- Local Docker image overrides

## Usage

Create a file matching your inventory hostname to override variables for that host:

```yaml
# host_vars/benchmark-server.yml
benchmark_clients:
  - nethermind
  - geth

benchmark_images:
  nethermind: "custom-registry/nethermind:latest"
  geth: "custom-registry/geth:latest"
```

## When to Use Host Vars

Use host vars when you need different configuration for different execution environments:

- **Development**: Fewer clients, fewer runs
- **CI/CD**: Specific image versions, test filters
- **Production**: Full client list, PostgreSQL enabled

## Priority

Host vars override group vars but are overridden by extra vars:

```
role defaults < group_vars < host_vars < extra_vars
```
