# Installing FabricNavigator

This guide covers the first installation on Windows with Docker Desktop and secure updates from the private GitHub repository. For Proxmox VE, use the [Ubuntu template builder guide](proxmox/README-Proxmox.md). For VMware ESXi 7/8, use the [OVA appliance installer guide](esxi/README-ESXi.md).

## Requirements

- Windows 10 or Windows 11 x64
- Docker Desktop using Linux containers and Docker Compose v2
- At least 2 GB of free disk space
- Network access from the Windows host and Docker Desktop VM to the managed devices
- Local administrator rights for the host updater and optional firewall changes

Verify Docker before installation:

```powershell
docker version
docker compose version
```

## Install

1. Download `FabricNavigator-Installer-26.08.10.109.zip` and `SHA256SUMS` from the [v26.08.10.109 release](https://github.com/marlon82/FabricNavigator/releases/tag/v26.08.10.109). Sign in to GitHub because the repository is private.
2. Verify the package:

   ```powershell
   Get-FileHash -Algorithm SHA256 .\FabricNavigator-Installer-26.08.10.109.zip
   Get-Content .\SHA256SUMS
   ```

   Do not install the package if the hashes differ.

3. Extract it to a permanent directory:

   ```powershell
   New-Item -ItemType Directory -Force C:\FabricNavigator
   Expand-Archive .\FabricNavigator-Installer-26.08.10.109.zip C:\FabricNavigator -Force
   Get-ChildItem C:\FabricNavigator -Recurse | Unblock-File
   Set-Location C:\FabricNavigator
   ```

4. Start the installer:

   ```powershell
   Set-ExecutionPolicy -Scope Process Bypass
   .\Import-FabricNavigator.ps1 -GitHubRepository marlon82/FabricNavigator
   ```

The installer imports the Docker image, starts Compose, creates the **FabricNavigator Updater** scheduled task, and configures support for host-level static routes. Use `-SkipUpdater` only if automatic updates and host route management are not required.

## First start

Open <https://localhost:8443/> locally or `https://WINDOWS-HOST-IP:8443/` from the LAN. The first-run wizard creates the initial administrator and collects optional SNMP, SSH, and WebView credentials before opening topology discovery.

FabricNavigator initially creates a self-signed certificate. For production, upload a PKCS#12 certificate under **Administration → System** whose DNS name or IP address matches the server.

If the Windows firewall blocks access from the LAN, run as administrator:

```powershell
New-NetFirewallRule -DisplayName "FabricNavigator HTTPS" -Direction Inbound -Protocol TCP -LocalPort 8443 -Action Allow
```

Restrict the firewall rule to the required private networks.

## Private GitHub updates

Create a fine-grained personal access token restricted to `marlon82/FabricNavigator` with **Contents: Read-only**. Store or replace it under **Administration → Updates**. The token is sent once over HTTPS, stored by the privileged host updater with restricted permissions, never displayed again, and never logged.

The host updater independently validates the repository, version, and SHA-256 digest, imports the new image, restarts FabricNavigator, and performs a health check. If the health check fails, it restores the previous runtime configuration. Persistent users, credentials, devices, host keys, certificates, and preferences remain intact.

## Operations

Run these commands from the installation directory:

```powershell
docker compose ps
docker compose logs --tail 200 fabricnavigator
docker compose restart fabricnavigator
docker compose down
docker compose up -d
```

Do not use `docker compose down --volumes` unless you intentionally want to permanently delete FabricNavigator data.

Back up these volumes before major changes: `fabricnavigator_security`, `fabricnavigator_devices`, and `fabricnavigator_tls`. Treat backups as secrets.

For update compatibility, the existing `fabricnavigator_security` volume name is retained, but its application data is mounted at `/opt/fabricnavigator/data`. Older installations are migrated automatically when the container starts.

## Troubleshooting

- **Docker not found:** Start Docker Desktop and wait until it reports that it is running.
- **PowerShell blocks the script:** Run `Set-ExecutionPolicy -Scope Process Bypass` and `Get-ChildItem . -Recurse | Unblock-File`.
- **Site unavailable:** Check `docker compose ps`, `docker compose logs --tail 200 fabricnavigator`, and `Test-NetConnection localhost -Port 8443`.
- **Update check returns 401/404:** Verify repository access and replace the fine-grained token with one that has `Contents: Read-only` access to this private repository.
- **Updater task is not running:** Inspect the **FabricNavigator Updater** task in Windows Task Scheduler or rerun the installer without `-SkipUpdater`.
