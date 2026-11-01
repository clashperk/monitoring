# monitoring

Metrics stack for ClashPerk.

| Folder | Runs on | Public URL |
| --- | --- | --- |
| [victoria-metrics/](victoria-metrics/) | Docker Compose on a small VM (the "metrics host"), port 8428 | https://metrics.clashperk.com (basic auth) |
| [grafana/](grafana/) | a container platform, built from `grafana/Dockerfile` | https://grafana.clashperk.com (Google login, `clashperk.com` only) |

VictoriaMetrics stores every sample; Grafana stores only users and settings and queries VictoriaMetrics
through its public URL.

## Deploy

- **VictoriaMetrics:** the host holds `~/victoria-metrics/` plus a `.env` (mode 600) made from
  `.env.example`. With `$HOST` as its SSH alias:

  ```bash
  scp victoria-metrics/docker-compose.yml $HOST:victoria-metrics/
  ssh $HOST 'cd ~/victoria-metrics && docker compose up -d'
  ```

- **Grafana:** rebuild and redeploy the image; see [grafana/README.md](grafana/README.md).

Secrets never go in this repo.

## Who pushes metrics

Services push to `https://metrics.clashperk.com/api/v1/import/prometheus` with `extra_label=job=<service>`;
VictoriaMetrics can't reach the other servers to scrape them. Currently: the API and the worker
(clashperk-backend `MetricsPushModule`, `job=api` / `job=worker`, worker forks add `fork`) the Discord bot (clashperk-bot, `job=bot`, one series per `cluster`), and the Clash API
proxy (coc-api-proxy, `job=proxy`; it sits on the same private network, so it pushes to the private address). Keep label values
bounded: route patterns, never raw URLs or tags.
