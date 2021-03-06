FROM hairyhenderson/gomplate:v3.7.0-slim AS gomplate
FROM docker:19.03.12 AS docker
FROM docker/compose:1.26.2 AS compose
FROM vault:1.4.3 AS vault
FROM hashicorp/terraform:0.12.28 AS terraform
FROM hashicorp/packer:1.6.0 AS packer
FROM prom/prometheus:v2.19.2 AS prometheus
FROM prom/alertmanager:v0.21.0 AS alertmanager
FROM mikefarah/yq:2.4.2 AS yq
FROM lachlanevenson/k8s-kubectl:v1.18.5 AS kubectl
FROM lachlanevenson/k8s-helm:v2.16.9 AS helm
FROM lachlanevenson/k8s-helm:v3.2.4 AS helm3
FROM r.j3ss.co/img:v0.5.10 AS img
FROM gcr.io/google.com/cloudsdktool/cloud-sdk:285.0.1-alpine AS google-cloud-sdk

FROM alpine:3.12.0 AS cc-test-reporter

RUN wget -q -O /bin/cc-test-reporter https://codeclimate.com/downloads/test-reporter/test-reporter-latest-linux-amd64
RUN chmod +x /bin/cc-test-reporter

FROM alpine:3.12.0 AS packages

RUN apk add --no-cache \
    make \
    git \
    libgcc \
    python3 \
    py3-pip \
    bash

ENV GLIBC=2.31-r0

RUN wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub

COPY awscli-requirements.txt requirements.txt
RUN pip install -r requirements.txt
RUN rm -rf requirements.txt ~/.cache

RUN wget https://github.com/sgerrand/alpine-pkg-glibc/releases/download/$GLIBC/glibc-$GLIBC.apk \
  && apk add --no-cache glibc-$GLIBC.apk \
  && rm glibc-$GLIBC.apk \
  && ln -s /lib/libz.so.1 /usr/glibc-compat/lib \
  && ln -s /lib/libc.musl-x86_64.so.1 /usr/glibc-compat/lib \
  && ln -s /usr/lib/libgcc_s.so.1 /usr/glibc-compat/lib

RUN mkdir hub && \
    wget https://github.com/github/hub/releases/download/v2.12.2/hub-linux-amd64-2.12.2.tgz -O- | \
    tar xz --strip-components 1 --directory hub \
    && ./hub/install \
    && rm -r hub

FROM alpine:3.12.0 AS build

COPY --from=gomplate /gomplate /bin/gomplate
COPY --from=docker /usr/local/bin/docker /bin/docker
COPY --from=vault /bin/vault /bin/vault
COPY --from=terraform /bin/terraform /bin/terraform
COPY --from=packer /bin/packer /bin/packer
COPY --from=prometheus /bin/promtool /bin/promtool
COPY --from=alertmanager /bin/amtool /bin/amtool
COPY --from=yq /usr/bin/yq /usr/bin/yq
COPY --from=compose /usr/local/bin/docker-compose /bin/docker-compose
COPY --from=kubectl /usr/local/bin/kubectl /usr/local/bin/kubectl
COPY --from=helm /usr/local/bin/helm /usr/local/bin/helm
COPY --from=helm3 /usr/local/bin/helm /usr/local/bin/helm3
COPY --from=cc-test-reporter /bin/cc-test-reporter /bin/cc-test-reporter
COPY --from=google-cloud-sdk /google-cloud-sdk/bin/ /usr/local/bin/
COPY --from=google-cloud-sdk /google-cloud-sdk/lib/ /usr/local/lib/
COPY --from=google-cloud-sdk /google-cloud-sdk/platform/ /usr/local/platform/
COPY --from=google-cloud-sdk /google-cloud-sdk/.install/ /usr/local/.install/
COPY --from=img /usr/bin/img /usr/bin/img

COPY --from=packages / /
