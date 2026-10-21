# syntax=docker/dockerfile:1.7

FROM oven/bun:1.3.13-slim

WORKDIR /repo
ENV NODE_ENV=production

# Patched system CA bundle (>= 20250419+deb13u1 distrusts the eMudhra-chained
# roots disclosed on 2026-10-20). Bun's fetch/TLS still uses its own bundled
# Mozilla store, which keeps trusting "emSign ECC Root CA - G3" until a Bun
# release drops it; --use-system-ca only adds to that store, so it is not set.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends ca-certificates \
    && dpkg --compare-versions "$(dpkg-query -W -f='${Version}' ca-certificates)" ge 20250419+deb13u1

RUN --mount=type=cache,target=/root/.bun/install/cache,sharing=locked \
    bun install -g pnpm@10.28.1

# Workspace manifests first so dependency install is a cached layer. teamsbot
# depends on the @centaur/* workspace packages, so they must be present for
# pnpm to link the `workspace:*` ranges.
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY patches/ patches/
COPY packages/ packages/
COPY services/teamsbot/package.json services/teamsbot/package.json
RUN --mount=type=cache,target=/root/.local/share/pnpm/store,sharing=locked \
    pnpm install --filter teamsbot --prod --frozen-lockfile

COPY services/teamsbot/ services/teamsbot/
WORKDIR /repo/services/teamsbot

EXPOSE 3100
ENV PORT=3100
USER bun
CMD ["bun", "src/server.ts"]
