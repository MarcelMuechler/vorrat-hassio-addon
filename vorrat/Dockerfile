# This repo is a thin HA add-on wrapper — the actual app source lives in
# https://github.com/MarcelMuechler/vorrat. Cloning it at build time keeps
# that the single source of truth instead of vendoring a copy here.
# ponytail: no CI/registry pipeline yet, so version pinning is just a git ref.
# Bump VORRAT_REF (and the version above) to track a specific release once
# the main repo starts tagging them.
ARG VORRAT_REF=main

# Supervisor no longer auto-injects BUILD_FROM (since 2026.04.0), so it needs
# an explicit default here. Declared before any FROM so it's in scope for
# both build stages below.
ARG BUILD_FROM=ghcr.io/home-assistant/base:3.23

FROM ghcr.io/cirruslabs/flutter:stable AS webbuilder
ARG VORRAT_REF
RUN git clone --depth 1 --branch "${VORRAT_REF}" https://github.com/MarcelMuechler/vorrat.git /src
RUN cd /src/frontend && flutter build web

FROM ${BUILD_FROM} AS runtime

RUN apk add --no-cache python3 py3-pip \
    && pip install --break-system-packages --no-cache-dir \
        fastapi "uvicorn[standard]" sqlalchemy alembic httpx pydantic-settings

WORKDIR /app
COPY --from=webbuilder /src/backend /app
COPY --from=webbuilder /src/frontend/build/web /app/app/static
COPY rootfs /

LABEL \
    org.opencontainers.image.title="Vorrat" \
    org.opencontainers.image.description="Self-hosted grocery & stock management" \
    org.opencontainers.image.source="https://github.com/MarcelMuechler/vorrat" \
    org.opencontainers.image.licenses="MIT"
