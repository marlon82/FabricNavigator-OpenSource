# Publishing FabricNavigator on GitHub

Publish FabricNavigator primarily as a ready-to-run Docker application. The Windows installer ZIP contains the same container plus the privileged host updater. The bundled Extreme EDM application and `snmp4jdm` library are removed from the FabricNavigator runtime; SNMP communication uses Apache-2.0 licensed SNMP4J. Keep the repository private until redistribution rights for ACLI and optional Extreme product images have been confirmed.

## Prepare the repository

1. Commit source code, scripts, and documentation. Generated archives, runtime state, and `.env` remain excluded by `.gitignore`.
2. Set `FABRICNAVIGATOR_GITHUB_REPOSITORY=OWNER/FabricNavigator` on installation hosts.
3. For a private repository, create a fine-grained token restricted to this repository with **Contents: Read-only**. Never place it in the image, `.env`, JavaScript, Git, or release artifacts.

## Build a release

```powershell
.\Build-FabricNavigator-Release.ps1 `
  -Version 26.08.10.109 `
  -SourceImage fabricnavigator:26.08.10.109
```

The script creates the installer ZIP, update ZIP, checksums, and release notes under `dist`.

## Publish the GitHub release

1. Create tag `vVERSION`.
2. Publish a normal GitHub release, not a draft.
3. Use the matching English release notes as the description.
4. Attach the update ZIP, installer ZIP, Proxmox template builder ZIP, and checksum files.
5. Verify that GitHub exposes a `sha256:` digest for the update asset through the Releases API.

The Admin UI reads release metadata from GitHub. The host updater downloads and validates the asset independently before changing the running installation. Persistent volumes are not deleted or replaced.
