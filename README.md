# kea-dhcp4 (multi-arch)

Multi-architecture (`linux/amd64` + `linux/arm64`) build of ISC's
[Kea](https://www.isc.org/kea/) DHCPv4 server container image, for use by
[Adari](https://github.com/adaricorp/adarigod).

**Published image**: `ghcr.io/adaricorp/kea-dhcp4:<version>`

## Why this repo exists

ISC publishes an official kea-dhcp4 image at
`docker.cloudsmith.io/isc/docker/kea-dhcp4`, but **only for `linux/amd64`** —
their [kea-docker](https://gitlab.isc.org/isc-projects/kea-docker) CI does a
plain `docker build` with no `buildx`/arm runner, tracked upstream as work item
[#47](https://gitlab.isc.org/isc-projects/kea-docker/-/work_items/47) ("Docker
images for arm64", open since 2026-01-28). That amd64-only manifest means the
Adari DHCP CNF can never start on the arm64 lab/VM harness (Apple Silicon), so
fresh-install e2e can't exercise DHCP there.

ISC **does** publish signed `aarch64` Kea packages (Alpine/Debian/Ubuntu/RPM),
and their `kea-dhcp4/Dockerfile` is just `FROM alpine:3.24` +
`apk add isc-kea-dhcp4~=${VERSION} …` from their Cloudsmith package repo. So a
`docker buildx --platform linux/amd64,linux/arm64` of the **unmodified upstream
Dockerfile** yields a multi-arch image whose binaries are ISC's own signed
packages on both arches, with runtime conventions identical to what the fleet
already runs.

See adaricorp/adarigod issue
[#1976](https://github.com/adaricorp/adarigod/issues/1976).

## What's in here

| File | Purpose |
|------|---------|
| `Dockerfile` | ISC's upstream `kea-dhcp4/Dockerfile`, reproduced **verbatim** except the `VERSION` build arg is pinned to a full three-part version and carries a Renovate annotation. Header comment documents the provenance. |
| `kea-dhcp4.conf`, `hiddens` | ISC's upstream default config + control-agent credentials file, copied verbatim (Adari mounts its own `kea-dhcp4.conf` over these at runtime). |
| `.github/workflows/build-and-publish.yml` | Multi-arch build + push to `ghcr.io/adaricorp/kea-dhcp4`. The image tag is derived from the Dockerfile's `ARG VERSION`, so tag and installed binaries can never diverge. |
| `renovate.json` | Bumps the Dockerfile's `ARG VERSION` when ISC ships a new **stable** (even-minor) kea release. |

## Versioning & updates

1. Renovate watches ISC's `docker.cloudsmith.io/isc/docker/kea-dhcp4` tags and,
   on a new stable release, opens a PR bumping `ARG VERSION=` in the
   `Dockerfile`.
2. Merging that PR triggers this workflow, which builds and publishes
   `ghcr.io/adaricorp/kea-dhcp4:<new version>` (plus `<major.minor>`, `latest`,
   and a `sha-` tag) for both architectures.
3. Renovate on **adarigod** then sees the new `ghcr.io/adaricorp/kea-dhcp4` tag
   and bumps `KEA_VERSION` in `src/adari/services/sdn/kea.py`.

If ISC ever ships multi-arch images upstream (work item #47), point adarigod
back at `docker.cloudsmith.io/isc/docker/kea-dhcp4` and archive this repo.

## Licensing

The `Dockerfile`, `kea-dhcp4.conf`, and `hiddens` originate from ISC's
[kea-docker](https://gitlab.isc.org/isc-projects/kea-docker) repository and are
licensed **MPL-2.0** (`SPDX-License-Identifier: MPL-2.0`, preserved in the
`Dockerfile`). The Kea binaries in the image are ISC's own signed packages.
