# Solace Agent Mesh k8s CI/CD Skill

An agent skill for planning, implementing, upgrading, validating, and rolling back [Solace Agent Mesh](https://solace.com/products/agent-mesh/) deployments on Kubernetes.

## What this is

A knowledge-and-tooling skill that gives AI coding agents (Codex, Claude Code, etc.) the context they need to make correct, minimally disruptive changes to SAM on Kubernetes. It provides:

- **Decision framework** — a change matrix mapping every common request to the exact control surfaces, files, and commands that need to change.
- **Reference docs** — platform surfaces, deployment workflows, versioning policy, and validation/rollback procedures.
- **Discovery and backup scripts** — read-only shell scripts to inventory a namespace and snapshot Helm releases before making changes.

## Quick start

1. Drop this skill into your agent's skill directory (e.g. `skills/solace-agent-mesh-k8s-cicd/`).
2. Run the inventory script to discover live topology:
   ```bash
   scripts/collect_sam_inventory.sh <namespace>
   ```
3. Back up any release you plan to touch:
   ```bash
   scripts/backup_helm_release.sh <namespace> <release>
   ```
4. Refer to `SKILL.md` for the full workflow and decision tree.

## Repository structure

```
SKILL.md                                  # Skill definition and workflow entry point
agents/openai.yaml                        # Agent interface metadata
scripts/
  collect_sam_inventory.sh                 # Namespace inventory (read-only)
  backup_helm_release.sh                   # Helm release backup (read-only)
references/
  platform-surfaces.md                    # What each SAM control surface owns
  change-matrix.md                        # What to edit for each change type
  deployment-workflows.md                 # Step-by-step rollout patterns
  versioning-and-release-policy.md        # Versioning rules and upgrade order
  validation-and-rollback.md              # Smoke tests, checks, and rollback procedures
```

## Prerequisites

- `kubectl` configured with cluster access
- `helm` 3.x
- Namespace-level read access (the scripts only run `get`, `list`, `describe`, and `logs` commands)

## Security

This repository contains no credentials, cluster endpoints, or environment-specific configuration. All scripts are read-only. Secrets and connection details are expected to live in Kubernetes secrets and are referenced by name only.

## License

See [LICENSE](LICENSE) for details.
