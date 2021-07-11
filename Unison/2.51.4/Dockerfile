FROM alpine:3.13
LABEL org.opencontainers.image.authors="f.bovo@hobiri.com"

# Alpine doesn't ship with Bash.
RUN apk add --no-cache bash

# Install Unison from source with inotify support
ARG UNISON_VERSION=2.51.4
ARG OCAML_VERSION=4.12.0

RUN apk update \
    && apk add --no-cache --virtual .build-deps build-base coreutils \
    && wget http://caml.inria.fr/pub/distrib/ocaml-${OCAML_VERSION:0:4}/ocaml-${OCAML_VERSION}.tar.gz \
    && tar xvf ocaml-${OCAML_VERSION}.tar.gz -C /tmp \
    && cd /tmp/ocaml-${OCAML_VERSION} \
    && ./configure \
    && make world \
    && make opt \
    && umask 022 \
    && make install \
    && make clean \
    && apk del .build-deps  \
    && rm -rf /tmp/ocaml-${OCAML_VERSION} \
    && rm /ocaml-${OCAML_VERSION}.tar.gz

RUN apk add --no-cache --virtual .build-deps build-base curl && \
    apk add --no-cache inotify-tools && \
    curl -L https://github.com/bcpierce00/unison/archive/v$UNISON_VERSION.tar.gz | tar zxv -C /tmp && \
    cd /tmp/unison-${UNISON_VERSION} && \
    sed -i -e 's/GLIBC_SUPPORT_INOTIFY 0/GLIBC_SUPPORT_INOTIFY 1/' src/fsmonitor/linux/inotify_stubs.c && \
    make UISTYLE=text NATIVE=true STATIC=true && \
    cp src/unison src/unison-fsmonitor /usr/local/bin && \
    apk del .build-deps && \
    rm -rf /tmp/unison-${UNISON_VERSION}

ENV UNISON_USER="docker" \
    UNISON_GROUP="docker" \
    UNISON_UID="1000" \
    UNISON_GID="1000" \
    SYNC_SOURCE_BASE_PATH="/sync" \
    SYNC_DESTINATION_BASE_PATH="/var/www/html" \
    SYNC_PREFER="newer" \
    SYNC_SILENT="0" \
    SYNC_MAX_INOTIFY_WATCHES='' \
    SYNC_EXTRA_UNISON_PROFILE_OPTS='' \
    SYNC_NODELETE_SOURCE="0"

COPY ./docker-entrypoint.sh /usr/local/bin/docker-entrypoint.sh

COPY scripts/sync.sh /usr/local/bin/sync
COPY scripts/watch.sh /usr/local/bin/watch

ENTRYPOINT ["docker-entrypoint.sh"]