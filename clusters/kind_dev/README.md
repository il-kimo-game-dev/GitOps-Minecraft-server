# kind_dev cluster

Flux definitions for a local [kind](https://kind.sigs.k8s.io/) cluster running
the Minecraft server. It deploys only the `minecraft` app from
`apps/base/minecraft` (via `apps/kind_dev`).

## Exposing the Minecraft server to your machine

Kind nodes are Docker containers, so the server's NodePort is not reachable
from the host by default. The port mapping is set in the kind cluster config,
[`kind-config.yaml`](kind-config.yaml), which is not read by Flux:

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
kind create cluster --name kind-dev --config clusters/kind_dev/kind-config.yaml
```

Then connect your Minecraft client to `localhost:25565`.

Notes:

- Port mappings can only be set when the cluster is created. To change them,
  delete and recreate the cluster (`kind delete cluster --name kind-dev`).
- `containerPort` must match `minecraftServer.nodePort` in `release.yaml`
  (currently `30565`). If you enable the voice chat port, map its UDP nodePort
  the same way.
