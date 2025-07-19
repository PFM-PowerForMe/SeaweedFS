FROM golang:1-alpine AS builder
ARG TAG
ENV TAG=${TAG:-main}
ARG TAGS="5BytesOffset,elastic,gocdk,rclone,sqlite,tarantool,tikv,ydb"
RUN apk update && apk add --no-cache \
  git 
RUN apk update && apk add --no-cache \
  g++ fuse
RUN mkdir -p /go/src/github.com/pfm-powerforme/
RUN git -c advice.detachedHead=false clone --branch $TAG --depth=1 --recurse-submodules --shallow-submodules https://github.com/pfm-powerforme/seaweedfs.git /go/src/github.com/pfm-powerforme/seaweedfs
RUN cd /go/src/github.com/pfm-powerforme/seaweedfs/weed \
  && export LDFLAGS="-X github.com/pfm-powerforme/seaweedfs/weed/util/version.COMMIT=$(git rev-parse --short HEAD)" \
  && CGO_ENABLED=0 go install -tags "$TAGS" -ldflags "-extldflags -static ${LDFLAGS}"


FROM alpine:latest AS runtime
RUN mkdir -p /etc/seaweedfs
COPY --from=builder /go/bin/weed /usr/bin/
COPY --from=builder /go/src/github.com/pfm-powerforme/seaweedfs/docker/filer.toml /etc/seaweedfs/filer.toml
COPY --from=builder /go/src/github.com/pfm-powerforme/seaweedfs/docker/entrypoint.sh /entrypoint.sh
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
