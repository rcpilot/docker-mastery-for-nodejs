## Stage 1 (production base)
FROM node:18-slim as base

LABEL org.opencontainers.image.authors=cgreen@coretag.net
LABEL org.opencontainers.image.title="Ultimate Dockerfile"
LABEL org.opencontainers.image.url=https://hub.docker.com/r/coretag/ultimate-dockerfile
LABEL org.opencontainers.image.source=https://github.com/rcpilot/udemy-docker-mastery-for-nodejs
LABEL org.opencontainers.image.licenses=MIT
LABEL com.coretag.nodeversion=$NODE_VERSION

ENV NODE_ENV=production

EXPOSE 80
ENV PORT 80

RUN apt-get update -qq && apt-get install -qy \ 
    tini \
    --no-install-recommends \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app

COPY package*.json ./

RUN npm config list \
    && npm ci \
    && npm cache clean --force

ENV PATH /app/node_modules/.bin:$PATH

ENTRYPOINT ["/usr/bin/tini", "--"]

CMD ["node", "server.js"]

## Stage 2 (development)
FROM base as dev

ENV NODE_ENV=development

# NOTE: these apt dependencies are only needed
# for testing. they shouldn't be in production
RUN apt-get update -qq \
    && apt-get install -qy --no-install-recommends \
    bzip2 \
    ca-certificates \
    curl \
    libfontconfig \
    && rm -rf /var/lib/apt/lists/*
    
RUN npm config list

RUN npm install \
    && npm cache clean --force

USER node

CMD ["nodemon", "server.js"]

## Stage 3 (testing)
FROM dev as test

USER root

COPY . .

RUN npm audit

## Stage 4 (prune test files for production)
FROM test as clean
RUN rm -rf ./tests && rm -rf ./node_modules

## Stage 5 (production)
FROM base as prod

COPY --from=clean /app /app

HEALTHCHECK --interval=5m --timeout=3s \
  CMD curl -f http://localhost/ || exit 1

USER node