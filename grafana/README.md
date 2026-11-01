# Grafana

Dashboards for ClashPerk metrics at **https://grafana.clashperk.com**, reading from
[../victoria-metrics](../victoria-metrics/). Runs from `~/grafana/` on the metrics host.

## Login

- Google only, restricted to `clashperk.com` accounts (`allowed_domains` + `hd`). New users become **Viewer**;
  promote them in Administration → Users. Role sync is skipped, so roles set there survive later logins.
- `admin@clashperk.com` is org Admin and Grafana server admin.
- The login form is disabled. The local `admin` (password in `.env`) works only over the API, e.g.
  `curl -u admin:$GF_ADMIN_PASSWORD localhost:3000/api/...`.
- The Google OAuth client must list `https://grafana.clashperk.com/login/google` as an authorized redirect URI.

## Rebuild

VictoriaMetrics must be up first: it creates the shared `monitoring` Docker network.

```bash
ssh $HOST 'mkdir -p ~/grafana'
scp -r docker-compose.yml provisioning $HOST:grafana/
# then on the host: create ~/grafana/.env (mode 600) from .env.example, and
ssh $HOST 'cd ~/grafana && docker compose up -d'
```

The first start runs database migrations and takes a minute or two on a small VM.

## Behaviour

- `grafana/grafana:13.2.5`, SQLite in the `grafana-data` volume (dashboards, users) — back that up.
- `mem_limit: 300m`. At 200m the first start swapped and stalled a 1 GB host; steady state is ~70–150 MB.
- The VictoriaMetrics datasource is provisioned (`provisioning/datasources/`), read-only in the UI, and
  reaches `http://victoria-metrics:8428` over the `monitoring` network with basic auth from `.env`.

## Network

```
https://grafana.clashperk.com → Cloudflare → load balancer :3000 → metrics host :3000
```

Cloudflare Origin Rule for port 3000 plus Flexible SSL; the load balancer health-checks `GET /api/health`
and only accepts Cloudflare's IP ranges on 3000.
