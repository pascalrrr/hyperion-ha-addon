ARG BUILD_FROM
FROM ${BUILD_FROM:-ghcr.io/hassio-addons/debian-base/amd64:7.8.3} AS build

ARG BUILD_VERSION

WORKDIR /tmp
COPY patches/*.patch patches/

RUN apt-get update && apt-get install --no-upgrade \
    git cmake build-essential ninja-build qt6-base-dev \
    qt6-serialport-dev qt6-websockets-dev libxkbcommon-dev \
    libvulkan-dev libgl1-mesa-dev libusb-1.0-0-dev python3-dev \
    libasound2-dev libturbojpeg0-dev libjpeg-dev libssl-dev \
    libcec-dev libp8-platform-dev libudev-dev pkg-config \
    libftdi1-dev -y --no-install-recommends

# based on the AUR PKGBUILD for hyperion.ng, for the original see:
# https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=hyperion.ng-git
RUN git clone --branch ${BUILD_VERSION} https://github.com/hyperion-project/hyperion.ng.git hyperion/ && \
	for i in /tmp/patches/*.patch; do \
    patch -d /tmp/hyperion -p1 < "${i}"; done
WORKDIR /tmp/hyperion
RUN git submodule update --init --recursive && \
	test -d build || mkdir build
WORKDIR /tmp/hyperion/build
RUN	test -e bin/hyperiond || cmake -DCMAKE_BUILD_TYPE=Release .. && make

FROM ${BUILD_FROM:-ghcr.io/hassio-addons/debian-base/amd64:7.8.3} AS final

WORKDIR /data

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# final dependencies
RUN apt-get update && apt-get install --no-upgrade \
    nginx --no-install-recommends -y && \
	apt-get update && apt-get install --no-upgrade \
    libturbojpeg0 libasound2-dev qt6-serialport-dev \
    qt6-websockets-dev libpython3-dev libqt6sql6-sqlite \
    libcec-dev --no-install-recommends -y && \
	rm -rf /var/lib/apt/lists/*

COPY --from=build /tmp/hyperion/build/bin/* /usr/bin
RUN install -d /usr/share/hyperion/webconfig
COPY --from=build /tmp/hyperion/assets /usr/share/hyperion

COPY rootfs/ /
RUN chmod +x /etc/s6-overlay/s6-rc.d/*/*

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