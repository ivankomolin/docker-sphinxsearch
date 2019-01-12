# Supported tags and respective `Dockerfile` links

- [`latest`, `3.1.1` (*3.1.1/fat/Dockerfile*)](https://github.com/ivankomolin/docker-sphinxsearch/blob/3.1.1/fat/Dockerfile)
- [`3.1.1-slim` (*3.1.1/slim/Dockerfile*)](https://github.com/ivankomolin/docker-sphinxsearch/blob/3.1.1/slim/Dockerfile)

# Quick reference sphinxsearch

- **Documentation**:  
  [http://sphinxsearch.com/docs/](http://sphinxsearch.com/docs/)

- **Community**:  
  [http://sphinxsearch.com/community/](http://sphinxsearch.com/community/)

- **Enterprise Support**:  
  [http://sphinxsearch.com/services/support/](http://sphinxsearch.com/services/support/)

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

Test data, content test.csv from Dockerfile:
```console
1;test 1;1,2,3
2;test 2;4,5,2
3;test 3;6,7,8,1
4;test 4;1,9,8,3
5;test 5;3
```
Test config, content sphinx.conf from Dockerfile:
```bash
source test
{
    type = csvpipe
    csvpipe_command = cat /app/etc/test.csv
    csvpipe_delimiter = ;
    csvpipe_field = name
    csvpipe_attr_multi = tags
}

index test
{
    source = test
    path = /app/data/test
}

indexer
{
    mem_limit = 32M
}

searchd
{
    listen = 9306:mysql41
    log = /app/logs/searchd.log
    pid_file = /var/run/searchd.pid
    binlog_path =
}
```

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
Or use image as base for your image
```bash
FROM ivankomolin/sphinxsearch

COPY sphinx.conf /app/etc/sphinx.conf
```


## Slim image
Containts only utilites sphinxsearch
For works indexer with database install need drivers

Example Dockerfile for use with PostgreSQL:
```bash
FROM ivankomolin/sphinxsearch:3.1.1-slim

RUN apk add --no-cache postgresql-dev

COPY sphinx.conf /app/etc/sphinx.conf
```

Example simple sphinx.conf
```bash
source db
{
    type = pgsql
    sql_host = 127.0.0.1
    sql_user = user
    sql_pass = secret
    sql_db = test
    sql_port = 3306
}

source users:db
{
    sql_query = SELECT id, name FROM users
}

source posts:db
{
    sql_query = SELECT id, title, content FROM posts
}

index users
{
    source = users
    path = /app/data/users
    morphology = stem_enru
    min_stemming_len = 4
    min_word_len = 4
}

index posts
{
    source = posts
    path = /app/data/posts
    morphology = stem_enru, metaphone
    min_stemming_len = 3
    min_word_len = 3
    html_strip = 1
}

indexer
{
    mem_limit = 32M
}

searchd
{
    listen = 9306:mysql41
    log = /app/logs/searchd.log
    pid_file = /var/run/searchd.pid
    binlog_path =
}
```