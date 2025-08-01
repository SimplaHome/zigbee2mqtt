ARG TARGETPLATFORM

FROM alpine:3.22 AS base

ENV NODE_ENV=production
WORKDIR /app
RUN apk add --no-cache tzdata eudev tini nodejs

# Dependencies and build
FROM base AS deps

COPY package.json pnpm-lock.yaml ./
# Make and such are needed to compile serialport for riscv64
RUN apk add make gcc g++ python3 linux-headers npm && \
    npm install -g pnpm@$(sed -n 's/.*"packageManager": "pnpm@\([0-9.]*\)".*/\1/p' package.json) && \
    pnpm install --frozen-lockfile --no-optional && \
    # serialport has outdated prebuilds that appear to fail on some archs, force build on target platform
    rm -rf `find ./node_modules/.pnpm/ -wholename "*/@serialport/bindings-cpp/prebuilds" -type d` && \
    pnpm rebuild @serialport/bindings-cpp

# Release
FROM base AS release

ARG DATE
ARG VERSION
LABEL org.opencontainers.image.authors="Koen Kanters"
LABEL org.opencontainers.image.title="Zigbee2MQTT"
LABEL org.opencontainers.image.description="Zigbee to MQTT bridge using Zigbee-herdsman"
LABEL org.opencontainers.image.url="https://github.com/Koenkk/zigbee2mqtt"
LABEL org.opencontainers.image.documentation="https://www.zigbee2mqtt.io/"
LABEL org.opencontainers.image.source="https://github.com/Koenkk/zigbee2mqtt"
LABEL org.opencontainers.image.licenses="GPL-3.0"
LABEL org.opencontainers.image.created=${DATE}
LABEL org.opencontainers.image.version=${VERSION}

COPY --from=deps /app/node_modules ./node_modules
COPY dist ./dist
COPY package.json LICENSE index.js data/configuration.example.yaml ./

COPY docker/docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

RUN mkdir /app/data

ARG COMMIT
RUN echo "$COMMIT" > dist/.hash

ENTRYPOINT ["docker-entrypoint.sh"]
CMD [ "/sbin/tini", "--", "node", "index.js"]
