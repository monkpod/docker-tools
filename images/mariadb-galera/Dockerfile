FROM python:2.7.14-slim
MAINTAINER Rich Braun "docker@instantlinux.net"
ARG BUILD_DATE
ARG VCS_REF
LABEL org.label-schema.build-date=$BUILD_DATE \
    org.label-schema.license=GPL-2.0 \
    org.label-schema.name=mariadb-galera \
    org.label-schema.vcs-ref=$VCS_REF \
    org.label-schema.vcs-url=https://github.com/instantlinux/docker-tools

ENV DEBIAN_FRONTEND=noninteractive \
    CLUSTER_NAME=cluster01 \
    CLUSTER_SIZE=3 \
    DISCOVERY_SERVICE=etcd:2379 \
    ROOT_PASSWORD_SECRET=mysql-root-password \
    TTL=10 \
    TZ=US/Pacific \
    XTRABACKUP_PASSWORD= \
    XTRABACKUP_SECRET=xtradb-root-password

ARG MARIADB_VERSION=10.2
ARG APT_KEY=CBCB082A1BB943DB
ARG UID=212
ARG GID=212

COPY requirements/ /root/
COPY src/healthcheck.sh /usr/local/bin/

RUN apt-key adv --recv-keys --keyserver keyserver.ubuntu.com $APT_KEY && \
    echo "deb [arch=amd64] \
     http://nyc2.mirrors.digitalocean.com/mariadb/repo/$MARIADB_VERSION/debian jessie main" \
     > /etc/apt/sources.list.d/mariadb.list && \
    groupadd -g $GID mysql && \
    useradd -u $UID -g $GID -s /bin/false -c "MariaDB" -d /none mysql && \
    apt-get -yq update && apt-get -yq install --no-install-recommends \
     curl iputils-ping jq mariadb-server mariadb-client net-tools netcat && \
    pip install -r /root/common.txt && \
    echo "dash dash/sh boolean false" | debconf-set-selections && \
    dpkg-reconfigure dash && \
    chmod -R g+rX,o+rX /etc/mysql && \
    apt-get clean; rm -fr /var/log/* /var/lib/mysql/*

EXPOSE 3306 4444 4567/udp 4567 4568
VOLUME /var/lib/mysql

HEALTHCHECK --interval=10s --timeout=3s --retries=30 \
    CMD /bin/sh /usr/local/bin/healthcheck.sh || exit 1

COPY wsrep.cnf my.cnf /etc/mysql/
COPY src/entrypoint.py /usr/local/bin/
ENTRYPOINT ["/usr/local/bin/entrypoint.py"]

# TODO: fix healthcheck.sh to handle long-duration bootstrap
# TODO: percona-xtrabackup when wsrep_sst_method works in 10.2
