# Supported tags and respective `Dockerfile` links

-	[`latest`, `3.1.1`](https://github.com/ivankomolin/docker-sphinxsearch/blob/master/fat/Dockerfile)
- [`slim`, `3.1.1-slim`](https://github.com/ivankomolin/docker-sphinxsearch/blob/master/slim/Dockerfile)

# How to use this images

## Full image
Containts utilites sphinxsearch, database drivers for indexer, sample index and client for test queries 

Pull image and create new container sphinx
```console
$ docker run -d -p 9306:9306 --name sphinx ivankomolin/sphinxsearch
```
The searchd service is now available both inside and outside the container on port 9306

For convenience within the container is installed, the mysql client, which you can use to connect to searchd
```console
$ docker exec -it sphinx mysql -h127.0.0.1 -P9306
```
Or use an external client, for example console mysql-client, but you can use other clients, following the connection details
```console
$ mysql -h127.0.0.1 -P9306
```
For convenience, the container has already created a small index `test` for the presentation of the search
Example queries in `test` index
```console
MySQL> select id from test where tags in (1,3);
+------+
| id   |
+------+
|    1 |
|    3 |
|    4 |
|    5 |
+------+
4 rows in set, 1 warning (0.00 sec)

MySQL> select id from test where MATCH('test');
+------+
| id   |
+------+
|    1 |
|    2 |
|    3 |
|    4 |
|    5 |
+------+
5 rows in set (0.01 sec)

MySQL> select id from test where MATCH('test 2');
+------+
| id   |
+------+
|    2 |
+------+
1 row in set (0.01 sec)
```

You can use the following command to rotate all indexes
```console
docker exec -it sphinx indexer --rotate --all -c /app/etc/sphinx.conf
```

Use volume /app/etc for include your config:
```console
docker run -d -p 9306:9306 -v /etc/sphinxsearch/:/app/etc/ --name sphinx ivankomolin/sphinxsearch
```

## Slim image
Containts only utilites sphinxsearch
For works indexer in database install need drivers

Example Dockerfile with drivers for PostgreSQL:
```console
FROM ivankomolin/sphinxsearch:slim

RUN apk add --no-cache postgresql-dev
```