# Runbook 03: Docker Host & Nextcloud Deployment

**Target OS:** Ubuntu Server (Version 26.04)
**Storage Environment:** Samsung SSD (~500GB)

## VM Creation & OS Installation in Proxmox

### Upload the ISO
* Download the official Ubuntu Server ISO at `https://ubuntu.com/download/server`
* In the Proxmox Web GUI, select your node, go to **local (proxmox-node)** -> **ISO Images** -> **Upload**,  and upload your Ubuntu ISO.

### Create a Directory for Nextcloud Storage
* Head to **Node** -> **Disks**, and under Disks to **Directory**.
* At Directory click **Create:Directory**
* Disk: `/dev/sda` (256gb Innovation IT SSD)
* Filesystem: `ext4`
* Name: **nextcloud-storage** (or your choice).

### Create the Virtual Machine
* Click **Create VM** in the top-right of the Proxmox UI.
***VM ID:** `100`(or your choice)
* **OS:** Select the uploaded Ubuntu Server ISO.
* **Systen:** Leave defaults.
* **Disks:**
* scsi0: Storage -> `local-lvm`, Disk size (GiB): **35** (This is the Disk your OS will function on).
* At **Disks** click **Add**, then add your `nextcloud-storage` Directory (it will display as `scsi1`).
* scsi1 Storage -> `nextcloud-storage`, Disk size (GiB): **220** (leave 10-15 GB of the Disk *unallocated* to avoid Conflicts with ext4).
* **CPU:** Cores -> Allocate 2-4 vCPUs.
* **Memory:** Assign 4-8GB RAM depending on container load.
* **Network:** Set bridge to `vmbr0` (for Telekom Speedport DHCP/static mapping).

### Ubuntu Server OS Installation
* **Boot:** Start VM 100 and complete the Ubuntu Server installation wizard.
* **SSH Access:** Note the VM's internal IP and test an SSH fom your main PC.
* **System Updates:** Run:
```bash
sudo apt update && sudo apt upgrade -y #sudo will ask you for your password before proceeding.
```

### Storage & Docker Setup
* Find the 220GB `scsi1` Nextcloud Drive via `lsblk` -> **sdb** (can vary by machine).
* Format the Disk with an ext4 filesystem (if not done yet):
```bash
sudo mkfs.ext4 /dev/sdb
```
* Create the mount Directory:
```bash
sudo mkdir -p /mnt/nextcloud-storage
```

### Configure Persistent Mounting (so the drive stays mounted after a reboot).
* Find the unique UUID of your `sdb` Drive:
```bash
sudo blkid /dev/sdb
```
* Open the system mount configuration file:
```bash
sudo nano /etc/fstab
```
* Implement the following line at the very bottom (replace your-uuid-here with your actual UUiD string from the blkid command):
```text
UUID=your-uuid-here /mnt/nextcloud-storage ext4 defaults 0 2 
```
* Save and Exit in Nano: Ctrl + O, Enter to save -> Ctrl + X to exit 
* Test it so it actually works:
```bash
sudo mount -a
```

## Installing Docker Engine & Docker Compose
* **Certificates & Keys:** Update your package list and install the required tools to safely add external repositories:
```bash
sudo apt update && sudo apt install -y ca-certificates curl gnupg
```
* Create the Keyrings directory
```bash
sudo mkdir -p /etc/apt/keyrings
```
* Add Docker's Official GPG Key
```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

### Set Up the Official Docker Repository
```bash 
echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Install Docker Engine & Compose
```bash
sudo apt update
sudo apt install -y docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
* Avoid **sudo** for Every Docker Command (Optional but recommended)
```bash
sudo groupadd docker
sudo usermod -aG docker $USER
```

### Create a Project Directory
```bash 
sudo mkdir -p /opt/nextcloud && sudo chown -R $USER:$USER /opt/nextcloud
```

### Create the Docker Compose File
```bash
cd /opt/nextcloud
nano docker-compose.yml
```
* **Docker Compose Configuration** (docker-compose.yml)
```yaml
version: '3.8'

services:
  db:
    image: mariadb:10.6
    restart: always
    command: --transaction-isolation=READ-COMMITTED --binlog-format=ROW
    volumes:
      - db-data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: random_root_password_change_me
      MYSQL_PASSWORD: nextcloud_password_change_me
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud

  nextcloud:
    image: nextcloud:latest
    restart: always
    ports:
      - 8080:80
    volumes:
      - /opt/nextcloud/html:/var/www/html
      - /mnt/nextcloud-storage:/var/www/html/data
    environment:
      MYSQL_PASSWORD: nextcloud_password_change_me
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      MYSQL_HOST: db
    depends_on:
      - db

volumes:
  db-data:
```
* Verify its running:
```bash
docker compose ps
```
* Hand ownership of the nextcloud-storage drive over to Nextcloud:
```bash
sudo chown -R 33:33 /mnt/nextcloud-storage
```

### Setting Up Background Jobs (Cron)
```bash
sudo crontab -e
```
* Add this line at the bottom of the nano file:
```bash
*/5 * * * * docker exec -u 33 nextcloud-nextcloud-1 php -f /var/www/html/cron.php > /dev/null 2>&1 # (nextcloud name can vary, check using 'docker ps')
```
