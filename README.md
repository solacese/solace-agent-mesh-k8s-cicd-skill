# Solace Agent Mesh k8s CI/CD Skill

Agent skill for operating Solace Agent Mesh on Kubernetes. Gives AI coding agents the context to make correct, minimal changes to SAM deployments managed with Helm and Git.

## What it does

- Change matrix: maps each request type to the exact surfaces, files, and commands that need editing.
- Reference docs: platform surfaces, deployment workflows, versioning policy, validation and rollback.
- Scripts: read-only namespace inventory and Helm release backup.

No credentials, no cluster-specific config, no write operations.

## Usage

```bash
# Discover namespace state
scripts/collect_sam_inventory.sh <namespace>

# Snapshot a release before changing it
scripts/backup_helm_release.sh <namespace> <release>
```

Then follow the workflow in [SKILL.md](SKILL.md).

## Structure

```
SKILL.md                                   Skill definition, rules, decision tree
agents/openai.yaml                         Agent interface metadata
scripts/
  collect_sam_inventory.sh                  Namespace inventory (read-only)
  backup_helm_release.sh                   Helm release backup (read-only)
references/
  platform-surfaces.md                     Control surface definitions
  change-matrix.md                         What to edit for each change type
  deployment-workflows.md                  Step-by-step rollout patterns
  versioning-and-release-policy.md         Versioning rules and upgrade order
  validation-and-rollback.md               Smoke tests, checks, rollback procedures
```

## Requirements

- `kubectl` with cluster access
- `helm` 3.x
- Namespace-level read permissions
