# n8n Local Deployment with ngrok

Boilerplate for running a self-hosted [n8n](https://n8n.io) instance locally using Docker, with [ngrok](https://ngrok.com) to expose webhooks to the internet.

## How It Works

Two Docker containers run on a shared bridge network:

- **n8n** — the workflow automation engine, storing all data in a persistent Docker volume
- **ngrok** — creates a secure tunnel from a public domain to n8n, enabling external services to reach your webhooks

ngrok communicates with n8n using Docker's internal DNS (`n8n:5678`), not localhost. `5678` is n8n's default port — if you override it with `N8N_PORT`, update `addr` in `ngrok.yml` to match.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/) and Docker Compose
- A free [ngrok account](https://dashboard.ngrok.com/signup) with an auth token and a reserved free domain

---

## Setup

### 1. Clone the repo

```bash
git clone https://github.com/stolen-smile/n8n-ngrok-boilerplate.git
cd n8n-ngrok-boilerplate
```

### 2. Create `.env`

Create a `.env` file in the project root:

```env
URL=https://your-domain.ngrok-free.app
TIMEZONE=America/New_York
NGROK_TOKEN=your_ngrok_auth_token
```

| Variable | Description |
|---|---|
| `URL` | Your ngrok public domain (must match `domain` in `ngrok.yml` exactly) |
| `TIMEZONE` | Your local timezone ([TZ identifier](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)) |
| `NGROK_TOKEN` | Auth token from your [ngrok dashboard](https://dashboard.ngrok.com/get-started/your-authtoken) |

### 3. Create `ngrok.yml`

Create a `ngrok.yml` file in the project root:

```yaml
version: 2
log_level: debug
tunnels:
  n8n:
    proto: http
    addr: n8n:5678
    domain: your-domain.ngrok-free.app
```

> The `domain` value must exactly match the `URL` variable in `.env` (without the `https://` prefix).

To get a free static domain, go to **ngrok Dashboard → Cloud Edge → Domains**.

### 4. Start the containers

```bash
docker-compose up -d
```

### 5. Access n8n

- **Externally (webhooks):** `https://your-domain.ngrok-free.app`

> By default, n8n is not accessible at `localhost:5678` — no ports are exposed to the host. To enable local access, add the following to the `n8n` service in `docker-compose.yaml`:
> ```yaml
> ports:
>   - "5678:5678"
> ```
> Then restart with `docker-compose up -d`.

---

## Useful Commands

```bash
# View live logs
docker-compose logs -f n8n
docker-compose logs -f ngrok

# Stop containers (data is preserved)
docker-compose down

# Stop and delete all data
docker-compose down -v

# Update n8n to the latest version
docker-compose pull n8n && docker-compose up -d
```

---

## File Structure

```
.
├── docker-compose.yaml   # Service definitions
├── ngrok.yml             # ngrok tunnel config (not committed)
├── .env                  # Environment variables (not committed)
└── backup/               # Optional backups (not committed)
```

Both `.env` and `ngrok.yml` are listed in `.gitignore` and will not be committed.

---

## Troubleshooting

**Webhooks not receiving requests**
- Confirm `URL` in `.env` and `domain` in `ngrok.yml` match exactly
- Check the tunnel is active: `docker-compose logs ngrok | grep "started tunnel"`

**n8n not starting**
- Check for port conflicts or volume permission issues: `docker-compose logs n8n`

**Data loss prevention**
- All workflows, credentials, and settings live in the `n8n_data` Docker volume
- Back it up before running `docker-compose down -v`
