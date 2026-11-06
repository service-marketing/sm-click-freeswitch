# syntax=docker/dockerfile:1
# vim:set ft=dockerfile:
#
# Multi-stage build from source — zero dependency on SignalWire token.
# Stage 1 (builder): compiles sofia-sip + FreeSWITCH from source.
# Stage 2 (runtime): copies installed files + runtime libs only.

# ---------------------------------------------------------------------------
# Stage 1: builder
# ---------------------------------------------------------------------------
FROM debian:bookworm AS builder

ARG FREESWITCH_UID=499
ARG FREESWITCH_GID=499
ARG SOFIA_SIP_TAG=v1.13.17

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update -qq && apt-get install -y --no-install-recommends \
    # core build tools
    build-essential \
    autoconf \
    automake \
    libtool \
    libtool-bin \
    pkg-config \
    cmake \
    git \
    wget \
    curl \
    # crypto / TLS
    libssl-dev \
    # audio / codecs
    libjpeg-dev \
    libspeex-dev \
    libspeexdsp-dev \
    libopus-dev \
    libsndfile1-dev \
    libasound2-dev \
    libvorbis-dev \
    libogg-dev \
    libflac-dev \
    # scripting
    liblua5.2-dev \
    libpython3-dev \
    libperl-dev \
    # database / ODBC
    libpq-dev \
    unixodbc-dev \
    libgdbm-dev \
    libdb-dev \
    # networking / HTTP
    libcurl4-openssl-dev \
    libldns-dev \
    libpcre3-dev \
    libxml2-dev \
    libxslt1-dev \
    # misc runtime-needed
    libedit-dev \
    libtiff-dev \
    zlib1g-dev \
    uuid-dev \
    libyaml-dev \
    libncurses5-dev \
    libglib2.0-dev \
    libmemcached-dev \
    # locale
    locales \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

ENV LANG=en_US.utf8

# Build sofia-sip from source (public GitHub — no token)
RUN git clone --depth 1 --branch ${SOFIA_SIP_TAG} \
      https://github.com/freeswitch/sofia-sip.git /usr/src/sofia-sip \
    && cd /usr/src/sofia-sip \
    && ./autogen.sh \
    && ./configure.gnu \
    && make -j$(nproc) \
    && make install \
    && ldconfig

# Copy FreeSWITCH source from the fork (build context = repo root)
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
    # runtime shared libs (non-dev)
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
    # sofia-sip runtime
    libglib2.0-0 \
    # gosu for entrypoint privilege drop
    gosu \
    # locale
    locales \
    ca-certificates \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

RUN groupadd -r freeswitch --gid=${FREESWITCH_GID} \
    && useradd -r -g freeswitch --uid=${FREESWITCH_UID} freeswitch

# Copy FreeSWITCH install and sofia-sip libs from builder
COPY --from=builder /usr/local/freeswitch /usr/local/freeswitch
COPY --from=builder /usr/local/lib/libsofia-sip-ua* /usr/local/lib/
COPY --from=builder /usr/local/lib/pkgconfig/sofia-sip-ua* /usr/local/lib/pkgconfig/

RUN ldconfig

COPY docker/master/docker-entrypoint.sh /
COPY docker/master/build/freeswitch.limits.conf /etc/security/limits.d/

## Ports
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
