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

## Automated dependency updates

Install the Renovate GitHub App and explicitly select this repository (including
when maintaining it as a fork). `renovate.json` enables reviewable PRs for:

- Debian base-image digest updates for the active Terraria Dockerfiles.
- Vanilla Terraria releases from the official dedicated-server download API.
- TShock releases from GitHub, grouped with the compatible Terraria version
  extracted from the latest stable TShock release's title.
- GitHub Actions used by the image-build workflow.

Server-version updates change both `build.env` and the Dockerfile defaults.
Vanilla `SERVER_VERSION` and `GAME_VERSION` move together. TShock's
`GAME_VERSION` follows TShock release metadata, not the independent vanilla
release feed. Review the paired versions and .NET runtime requirements before
merging; automerge is disabled.

Renovate pins base images by digest and proposes new digests for the same
`bookworm-slim` tag. Debian distribution upgrades are held because the TShock
Dockerfile currently installs `libicu72`. Unused `base/` images are excluded;
the workflow builds only `games/`.

For changes that leave `SERVER_VERSION` unchanged, increment `BUILD_TAG` in
both `build.env` and the Dockerfile before merging to publish a fresh image tag.
Renovate does not change that packaging revision automatically. Game dependency
PRs exercise the existing build workflow, and merging publishes the resulting
images for downstream Zarf-package Renovate PRs to discover. Workflow-only
updates also exercise all current game builds.

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
