FROM oven/bun:alpine AS bun

RUN set -xe; \
    mv /usr/local/bin/bun /usr/bin/bun; \
    mkdir /self; \
    ln -sfn /usr/bin/bun /self/exe

RUN set -xe; \
    mkdir -p /target/etc; \
    mkdir -p /blank; \
    apk --no-cache add \
      ca-certificates \
      tzdata \
    ; \
    update-ca-certificates; \
    ln -sf /usr/share/zoneinfo/Etc/UTC /target/etc/localtime; \
    echo "Etc/UTC" > /target/etc/timezone;

FROM scratch

COPY --from=bun /self /proc/self
COPY --from=bun /usr/bin/bun /usr/bin/bun

COPY --from=bun /lib/ld-musl-x86_64.so.1 /lib/ld-musl-x86_64.so.1
COPY --from=bun /usr/lib/libstdc++.so.6 /usr/lib/libstdc++.so.6
COPY --from=bun /lib/ld-musl-x86_64.so.1 /lib/ld-musl-x86_64.so.1
COPY --from=bun /usr/lib/libgcc_s.so.1 /usr/lib/libgcc_s.so.1

COPY --from=bun /target/etc /etc
COPY --from=bun /usr/share/zoneinfo/UTC /usr/share/zoneinfo/UTC
COPY --from=bun /usr/share/zoneinfo/Etc/UTC /usr/share/zoneinfo/Etc/UTC
COPY --from=bun /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/ca-certificates.crt
COPY --from=bun /blank /tmp

COPY ./server.ts /usr/src/server.ts
