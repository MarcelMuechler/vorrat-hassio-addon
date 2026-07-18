# This repo is a thin HA add-on wrapper — the actual app source lives in
# https://github.com/MarcelMuechler/vorrat. Cloning it at build time keeps
# that the single source of truth instead of vendoring a copy here.
# ponytail: no CI/registry pipeline yet, so version pinning is a git tag.
# Bump VORRAT_REF (and the version above) to the next tag on every release —
# this also busts Docker's build cache for the RUN below, since the resolved
# command text changes with it. A floating ref like "main" would cache-hit
# forever and silently keep serving whatever was cloned on the first build.
ARG VORRAT_REF=v0.20.0

# Supervisor no longer auto-injects BUILD_FROM (since 2026.04.0), so it needs
# an explicit default here. Declared before any FROM so it's in scope for
# both build stages below.
ARG BUILD_FROM=ghcr.io/home-assistant/base:3.23

# Cloned once and reused by both the web build and the backend dependency
# resolution below, so the pinned tag is only fetched from GitHub a single
# time per build.
FROM ${BUILD_FROM} AS source
RUN apk add --no-cache git
ARG VORRAT_REF
RUN git clone --depth 1 --branch "${VORRAT_REF}" https://github.com/MarcelMuechler/vorrat.git /src

FROM ghcr.io/cirruslabs/flutter:stable AS webbuilder
COPY --from=source /src/frontend /src/frontend
# Matches the same marker main.py expects for the HA Ingress base-href
# rewrite (see the main repo's vorrat/Dockerfile) — without this flag,
# Flutter's default base href ships instead and Ingress breaks.
RUN cd /src/frontend && flutter build web --base-href=/__INGRESS_BASE__/

# uv's official distroless image ships a statically-linked (musl-compatible)
# binary — copying just the executable out of it is the documented way to
# add uv to another base image.
FROM ghcr.io/astral-sh/uv:0.11.28@sha256:0f36cb9361a3346885ca3677e3767016687b5a170c1a6b88465ec14aefec90aa AS uv

# Resolve the backend's *locked* dependency set (backend/uv.lock at the
# pinned tag) into a venv instead of a hand-maintained pip install list here
# — a hardcoded list silently drifts from the real dependencies, which is
# exactly what broke python-multipart after the main repo added it (#210,
# #212) but this Dockerfile's old list was never updated to match.
FROM ${BUILD_FROM} AS backend-builder
RUN apk add --no-cache python3
COPY --from=uv /uv /usr/local/bin/uv
WORKDIR /app
COPY --from=source /src/backend/pyproject.toml /src/backend/uv.lock /app/
RUN uv sync --frozen --no-install-project --no-dev

FROM ${BUILD_FROM} AS runtime

RUN apk upgrade --no-cache && apk add --no-cache python3

WORKDIR /app
COPY --from=source /src/backend /app
COPY --from=backend-builder /app/.venv /app/.venv
ENV PATH="/app/.venv/bin:${PATH}"
COPY --from=webbuilder /src/frontend/build/web /app/app/static
COPY rootfs /

LABEL \
    org.opencontainers.image.title="Vorrat" \
    org.opencontainers.image.description="Self-hosted grocery & stock management" \
    org.opencontainers.image.source="https://github.com/MarcelMuechler/vorrat" \
    org.opencontainers.image.licenses="MIT"
