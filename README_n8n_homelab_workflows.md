# n8n Homelab Workflow Templates (10)
Below are 10 n8n workflow templates created for your homelab stack: Proxmox, Portainer, Glance, Vaultwarden, n8n, Pi-hole, and social/portfolio automation.

Files are in this directory: `/mnt/data/n8n_workflows_homelab/`

## What you will find
1. 01_Proxmox_-_VM_Status_Alert_(single_VM_template).json
2. 02_Portainer_-_Container_Watchdog_(single_container_template).json
3. 03_Glance_Dashboard_-_Auto_Update_(Portainer_->_Glance).json
4. 04_Vaultwarden_-_Backup_(SSH_+_upload_placeholder).json
5. 05_Failed_Login_Alerts_(SSH_log_tail_->_Discord).json
6. 06_Pi-hole_-_Blocklist_Sync_(weekly).json
7. 07_Pi-hole_-_DNS_Health_Watchdog.json
8. 08_GitHub_->_Portfolio_Auto-Update_(Webhook_->_Notion_+_Social).json
9. 09_Homelab_Status_Aggregator_(Proxmox_+_Portainer_+_Pi-hole).json
10. 10_Cloudflare_Tunnel_Watchdog.json

## How to import
1. Open your n8n instance.
2. Click on **Workflows** -> **Import** -> choose the desired JSON file.
3. After import, **edit each workflow** to:
   - Update placeholder variables (hosts, IDs, URLs).
   - Configure credentials (SSH, API keys).
   - Optionally set workflow to active.

## Common Credential & Connectivity Setup (per workflow)
Below are step-by-step notes and credential recommendations for each workflow.

### 1) Proxmox - VM Status Alert (single VM template)
- Purpose: Check a single VM and start it if not running.
- Required:
  - Proxmox API token (preferred) or username/password.
  - Make sure Proxmox API is reachable from n8n (open port 8006).
- How to use:
  - After import, add a **Set** node before "Check VM Status" to set these JSON fields:
    - `proxmoxHost` (e.g. proxmox.example.com:8006)
    - `nodeName` (Proxmox node)
    - `vmid` (VM numeric id)
  - Configure the "Check VM Status" and "Start VM" HTTP Request nodes to use your Proxmox API token.
  - If you want to check multiple VMs, duplicate the block and/or use SplitInBatches.

### 2) Portainer - Container Watchdog (single container template)
- Purpose: Restart container if it’s not running.
- Required:
  - Portainer host URL, endpointId (find in Portainer API), containerId or name.
  - API key for Portainer (use HTTP header `Authorization: Bearer <API_KEY>`).
- How to use:
  - Set `portainerHost`, `endpointId`, `containerId`.
  - Add Authorization header under each HTTP Request node (or create an n8n credential).

### 3) Glance Dashboard - Auto Update
- Purpose: Pull container list from Portainer and POST a payload to Glance.
- Required:
  - Portainer API access.
  - Glance dashboard "API" endpoint (this may require a simple HTTP endpoint you host that accepts JSON; adjust according to your Glance config).
- How to use:
  - Set `portainerHost`, `endpointId`, and `glanceUrl`.
  - Tweak "Build Glance Payload" to match the structure your Glance instance accepts.

### 4) Vaultwarden - Backup
- Purpose: Create a daily DB backup and upload it to remote storage.
- Required:
  - SSH access to the host running Vaultwarden (key or password) configured in n8n credentials for the SSH node.
  - Upload destination (S3 presigned URL, SFTP server, or any HTTP endpoint that accepts file uploads).
- How to use:
  - Update the SSH command if your Vaultwarden DB path differs (the template assumes sqlite in `/data/vaultwarden/db.sqlite3`).
  - Set `uploadUrl` (S3 presigned URL) or replace the final HTTP Request node with an SFTP node or S3 node (if you have an S3 credential configured in n8n).

### 5) Failed Login Alerts
- Purpose: Tail auth logs and alert on suspicious entries (failed password).
- Required:
  - SSH credentials to the server(s) you want to monitor.
  - Discord webhook (or another alert endpoint).
- How to use:
  - Configure SSH credential in SSH node.
  - Set `discordWebhookUrl` in a Set node or environment variable.
  - Consider adding IP auto-blocking: call an API (firewall device, Pi-hole, or iptables via SSH) when a threshold is reached.

### 6) Pi-hole Blocklist Sync
- Purpose: Fetch blocklist and push to Pi-hole.
- Required:
  - Pi-hole admin interface reachable from n8n.
  - If Pi-hole protected by password, you may need to enable API token or call the gravity update method.
- How to use:
  - Replace `blocklistUrl` with your source(s).
  - Adjust the final HTTP Request to match Pi-hole’s method for adding or updating lists; you can also use GitHub Actions or rclone+gravity for heavy duty lists.

### 7) Pi-hole DNS Health Watchdog
- Purpose: Watch for spikes in blocked domains and alert.
- Required:
  - Pi-hole host URL.
  - Discord webhook or other notifier.
- How to use:
  - Adjust the threshold in the "Spike?" IF node.
  - Optionally send a report with top blocked domains (Pi-hole has endpoints for top clients/domains).

### 8) GitHub -> Portfolio Auto-Update (Webhook -> Notion + Social)
- Purpose: On GitHub push, update Notion project and post to socials.
- Required:
  - GitHub webhook (set repo Settings -> Webhooks -> Payload URL: `https://<your-n8n-host>/webhook/webhook-github-portfolio`)
  - Notion integration token and pageId
  - Social posting: Buffer API or direct Twitter/LinkedIn API credentials (the template uses a placeholder `socialApiUrl`)
- How to use:
  - Import workflow, then configure your GitHub webhook to hit your n8n webhook URL.
  - Create Notion integration and share the project page with it; set `notionApiUrl` and `notionPageId`.
  - For social posting, either plug into Buffer (recommended) or implement the platform-specific OAuth.

### 9) Homelab Status Aggregator
- Purpose: Aggregate Proxmox, Portainer and Pi-hole snapshots and store/send to Glance or storage.
- Required:
  - API access to each service (Proxmox API token, Portainer API, Pi-hole).
  - A storage endpoint (could be a small webhook on a local webserver, a simple Gist, or Glance API).
- How to use:
  - Update hosts and credentials.
  - Adapt the "Build Status JSON" function to capture only the fields you care about.
  - Use a storage endpoint (e.g., your Glance instance) to display. Alternatively, store to an S3 bucket or database.

### 10) Cloudflare Tunnel Watchdog
- Purpose: Check your tunnel URL, attempt restart via SSH if down, and notify via Discord.
- Required:
  - SSH credentials to the host running `cloudflared`.
  - Discord webhook.
  - Public tunnel URL to check (`tunnelUrl`).
- How to use:
  - Update `tunnelUrl` and set SSH credential for the SSH node.
  - If your host requires `sudo` for restarting cloudflared, ensure the SSH user has passwordless sudo or use the appropriate restart command/container restart command (e.g., docker restart cloudflared).

## Tips & Next steps
- **Credentials**: Create credentials in n8n (SSH, HTTP headers for API tokens, OAuth for Notion/Twitter if you want). When a node requires credentials, open it and link the proper credential.
- **Testing**: Use manual triggers or set short cron expressions while testing.
- **Scaling**: For multiple VMs/containers, you can add a `SplitInBatches` node or create a "List" function to produce items and let the flow iterate.
- **Security**: Keep API tokens and SSH keys secure. Use environment variables or n8n credentials rather than hardcoding secrets in nodes.

