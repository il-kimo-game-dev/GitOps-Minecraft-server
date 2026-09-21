# Local mods for the kind_dev cluster

Put the mod jars you want to test on the server in this directory. Everything
here except this README is git-ignored.

The directory is mounted into the kind node (see `../kind_dev-config.yaml`) and
from there into the Minecraft pod at `/mods`, which the itzg image copies into
`/data/mods` on start. Fabric loads mods at startup only, so after changing a
jar restart the server:

```sh
kubectl -n minecraft rollout restart deploy/minecraft-server
```

Copy only the remapped mod jar (for example
`build/libs/<name>-<version>.jar`), not the `-sources.jar`.

Dependencies such as Fabric API and Fabric Language Kotlin are installed from
Modrinth by `apps/kind_dev/minecraft-values.yaml`; do not put them here.
