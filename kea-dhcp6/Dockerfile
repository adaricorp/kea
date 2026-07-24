# SPDX-License-Identifier: MPL-2.0
#
# This Dockerfile is ISC's upstream kea-docker `kea-dhcp6/Dockerfile`
# (https://gitlab.isc.org/isc-projects/kea-docker, MPL-2.0), reproduced here
# UNMODIFIED except for one thing: the `VERSION` build arg below is pinned to a
# full three-part version and carries a Renovate annotation so this repo can
# bump it and publish a matching `ghcr.io/adaricorp/kea-dhcp6:<version>` tag.
#
# Why this repo exists: ISC only publishes amd64 manifests for
# `docker.cloudsmith.io/isc/docker/kea-dhcp6`, so the DHCPv6 CNF could never
# start on the arm64 lab/VM harness. ISC *does* publish signed aarch64 kea
# packages, and their Dockerfile is just `FROM alpine + apk add isc-kea-dhcp6`
# from their Cloudsmith package repo, so a
# `docker buildx --platform linux/amd64,linux/arm64` of the unmodified upstream
# Dockerfile produces a multi-arch image whose binaries are ISC's own signed
# packages on both arches. See adaricorp/adarigod issue #1976.
#
# Revert to `docker.cloudsmith.io` if ISC ever ships multi-arch images.

# This Kea docker image provides the following functionality:
# - running Kea DHCPv6 service
# - running Kea control agent (exposes REST API over http)
# - open source hooks
# - possible to build with premium hooks

FROM alpine:3.24
LABEL org.opencontainers.image.authors="Kea Developers <kea-dev@lists.isc.org>"
LABEL org.opencontainers.image.source="https://github.com/adaricorp/kea"
LABEL org.opencontainers.image.description="Multi-arch (amd64+arm64) build of ISC's kea-dhcp6 image for Adari"
LABEL org.opencontainers.image.licenses="MPL-2.0"

# Add Kea packages from cloudsmith. Make sure the version matches that of the Alpine version.
# Also, install all the open source hooks. When updating, new instructions can
# be found at: https://cloudsmith.io/~isc/repos/kea-2-5/setup/#formats-alpine
#
# renovate: datasource=docker depName=docker.cloudsmith.io/isc/docker/kea-dhcp6 registryUrl=https://docker.cloudsmith.io
ARG VERSION=3.2.0
ARG TOKEN
ARG PREMIUM

SHELL ["/bin/ash", "-o", "pipefail", "-c"]
RUN cp /etc/apk/repositories /etc/apk/repositories_backup && \
    # Install curl and bash for cloudsmith's script. Tzdata to allow timezone change.
    apk update && apk add --no-cache curl bash tzdata && \
    # Setup Cloudsmith repo
    if test "$(expr "$(echo "${VERSION}" | cut -d '.' -f 2)" % 2)" = 0; then \
        repo="kea-$(echo "${VERSION}" | cut -d '.' -f 1)-$(echo "${VERSION}" | cut -d '.' -f 2)"; \
    else \
        repo='kea-dev'; \
    fi && \
    curl -1sLf "https://dl.cloudsmith.io/public/isc/${repo}/setup.alpine.sh" | bash && \
    apk update && \
    # Install open-source Kea packages.
    apk add --no-cache \
        isc-kea-dhcp6~=${VERSION} \
        isc-kea-mysql~=${VERSION} \
        isc-kea-pgsql~=${VERSION} \
        isc-kea-hooks~=${VERSION} && \
    # If token is provided add premium Cloudsmith repository
    if [ -n "$TOKEN" ]; then \
        curl -1sLf "https://dl.cloudsmith.io/${TOKEN}/isc/${repo}-prv/setup.alpine.sh" | bash && \
        apk update && \
        # Install subscriber Kea hooks (provided TOKEN should have access to those pkgs)
        if [ "$PREMIUM" = "SUBSCRIBER" ]; then \
            apk add --no-cache \
                isc-kea-subscriber-cb-cmds~=${VERSION} \
                isc-kea-subscriber-rbac~=${VERSION}; \
        fi \
    fi && \
    # Revert Cloudsmith repositories
    mv /etc/apk/repositories_backup /etc/apk/repositories && \
    # remove curl and bash
    apk del curl bash


VOLUME ["/etc/kea", "/var/lib/kea/"]

COPY kea-dhcp6.conf /etc/kea/kea-dhcp6.conf
COPY hiddens /etc/kea/hiddens

# 8000 command control channel
# 8001 HA MT
# 547 DHCPv6
EXPOSE 8000-8001/tcp 547/tcp 547/udp

CMD ["/usr/sbin/kea-dhcp6", "-c", "/etc/kea/kea-dhcp6.conf"]
