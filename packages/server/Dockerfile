FROM node:18-alpine AS build

WORKDIR /app

COPY package.json yarn.lock lerna.json tsconfig.json ./
COPY ./packages/lib/package.json ./packages/lib/package.json
COPY ./packages/server/package.json ./packages/server/package.json

RUN yarn install --frozen-lockfile --ignore-scripts > /dev/null

COPY ./packages/lib ./packages/lib
COPY ./packages/server ./packages/server

RUN yarn workspaces run build --update-main-to-dist > /dev/null

RUN yarn install --production --ignore-scripts --prefer-offline > /dev/null

WORKDIR /app/packages/server/service
RUN yarn install --frozen-lockfile > /dev/null

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
