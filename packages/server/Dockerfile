FROM node:18-alpine AS deps

WORKDIR /app

COPY package.json yarn.lock lerna.json tsconfig.json ./
COPY ./packages/lib/package.json ./packages/lib/package.json
COPY ./packages/server/package.json ./packages/server/package.json
COPY ./packages/server/service/package.json ./packages/server/service/package.json
COPY ./packages/server/service/yarn.lock ./packages/server/service/yarn.lock

RUN --mount=type=cache,target=/root/.yarn,sharing=locked \
    --mount=type=cache,target=/app/node_modules/.cache,sharing=locked \
    yarn install --frozen-lockfile --ignore-scripts > /dev/null

FROM node:18-alpine AS build

WORKDIR /app

COPY --from=deps /app ./

COPY ./packages/lib ./packages/lib
COPY ./packages/server ./packages/server

RUN --mount=type=cache,target=/app/node_modules/.cache,sharing=locked \
    yarn workspaces run build > /dev/null && \
    yarn workspace @did-plc/server run update-main-to-dist

RUN --mount=type=cache,target=/root/.yarn,sharing=locked \
    --mount=type=cache,target=/app/node_modules/.cache,sharing=locked \
    yarn install --production --ignore-scripts --prefer-offline > /dev/null

WORKDIR /app/packages/server/service
RUN --mount=type=cache,target=/root/.yarn,sharing=locked \
    yarn install --frozen-lockfile > /dev/null

# Uses assets from build stage to reduce build size
FROM node:18-alpine

# RUN npm install -g yarn
RUN apk add --update dumb-init

# Avoid zombie processes, handle signal forwarding
ENTRYPOINT ["dumb-init", "--"]

WORKDIR /app/packages/server/service
COPY --from=build /app /app

EXPOSE 3000
ENV PORT=3000
ENV NODE_ENV=production

# https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md#non-root-user
USER node
CMD ["node", "--enable-source-maps", "index.js"]

LABEL org.opencontainers.image.source=https://github.com/did-method-plc/did-method-plc
LABEL org.opencontainers.image.description="DID PLC server"
LABEL org.opencontainers.image.licenses=MIT
