# Grafana

Dashboards for ClashPerk metrics at **https://grafana.clashperk.com**, reading from
[../victoria-metrics](../victoria-metrics/) over `https://metrics.clashperk.com`. It runs on a container
platform, built from the [Dockerfile](Dockerfile); [docker-compose.yml](docker-compose.yml) is kept for
running it next to VictoriaMetrics instead.

## Login

- Google only, restricted to `clashperk.com` accounts (`allowed_domains` + `hd`). New users become **Viewer**;
  promote them in Administration → Users. Role sync is skipped, so roles set there survive later logins.
- The login form is disabled. The local `admin` (password in the platform's variables) works only over the
  API, e.g. `curl -u admin:$GF_ADMIN_PASSWORD https://grafana.clashperk.com/api/...`.
- The Google OAuth client must list `https://grafana.clashperk.com/login/google` as an authorized redirect URI.

## Deploy

- The image is stock `grafana/grafana:13.2.5` plus `provisioning/`. Rebuild and redeploy it after changing a
  dashboard or the datasource.
- Configuration comes from environment variables: the `GF_*` settings in
  [docker-compose.yml](docker-compose.yml), plus the values in [.env.example](.env.example). `VM_URL` must
  be the public VictoriaMetrics URL when Grafana isn't on the `monitoring` Docker network.
- Mount a persistent volume at `/var/lib/grafana` (SQLite: users, roles, preferences). Dashboards and the
  datasource are provisioned, so losing it only loses users.
- Give it at least ~400 MB of memory: Grafana 13 settles around 340 MB, and below that it swaps until
  requests time out.

## Run locally

For editing dashboards: anonymous Admin, bound to localhost, `provisioning/` mounted live, so a saved JSON file
shows up within 30s. Put `VM_URL`, `VM_USER` and `VM_PASSWORD` in an env file outside the repo, then:

```bash
docker run -d --name grafana-local -p 127.0.0.1:3001:3000 --env-file /path/to/grafana-local.env \
  -e GF_AUTH_ANONYMOUS_ENABLED=true -e GF_AUTH_ANONYMOUS_ORG_ROLE=Admin -e GF_AUTH_DISABLE_LOGIN_FORM=true \
  -v "$PWD/provisioning:/etc/grafana/provisioning:ro" grafana/grafana:13.2.5
```

Open http://localhost:3001; `docker rm -f grafana-local` when done.

## Datasource

`provisioning/datasources/victoria-metrics.yml` is read-only in the UI and uses basic auth from `VM_USER` /
`VM_PASSWORD`. Queries go Grafana → `VM_URL`, so dashboard speed depends on that hop, not on the browser.

## Dashboards

- JSON files in `provisioning/dashboards/` load into the **ClashPerk** folder and are read-only in the UI
  (`allowUiUpdates: false`), so the files stay the source of truth.
- To change one visually: edit in the UI, use *Save as copy* or *Export → JSON*, replace the file here and
  redeploy.
- `api.json` (uid `api`): the API's traffic, latency and errors per route, Clash API calls, and Node runtime,
  filtered on `job="api"`. Rates use `$__rate_interval`, since metrics arrive every 30s.
- `worker.json` (uid `worker`): loop durations, tracked items, events to the bot, maintenance, Clash API calls
  and per-fork runtime, filtered on `job="worker"`. Loop panels average over 1h with MetricsQL's
  `increase_pure`: loops finish every few minutes to ~15 min, and `increase()` drops the first pass of a
  series that appears after a restart.
- `proxy.json` (uid `proxy`): the Clash API proxy's requests by status and endpoint, latency to upstream
  headers, per-key responses (one key far above the rest in non-2xx is dead), connections against the
  `MAX_CONNECTIONS` cap and process runtime, filtered on `job="proxy"`. Ratio panels are empty while idle (0/0).
