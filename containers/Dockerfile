FROM docker.io/library/alpine:edge AS git
ARG TAG
ENV TAG=${TAG:-main}
RUN apk update && apk add --no-cache \
	git
WORKDIR /
RUN git -c advice.detachedHead=false clone --branch $TAG --depth=1 --recurse-submodules https://github.com/pfm-powerforme/seaweedfs.git source
WORKDIR /source/


FROM docker.io/library/golang:alpine AS builder
ARG TAG
ENV TAG=${TAG:-main}
ENV GO111MODULE=on \
    CGO_ENABLED=0 \
    GOOS=linux
WORKDIR /
COPY --from=git /source/ /build/
WORKDIR /build/
ARG TAGS="5BytesOffset,elastic,gocdk,rclone,sqlite,tarantool,tikv,ydb"
RUN apk update && apk add --no-cache \
  git 
RUN apk update && apk add --no-cache \
  g++ fuse
RUN export COMMIT="$(git rev-parse --short HEAD)"
WORKDIR /build/weed/
RUN go build -tags "$TAGS" -ldflags "-s -w -X 'weed/util/version.COMMIT=$COMMIT' -extldflags '-static'" -o weed


FROM docker.io/library/alpine:latest AS runtime
COPY --from=builder /build/weed/weed /usr/bin/weed
COPY --from=git /source/docker/filer.toml /etc/seaweedfs/filer.toml
COPY --from=git /source/docker/entrypoint.sh /entrypoint.sh
RUN apk update && apk add --no-cache \
  fuse
# volume server gprc port
EXPOSE 18080
# volume server http port
EXPOSE 8080
# filer server gprc port
EXPOSE 18888
# filer server http port
EXPOSE 8888
# master server shared gprc port
EXPOSE 19333
# master server shared http port
EXPOSE 9333
# s3 server http port
EXPOSE 8333
# webdav server http port
EXPOSE 7333
RUN mkdir -p /data/filerldb2
VOLUME /data
WORKDIR /data
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
