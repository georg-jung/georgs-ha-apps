# Music Capsule Sidecar

## About

Music Capsule is a private, self-hosted music library web app. The import
sidecar is a small helper that runs on your home network. It dials out to
your Music Capsule server over HTTPS (no inbound port, no tunnel), long-polls
for import work, fetches the media for an import from your home connection,
and uploads the result back to the server. The server keeps all the
decisions — naming, tagging, storing — the sidecar only fetches and uploads.

Run this add-on if you want imports to leave from your home connection
rather than from the server.

## Installation

1. The image is private, so first add registry credentials: **Settings →
   Apps** (older Home Assistant versions: **Add-ons**) **→ App store → ⋮ →
   Registries**, then add registry `ghcr.io` with your GitHub username as
   the username and a personal access token (classic) with the
   `read:packages` scope as the password — fine-grained tokens do not work
   for packages. If the pull is still refused, add the `repo` scope.
2. Add this repository under **⋮ → Repositories** using
   `https://github.com/georg-jung/georgs-ha-apps`, then find "Music Capsule
   Sidecar" in the app store and install it.
3. Open the app's **Configuration** tab and fill in `server_url` and
   `token` (see below).
4. Start the app.

## Configuration

```yaml
server_url: https://music.example.com
token: 0123456789abcdef0123456789abcdef
# The ones below are optional; leave them out unless you need them.
additional_server_urls:
  - https://kids.example.com
proxy: socks5://127.0.0.1:1080
allow_http: false
```

| Option | Required | Description |
| --- | --- | --- |
| `server_url` | yes | Your Music Capsule server's address, scheme and host only, e.g. `https://music.example.com`. No path, no trailing slash. |
| `token` | yes | Shared secret that authenticates the sidecar to your server. At least 32 characters, and must match `MUSIC_CAPSULE_SIDECAR_TOKEN` on the server. Generate one with `openssl rand -hex 16`. |
| `additional_server_urls` | no | Further Music Capsule servers the same sidecar serves, each written like `server_url`. At most 15. All of them use `token`, so each of those servers needs the same `MUSIC_CAPSULE_SIDECAR_TOKEN`. |
| `proxy` | no | An `http://`, `https://`, `socks5://` or `socks5h://` proxy URL. Only the fetch of import media is routed through it, never the connection to your server. |
| `allow_http` | no | Permit an `http://` `server_url`, for LAN-only setups. Defaults to `false`, which requires HTTPS. |

With additional servers, every server is served at the same time rather
than in turn, so a long import into one does not hold up the others. Each
server's Import view shows the sidecar as connected on its own.

Any data the sidecar writes is scratch space only. It is swept at every
start, so there is nothing to back up.

## Checking it works

Once the sidecar is running and connected, your Music Capsule server's
Import view shows:

```
Downloader: home sidecar … seen Ns ago
```

If it shows "No sidecar is connected", the sidecar is not reaching your
server — check the configuration and the log.

The add-on's log is JSON lines and can be read from the app's **Log** tab.

## Troubleshooting

- **Token mismatch**: if `token` does not match `MUSIC_CAPSULE_SIDECAR_TOKEN`
  on the server, the server refuses the connection. This shows up in the log
  as a refused token, retried every 30 seconds. Make sure both sides use the
  exact same value.
- **The update fails to install**: Home Assistant learns about a new version from this repository, but pulls the image from `ghcr.io` — an expired or revoked GitHub token under **⋮ → Registries** stops the pull, not the notification.
- **A killed import**: stopping the sidecar mid-import only fails that one
  item. The server continues with the rest, and re-running the import only
  fetches what is still missing.

## Updates

The add-on's version follows Music Capsule's own releases: a new Music
Capsule release publishes a matching sidecar image and updates the version
in this repository automatically. Updating the add-on in Home Assistant
pulls the image tag that matches the current Music Capsule release.
