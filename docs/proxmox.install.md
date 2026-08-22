# Runbook 01: Proxmox VE Installation

## Installation Notes

* **Tool Used to Flash USB:** Rufus
* **OS Drive/Target Drive:** Samsung 840 EVO SSD (~500GB)

* **Network Setup:**
  * Static IP assigned: 192.168.2.218
  * CIDR notation: 24
  * Gateway: 192.168.2.1
  * DNS: 1.1.1.1
  * Hostname(FQND): proxmox-node.local

* **Filesystem:** ext4 (Chosen to protect consumer SSD endurance over ZFS)

* **Initial Hiccup:** The first installation attempt failed during package extraction (likely a minor packet/USB write error). Re-running the installation wizard completed successfully on the second try.

## Next Steps
* Access the web UI at `https://192.168.2.218:8006`
* Configure the `pve-no-subscription` repository.
