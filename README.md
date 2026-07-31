# Proxmox Homelab & Infrastructure

> **Overview:** A custom home server/VE built from repurposed Hardware to learn Linuxsystem administration, container hosting, and network security

## Project Goals

* **Hands on Learning:** Skip most theory to manage real hardware and vms.
* **Smart Storage:** Split OS, app data, and backups across several dedicated physical drives.
* **First comes Documentation:** Log every setup step, configuration, and fix on GitHub.

## Server Hardware

* **Power:** 250W OEM Fujitsu Small Form Factor PSU.
* **CPU:**  Intel Core i7 4790 (4 Cores / 8 Threads @ 3.6 Ghz)
* **Memory:** 32 GB DDR3 1600 Mhz
* **Storage:**
* **Drive 1:** Samsung 840 EVO SSD (~500GB),(512MB LPDDR3 onboard DRAM Cache) - Planned for Proxmox OS ('ext4')
* **Drive 2:** Innovation IT SSD (~256GB),(No onboard DRAM cache) - Planned for VM storage ('LVM-Thin')
* **Drive 3:** Toshiba 7200rpm HDD (~500GB) - Planned for Cloud Storage, Backups, and ISOs.
* **GPU:** No dedicated desktop GPU (Not needed for a text based environment)
* **Network Interface:** Integrated Gigabit Ethernet.