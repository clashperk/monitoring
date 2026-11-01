# VictoriaMetrics

Metrics store for ClashPerk backends; Grafana reads it as a Prometheus datasource. Runs from
`~/victoria-metrics/` on the metrics host (`$HOST` below is its SSH alias).

## Rebuild

```bash
ssh $HOST 'mkdir -p ~/victoria-metrics'
scp docker-compose.yml $HOST:victoria-metrics/
ssh $HOST 'cd ~/victoria-metrics && umask 077 && \
  printf "VM_USER=clashperk\nVM_PASSWORD=%s\n" "$(openssl rand -hex 24)" > .env && \
  docker compose up -d'
```

A new password means updating every client that pushes or queries (Grafana datasource, services).

## Behaviour

- Port 8428. Basic auth on everything except `/health`. The flags take the credentials as `%{VM_USER}` /
  `%{VM_PASSWORD}` placeholders, so the password never shows in process args.
- 6 months retention (`-retentionPeriod`), data in the `victoria-metrics-data` Docker volume (fixed name, independent of the folder).
- `mem_limit: 250m`; VictoriaMetrics sizes its caches from the cgroup limit, so raising it is safe.
- Accepts OTLP (`/opentelemetry/v1/metrics`) and Prometheus remote write / import; Grafana queries
  `/api/v1/query*` like Prometheus.

## Network

```
https://metrics.clashperk.com → Cloudflare → load balancer :8428 → metrics host :8428
```

- Cloudflare: proxied record to the load balancer, an Origin Rule for port 8428, and Flexible SSL (the
  load balancer speaks plain HTTP). It is outside the API rate limit.
- The load balancer health-checks `GET /health` and only accepts Cloudflare's IP ranges on 8428.
- Docker publishes 8428 on all interfaces, so the cloud firewall is what keeps the host's public IP closed.
  Hosts on the same private network can use the private address directly.

## Checks

```bash
ssh $HOST 'cd ~/victoria-metrics && set -a && . ./.env && \
  curl -s -u "$VM_USER:$VM_PASSWORD" "localhost:8428/api/v1/query?query=up"'
```

Label by route pattern, never raw URLs or player/clan tags, to keep active series low.
