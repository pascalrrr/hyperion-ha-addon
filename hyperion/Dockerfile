ARG BUILD_FROM
FROM ${BUILD_FROM:-ghcr.io/home-assistant/amd64-base-debian:bookworm}

# Static build args
ARG TARGETARCH
ARG HYPERION_VERSION="2.1.1"
ARG HYPERION_URL="https://github.com/hyperion-project/hyperion.ng"
ARG OPENSSL_VERSION="1.1.1w-0+deb11u4"
ARG LIBSSL_VERSION="1.1.1w-0+deb11u4"

WORKDIR /tmp
COPY patches/*.patch patches/

# Primary dependency step
# hadolint ignore=DL3008
RUN apt-get update && apt-get install --no-upgrade \
    nginx wget git patch libpython3.11 libturbojpeg0 \
    libasound2 libusb-1.0-0 libftdi1 libcec6 libcec-dev \
    libqt5serialport5 libqt5websockets5 libqt5sql5-sqlite \
    libqt5svg5  -y --no-install-recommends && \
    rm -rf /var/lib/apt/lists/* && \

# Pull lib/openssl1.1 because Hyperion is built against these
RUN wget http://security.debian.org/debian-security/pool/updates/main/o/openssl/openssl_${OPENSSL_VERSION}_${TARGETARCH}.deb \
    --progress=dot:giga -O /tmp/openssl1.1.deb && \
    wget http://security.debian.org/debian-security/pool/updates/main/o/openssl/libssl1.1_${LIBSSL_VERSION}_${TARGETARCH}.deb \
    --progress=dot:giga -O /tmp/libssl1.1.deb && \
    apt-get install --no-upgrade \
    /tmp/libssl1.1.deb /tmp/openssl1.1.deb \
    -y --no-install-recommends --allow-downgrades

# Download and patch Hyperion Web UI assets
RUN git clone --branch ${HYPERION_VERSION} --depth 1 ${HYPERION_URL}.git hyperion/ && \
	for i in /tmp/patches/*.patch; do \
    patch -d /tmp/hyperion -p1 < "${i}"; done && \
    # Determine build architechure for Hyperion
    if [ "${TARGETARCH}" = "arm64" ]; then HYPERION_ARCH=arm64; fi && \
    if [ "${TARGETARCH}" = "amd64" ]; then HYPERION_ARCH=x86_64; fi && \
    # Download precompiled binaries for Hyperion
    wget ${HYPERION_URL}/releases/download/${HYPERION_VERSION}/Hyperion-${HYPERION_VERSION}-Linux-$HYPERION_ARCH.deb \
    --progress=dot:giga -O /tmp/Hyperion.deb

# Cleanup step
# hadolint ignore=DL3008
RUN apt-get update && apt-get install --no-upgrade \
    /tmp/Hyperion.deb \
    -y --no-install-recommends && \
    apt-get remove wget git -y && \
    apt-get autoremove --purge -y && \
    rm -rf /var/lib/apt/lists/* && \
    cp -r /tmp/hyperion/assets/* \
    /usr/share/hyperion/ && \
    rm -rf /tmp/*


# Static configuration
COPY rootfs/ /
RUN chmod +x /etc/s6-overlay/s6-rc.d/*/*

WORKDIR /data

SHELL ["/bin/bash", "-o", "pipefail", "-c"]
HEALTHCHECK --interval=5s  --timeout=3s --retries=3 \
    CMD curl --fail http://127.0.0.1:8090/health || exit 1


ARG BUILD_VERSION
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY

LABEL \
    io.hass.name=${BUILD_NAME} \
    io.hass.description=${BUILD_DESCRIPTION} \
    io.hass.arch=${BUILD_ARCH} \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Cayden (Kay) Haun <me@pascalrr.gay>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Pascalrr" \
    org.opencontainers.image.authors="Cayden (Kay) Haun <me@pascalrr.gay>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://pascalrr.gay/hyperion-ha-addon" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}