# This repo is a thin HA add-on wrapper — the actual app source lives in
# https://github.com/MarcelMuechler/vorrat. Cloning it at build time keeps
# that the single source of truth instead of vendoring a copy here.
# ponytail: no CI/registry pipeline yet, so version pinning is a git tag.
# Bump VORRAT_REF (and the version above) to the next tag on every release —
# this also busts Docker's build cache for the RUN below, since the resolved
# command text changes with it. A floating ref like "main" would cache-hit
# forever and silently keep serving whatever was cloned on the first build.
ARG VORRAT_REF=v0.17.1

# Supervisor no longer auto-injects BUILD_FROM (since 2026.04.0), so it needs
# an explicit default here. Declared before any FROM so it's in scope for
# both build stages below.
ARG BUILD_FROM=ghcr.io/home-assistant/base:3.23

FROM ghcr.io/cirruslabs/flutter:stable AS webbuilder
ARG VORRAT_REF
RUN git clone --depth 1 --branch "${VORRAT_REF}" https://github.com/MarcelMuechler/vorrat.git /src
# Matches the same marker main.py expects for the HA Ingress base-href
# rewrite (see the main repo's vorrat/Dockerfile) — without this flag,
# Flutter's default base href ships instead and Ingress breaks.
RUN cd /src/frontend && flutter build web --base-href=/__INGRESS_BASE__/

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
