# syntax=docker/dockerfile:1
# vim:set ft=dockerfile:
ARG DEBIAN_VERSION=bookworm
FROM debian:${DEBIAN_VERSION}

ARG DEBIAN_VERSION
ARG FS_META_PACKAGE=freeswitch-meta-all
ARG FREESWITCH_UID=499
ARG FREESWITCH_GID=499

RUN groupadd -r freeswitch --gid=${FREESWITCH_GID} \
    && useradd -r -g freeswitch --uid=${FREESWITCH_UID} freeswitch

RUN apt-get update -qq \
    && apt-get install -y --no-install-recommends ca-certificates gnupg2 gosu locales wget \
    && localedef -i en_US -c -f UTF-8 -A /usr/share/locale/locale.alias en_US.UTF-8

ENV LANG en_US.utf8

RUN --mount=type=secret,id=signalwire_token \
    TOKEN=$(cat /run/secrets/signalwire_token) \
    && wget --no-verbose --http-user=signalwire --http-password="${TOKEN}" \
         -O /usr/share/keyrings/signalwire-freeswitch-repo.gpg \
         https://freeswitch.signalwire.com/repo/deb/debian-release/signalwire-freeswitch-repo.gpg \
    && echo "machine freeswitch.signalwire.com login signalwire password ${TOKEN}" > /etc/apt/auth.conf \
    && echo "deb [signed-by=/usr/share/keyrings/signalwire-freeswitch-repo.gpg] https://freeswitch.signalwire.com/repo/deb/debian-release/ ${DEBIAN_VERSION} main" \
         > /etc/apt/sources.list.d/freeswitch.list \
    && apt-get -qq update \
    && apt-get install -y ${FS_META_PACKAGE} \
    && apt-get purge -y --auto-remove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && rm -f /etc/apt/auth.conf

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
    CMD fs_cli -x status | grep -q ^UP || exit 1

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["freeswitch"]
