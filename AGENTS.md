# AGENTS.md — vm-k3s-agent

Standalone candy repo for the `k3s-agent` layer — a k3s worker node that joins an
existing `k3s-server` cluster via a pre-shared token. The candy lives in
`charly.yml` at the repo root: the `require:` deps (`layer-k3s`), the
`secret_require:` (`K3S_CLUSTER_TOKEN`) and `env_require:` (`K3S_SERVER_URL`),
the config-render + systemd-unit `plan:` steps, the `check:` assertions, and the
embedded `skill:` entity projected into the marketplace corpus as
`/charly-infrastructure:k3s-agent`.

Canonical files:

- `charly.yml` — the `k3s-agent:` candy entity and the `k3s-agent-skill:` skill
  entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — user overview only; never agent guidance.

## Load these skills first (R0)

- `/charly-infrastructure:k3s-agent` — the owning skill. The worker-node config,
  the token/URL contract, and verification. Load before editing or
  troubleshooting the layer.
- `/charly-infrastructure:k3s` — the base candy that installs the `k3s` binary
  this layer requires.
- `/charly-image:layer` — the candy authoring reference (`charly.yml` schema,
  `plan:` step verbs incl. `check:`, per-distro `distro:` arms, package/repo
  sections, `secret_require:` / `env_require:`, and service declarations). Load
  before editing any entity field or plan step.
- `/charly-check:check` — the check-verb reference for the deploy-scope
  `kube:` step (the cluster probe is served out-of-process by
  `candy/plugin-kube`; there is no host `charly check kube` command).

## Build / validate / test

- The merge gate is the **org-wide** `charly/pr-validator` (required check
  `validate / validate`, defined in `opencharly/.github`); this repo has **no
  per-repo candy gate** and ships only `.github/workflows/tag-on-merge.yml`.
- Validate the manifest with `charly box validate` at the repo root.
- The candy's `plan:` `check:` steps are the functional evidence; the live
  deploy-scope `kube: wait-nodes` step proves the node registers and reaches
  `Ready`.

## Modify this repo

- Edit the `k3s-agent:` candy entity AND the `k3s-agent-skill:` skill entity in
  `charly.yml` together. The skill is the projected usage source, so a
  behaviour change not mirrored in the skill leaves the corpus stale.
- Keep `K3S_SERVER_URL` a deploy-time `env_require:` and `K3S_CLUSTER_TOKEN` a
  credential-store `secret_require:` — do not inline either value into the
  entity.
- New behaviour claims belong in the `plan:` as an observable `check:` step, and
  in the skill body.

## Landing

- The authoritative landing mechanics are `/charly-internals:git-workflow` and
  the umbrella `AGENTS.md` in `opencharly/opencharly`; this signpost does not
  restate them.
- History lives in `CHANGELOG/` (written by `tag-on-merge` at merge time).
