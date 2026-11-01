# CLAUDE.md

Guidance for Claude Code in **monitoring** (`clashperk/monitoring`): compose files and provisioning for
VictoriaMetrics + Grafana on one small VM. Start with [README.md](README.md). Host names, IPs and cloud
resource names live in the gitignored `CLAUDE.local.md`.

- Never write server names, IP addresses or cloud resource names into tracked files, commit messages or PR
  text — the repo may be open-sourced. Put them in `CLAUDE.local.md`.
- No build step and no images of our own: stock images, configured by compose, `.env` and `grafana/provisioning/`.
- `.env` files live only on the host (mode 600). Never commit, print or echo their values.
- Deploy = copy the changed files to the matching folder on the host, then `docker compose up -d` there.
  Restarting Grafana takes ~70s and the load balancer returns 522 meanwhile; dashboard JSON reloads without a restart.
- Grafana needs `mem_limit` ≥ 300m on a 1 GB host (200m swapped and stalled it on first start).
- Opening ports (load balancer listeners, firewall rules) needs the user's approval first.
