FROM alpine

ARG BUILD_DATE
ARG VCS_REF

LABEL maintainer="Mister Trololo"

RUN apk add --no-cache mariadb mariadb-client mariadb-server-utils pwgen mc && \
    rm -f /var/cache/apk/*

ADD files/run.sh /scripts/run.sh
RUN mkdir /docker-entrypoint-initdb.d && \
    mkdir /scripts/pre-exec.d && \
    mkdir /scripts/pre-init.d && \
    chmod -R 755 /scripts

EXPOSE 3306

VOLUME ["/var/lib/mysql"]

ENTRYPOINT ["/scripts/run.sh"]