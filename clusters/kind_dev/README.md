# kind_dev cluster

Flux definitions for a local [kind](https://kind.sigs.k8s.io/) cluster running
only the Minecraft server (`apps/base/minecraft`, via `apps/kind_dev`).

## Table of contents

- [Create the cluster](#create-the-cluster)
- [Connect to the server](#connect-to-the-server)
- [Test a local mod](#test-a-local-mod)
- [Stop and restart](#stop-and-restart)

## Create the cluster

Run from the repository root, since paths in the kind config are relative:

```sh
kind create cluster --name minecraft-dev \
  --config kind/kind_dev-config.yaml \
  --kubeconfig ~/.kube/clusters/kind/minecraft_dev/config
```

The config lives in [`kind/`](../../kind/kind_dev-config.yaml), not here: Flux
syncs every YAML file under `clusters/kind_dev/`, and a kind `Cluster` is not a
Kubernetes resource.

Mounts and port mappings are fixed at creation. To change them, delete the
cluster (`kind delete cluster --name minecraft-dev --kubeconfig <same file>`)
and recreate it.

## Connect to the server

Kind nodes are containers, so the NodePort is not reachable from the host by
default. `extraPortMappings` maps host port `25565` to NodePort `30565`
(`nodePort` in `apps/base/minecraft/release.yaml`; keep them in sync). Connect
your client to `localhost:25565`.

## Test a local mod

Jars in [`kind/mods/`](../../kind/mods/) (git-ignored) are mounted into the
pod at `/mods` through `extraMounts`. Fabric loads mods at startup, so restart
after changing a jar:

```sh
kubectl -n minecraft rollout restart deploy/minecraft-server
```

Fabric API and Fabric Language Kotlin come from Modrinth
(`apps/kind_dev/minecraft-values.yaml`).

## Stop and restart

Stop the node container to free resources. The cluster, Flux and the world
data are kept:

```sh
docker stop minecraft-dev-control-plane
```

Start it again in the next session:

```sh
docker start minecraft-dev-control-plane
```

The API server's host port can change after a restart. If `kubectl` can no
longer connect, refresh the kubeconfig:

```sh
kind export kubeconfig --name minecraft-dev \
  --kubeconfig ~/.kube/clusters/kind/minecraft_dev/config
```

`kind delete cluster` (see above) removes everything, including the world.
