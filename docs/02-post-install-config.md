# Runbook 02: Post-Installation & Repository Configuration

* **Node:** `proxmox-node.local` (`192.168.2.218`)

## Objective
* Disable the paid Proxmox enterprise repositories.
* Enable the free `pve-no-subscription` repository to allow system updates.
* Run initial system package upgrades.
* (Optional but Recommended) Supress the "No valid subscriptions" screen.

## Steps Performed

### Repository Fix
* Logged into the Proxmox Web UI (`https://192.168.2.218:8008`)
* Navigated to **Node (`proxmox-node.local`)** -> **Updates** -> **Repositories**.
* Disabled the `pve-enterprise`repository.
* Added the `pve-no-subscription`repository.

### System Update
* Opened the **Shell** directly inside the proxmox web UI
* Executed the package update and upgrade commands:
```bash
apt update && apt upgrade -y