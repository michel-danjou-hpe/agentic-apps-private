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
    uv sync --frozen --no-dev --no-install-project --package noa-user-proxy --no-install-workspace

# Copy code into builder
COPY . /src
WORKDIR /src

# Install just the noa-user-proxy package
RUN --mount=type=cache,target=/root/.cache/uv \
  uv sync --frozen --no-dev --no-editable --package noa-user-proxy

FROM python:3.13-slim-bookworm AS runner

# Add Zscaler CA certificate for corporate proxy (needed for runtime HTTPS requests)
COPY zscaler-ca.cr[t] /usr/local/share/ca-certificates/
RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates && \
    update-ca-certificates && \
    rm -rf /var/lib/apt/lists/*

RUN useradd -m user

# Copy venv from builder with just the dependencies we need + our package
COPY --from=builder --chown=user:user /app /app

USER user

CMD ["/app/bin/noa-user-proxy"]
