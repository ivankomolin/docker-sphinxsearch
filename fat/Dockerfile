FROM alpine:3.8

ENV VERSION 3.1.1
ENV REVISION 612d99f

RUN apk add --no-cache mariadb-dev postgresql-dev unixodbc-dev mysql-client && \
    wget -q http://sphinxsearch.com/files/sphinx-${VERSION}-${REVISION}-linux-amd64-musl.tar.gz && \
    tar zxf sphinx-${VERSION}-${REVISION}-linux-amd64-musl.tar.gz && \
    mv sphinx-${VERSION}/bin/* /usr/local/bin/ && \
    rm -rf sphinx-${VERSION} && \
    rm sphinx-${VERSION}-${REVISION}-linux-amd64-musl.tar.gz && \
    mkdir -p /app/etc /app/data /app/logs && \
    ln -sf /dev/stdout /app/logs/*

COPY ./etc/sphinx.conf /app/etc/sphinx.conf
COPY ./etc/test.csv /app/etc/test.csv

RUN indexer --all -c /app/etc/sphinx.conf

VOLUME ["/app/data", "/app/etc"]

STOPSIGNAL SIGKILL

CMD ["searchd", "--nodetach", "-c", "/app/etc/sphinx.conf"]