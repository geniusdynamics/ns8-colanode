# ns8-colanode

Self-hosted team workspace for chat, documents and tasks, running on your own server with its web app.

Module id: `colanode`. Images are pinned in `build-images.sh` and updated through reviewed changes.

## Install

```bash
add-module ghcr.io/geniusdynamics/colanode:latest 1
```

The command returns the instance id, for example `colanode1`. Use the id NS8 returns.

## Configure

```bash
api-cli run configure-module --agent module/colanode1 --data - <<'EOF'
{
  "host": "colanode.example.org",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

| Parameter | Required | Type | Description |
|---|---|---|---|
| `host` | yes | string | Host name for the application, like 'colanode.domain.org' |
| `lets_encrypt` | yes | boolean | Request a valid Let's Encrypt certificate. |
| `http2https` | yes | boolean | Redirect all the HTTP requests to HTTPS |

## Configuration output

`get-configuration` returns: `host`, `lets_encrypt`, `http2https`.

## Images

| Image | Version |
|---|---|
| `docker.io/pgvector/pgvector` | `0.8.7-pg17` |
| `docker.io/valkey/valkey` | `9.2` |
| `ghcr.io/colanode/server` | `0.4.7` |
| `ghcr.io/colanode/web` | `0.4.7` |

Requires NS8 core `3.12.4-0` or newer.

## Network and access

- Reserves 1 TCP port(s) on the node. The web application is published through Traefik on the configured host name.
- Runs rootless.
- Declares these NS8 authorizations: `traefik@node:routeadm`.

## Backup

The following paths are included in the NS8 backup (`imageroot/etc/state-include.conf`):

- `state/web.env`
- `state/postgres.env`
- `volumes/postgresdata`
- `state/valkey.env`
- `volumes/valkeydata`
- `state/minio.env`
- `volumes/miniodata`
- `state/server.env`

## Services

- `colanode.service`
- `postgres-app.service`
- `server-app.service`
- `valkey-app.service`
- `web-app.service`

## Get the configuration

```bash
api-cli run get-configuration --agent module/colanode1
```

## Update and uninstall

```bash
api-cli run update-module --data '{"module_url":"ghcr.io/geniusdynamics/colanode:latest","instances":["colanode1"],"force":true}'
remove-module --no-preserve colanode1
```

## Testing

Continuous integration:

- `Publish images` builds the module image on every push.
- `Test module` installs the image on a QEMU NS8 node and runs the Robot Framework suite in `tests/`. It runs after `Publish images` succeeds on `dev` and on `main`.
- `Build API JSON Schema docs` and the security scan run alongside.

The suite installs the module, configures a host name, checks that the web application answers through Traefik, takes UI screenshots and removes the module.

Run it locally with the NS8 standard infrastructure described in the [ns8-github-actions README](https://github.com/NethServer/ns8-github-actions/blob/v1/README.md#running-tests-locally).

## Open items

Found by the NS8 module standards check. These must be resolved before a stable release:

- README still has the template boilerplate
- imageroot/actions/restore-module/80start_services: smarthost lines removed, check the unit still starts
- imageroot/actions/configure-module/80start_services: smarthost lines removed, check the unit still starts
- MARIADB_IMAGE is used by the module but no image in the label provides it: leftover from the template?

## Upstream

Application documentation: https://github.com/colanode/colanode

## License

GPL-3.0-or-later. See [LICENSE](LICENSE).
