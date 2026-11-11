# MOSIP Asia Standardized GitHub Actions

Centralized repository containing reusable GitHub Composite Actions for CI/CD deployment pipelines across the `mosip-asia` organization.

---

## 📦 Available Actions

| Action | Description | Usage |
|---|---|---|
| [`preflight-check`](./preflight-check) | Validates environment variables and deployment secrets before execution | `uses: mosip-asia/actions/preflight-check@v1` |
| [`netbird-connect`](./netbird-connect) | Connects the GitHub runner to the self-hosted NetBird VPN overlay mesh | `uses: mosip-asia/actions/netbird-connect@v1` |
| [`ssh-setup`](./ssh-setup) | Configures private SSH keys and scans target host into `known_hosts` | `uses: mosip-asia/actions/ssh-setup@v1` |
| [`ssh-exec`](./ssh-exec) | Executes deployment scripts or commands on the remote target host over SSH | `uses: mosip-asia/actions/ssh-exec@v1` |

---

## 🚀 Quickstart & Usage Example

Use these actions together in your continuous deployment (`cd.yml`) workflow:

```yaml
name: 🚀 Deployment

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: dev
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: 🔍 Validate Deployment Secrets & Variables
        uses: mosip-asia/actions/preflight-check@v1
        with:
          environment: dev
          netbird-setup-key: ${{ secrets.NETBIRD_SETUP_KEY }}
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
          netbird-management-url: ${{ vars.NETBIRD_MANAGEMENT_URL }}
          server-host: ${{ vars.SERVER_HOST }}
          server-user: ${{ vars.SERVER_USER }}
          server-project-dir: ${{ vars.SERVER_PROJECT_DIR }}

      - name: 🌐 Connect to NetBird VPN Mesh
        uses: mosip-asia/actions/netbird-connect@v1
        with:
          management-url: ${{ vars.NETBIRD_MANAGEMENT_URL }}
          setup-key: ${{ secrets.NETBIRD_SETUP_KEY }}
          server-host: ${{ vars.SERVER_HOST }}

      - name: 🔑 Setup SSH Authentication
        uses: mosip-asia/actions/ssh-setup@v1
        with:
          ssh-key: ${{ secrets.SSH_PRIVATE_KEY }}
          server-host: ${{ vars.SERVER_HOST }}

      - name: 🚀 Remote Deploy via SSH
        uses: mosip-asia/actions/ssh-exec@v1
        with:
          server-host: ${{ vars.SERVER_HOST }}
          server-user: ${{ vars.SERVER_USER }}
          cmd: |
            cd ${{ vars.SERVER_PROJECT_DIR }}
            git fetch origin
            git checkout ${{ github.sha }}
            bash scripts/deploy.sh
```

---

## 🛠️ Actions Reference

### 1. `preflight-check`
Inspects required deployment secrets and variables early to fail fast with helpful diagnostic messages before attempting network handshakes.

**Inputs:**
- `environment` *(required)*: The GitHub environment name (`dev`, `uat`, `production`).
- `netbird-setup-key`: NetBird ephemeral setup key.
- `ssh-private-key`: SSH private deploy key.
- `netbird-management-url`: NetBird self-hosted management URL.
- `server-host`: Target host FQDN or IP on NetBird mesh.
- `server-user`: Deploy username on target host.
- `server-project-dir`: Project path on target host.

---

### 2. `netbird-connect`
Installs NetBird daemon on the GitHub runner, connects using the setup key, and verifies ping reachability to `server-host`.

**Inputs:**
- `management-url` *(required)*: NetBird management server URL.
- `setup-key` *(required)*: NetBird setup key.
- `server-host` *(required)*: Target host to verify reachability.
- `max-attempts` *(optional, default: `10`)*: Ping retry attempts.

---

### 3. `ssh-setup`
Configures `~/.ssh/id_ed25519` with strict permissions (`600`) and runs `ssh-keyscan` against `server-host`.

**Inputs:**
- `ssh-key` *(required)*: Private SSH key content.
- `server-host` *(required)*: Target server hostname/IP.

---

### 4. `ssh-exec`
Runs commands or multiline bash scripts on the remote host over SSH without host checking prompts.

**Inputs:**
- `server-host` *(required)*: Remote server host.
- `server-user` *(required)*: Remote user.
- `cmd` *(required)*: Command or script block to execute.
