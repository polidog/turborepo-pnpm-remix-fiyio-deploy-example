# syntax = docker/dockerfile:1

# Adjust NODE_VERSION as desired
ARG NODE_VERSION=18.4.0

FROM node:${NODE_VERSION}-slim as base
ARG SCOPE

LABEL fly_launch_runtime="Remix"

# ENV NODE_ENV=production

# Install packages needed to build node modules
RUN apt-get update -qq && \
   apt-get install -y python-is-python3 pkg-config build-essential 

ENV PNPM_HOME="/root/.local/share/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN npm i -g pnpm

RUN pnpm add turbo -g


WORKDIR /app
COPY . .

# setup
FROM base as setup
ARG SCOPE

RUN turbo prune --scope=web --docker

# Install & Build packages
FROM base as build

COPY .gitignore .gitignore
COPY --from=setup /app/out/pnpm-lock.yaml ./pnpm-lock.yaml
COPY --from=setup /app/out/pnpm-workspace.yaml ./pnpm-workspace.yaml
RUN pnpm install --prod=false

COPY --from=setup /app/out/full/ .
COPY turbo.json turbo.json
RUN pnpm run build --filter=web

# Runner 
FROM base as runner
ARG SCOPE
ENV NODE_ENV=production

COPY .gitignore .gitignore
COPY --from=build /app/pnpm-lock.yaml ./pnpm-lock.yaml
COPY --from=build /app/pnpm-workspace.yaml ./pnpm-workspace.yaml

RUN pnpm install --prod --filter=web

COPY --from=build /app/apps/web ./apps/web

WORKDIR /app/apps/web
CMD [ "pnpm", "run", "start", "--filter", "web" ]