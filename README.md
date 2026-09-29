# vm-k3s-agent

The `k3s-agent` candy — a k3s **worker node** that joins an existing
`k3s-server` cluster via a pre-shared token — plus its owning skill, projected
into the marketplace corpus as `/charly-infrastructure:k3s-agent`.

The candy renders `/etc/rancher/k3s/config.yaml` (mode `0600`) with the cluster
server URL and the shared token, then installs and enables a
`k3s-agent.service` systemd unit that runs `k3s agent`. On a deployed node the
agent registers with the control plane and the node reaches `Ready` — no
join-token handoff, only the server URL plus the credential-store token.

## What it provides

| Property | Value |
|---|---|
| Layer / candy | `k3s-agent` |
| Requires | `layer-k3s` (installs the `k3s` binary) |
| Secrets | `K3S_CLUSTER_TOKEN` — pre-shared token; auto-generated on first deploy and shared with `k3s-server` via the credential store |
| Env | `K3S_SERVER_URL` (required) — e.g. `https://k3s-srv.lan:6443` |
| Service | `k3s-agent.service` (system scope, enabled) |
| Files | `/etc/rancher/k3s/config.yaml` (0600), `/etc/systemd/system/k3s-agent.service` |

`K3S_CLUSTER_TOKEN` is provisioned automatically: whichever of `k3s-server` /
`k3s-agent` resolves the `secret_require:` first generates a 32-byte hex token
and the other reads the persisted value. Override only when reproducing a
specific cluster identity:

```bash
charly secrets set charly/secret/K3S_CLUSTER_TOKEN $(openssl rand -hex 32)
```

## How to use it

Compose the candy into a VM deploy. The VM hardware template is a `kind: vm`
entity; the disposable deploy overlays the candy via `add_candy:` and supplies
`K3S_SERVER_URL`:

```yaml
k3s-ag1-vm:
  vm:
    source: {kind: cloud_image, distro: arch, url: "…", base_user: arch}
    ram: 4G
    cpu: 2

k3s-ag1:
  vm:
    from: k3s-ag1-vm
    disposable: true
    add_candy: [k3s-agent]
    env:
      K3S_SERVER_URL: https://k3s-srv.lan:6443
      K3S_CLUSTER: k3s-srv
```

```bash
charly fleet add vm:k3s-ag1
```

The agent registers; the candy's deploy-scope check confirms the join with the
declarative `kube: wait-nodes` verb (served out-of-process by
`candy/plugin-kube` — there is no host `charly check kube` command).

## Layout

- `charly.yml` — the `k3s-agent:` candy entity and the `k3s-agent-skill:` skill
  entity.
- `.github/workflows/tag-on-merge.yml` — CalVer tag + `CHANGELOG/` on merge.
- `README.md` — this user overview.

## Related

- Owning skill: `/charly-infrastructure:k3s-agent` — the candy and its checks.
- Control plane: `/charly-infrastructure:k3s-server` — the server this agent joins.
- Base binary: `/charly-infrastructure:k3s` — installs the `k3s` binary.
- Cluster probing: `/charly-kubernetes:check-k8s` — the `kube:` check verb.
- [`opencharly/charly`](https://github.com/opencharly/charly) — the charly CLI and image builder.
- [`opencharly/opencharly`](https://github.com/opencharly/opencharly) — the umbrella.
