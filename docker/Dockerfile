ARG SERVE_U=amd64
ARG SERVE_R=alpine

### Build ###
FROM golang:1.11 AS builder

ARG BUILD_GOOS=linux
ARG BUILD_GOARCH=amd64

WORKDIR /go/src/github.com/syncthing/syncthing
COPY . .

ENV CGO_ENABLED=0
ENV BUILD_HOST=syncthing.net
ENV BUILD_USER=docker

RUN go run build.go -no-upgrade -goos=$BUILD_GOOS -goarch=$BUILD_GOARCH build

### Serve ###
FROM $SERVE_U/$SERVE_R

EXPOSE 8384 22000 21027/udp

VOLUME ["/var/syncthing"]

RUN apk add --no-cache ca-certificates su-exec

COPY --from=builder /go/src/github.com/syncthing/syncthing/syncthing /bin/syncthing

ENV PUID=1000 PGID=1000

HEALTHCHECK --interval=1m --timeout=10s \
  CMD nc -z localhost 8384 || exit 1

ENTRYPOINT \
  chown "${PUID}:${PGID}" /var/syncthing \
  && su-exec "${PUID}:${PGID}" \
     env HOME=/var/syncthing \
     /bin/syncthing \
       -home /var/syncthing/config \
       -gui-address 0.0.0.0:8384
