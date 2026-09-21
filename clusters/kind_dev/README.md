# kind_dev cluster

Flux definitions for a local [kind](https://kind.sigs.k8s.io/) cluster running
the Minecraft server. It deploys only the `minecraft` app from
`apps/base/minecraft` (via `apps/kind_dev`).

## Exposing the Minecraft server to your machine

Kind nodes are Docker containers, so the server's NodePort is not reachable
from the host by default. The port mapping is set in the kind cluster config,
[`kind/kind_dev-config.yaml`](../../kind/kind_dev-config.yaml). It is kept outside `clusters/kind_dev/` on purpose: Flux syncs every YAML file in that directory, and a kind `Cluster` is not a Kubernetes resource:

```yaml
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30565   # nodePort in apps/base/minecraft/release.yaml
        hostPort: 25565        # port on your machine
        protocol: TCP
```

Create the cluster with it:

```sh
kind create cluster --name minecraft-dev \
  --config kind/kind_dev-config.yaml \
  --kubeconfig ~/.kube/clusters/kind/minecraft_dev/config
```

Then connect your Minecraft client to `localhost:25565`.

Notes:

- Port mappings can only be set when the cluster is created. To change them,
  delete and recreate the cluster (`kind delete cluster --name minecraft-dev --kubeconfig ~/.kube/clusters/kind/minecraft_dev/config`).
- `containerPort` must match `minecraftServer.nodePort` in `release.yaml`
  (currently `30565`). If you enable the voice chat port, map its UDP nodePort
  the same way.

## Testing a local mod

Jars placed in [`kind/mods/`](../../kind/mods/) (git-ignored, see its README)
are mounted into the kind node via `extraMounts` and from there into the
pod at `/mods`. Create the cluster from the repository root, because the
mount path in `kind/kind_dev-config.yaml` is relative. After copying or
rebuilding a jar, restart the server so Fabric reloads it:

```sh
kubectl -n minecraft rollout restart deploy/minecraft-server
```

Fabric API and Fabric Language Kotlin are installed from Modrinth by
`apps/kind_dev/minecraft-values.yaml`.
