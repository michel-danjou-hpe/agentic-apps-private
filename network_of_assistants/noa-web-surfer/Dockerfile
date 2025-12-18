# Copyright AGNTCY Contributors (https://github.com/agntcy)
# SPDX-License-Identifier: Apache-2.0

FROM python:3.13-slim-bookworm AS builder

ENV PYTHONUNBUFFERED=1
ENV PYTHONDONTWRITEBYTECODE=1

COPY --from=ghcr.io/astral-sh/uv:0.5.26 /uv /bin/uv

# Add Zscaler CA certificate for corporate proxy (optional - wildcard handles missing file)
COPY zscaler-ca.cr[t] /usr/local/share/ca-certificates/
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates && \
    update-ca-certificates && \
    rm -rf /var/lib/apt/lists/*

ENV UV_LINK_MODE=copy \
    UV_COMPILE_BYTECODE=1 \
    UV_PYTHON_DOWNLOADS=never \
    UV_PYTHON=python3.13 \
    UV_PROJECT_ENVIRONMENT=/app

# Install dependencies
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --frozen --no-dev --no-install-project --package noa-web-surfer --no-install-workspace

# Copy code into builder
COPY . /src
WORKDIR /src

# Install just the noa-web-surfer package
RUN --mount=type=cache,target=/root/.cache/uv \
  uv sync --frozen --no-dev --no-editable --package noa-web-surfer

FROM python:3.13-slim-bookworm AS runner

# Add Zscaler CA certificate for corporate proxy (needed for playwright install)
COPY zscaler-ca.cr[t] /usr/local/share/ca-certificates/
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates && \
    update-ca-certificates && \
    rm -rf /var/lib/apt/lists/*

# Set NODE_EXTRA_CA_CERTS for Node.js (used by Playwright)
ENV NODE_EXTRA_CA_CERTS=/etc/ssl/certs/ca-certificates.crt

RUN useradd -m user

# Copy venv from builder with just the dependencies we need + our package
COPY --from=builder --chown=user:user /app /app

# Ensure the environment is set up for the user
ENV PATH="/app/bin:${PATH}"

RUN playwright install-deps

USER user

# Install playwright dependencies
RUN playwright install

CMD ["/app/bin/noa-web-surfer"]