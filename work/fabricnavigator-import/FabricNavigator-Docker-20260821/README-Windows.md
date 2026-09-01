# FabricNavigator for Windows Docker Desktop

The installer contains a self-contained Linux/amd64 Docker application and the Windows host updater. Production users, password hashes, credentials, token keys, device assignments, approved host keys, TLS private keys, logs, and machine identities must not be included in a published image.

## Requirements and installation

- Windows 10/11 x64
- Docker Desktop using Linux containers and Docker Compose v2
- Network access from the Docker Desktop VM to the managed devices

Extract `FabricNavigator-Installer-VERSION.zip` to a permanent directory, start Docker Desktop, and run:

```powershell
Set-ExecutionPolicy -Scope Process Bypass
.\Import-FabricNavigator.ps1 -GitHubRepository OWNER/FabricNavigator
```

FabricNavigator is then available at <https://localhost:8443/>. The first start creates a self-signed TLS certificate. The installer also creates the **FabricNavigator Updater** scheduled task, which performs validated updates and applies routes configured under **Administration → System** to the Windows Docker host. The web application itself has no Docker socket access.

## GitHub updates

Under **Administration → Updates**, store a fine-grained GitHub token restricted to the FabricNavigator repository with **Contents: Read-only**. Releases use tags such as `v26.08.10.109` and contain `FabricNavigator-Update-26.08.10.109.zip`.

The updater independently validates repository, version, and GitHub's SHA-256 digest before importing the image and restarting the service. A failed health check restores the previous runtime configuration. Persistent data remains intact.

## Persistent data and operations

Compose uses `fabricnavigator_security`, `fabricnavigator_devices`, and `fabricnavigator_tls`. Only `docker compose down --volumes` deletes them.

The legacy-compatible `fabricnavigator_security` volume name now mounts at `/opt/fabricnavigator/data`; existing credentials and settings are migrated without recreating the volume.

```powershell
docker compose ps
docker compose logs --tail 200 fabricnavigator
docker compose restart fabricnavigator
docker compose down
```

`NET_ADMIN` is required for the container's outbound nftables policy; the container is not privileged. Allow TCP 8443 in Windows Firewall when LAN access is required.
