FROM ivankomolin/sphinxsearch:3.1.1-slim

RUN apk add --no-cache \
    mariadb-dev \
    postgresql-dev \
    unixodbc-dev \
    mysql-client

COPY ./etc/sphinx.conf /app/etc/sphinx.conf
COPY ./etc/test.csv /app/etc/test.csv

RUN indexer --all -c /app/etc/sphinx.conf

VOLUME ["/app/data", "/app/etc"]