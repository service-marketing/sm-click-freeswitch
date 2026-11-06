# syntax=docker/dockerfile:1
# vim:set ft=dockerfile:
#
# Full from-source build — zero dependency on SignalWire services or images.
# Deps not in Debian repos (spandsp4, sofia-sip >= 1.13.19) are built from
# their public GitHub sources before FreeSWITCH is compiled.
#
# Build time: ~30-60 min on a 4-core GitHub Actions runner.

# ---------------------------------------------------------------------------
# Stage 1: builder  (plain Debian — no SignalWire image)
# ---------------------------------------------------------------------------
FROM debian:bookworm AS builder

ARG SOFIA_SIP_TAG=v1.13.19

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update -qq && apt-get install -y --no-install-recommends \
    # toolchain
    build-essential autoconf automake libtool libtool-bin pkg-config cmake wget \
    # TLS / crypto
    libssl-dev \
    # audio / codecs
    libjpeg-dev libspeex-dev libspeexdsp-dev libopus-dev \
    libsndfile1-dev libasound2-dev libvorbis-dev libogg-dev \
    # scripting
    liblua5.2-dev libpython3-dev libperl-dev \
    # database / ODBC
    libpq-dev unixodbc-dev libgdbm-dev libdb-dev \
    # network / HTTP
    libcurl4-openssl-dev libldns-dev libpcre3-dev \
    libxml2-dev libxslt1-dev \
    # misc
    libedit-dev libtiff-dev zlib1g-dev uuid-dev \
    libyaml-dev libncurses5-dev libglib2.0-dev libmemcached-dev \
    # certs + locale
    ca-certificates locales \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

ENV LANG=en_US.utf8

# spandsp4 — not in Debian repos, required >= 3.1.1
# --no-check-certificate: CA path not resolved by wget in Buildkit on bookworm
RUN wget -q --no-check-certificate \
      https://github.com/freeswitch/spandsp/archive/refs/heads/master.tar.gz \
      -O /tmp/spandsp.tar.gz \
    && tar -xzf /tmp/spandsp.tar.gz -C /usr/src/ \
    && rm /tmp/spandsp.tar.gz \
    && cd /usr/src/spandsp-master \
    && ./bootstrap.sh \
    && ./configure \
    && make -j$(nproc) \
    && make install \
    && ldconfig

# sofia-sip >= 1.13.19
RUN SOFIA_CLEAN=$(echo "${SOFIA_SIP_TAG}" | sed 's/^v//') \
    && wget -q --no-check-certificate \
         "https://github.com/freeswitch/sofia-sip/archive/refs/tags/${SOFIA_SIP_TAG}.tar.gz" \
         -O /tmp/sofia-sip.tar.gz \
    && tar -xzf /tmp/sofia-sip.tar.gz -C /usr/src/ \
    && mv "/usr/src/sofia-sip-${SOFIA_CLEAN}" /usr/src/sofia-sip \
    && rm /tmp/sofia-sip.tar.gz \
    && cd /usr/src/sofia-sip \
    && ./autogen.sh \
    && ./configure.gnu \
    && make -j$(nproc) \
    && make install \
    && ldconfig

# FreeSWITCH from the fork source
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
# Stage 2: runtime  (plain Debian — only runtime libs)
# ---------------------------------------------------------------------------
FROM debian:bookworm AS runtime

ARG FREESWITCH_UID=499
ARG FREESWITCH_GID=499

ENV DEBIAN_FRONTEND=noninteractive \
    LANG=en_US.utf8 \
    PATH=/usr/local/freeswitch/bin:$PATH

RUN apt-get update -qq && apt-get install -y --no-install-recommends \
    libssl3 libjpeg62-turbo libspeex1 libspeexdsp1 libopus0 \
    libsndfile1 libasound2 libvorbis0a libogg0 liblua5.2-0 \
    libpq5 unixodbc libcurl4 libldns3 libpcre3 libxml2 libxslt1.1 \
    libedit2 libtiff6 zlib1g uuid-runtime libyaml-0-2 \
    libglib2.0-0 libmemcached11 \
    gosu locales ca-certificates \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8 \
    && rm -rf /var/lib/apt/lists/*

RUN groupadd -r freeswitch --gid=${FREESWITCH_GID} \
    && useradd -r -g freeswitch --uid=${FREESWITCH_UID} freeswitch

COPY --from=builder /usr/local/freeswitch    /usr/local/freeswitch
COPY --from=builder /usr/local/lib/libsofia-sip-ua*  /usr/local/lib/
COPY --from=builder /usr/local/lib/libspandsp*       /usr/local/lib/

RUN ldconfig

COPY docker/master/docker-entrypoint.sh /
COPY docker/master/build/freeswitch.limits.conf /etc/security/limits.d/

EXPOSE 8021/tcp
EXPOSE 5060/tcp 5060/udp 5080/tcp 5080/udp
EXPOSE 5061/tcp 5061/udp 5081/tcp 5081/udp
EXPOSE 5066/tcp 7443/tcp 8081/tcp 8082/tcp
EXPOSE 64535-65535/udp
EXPOSE 16384-32768/udp

VOLUME ["/etc/freeswitch"]
VOLUME ["/tmp"]

SHELL ["/bin/bash", "-c"]
HEALTHCHECK --interval=15s --timeout=5s \
    CMD /usr/local/freeswitch/bin/fs_cli -x status | grep -q ^UP || exit 1

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["freeswitch"]
