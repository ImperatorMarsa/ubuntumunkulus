FROM ubuntu:24.04

ARG YQ_VERSION=4.45.1

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install --yes --no-install-recommends \
        bash \
        ca-certificates \
        coreutils \
        curl \
        grep \
        mawk \
        openssl \
        sed \
        xorriso \
    && curl --fail --location --retry 3 \
        --output /usr/local/bin/yq \
        "https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_linux_amd64" \
    && chmod 0755 /usr/local/bin/yq \
    && yq --version \
    && rm -rf /var/lib/apt/lists/*
