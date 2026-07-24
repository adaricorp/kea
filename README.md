# kea (multi-arch)

Multi-architecture (`linux/amd64` + `linux/arm64`) builds of ISC's
[Kea](https://www.isc.org/kea/) DHCP server container images, for use by
[Adari](https://github.com/adaricorp/adarigod).

**Published images**:
- `ghcr.io/adaricorp/kea-dhcp4:<version>` — DHCPv4 server
- `ghcr.io/adaricorp/kea-dhcp6:<version>` — DHCPv6 server

## Why this repo exists

ISC publishes official kea images at `docker.cloudsmith.io/isc/docker/kea-dhcp4`
and `…/kea-dhcp6`, but **only for `linux/amd64`** — their
[kea-docker](https://gitlab.isc.org/isc-projects/kea-docker) CI does a plain
`docker build` with no `buildx`/arm runner, tracked upstream as work item
[#47](https://gitlab.isc.org/isc-projects/kea-docker/-/work_items/47) ("Docker
images for arm64", open since 2026-01-28). That amd64-only manifest means the
Adari DHCP CNFs can never start on the arm64 lab/VM harness (Apple Silicon), so
fresh-install e2e can't exercise DHCP there.

ISC **does** publish signed `aarch64` Kea packages (Alpine/Debian/Ubuntu/RPM),
and their Dockerfiles are just `FROM alpine:3.24` +
`apk add isc-kea-dhcpN~=${VERSION} …` from their Cloudsmith package repo. So a
`docker buildx --platform linux/amd64,linux/arm64` of the **unmodified upstream
Dockerfiles** yields multi-arch images whose binaries are ISC's own signed
packages on both arches, with runtime conventions identical to what the fleet
already runs.

See adaricorp/adarigod issue
[#1976](https://github.com/adaricorp/adarigod/issues/1976).

## Layout

Each subdirectory is one published image; the directory name is the image name.

```
kea-dhcp4/   → ghcr.io/adaricorp/kea-dhcp4   (Dockerfile, kea-dhcp4.conf, hiddens)
kea-dhcp6/   → ghcr.io/adaricorp/kea-dhcp6   (Dockerfile, kea-dhcp6.conf, hiddens)
```

Each `Dockerfile` is ISC's upstream `kea-docker/<component>/Dockerfile`,
reproduced **verbatim** except the `VERSION` build arg is pinned to a full
three-part version and carries a Renovate annotation (header comment documents
the provenance). The `*.conf` / `hiddens` are ISC's upstream defaults, copied
verbatim (Adari mounts its own config over these at runtime).

The `.github/workflows/build-and-publish.yml` builds both components in a matrix
and pushes to `ghcr.io/adaricorp/<component>`. Each image's tag is derived from
its Dockerfile's `ARG VERSION`, so tag and installed binaries can never diverge.

## Versioning & updates

1. Renovate watches ISC's `docker.cloudsmith.io/isc/docker/kea-dhcp4` and
   `…/kea-dhcp6` tags and, on a new **stable** (even-minor) release, opens a PR
   bumping `ARG VERSION=` in both Dockerfiles (grouped — both track the same Kea
   release).
2. Merging that PR triggers this workflow, which builds and publishes
   `ghcr.io/adaricorp/kea-dhcp4:<new version>` and `…/kea-dhcp6:<new version>`
   (plus `<major.minor>`, `latest`, and `sha-` tags) for both architectures.
3. Renovate on **adarigod** then sees the new `ghcr.io/adaricorp/kea-dhcp4` tag
   and bumps `KEA_VERSION` in `src/adari/services/sdn/kea.py`. (adarigod is
   DHCPv4-only today; the kea-dhcp6 image is published and ready for when DHCPv6
   support lands — see the `# TODO: IPv6 compatibility` in that file.)

If ISC ever ships multi-arch images upstream (work item #47), point adarigod back
at `docker.cloudsmith.io` and archive this repo.

## Licensing

The Dockerfiles, `*.conf`, and `hiddens` files originate from ISC's
[kea-docker](https://gitlab.isc.org/isc-projects/kea-docker) repository and are
licensed **MPL-2.0** (`SPDX-License-Identifier: MPL-2.0`, preserved in each
`Dockerfile`). The Kea binaries in the images are ISC's own signed packages.
