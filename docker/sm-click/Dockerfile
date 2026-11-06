# syntax=docker/dockerfile:1
# vim:set ft=dockerfile:
#
# Multi-stage build from source — zero dependency on SignalWire token.
#
# Stage 1 (builder): signalwire/freeswitch-public-ci-base is a PUBLIC image
# (no credentials) that ships all build deps (spandsp4, sofia-sip, etc.)
# pre-installed. We only compile the FreeSWITCH source from this fork.
#
# Stage 2 (runtime): copies installed binaries + installs runtime libs.

# ---------------------------------------------------------------------------
# Stage 1: builder
# ---------------------------------------------------------------------------
FROM signalwire/freeswitch-public-ci-base:bookworm-amd64 AS builder

COPY . /usr/src/freeswitch
WORKDIR /usr/src/freeswitch

RUN ./bootstrap.sh -j \
    && ./configure \
         --prefix=/usr/local/freeswitch \
         --exec-prefix=/usr/local/freeswitch \
    && make -j$(nproc) \
    && make install \
    && make sounds-install \
    && make moh-install

# ---------------------------------------------------------------------------
# Stage 2: runtime
# ---------------------------------------------------------------------------
FROM debian:bookworm AS runtime

ARG FREESWITCH_UID=499
ARG FREESWITCH_GID=499

ENV DEBIAN_FRONTEND=noninteractive \
    LANG=en_US.utf8 \
    PATH=/usr/local/freeswitch/bin:$PATH

RUN apt-get update -qq && apt-get install -y --no-install-recommends \
    libssl3 \
    libjpeg62-turbo \
    libspeex1 \
    libspeexdsp1 \
    libopus0 \
    libsndfile1 \
    libasound2 \
    libvorbis0a \
    libogg0 \
    liblua5.2-0 \
    libpq5 \
    unixodbc \
    libcurl4 \
    libldns3 \
    libpcre3 \
    libxml2 \
    libxslt1.1 \
    libedit2 \
    libtiff6 \
    zlib1g \
    uuid-runtime \
    libyaml-0-2 \
    libglib2.0-0 \
    libmemcached11 \
    gosu \
    locales \
    ca-certificates \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

RUN groupadd -r freeswitch --gid=${FREESWITCH_GID} \
    && useradd -r -g freeswitch --uid=${FREESWITCH_UID} freeswitch

COPY --from=builder /usr/local/freeswitch /usr/local/freeswitch
COPY --from=builder /usr/local/lib/libsofia-sip-ua* /usr/local/lib/
COPY --from=builder /usr/local/lib/pkgconfig/sofia-sip-ua* /usr/local/lib/pkgconfig/

RUN ldconfig

COPY docker/master/docker-entrypoint.sh /
COPY docker/master/build/freeswitch.limits.conf /etc/security/limits.d/

EXPOSE 8021/tcp
EXPOSE 5060/tcp 5060/udp 5080/tcp 5080/udp
EXPOSE 5061/tcp 5061/udp 5081/tcp 5081/udp
EXPOSE 5066/tcp
EXPOSE 7443/tcp
EXPOSE 8081/tcp 8082/tcp
EXPOSE 64535-65535/udp
EXPOSE 16384-32768/udp

VOLUME ["/etc/freeswitch"]
VOLUME ["/tmp"]

SHELL ["/bin/bash", "-c"]
HEALTHCHECK --interval=15s --timeout=5s \
    CMD /usr/local/freeswitch/bin/fs_cli -x status | grep -q ^UP || exit 1

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["freeswitch"]
