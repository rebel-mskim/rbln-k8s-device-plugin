FROM golang:1.22-alpine AS builder

ARG http_proxy
ARG https_proxy

COPY . /usr/src/sriov-network-device-plugin

ENV HTTP_PROXY=$http_proxy
ENV HTTPS_PROXY=$https_proxy
RUN apk add --no-cache --virtual build-dependencies build-base

WORKDIR /usr/src/sriov-network-device-plugin
ENV CGO_ENABLED=0 GOOS=linux GOARCH=amd64
RUN make clean && \
    make build

FROM golang:1.20-alpine3.16 AS ddp-builder

ARG http_proxy
ARG https_proxy

ADD images/ddptool-1.0.1.12.tar.gz /tmp/ddptool/

ENV HTTP_PROXY=$http_proxy
ENV HTTPS_PROXY=$https_proxy
RUN apk add --no-cache --virtual build-dependencies build-base linux-headers

WORKDIR /tmp/ddptool
RUN make

FROM redhat/ubi9-minimal:9.6

ARG http_proxy
ARG https_proxy

RUN microdnf install -y \
      pciutils \
      hwdata \
  && microdnf clean all

COPY --from=builder     /usr/src/sriov-network-device-plugin/build/sriovdp /usr/bin/
COPY --from=ddp-builder /tmp/ddptool/ddptool /usr/bin/

COPY ./images/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

WORKDIR /

LABEL io.k8s.display-name="SRIOV Network Device Plugin (UBI9)"

ENTRYPOINT ["/entrypoint.sh"]
