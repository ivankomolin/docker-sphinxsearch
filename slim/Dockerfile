FROM alpine:3.8

ENV VERSION 3.1.1
ENV REVISION 612d99f

RUN wget -q http://sphinxsearch.com/files/sphinx-${VERSION}-${REVISION}-linux-amd64-musl.tar.gz && \
    tar zxf sphinx-${VERSION}-${REVISION}-linux-amd64-musl.tar.gz && \
    mv sphinx-${VERSION}/bin/* /usr/local/bin/ && \
    rm -rf sphinx-${VERSION} && \
    rm sphinx-${VERSION}-${REVISION}-linux-amd64-musl.tar.gz && \
    mkdir -p /app/etc /app/data /app/logs && \
    ln -sf /dev/stdout /app/logs/*

STOPSIGNAL SIGKILL

CMD ["searchd", "--nodetach", "-c", "/app/etc/sphinx.conf"]