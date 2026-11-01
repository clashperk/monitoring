# CLAUDE.md

Guidance for Claude Code in **monitoring** (`clashperk/monitoring`): VictoriaMetrics (compose on a small VM)
and Grafana (container image). Start with [README.md](README.md). Host names, IPs, platform project names and
cloud resource names live in the gitignored `CLAUDE.local.md`.

- Never write server names, IP addresses, container names or cloud resource names into tracked files, commit
  messages or PR text — the repo may be open-sourced. Put them in `CLAUDE.local.md`.
- No images of our own except `grafana/Dockerfile` (stock Grafana + `provisioning/`). VictoriaMetrics is
  configured by compose and `.env`.
- `.env` files and platform variables hold the secrets. Never commit, print or echo their values.
- Dashboard or datasource changes need a Grafana image rebuild and redeploy.
- Grafana needs ~400 MB of memory (it settles around 340 MB; at 300 MB it swapped until requests timed out).
- Opening ports (load balancer listeners, firewall rules) needs the user's approval first.
