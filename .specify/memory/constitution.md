<!--
SYNC IMPACT REPORT
==================
Version change: Initial → 1.0.0
Modified principles: N/A (initial creation)
Added sections:
  - Core Principles (I-V)
  - Automation & Orchestration
  - Quality Standards
  - Governance
Removed sections: N/A
Templates requiring updates:
  ✅ .specify/templates/plan-template.md (reviewed, Constitution Check section aligns)
  ✅ .specify/templates/spec-template.md (reviewed, requirements align)
  ✅ .specify/templates/tasks-template.md (reviewed, task structure aligns)
Follow-up TODOs: None
-->

# Gas Benchmarks v2 Constitution

## Core Principles

### I. Ansible-First Orchestration

**All benchmark workflows MUST be orchestrated through Ansible playbooks.** Individual scripts remain as reusable components, but execution, coordination, and environment management flow through Ansible roles and tasks. This principle addresses the core problem of v1: scattered scripts with unclear dependencies and manual orchestration.

**Rationale**: Ansible provides declarative infrastructure management, idempotency, clear dependency ordering, and inventory-based configuration. This eliminates the fragility of shell script orchestration and makes the entire pipeline reproducible across environments.

**Non-negotiable rules**:
- New workflow steps are added as Ansible tasks or roles, not standalone scripts
- Scripts may be invoked by Ansible but MUST NOT orchestrate other scripts
- Environment setup, client deployment, test execution, and result collection go through Ansible
- Configuration is managed via Ansible inventory and variables, not hardcoded in scripts

### II. Externalized Test Artifacts

**Benchmark test definitions (payloads, test files) MUST be distributed as versioned releases external to this repository.** This repository contains orchestration code, configuration templates, and result processing—not the large test datasets themselves.

**Rationale**: Test artifacts (EELS payloads, captured tests) are large, change independently of orchestration logic, and should be versioned separately. Using GitHub releases or artifact repositories enables clean versioning, reduces repository bloat, and allows test sets to evolve without code changes.

**Non-negotiable rules**:
- Test files are NOT committed to the main repository
- Tests are fetched from GitHub releases or external artifact stores at runtime
- Each Ansible playbook specifies test artifact version/tag explicitly
- Documentation MUST include instructions for publishing new test releases

### III. Determinism & Reproducibility

**Every benchmark run MUST be fully reproducible given the same inputs.** Client versions, test sets, configurations, and execution parameters MUST be captured in run metadata to enable exact reproduction of any historical result.

**Rationale**: Performance benchmarks are only valuable if results are reproducible. Deterministic execution enables regression detection, cross-client comparison validity, and forensic analysis of performance changes.

**Non-negotiable rules**:
- All client images are tagged (no `:latest` in production runs)
- Test artifact versions are pinned and recorded in results
- Environment specifications (CPU, memory, OS) captured in metadata
- Genesis files, warmup configurations, and run parameters saved with results

### IV. Multi-Client Support

**The framework MUST support benchmarking any Ethereum execution client through a consistent interface.** While Nethermind receives primary focus, the system architecture treats all clients (Geth, Reth, Besu, Erigon, etc.) as equals.

**Rationale**: Valid performance comparison requires testing multiple clients under identical conditions. Client-agnostic design prevents vendor lock-in and ensures the tool remains useful to the broader Ethereum ecosystem.

**Non-negotiable rules**:
- Client-specific logic is isolated in Ansible roles or client adapters
- Test execution, metric collection, and reporting are client-agnostic
- New client support requires only adding a new Ansible role + configuration
- Breaking changes to client interfaces MUST maintain backward compatibility via versioned roles

### V. Observability & Data Pipeline

**Raw results, aggregated metrics, and reports MUST be generated in structured formats enabling both human analysis and automated ingestion.** The system produces CSV outputs, timestamped reports, and PostgreSQL-compatible data for Grafana dashboards.

**Rationale**: Benchmark data has multiple consumers: engineers reviewing reports, CI systems checking regressions, and Grafana displaying trends. Structured output enables all use cases without manual data transformation.

**Non-negotiable rules**:
- Raw results saved as CSV with full metadata (client, version, test, timestamp)
- Aggregated metrics computed and stored separately (MGas/s, percentiles, etc.)
- PostgreSQL schema and ingestion scripts maintained alongside benchmark code
- Reports generated in both human-readable (HTML/Markdown) and machine-readable (JSON) formats

## Automation & Orchestration

### Workflow Structure

All benchmark workflows follow a standard pattern:

1. **Environment Preparation** (Ansible): Validate prerequisites, fetch test artifacts, configure inventory
2. **Client Deployment** (Ansible): Deploy execution client containers with versioned images
3. **Test Execution** (Ansible → Scripts): Run benchmarks via orchestrated script invocation
4. **Result Collection** (Ansible): Aggregate outputs, compute metrics, generate reports
5. **Data Ingestion** (Optional, Ansible): Push results to PostgreSQL for Grafana

Each stage MUST be idempotent and resumable. Failed runs can restart from the last successful checkpoint.

### Configuration Management

- **Inventory**: Defines target environments (local, remote servers, CI)
- **Group Variables**: Client-specific settings (image tags, RPC ports, resource limits)
- **Playbook Parameters**: Test artifact versions, run counts, warmup configurations
- **Secrets**: Managed via Ansible Vault or environment variables (database credentials, API keys)

## Quality Standards

### Testing Requirements

- **Contract Tests**: Verify script outputs match expected schemas (CSV columns, JSON structure)
- **Integration Tests**: End-to-end playbook execution in isolated environments
- **Smoke Tests**: Quick validation runs with minimal test sets before full benchmarks

Testing is NOT optional for changes to orchestration logic, metric computation, or report generation. Scripts that are purely wrappers around external tools may skip dedicated tests if covered by integration tests.

### Documentation Requirements

Every Ansible role MUST include:
- README describing role purpose and parameters
- Example playbook demonstrating usage
- Variable documentation (defaults, required values, types)

Every script invoked by Ansible MUST include:
- Usage documentation (arguments, exit codes, outputs)
- Error handling with meaningful exit codes

### Performance Expectations

- **Benchmark Execution**: Optimized for throughput, not latency (batch processing is acceptable)
- **Result Processing**: Sub-minute aggregation for typical test sets (<1000 tests)
- **Report Generation**: Near-instant for cached data, <5min for full re-computation

## Governance

### Amendment Process

This constitution may be amended when:
1. Core architectural principles change (e.g., adopting a new orchestration tool)
2. New non-negotiable requirements emerge (e.g., compliance, security mandates)
3. Existing principles prove unworkable in practice

Amendments require:
- Documented rationale explaining why existing principles are insufficient
- Migration plan for existing playbooks/scripts
- Updated templates and command files
- Version bump following semantic versioning (see below)

### Versioning Policy

Constitution versions follow **MAJOR.MINOR.PATCH**:
- **MAJOR**: Backward-incompatible changes (e.g., removing a core principle, changing orchestration tool)
- **MINOR**: New principles added, material expansions to existing sections
- **PATCH**: Clarifications, wording improvements, non-semantic fixes

All templates in `.specify/templates/` MUST align with the current constitution version. Misalignment blocks feature planning.

### Compliance Review

- **Plan Phase**: Constitution Check gate in `plan-template.md` MUST be completed
- **Task Phase**: Tasks violating principles require explicit justification in plan.md Complexity Tracking
- **Implementation**: Code reviews MUST verify adherence to Ansible-first and externalized artifacts principles

Violations without documented justification are rejected.

**Version**: 1.0.0 | **Ratified**: 2025-11-27 | **Last Amended**: 2025-11-27
