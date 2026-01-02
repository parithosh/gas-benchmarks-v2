# Gas Benchmarks V2

Ansible-orchestrated gas benchmark suite for Ethereum execution clients.

## Quick Start

```bash
# 1. Install dependencies
make install
source .venv/bin/activate
ansible-galaxy collection install -r requirements.yml

# 2. Initialize test submodule
make init-submodule

# 3. Run a benchmark
ansible-playbook collections/ansible_collections/local/main/playbooks/run_benchmarks.yml \
  -i inventory/hosts.yml \
  -e "benchmark_clients=['nethermind']" \
  -e "benchmark_filter='bn128'"
```

## Requirements

| Tool | Version | Purpose |
|------|---------|---------|
| Docker | 20.10+ | Container runtime |
| Docker Compose | 2.0+ | Container orchestration |
| Python | 3.10+ | Ansible and scripts |
| jq | 1.6+ | JSON processing |

## Usage

### Basic Commands

```bash
# Single client
ansible-playbook collections/ansible_collections/local/main/playbooks/run_benchmarks.yml \
  -i inventory/hosts.yml \
  -e "benchmark_clients=['nethermind']"

# Multiple clients
ansible-playbook collections/ansible_collections/local/main/playbooks/run_benchmarks.yml \
  -i inventory/hosts.yml \
  -e "benchmark_clients=['nethermind','geth','reth']"

# With test filter
ansible-playbook collections/ansible_collections/local/main/playbooks/run_benchmarks.yml \
  -i inventory/hosts.yml \
  -e "benchmark_clients=['nethermind']" \
  -e "benchmark_filter='bn128'"

# Custom Docker image
ansible-playbook collections/ansible_collections/local/main/playbooks/run_benchmarks.yml \
  -i inventory/hosts.yml \
  -e "benchmark_clients=['nethermind']" \
  -e "benchmark_images={'nethermind': 'ethpandaops/nethermind:1.25.0'}"
```

### Remote Execution

```bash
# Edit inventory with your remote host
cp inventory/hosts.yml.example inventory/hosts.yml
vi inventory/hosts.yml

# Run on remote
ansible-playbook collections/ansible_collections/local/main/playbooks/run_benchmarks.yml \
  -i inventory/hosts.yml
```

### Cleanup

```bash
# Clean stale files after cancelled runs
ansible-playbook collections/ansible_collections/local/main/playbooks/cleanup.yml \
  -i inventory/hosts.yml
```

## Configuration

### Key Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `benchmark_clients` | `['nethermind','geth','reth']` | Clients to benchmark |
| `benchmark_filter` | `""` | Test name filter pattern |
| `benchmark_runs` | `1` | Number of iterations |
| `benchmark_images` | `{client: "default"}` | Docker image overrides |

Configuration files:
- `inventory/group_vars/all.yml` - Default settings
- `inventory/host_vars/<host>.yml` - Host-specific overrides

### PostgreSQL Integration (Optional)

```bash
# Set credentials via environment
export DB_HOST=db.example.com
export DB_USER=benchmark_user
export DB_PASSWORD=secret

# Run with PostgreSQL ingestion
ansible-playbook collections/ansible_collections/local/main/playbooks/run_benchmarks.yml \
  -i inventory/hosts.yml \
  -e "benchmark_clients=['nethermind']" \
  --tags benchmarks,postgres
```

## Results

Results are stored in timestamped directories:
- `results/<timestamp>/` - CSV benchmark data
- `reports/<timestamp>/` - HTML reports
- `logs/<timestamp>/` - Execution logs

View HTML report:
```bash
open reports/*/index.html  # macOS
xdg-open reports/*/index.html  # Linux
```

## Supported Clients

- nethermind
- geth
- reth
- besu
- erigon
- nimbus
- ethrex

## Project Structure

```
gas-benchmarks-v2/
├── collections/ansible_collections/local/main/
│   ├── playbooks/
│   │   ├── run_benchmarks.yml    # Main playbook
│   │   └── cleanup.yml           # Cleanup playbook
│   └── roles/
│       ├── environment_setup/    # Phase 1: Setup
│       ├── benchmark_validator/  # Phase 2: Validation
│       ├── benchmark_runner/     # Phase 3: Execution
│       └── postgres_publisher/   # Phase 4: DB ingestion
├── inventory/
│   ├── hosts.yml                 # Ansible inventory
│   ├── group_vars/all.yml        # Default config
│   └── host_vars/                # Host overrides
├── gas-benchmarks/               # Test submodule
├── results/                      # Benchmark results
├── reports/                      # HTML reports
└── logs/                         # Execution logs
```


## Troubleshooting

### Common Issues
**Stale lock file:**
```bash
rm /tmp/gas_benchmark.lock
# Or run cleanup playbook
```
