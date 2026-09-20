FROM ubuntu:24.04

ARG YQ_VERSION=4.45.1
ARG NODE_MAJOR=20

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install --yes --no-install-recommends \
        bash \
        ca-certificates \
        coreutils \
        curl \
        gnupg \
        grep \
        mawk \
        openssl \
        sed \
        xorriso \
    && curl --fail --location --retry 3 \
        "https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key" \
        | gpg --dearmor --output /usr/share/keyrings/nodesource.gpg \
    && printf '%s\n' \
        "deb [signed-by=/usr/share/keyrings/nodesource.gpg] https://deb.nodesource.com/node_${NODE_MAJOR}.x nodistro main" \
        > /etc/apt/sources.list.d/nodesource.list \
    && apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install --yes --no-install-recommends nodejs \
    && curl --fail --location --retry 3 \
        --output /usr/local/bin/yq \
        "https://github.com/mikefarah/yq/releases/download/v${YQ_VERSION}/yq_linux_amd64" \
    && chmod 0755 /usr/local/bin/yq \
    && node --version \
    && yq --version \
    && rm -rf /var/lib/apt/lists/*
