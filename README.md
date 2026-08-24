# Terraria Docker Containers

Versioned Terraria dedicated-server images designed for Kubernetes deployment.
The server binaries are downloaded while the images are built, so containers do
not need deployment-time network access to install the server.

## Images

| Image | Server version | Tag |
|---|---|---|
| Vanilla Terraria | 1.4.5.6 | `ghcr.io/jbmay/terraria-vanilla:1.4.5.6-build.1` |
| TShock | 6.1.0 for Terraria 1.4.5.6 | `ghcr.io/jbmay/terraria-tshock:6.1.0-build.1` |

Each image's `build.env` is the source of truth for:

- `SERVER_VERSION`: the packaged server release
- `GAME_VERSION`: the compatible Terraria release
- `BUILD_TAG`: the packaging revision for that release

Increment `BUILD_TAG` when rebuilding the same server release after changing
the image itself.

## Local builds

```bash
docker build \
  --build-arg SERVER_VERSION=1.4.5.6 \
  --build-arg GAME_VERSION=1.4.5.6 \
  --build-arg BUILD_TAG=1 \
  --tag terraria-vanilla:1.4.5.6-build.1 \
  games/terraria-vanilla

docker build \
  --build-arg SERVER_VERSION=6.1.0 \
  --build-arg GAME_VERSION=1.4.5.6 \
  --build-arg BUILD_TAG=1 \
  --tag terraria-tshock:6.1.0-build.1 \
  games/terraria-tshock
```

## CI/CD

The game-image workflow discovers directories beneath `games/` dynamically.
On pull requests it builds changed games without pushing. On merges to `main`,
it builds changed games and publishes these tags to GHCR:

- `latest`
- `sha-<commit>`
- branch or pull-request name
- `<server-version>-build.<revision>`

Use the manual `game` input to build one directory, such as
`terraria-vanilla`, or select `rebuild_all` to enumerate and build every current
directory beneath `games/`.
