---
name: solace-agent-mesh-k8s-cicd
description: Use when planning, modifying, reviewing, upgrading, or validating Solace Agent Mesh deployments on Kubernetes, including core releases, gateways, standalone agents, RBAC/users, prompts/projects, secrets, persistence, and custom Python tool images.
---

# Solace Agent Mesh k8s CI/CD

## What this skill does

Gives agents the operational knowledge to make correct, minimal changes to Solace Agent Mesh on Kubernetes. Covers Git-managed SAM environments using Helm, Kubernetes secrets/config, and optionally custom Python tool packages.

Workflow: discover live topology, classify the change, edit only the required surfaces, dry-run, deploy, verify, document rollback.

## Quick start

1. Run `scripts/collect_sam_inventory.sh <namespace> [output-dir]` before making any assumptions about release names, pods, services, secrets, or chart mode.
2. Snapshot every release you will touch: `scripts/backup_helm_release.sh <namespace> <release> [output-dir]`.
3. Classify the request into one or more change surfaces:
   - standalone agent
   - gateway
   - SAM core / control plane
   - users / auth / RBAC
   - prompts / projects / platform objects
   - secrets / runtime contracts
   - custom Python tools / image build
4. Read the matching reference file before editing anything.
5. Run `helm upgrade --dry-run` or `helm template` before any live rollout.
6. Verify both Kubernetes health and a functional SAM scenario after the change.

## Non-negotiable rules

- Never inline real credentials in YAML. Use placeholders or Kubernetes secrets.
- Keep stable identities stable unless the task is an intentional replacement.
  Stable identities: Helm release name, `id`, `agent_name`, `gateway_id`, project IDs, secret names referenced by other components.
- If Python code or dependencies change, build a new image tag. YAML-only changes do not justify reusing a stale image.
- If only prompt text, instructions, allow-lists, routing rules, or built-in tool wiring changes, prefer config-only redeploy. Do not rebuild the image.
- Upgrade SAM core first, then gateways, then standalone agents, unless release notes explicitly require a different order.
- Back up live values and rendered manifests before every upgrade or replacement.
- After any live change, verify one end-to-end task, not just pod readiness.

## Workflow decision tree

### 1. Inventory and classify

Run the inventory script first. Then answer:
- Is this change in core, a gateway, a standalone agent, RBAC/auth, prompts/projects, or custom tool code?
- Is the deployment Git-managed, platform/deployer-managed, or mixed?
- Is the current deployment using bundled persistence or external services?

Read [`references/platform-surfaces.md`](references/platform-surfaces.md) and [`references/change-matrix.md`](references/change-matrix.md).

### 2. Pick the right workflow

| Change type | Reference |
|---|---|
| New or updated standalone agent | [`deployment-workflows.md`](references/deployment-workflows.md) |
| Gateway changes | [`deployment-workflows.md`](references/deployment-workflows.md) |
| Core or chart upgrade | [`versioning-and-release-policy.md`](references/versioning-and-release-policy.md) |
| RBAC, users, auth wiring | [`platform-surfaces.md`](references/platform-surfaces.md) |
| Prompt library, projects, platform API objects | [`deployment-workflows.md`](references/deployment-workflows.md) |
| Validation and rollback | [`validation-and-rollback.md`](references/validation-and-rollback.md) |

### 3. Edit only the required surfaces

Use the change matrix to determine whether the task requires changes in:
- app `config.yaml`
- Helm values
- secret contract
- image tag / package version
- platform API seed data
- RBAC / auth config

If the request touches multiple surfaces, edit them together and keep versions aligned.

### 4. Validate before and after rollout

Minimum:
- rendered config is correct
- release dry-run succeeds
- rollout completes
- logs show healthy startup
- platform health is green
- target agent/gateway is discoverable
- one real scenario completes

See [`references/validation-and-rollback.md`](references/validation-and-rollback.md).

## When to rebuild the image

Rebuild and retag when any of these change:
- Python source under `src/`
- `pyproject.toml`, requirements, or wheel contents
- vendored plugin or tool modules
- base image version

Do NOT rebuild for:
- instruction or prompt edits
- model endpoint or model-name env references
- allow-lists and peer names
- routing rules
- RBAC changes
- prompt library entries

## When to change identity vs version only

- Keep the same release name and `id` for in-place upgrades.
- Keep the same `agent_name` or `gateway_id` if downstream peers, projects, or users already reference it.
- Change the identity only when intentionally replacing a component or running old and new side by side.
- Bump image tags for code changes. Bump chart/app config revision in Git for config changes. Use semantic versions or immutable git-SHA tags. Do not reuse mutable image tags in production.

## Scripts

### `scripts/collect_sam_inventory.sh`

Run first. Inventories namespace state, Helm releases, workloads, services, ingress, config maps, secrets, role bindings, and likely SAM-related releases. Read-only.

### `scripts/backup_helm_release.sh`

Run before changing a Helm-managed component. Saves `helm status`, `helm get values`, `helm get manifest`, `kubectl get` outputs, and recent logs. Read-only.

## References

| File | What it covers |
|---|---|
| [`platform-surfaces.md`](references/platform-surfaces.md) | What each SAM control surface owns |
| [`change-matrix.md`](references/change-matrix.md) | What to edit for each change type |
| [`deployment-workflows.md`](references/deployment-workflows.md) | Step-by-step rollout patterns |
| [`versioning-and-release-policy.md`](references/versioning-and-release-policy.md) | Versioning rules, upgrade order, replacement strategy |
| [`validation-and-rollback.md`](references/validation-and-rollback.md) | Smoke tests, functional checks, rollback procedures |

## Output expectations

When using this skill, return:
1. Discovered topology and assumptions
2. Exact change surfaces being modified
3. Files or Kubernetes objects to edit
4. Rollout plan
5. Verification plan
6. Rollback path

If the environment already contains live values, manifests, or runbooks, use them as evidence. Do not overfit to one cluster unless the task is explicitly cluster-specific.
