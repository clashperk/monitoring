# monitoring

Metrics stack for ClashPerk, self-hosted with Docker Compose on one small VM (the "metrics host").

| Folder | Port | Public URL |
| --- | --- | --- |
| [victoria-metrics/](victoria-metrics/) | 8428 | https://metrics.clashperk.com (basic auth) |
| [grafana/](grafana/) | 3000 | https://grafana.clashperk.com (Google login, `clashperk.com` only) |

Both sit behind Cloudflare and a TCP load balancer. Grafana reaches VictoriaMetrics over the shared
`monitoring` Docker network, which the VictoriaMetrics compose creates, so start that one first.

## Deploy

The host holds each folder at `~/victoria-metrics/` and `~/grafana/`, plus a `.env` (mode 600) made from
`.env.example`. Secrets never go in this repo. With `$HOST` as the host's SSH alias:

```bash
scp -r grafana/docker-compose.yml grafana/provisioning $HOST:grafana/
ssh $HOST 'cd ~/grafana && docker compose up -d'
```

See each folder's README for rebuild steps, settings and checks.

## Who pushes metrics

Services push to `https://metrics.clashperk.com/api/v1/import/prometheus` with `extra_label=job=<service>`;
VictoriaMetrics can't reach the other servers to scrape them. Currently: the API
(clashperk-backend `MetricsPushService`, `job=api`). Keep label values bounded: route patterns, never raw URLs or tags.
