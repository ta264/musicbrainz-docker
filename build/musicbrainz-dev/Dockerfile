ARG METABRAINZ_BASE_IMAGE_VERSION=focal-1.0.0-alpha1
ARG METABRAINZ_BASE_IMAGE_VARIANT=
ARG METABRAINZ_BASE_IMAGE_TAG=${METABRAINZ_BASE_IMAGE_VERSION}${METABRAINZ_BASE_IMAGE_VARIANT}
FROM metabrainz/base-image:${METABRAINZ_BASE_IMAGE_TAG}

ARG METABRAINZ_BASE_IMAGE_VERSION
ARG METABRAINZ_BASE_IMAGE_VARIANT
ARG METABRAINZ_BASE_IMAGE_TAG
LABEL org.metabrainz.based-on-image="metabrainz/base-image:${METABRAINZ_BASE_IMAGE_TAG}"

ARG APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=1
ARG DEBIAN_FRONTEND=noninteractive

ARG DOCKERIZE_VERSION=v0.6.1
RUN curl -sSLO --retry 5 https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz && \
    tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz && \
    rm -f dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

ARG NODE_VERSION=16.1.0
ARG PGP_SERVERS="ha.pool.sks-keyservers.net ipv4.pool.sks-keyservers.net p80.pool.sks-keyservers.net:80"
ARG POSTGRES_VERSION=12
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN curl -sSL --retry 5 https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - && \
    echo "deb http://apt.postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list && \
    curl -sSL --retry 5 https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
    \
    # Refresh Yarn signing keys
    refreshed_keys=''; \
    for pgp_server in $(tr ' ' '\n' <<<"$PGP_SERVERS"); do \
        for attempt in {1..3}; do \
            apt-key adv --keyserver "$pgp_server" --refresh-keys Yarn --textmode 2>&1 && \
            { refreshed_keys='yes'; break; } || \
            echo "Temporary failure: apk-key/gpg returned error code '$?' on attempt #$attempt to reach '$pgp_server'."; \
        done; \
        if [[ $refreshed_keys == yes ]]; then break; fi; \
    done; \
    if [[ $refreshed_keys != yes ]]; then \
        echo >&2 'Fatal error: Failed all attempts to refresh PGP keys.'; \
        echo >&2 'Try passing a list of PGP servers that work for you as build argument:'; \
        echo >&2 ''; \
        echo >&2 '    docker-compose build --build-arg PGP_SERVERS="LIST" musicbrainz'; \
        echo >&2 ''; \
        echo >&2 "Current LIST is \"$PGP_SERVERS\""; \
        EX_TEMPFAIL=75; \
        exit $EX_TEMPFAIL; \
    fi; \
    unset attempt pgp_server refreshed_keys; \
    \
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
    apt-get update && \
    apt-get install --no-install-recommends -qy \
        python2-minimal && \
    curl -sSLO --retry 5 https://deb.nodesource.com/node_16.x/pool/main/n/nodejs/nodejs_${NODE_VERSION}-deb-1nodesource1_amd64.deb && \
    dpkg -i nodejs_${NODE_VERSION}-deb-1nodesource1_amd64.deb && \
    apt-get install --no-install-recommends -qy \
        cpanminus \
        bash-completion \
        build-essential \
        bzip2 \
        gettext \
        g++ \
        git \
        language-pack-de \
        language-pack-el \
        language-pack-es \
        language-pack-et \
        language-pack-fi \
        language-pack-fr \
        language-pack-it \
        language-pack-ja \
        language-pack-nl \
        language-pack-sq \
        # Needed for Cache in DB_File
        libdb-dev \
        libexpat1-dev \
        libicu-dev \
        liblocal-lib-perl \
        libpq-dev \
        libssl-dev \
        # Needed for XML::LibXML
        libxml2-dev \
        make \
        memcached \
        # Needed for Unicode::ICU::Collator
        pkg-config \
        postgresql-${POSTGRES_VERSION} \
        # Needed to decompress sample data
        xz-utils \
        yarn \
        # Needed for XML:LibXML
        zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /musicbrainz-server

COPY DBDefs.pm /
COPY scripts/* /usr/local/bin/
RUN cat /usr/local/bin/snippet.perllocallib.bashrc >> ~/.bashrc \
    && rm /usr/local/bin/snippet.perllocallib.bashrc \
    && ln -s /usr/local/bin/docker-entrypoint.sh /

# Postgres user/password would be solely needed to compile tests
ARG POSTGRES_USER=doesntmatteraslongasyoudontcompiletests
ARG POSTGRES_PASSWORD=doesntmatteraslongasyoudontcompiletests

ENV BASH_ENV=/noninteractive.bash_env \
    MUSICBRAINZ_BASE_FTP_URL=ftp://ftp.eu.metabrainz.org/pub/musicbrainz \
    MUSICBRAINZ_CATALYST_DEBUG=0 \
    MUSICBRAINZ_DEVELOPMENT_SERVER=1 \
    MUSICBRAINZ_SEARCH_SERVER=search:8983/solr \
    MUSICBRAINZ_SERVER_PROCESSES=1 \
    MUSICBRAINZ_STANDALONE_SERVER=1 \
    MUSICBRAINZ_WEB_SERVER_HOST=localhost \
    MUSICBRAINZ_WEB_SERVER_PORT=5000 \
    # Needed for yarn to install devDependencies too
    NODE_ENV=test \
    POSTGRES_USER=musicbrainz \
    POSTGRES_PASSWORD=musicbrainz

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["start.sh"]
