# Runbook 04: Cloudflare Tunnel & Domain Integration

* **Objective:** Securely expose your local Docker-hosted Nextcloud instance to the internet via an encrypted Cloudflare Tunnel (`cloudflared`) without opening router ports.

## Cloudflare Dashboard & Zero Trust Setup
* Check if your domain is active on your Cloudflare account (if purchased on Cloudflare) if not ensure that your paid domain is added to your Cloudflare account.

### Create the Tunnel
* Log into the **Cloudflare Dashboard** -> **Zero Trust**.
* Choose **Cloudflared** as the connector type and name your tunnel (e.g cloud-tunnel).
* Choose Docker as the Server's operating system.
* Copy your unique **Tunnel Token** from the installation command snippet provided.

### Run it inside your docker-compose.yml

```bash
cd /opt/nextcloud
nano docker compose yml
```
* Paste the updated YAML file:
```yaml

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

  cloudflared:
    image: cloudflare/cloudflared:latest
    restart: always
    command: tunnel --no-autoupdate run --token YOUR_ACTUAL_TUNNEL_TOKEN_HERE
    depends_on:
      - nextcloud

volumes:
  db-data:
```

### Spin up the updated stack in the background
```bash 
docker compose up -d
```
* Verify the Tunnel Status
```bash
docker compose ps
```

### Configure the Public Hostname
* At Zero Trust head to your tunnel's **Public Hostname/Published application routes** tab, and add a route.

* At **Hostname:**
* Add your Subdomain: `cloud` (or your choice).
* Domain: Choose your paid domain -> `civtact.com`
* Path: (you can leave empty)
* At **Service:**
* Type: `HTTP`
* URL: `nextcloud:80`

### Fix the Untrusted Domain Error
* Docker often auto-creates host bind-mounts as `root`,locking out the Nextcloud web service (33:33). Always create and pre-permission your folders *before* bringing up the stack for the first time:
```bash
sudo mkdir -p /opt/nextcloud/html /mnt/nextcloud-storage
sudo chown -R 33:33 /opt/nextcloud/html /mnt/nextcloud-storage
sudo chmod -R 755 /opt/nextcloud/html
```
* Handling the `CAN_INSTALL` Lock (Clean Reinstall):
```bash
sudo touch /opt/nextcloud/html/config/CAN_INSTALL
sudo chown 33:33 /opt/nextcloud/html/config/CAN_INSTALL
sudo chmod 644 /opt/nextcloud/html/config/CAN_INSTALL
```
* restart docker compose.
* Fill in the Admin login and the MySQL/MariaDB Databank credentials.
* For Databank-Host use `db` (or your choice).
* For Databank-Password use the same as in your docker-compose.yml file.
* Fix MIME-Type Migration (Entirely optional):
```bash
docker exec -it -u 33 nextcloud-nextcloud-1 php occ maintenance:repair --include-expensive
```
* Add this code to your config.php file to fix possible freezing at the login screen:
```yaml
'overwritehost' => 'cloud.civtact.com',
  'overwriteprotocol' => 'https',
  ```
* Restart docker:
```bash
docker compose restart
```