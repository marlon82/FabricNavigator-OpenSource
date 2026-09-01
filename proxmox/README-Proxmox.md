# FabricNavigator Proxmox VM Template

The builder creates an Ubuntu 24.04 LTS Cloud-Init template on a Proxmox VE node. The FabricNavigator release is embedded in the template. Docker Engine, Compose, QEMU Guest Agent, FabricNavigator, and the privileged Linux updater are configured automatically during the first boot of a clone. Static routes configured in FabricNavigator are applied to the Ubuntu Docker host rather than the container.

## Requirements

- Proxmox VE 8 or 9
- Internet access from the Proxmox node and future VM
- VM disk storage such as `local-lvm` and a bridge such as `vmbr0`
- About 4 GB of temporary free space
- A fine-grained GitHub token restricted to the private repository with **Contents: Read-only**, unless a local installer ZIP is used

The builder does not install or remove Proxmox packages. It uses the tools already supplied by Proxmox and aborts if required components are missing. The GitHub token is used only to download the release and is never copied into the image or template.

## Create the template

Download these release assets:

- `FabricNavigator-Proxmox-Template-Builder-26.08.10.116.zip`
- `SHA256SUMS-Proxmox-26.08.10.109`

Copy them to the Proxmox host and prepare the builder:

```powershell
scp .\FabricNavigator-Proxmox-Template-Builder-26.08.10.116.zip root@PROXMOX-IP:/root/
scp .\SHA256SUMS-Proxmox-26.08.10.109 root@PROXMOX-IP:/root/
```

```bash
cd /root
sha256sum --check SHA256SUMS-Proxmox-26.08.10.109
unzip FabricNavigator-Proxmox-Template-Builder-26.08.10.116.zip -d /root
# Alternative when unzip is unavailable:
# python3 -m zipfile -e FabricNavigator-Proxmox-Template-Builder-26.08.10.116.zip /root
cd /root/FabricNavigator-Proxmox-26.08.10.109
chmod +x create-fabricnavigator-template.sh
```

Store the read-only token as a single line and run the builder:

```bash
chmod 600 /root/github-update-token.txt
./create-fabricnavigator-template.sh \
  --token-file /root/github-update-token.txt \
  --vmid 9000 \
  --storage local-lvm \
  --bridge vmbr0
```

Alternatively, avoid a token during the build by using the downloaded installer:

```bash
./create-fabricnavigator-template.sh \
  --installer /root/FabricNavigator-Installer-26.08.10.109.zip \
  --vmid 9000 \
  --storage local-lvm \
  --bridge vmbr0
```

Existing VM IDs are never overwritten. Run `./create-fabricnavigator-template.sh --help` for all options.

## SSH username and password access

The template enables password authentication for the Cloud-Init user `fabricnavigator`, but does not embed a default password. Supply an initial password from a protected file:

```bash
printf '%s' 'USE-A-LONG-UNIQUE-PASSWORD' > /root/fabricnavigator-vm-password.txt
chmod 600 /root/fabricnavigator-vm-password.txt
./create-fabricnavigator-template.sh \
  --token-file /root/github-update-token.txt \
  --ssh-user fabricnavigator \
  --ssh-password-file /root/fabricnavigator-vm-password.txt \
  --vmid 9000 --storage local-lvm --bridge vmbr0
rm -f /root/fabricnavigator-vm-password.txt
```

Without `--ssh-password-file`, configure the user and password or SSH public key in the clone's **Cloud-Init** settings. Root password login remains disabled.

## Create a VM from the template

1. Select `fabricnavigator-ubuntu-template` in Proxmox.
2. Create a **Full Clone**.
3. Configure the Cloud-Init user, password or SSH key, and IP settings.
4. Start the VM.
5. Monitor first boot with `journalctl -u fabricnavigator-firstboot -f` if required.
6. Open `https://VM-IP:8443/`.

Recommended resources are 2 vCPU, 8 GB RAM, and a 30 GB disk.

## Enable private updates

The builder token is deliberately excluded from the VM template. On the final clone, install a read-only token without printing it:

```bash
sudo fabricnavigator-token /path/github-update-token.txt
rm -f /path/github-update-token.txt
```

Update checks, release notes, and administrator-approved installation are then available under **Administration → Updates**. The Linux updater validates the release version and GitHub SHA-256 digest, restarts Compose, performs a health check, and preserves persistent volumes.

## Troubleshooting

A GitHub `404 Not Found` for a private repository normally means the token cannot see the repository. Create a fine-grained token with resource owner `marlon82`, repository access limited to `FabricNavigator`, and **Contents: Read-only**. The token file must contain only the token, without quotes, `Token=`, or extra whitespace.

Connectivity checks:

```bash
getent ahostsv4 api.github.com
curl --ipv4 --http1.1 --connect-timeout 15 --max-time 30 --fail --show-error --location --output /dev/null https://api.github.com/
curl --ipv4 --http1.1 --connect-timeout 15 --max-time 30 --fail --show-error --location --output /dev/null https://cloud-images.ubuntu.com/noble/current/SHA256SUMS
```

Operations and diagnostics:

```bash
sudo docker compose --project-directory /opt/fabricnavigator ps
sudo docker compose --project-directory /opt/fabricnavigator logs --tail 200 fabricnavigator
sudo systemctl status fabricnavigator-firstboot fabricnavigator-updater
sudo journalctl -u fabricnavigator-updater -n 100
```

First boot is complete when `/var/lib/fabricnavigator-firstboot.complete` exists. Do not delete Docker volumes if users, devices, credentials, TLS data, and preferences must be retained.
