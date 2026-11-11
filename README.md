
# Certification Bot

A self-hosted study bot that sends RHCSA exam questions to users at random
intervals throughout the day, scores their answers, and tracks progress.

## Description

Certification Bot posts curated RHCSA practice questions over Mattermost at
random times during the day. Each question has a limited window to respond;
users earn points based on accuracy. The stack is self-hosted on a RHEL-family
server with Podman, each service running as a Quadlet systemd unit behind a
Caddy reverse proxy.

Starting with RHCSA; additional certifications planned.

## Architecture

A user connects over HTTPS to Caddy, the only externally exposed service. Caddy
reverse-proxies to Mattermost and n8n by container name over a shared Podman
network. Mattermost stores data in PostgreSQL, which is never exposed outside
the network. n8n holds the quiz logic: scheduling questions, catching replies,
scoring, and tracking per-user progress.

## Getting Started

### Host requirements

* RHEL / Rocky Linux / AlmaLinux
* SELinux enforcing
* Podman (with Quadlet support)
* firewalld

### Services

* Caddy (reverse proxy, TLS)
* Mattermost (chat server)
* PostgreSQL (Mattermost database)
* n8n (quiz bot logic)

### Installing

Quadlet unit files live in `quadlets/` and are placed in
`/etc/containers/systemd/`. Service configuration (passwords, connection
strings) is supplied via a gitignored `.env` file; see `.env_example` for the
required keys.

1. Copy the `.container` and `.network` files to `/etc/containers/systemd/`
2. Create `.env` from `.env_example` and fill in your values
3. Create the host directories referenced by the volume mounts
4. `sudo systemctl daemon-reload`
5. Start the services (PostgreSQL first, then the rest)


## Troubleshooting / Lessons Learned

### Logging
* **No journal logs** — journald persists logs only if `/var/log/journal`
  exists. Created the directory, then `systemctl restart
  systemd-journal-flush.service`.

### Caddy / networking
* **Rootless can't bind ports <1024** — hit on ports 80/443.
* **Port 80 in use** — a leftover native Caddy package and `httpd` were
  squatting it; removed both.
* **Port 443 in use** — tailscale binds :443 on its own interface. Scoped
  Caddy's `PublishPort` to the LAN IP to avoid the collision.
* **Empty Caddyfile** — the default image only serves static files; added a
  `reverse_proxy` config.

### Rootful vs rootless
* **Rootful and rootless containers can't share a Podman network.** Moved the
  stack to rootful for consistency. (Revisiting rootless is a future hardening
  task.)
* **`%h` under rootful** resolves to `/root`, not the user's home — all volume
  and env paths had to become absolute.

### Mattermost
* **Permission denied on config.json** — container runs as UID 2000; set
  `PUID`/`PGID=2000` and chowned host folders to 2000.
* **Invalid SQL driver** — `MM_SQLSETTINGS_DRIVERNAME` must be exactly
  `postgres`, not `postgresql`.
* **`no such host`** — the datasource hostname must match the DB container name.
* **Password auth failed** — PostgreSQL only sets credentials on first volume
  init; wiped the empty data volume to re-init with current credentials.

## Version History

* 0.2
    * Added Caddy, PostgreSQL, Mattermost, and n8n as Quadlet units
    * See [commit history](https://github.com/thedonnyb/Certification-bot/commits/main)
* 0.1
    * Initial release


## Acknowledgments

* [Mattermost](https://mattermost.com/), [n8n](https://n8n.io/),
  [Caddy](https://caddyserver.com/), [PostgreSQL](https://www.postgresql.org/)
