## postgresql-vacuum-example

VACUUM reclaims storage occupied by dead tuples.

### PostgreSQL Vacuum Practices

To simply run the container using the Postgresql latest image, we can execute the following command:

```plaintext
docker run --name postgres-vacuum --rm -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e PGDATA=/var/lib/postgresql/data/pgdata -v /tmp:/var/lib/postgresql/data -p 5432:5432 -it postgres:latest
```

to Execute to docker:

```plaintext
docker exec -it postgres-vacuum /bin/sh
```

### OR

for docker-compose.yaml:

```plaintext
version: '3.8'
services:
  db:
    image: postgres:latest
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
volumes:
  db:
    driver: local
```

to run PostgreSQL via docker-compose:

```plaintext
docker-compose up -d
```

## Practice Time

go into the docker container

```plaintext
docker exec -it postgres-vacuum /bin/bash
```

and access the Postgres CLI:

```plaintext
psql --username postgres
```

Let's create a pg\_visibility extension to see the dead tuples.

```plaintext
CREATE EXTENSION pg_visibility;
```

Let's do a delete operation example with pg\_visibility.

Create a table:

```plaintext
postgres=# create table gokhanadev(id int, id2 int);
CREATE TABLE
```

And insert some data with generate\_series:

```plaintext
postgres=# insert into gokhanadev values(generate_series(1,1000000), generate_series(1,1000000));
INSERT 0 1000000
```

Check all visible tuples. **f** means visibility is false for the pages. **t** means visibility “true” for the pages. :

```plaintext
postgres=# select count(blkno), all_visible from pg_visibility_map('gokhanadev') group by all_visible;
count | all_visible
-------+-------------
4425 | f
(1 row)
```

Now we can update the data to see dead tuples:

```plaintext
postgres=# Update gokhanadev set id = id + 1 where id > 50000;
UPDATE 950000
```

Let's check the dead tuples via pg\_visibility\_map:

```plaintext
postgres=# select count(blkno), all_visible from pg_visibility_map('gokhanadev') group by all_visible;
 count | all_visible 
-------+-------------
  8408 | f
   221 | t
(2 rows)
```

You can wait a few minutes for the auto-vacuum or execute the Vacuum comment

```plaintext
postgres=# VACUUM gokhanadev;
VACUUM
```

Take a look, what happened after the vacuum:

```plaintext
postgres=# select count(blkno), all_visible from pg_visibility_map('gokhanadev') group by all_visible;
 count | all_visible 
-------+-------------
  8629 | t
(1 row)
```

## More about Vacuum

*   **Verbose** the Vacuum to Print a detailed vacuum activity report for each table.

```plaintext
VACUUM VERBOSE table_name
```

*   **ANALYZE** the Vacuum to Update statistics used by the planner to determine the most efficient way to execute a query.

```plaintext
VACUUM ANALYZE table_name
```

*   **Full VACUUM**: Locks the database table, and reclaims more space than a plain VACUUM. Be careful for production.

```plaintext
VACUUM(FULL) table_name
```

*   Set the auto vacuum

```plaintext
ALTER TABLE table_name SET (autovacuum_enabled = false);
```

[More about the vacuum](https://www.postgresql.org/docs/current/sql-vacuum.html)