---
title: Put Kept on HTTPS
description: Why Kept wants HTTPS, and how to get it - Caddy from compose.full.yaml, Traefik, Pangolin, Tailscale, NetBird, Twingate, Cloudflare Tunnel or your own proxy.
sidebar:
  order: 2
---

Kept works over plain HTTP, but a phone treats a plain-HTTP address as unsafe, and several things
stop working:

- **the camera** (Kept falls back to a file picker),
- **installing Kept** to the home screen, and working offline from it,
- **push notifications**,
- **location**.

`localhost` counts as safe; a LAN address over HTTP does not. With an `https:` `KEPT_PUBLIC_URL`,
Kept also marks its cookies `Secure` and sends HSTS; `KEPT_HSTS=off` clears it when plain-http
services share the hostname ([HSTS](/install/reverse-proxy/#hsts)).

Some things Kept refuses outright while `KEPT_PUBLIC_URL` is `http:`, even on `localhost`, because
they hand out or change a secret: showing a secret value, an export with secret values, opening
an export's secrets when importing it, making an access token in the browser, connecting an app
with OAuth, downloading the recovery kit, and changing or testing the backup settings. An export
without secret values works, and so does an import without them. Sessions are also shorter over
HTTP. While Kept is served over HTTP, the Get started checklist on Home puts "Put Kept on HTTPS"
first, until an admin chooses "Staying on my home network". Everything that works over plain HTTP,
in one table: [Kept on a home network](/install/home-network/).

Whichever way you choose, set `KEPT_PUBLIC_URL` to the `https://` address people will open, and
restart Kept.

## Compose's Caddy profile

For a domain name that points at this machine, with ports 80 and 443 reachable from the internet
(Caddy proves the domain to Let's Encrypt or ZeroSSL over them). Caddy comes with
`compose.full.yaml`, not `compose.yaml` ([Caddy and Ollama](/install/compose/#caddy-and-ollama)):

```sh
# in .env
COMPOSE_FILE=compose.full.yaml
KEPT_DOMAIN=kept.example.org
KEPT_PUBLIC_URL=https://kept.example.org
```

Caddy reads `docker/caddy/Caddyfile` next to the Compose file. A checkout of the repository has
both files; an install without one downloads them once:

```sh
curl -fsSLO https://raw.githubusercontent.com/keptinventory/kept/v1.14.5/compose.full.yaml
mkdir -p docker/caddy
curl -fsSLo docker/caddy/Caddyfile https://raw.githubusercontent.com/keptinventory/kept/v1.14.5/docker/caddy/Caddyfile
docker compose --profile https up -d
```

Without the file, `docker compose --profile https up -d` stops with "bind source path does not
exist". Any other proxy works instead; it forwards to Kept's port, as in the sections below.

Caddy gets and renews the certificate and forwards everything to Kept. Compose gives Caddy a fixed
address on its own network and passes that address to Kept as `KEPT_TRUSTED_PROXIES`, so Kept
believes Caddy's `X-Forwarded-For` and nothing else's. You can then bind plain HTTP to loopback
only, with `KEPT_HTTP_PORT=127.0.0.1:8080`.

If `172.29.72.0/24` clashes with your LAN, change the network's subnet, its `ip_range`, Caddy's
`ipv4_address` and `KEPT_TRUSTED_PROXIES` together in `compose.full.yaml`.

## Tailscale

For a Kept that only your own devices reach, with no port open to the internet: on the machine
running Kept, with HTTPS certificates enabled for your tailnet,

```sh
tailscale serve --bg 8080
```

serves Kept at `https://<machine>.<tailnet>.ts.net` with a real certificate, to any device on the
tailnet. Set `KEPT_PUBLIC_URL` to that address.

Tailscale's proxy runs on the same machine and connects to Kept from `127.0.0.1`; Kept sees every
request as coming from there unless you trust it. On a single-household instance that only
matters for [the setup lockout](/install/reverse-proxy/#the-setup-lockout).

## NetBird

NetBird's reverse proxy gives a peer's port a public `https://` address with a certificate:

```sh
netbird expose 8080
```

It prints the address. Set `KEPT_PUBLIC_URL` to it (or to your own domain, with
`--with-custom-domain kept.example.org`, set up in NetBird first). The address is public unless
you choose **NetBird-Only Access** for the service in the dashboard, which limits it to your peers.
An account admin turns on **Peer Expose** under Settings → Clients first.

## Twingate

Twingate makes Kept reachable to your devices without opening a port, but the certificate is
yours: give Kept a hostname on a domain you own (`kept.example.org`), get a certificate for it with
a DNS challenge (Caddy, Traefik or cert-manager can), and add that hostname as a Twingate resource.
Set `KEPT_PUBLIC_URL` to `https://kept.example.org`.

## Cloudflare Tunnel

For a public address with no open port. Cloudflare serves the hostname over HTTPS and forwards to
Kept through `cloudflared`:

```sh
cloudflared tunnel login
cloudflared tunnel create kept
cloudflared tunnel route dns kept kept.example.org
cloudflared tunnel run kept
```

with `~/.cloudflared/config.yml`:

```yaml
tunnel: kept
credentials-file: <home>/.cloudflared/<tunnel id>.json
ingress:
  - hostname: kept.example.org
    service: http://localhost:8080
  - service: http_status:404
```

Set `KEPT_PUBLIC_URL` to `https://kept.example.org`. Cloudflare Access can put a sign-in in front
of it. A public address is also what a web app such as claude.ai needs to
[connect to Kept's MCP server](/users/mcp-clients/#connect-as-a-connector-oauth).

`cloudflared`, like the other tunnels here, reaches Kept from the same machine; trust it as in
[reverse proxies](/install/reverse-proxy/#trust-the-proxy-and-only-the-proxy).

## Traefik

For a Traefik that already runs on this machine and finds containers by their Docker labels.
Kept joins Traefik's network and carries the labels; `compose.override.yaml` next to
`compose.yaml` adds both without changing it (`proxy` is the name of Traefik's network,
`websecure` its https entrypoint and `letsencrypt` its certificate resolver: use yours):

```yaml
# compose.override.yaml
services:
  kept:
    networks: [kept, proxy]
    labels:
      - traefik.enable=true
      - traefik.docker.network=proxy
      - traefik.http.routers.kept.rule=Host(`kept.example.org`)
      - traefik.http.routers.kept.entrypoints=websecure
      - traefik.http.routers.kept.tls.certresolver=letsencrypt
      - traefik.http.services.kept.loadbalancer.server.port=8080

networks:
  proxy:
    external: true
```

```sh
# in .env
KEPT_PUBLIC_URL=https://kept.example.org
KEPT_TRUSTED_PROXIES=172.30.0.2
```

`KEPT_TRUSTED_PROXIES` is Traefik's address on that network (`docker network inspect proxy`); give
Traefik a fixed address there, or it can change when Traefik is recreated. Then
`docker compose up -d`. Traefik sets no upload limit of its own, so Kept's applies.

## Pangolin

For a public address through a Pangolin server, with no port open at home. Newt, Pangolin's site
connector, runs next to Kept and reads the resource from Kept's labels
([Pangolin's blueprints](https://docs.pangolin.net/manage/blueprints)). Create the site in
Pangolin first, for its ID and secret, and give Newt a fixed address on Kept's network:

```yaml
# compose.override.yaml
services:
  kept:
    labels:
      - pangolin.public-resources.kept.name=Kept
      - pangolin.public-resources.kept.full-domain=kept.example.org
      - pangolin.public-resources.kept.protocol=http
      - pangolin.public-resources.kept.ssl=true
      - pangolin.public-resources.kept.auth.sso-enabled=false
      - pangolin.public-resources.kept.targets[0].method=http
      - pangolin.public-resources.kept.targets[0].hostname=kept
      - pangolin.public-resources.kept.targets[0].port=8080
      - pangolin.public-resources.kept.targets[0].healthcheck.enabled=true
      - pangolin.public-resources.kept.targets[0].healthcheck.hostname=kept
      - pangolin.public-resources.kept.targets[0].healthcheck.port=8080
      - pangolin.public-resources.kept.targets[0].healthcheck.path=/readyz

  newt:
    image: fosrl/newt
    restart: unless-stopped
    environment:
      PANGOLIN_ENDPOINT: https://pangolin.example.org
      NEWT_ID: ${NEWT_ID}
      NEWT_SECRET: ${NEWT_SECRET}
      DOCKER_SOCKET: /var/run/docker.sock
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      kept:
        ipv4_address: 172.29.72.201
```

```sh
# in .env
NEWT_ID=…
NEWT_SECRET=…
KEPT_PUBLIC_URL=https://kept.example.org
KEPT_TRUSTED_PROXIES=172.29.72.201
```

Then `docker compose up -d`; the resource appears in Pangolin's dashboard.

- **`sso-enabled=false`:** Kept has its own accounts. Pangolin's sign-in in front of it would also
  stop the phone app, scanned labels, the Home Assistant integration and AI apps, which can't
  sign in to Pangolin.
- **Newt's address:** Newt makes the connections to Kept, so it is the proxy Kept trusts. A fixed
  address above `172.29.72.127` stays out of the range Docker hands out on Kept's network.
- **The Docker socket:** Newt reads the labels through it, and whoever can read it can run
  containers on this machine. Leave Newt's labels out and the socket off if you would rather add
  the resource in Pangolin's dashboard, with `kept` and port 8080 as its target.
- **Labels belong to the resource:** Pangolin applies them again when Newt starts, over changes
  made in its dashboard.

## Your own reverse proxy

Any proxy that terminates TLS and forwards to port 8080 works. Read
[reverse proxies](/install/reverse-proxy/) for the three settings that matter.

## A certificate only your devices trust

On a LAN with no domain, mkcert can make a certificate
for the machine's address, served through any TLS proxy. Every phone must then trust mkcert's root
certificate; on an iPhone that means installing it as a profile and turning on full trust in
Settings → General → About → Certificate Trust Settings. Tailscale is usually less work.
