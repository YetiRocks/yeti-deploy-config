<p align="center">
  <img src="https://cdn.prod.website-files.com/68e09cef90d613c94c3671c0/697e805a9246c7e090054706_logo_horizontal_grey.png" alt="Yeti" width="200" />
</p>

---

# Yeti Deploy Config

[![Yeti](https://img.shields.io/badge/Yeti-Deploy-blue)](https://yetirocks.com)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Deploy](https://img.shields.io/badge/CI-GitHub_Actions-orange)](https://github.com/yetirocks/yeti-deploy-config/actions)

> **[Yeti](https://yetirocks.com)** — The Performance Platform for Agent-Driven Development.

Deployment configuration for a Yeti instance. Push to `main` to deploy.

## How It Works

1. **yeti-core** is Yeti's precompiled binary — it compiles and runs your applications. Hosted on public Object Storage and pulled by the GitHub Action.
2. This repo defines your server configuration (`yeti-config.yaml`) and lists the application repos to deploy.
3. The GitHub Action packages everything and deploys to your server.

## Setup

### 1. Create a Linux VM

Any Linux VM works. Minimum: 1GB RAM, 1 CPU. Recommended: 2+ CPU, 4GB+ RAM.

### 2. Add GitHub Secrets

| Secret | Value |
|--------|-------|
| `DEPLOY_HOST` | Your server's IP address |
| `DEPLOY_SSH_KEY` | SSH private key with root access to the server |

### 3. Configure and Deploy

Edit `yeti-config.yaml`, push to `main`, and the workflow handles the rest.

```bash
git add -A && git commit -m "deploy" && git push
```

To set up multiple environments, copy `.github/workflows/deploy.yml`, change the branch trigger, and use different secrets.

### 4. Verify

```bash
curl -sk https://YOUR_SERVER_IP/health
```

## Configuration

The `yeti-config.yaml` file contains server settings and the list of applications to deploy. Only include what you want to override — everything else uses sensible defaults.

```yaml
environment: production
rootDirectory: /opt/yeti
http:
  port: 443
storage:
  mode: embedded
logging:
  level: info
tls:
  autoGenerate: true

# Git repos cloned and deployed by the GitHub Action
applications:
  - https://github.com/your-org/your-app
```

| Field | Description |
|-------|-------------|
| `environment` | `production` disables dev features. `development` enables wide-open CORS and debug logging. |
| `rootDirectory` | Base directory on the server. Data, certs, cache, and apps live under this path. |
| `http.port` | HTTPS port. Yeti always serves over TLS. |
| `storage.mode` | `embedded` uses local RocksDB. `cluster` uses distributed TiKV. |
| `logging.level` | Minimum log level. `info` recommended for production. |
| `tls.autoGenerate` | Generates self-signed certs on first run. Set `false` and provide `tls.privateKey` / `tls.certificate` for CA-signed certs. |
| `applications` | Git repo URLs cloned by the deploy Action. Public repos work out of the box. For private repos, add a deploy key or enable Actions access. |

For the full server configuration reference, see the [documentation](https://yetirocks.com/documentation/reference/server-config.html).

## Deployment Pipeline

The GitHub Actions workflow (`.github/workflows/deploy.yml`) runs on every push to `main`:

1. Downloads the latest `yeti` binary from Object Storage
2. Clones each application repo listed in `yeti-config.yaml`
3. Packages binary, config, and applications into a tarball
4. SCPs the tarball to the server
5. Extracts, installs the systemd service, and starts Yeti
6. Waits for the health check to pass

## Server Management

```bash
ssh root@YOUR_SERVER_IP

systemctl status yeti          # check status
systemctl restart yeti         # restart
journalctl -u yeti -f          # follow logs

curl -sk https://localhost/health
```

## Project Structure

```
yeti-deploy-config/
├── .github/
│   └── workflows/
│       └── deploy.yml         # CI/CD pipeline
├── yeti-config.yaml           # Server config + app list
└── README.md
```

---

Built with [Yeti](https://yetirocks.com) | The Performance Platform for Agent-Driven Development
