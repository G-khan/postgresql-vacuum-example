
# postgresql-vacuum-example

VACUUM reclaims storage occupied by dead tuples.

### PostgreSQL Vacuum Practices

To simply run the container using the Postgres latest image we can execute the following command:

    docker run --name postgres-vacuum --rm -e POSTGRES_USER=postgres -e POSTGRES_PASSWORD=postgres -e PGDATA=/var/lib/postgresql/data/pgdata -v /tmp:/var/lib/postgresql/data -p 5432:5432 -it postgres:latest

to Execute to docker:

    docker exec -it postgres-vacuum /bin/sh

  

### OR

for docker-compose.yaml:

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

  

to run the postgresql via docker-compose:

    docker-compose up -d

  

## Practice Time

  go into docker container

    docker exec -it postgres-vacuum /bin/bash

and access the Postgres CLI:

    psql --username postgres

Let's create a pg_visibility extension for see the dead tuples.

    CREATE EXTENSION pg_visibility;
    
    CREATE EXTENSION pg_freespacemap ;

 Let's make a delete operation example with pg_visibility.

Create a table:

    postgres=# create table gokhanadev(id int, id2 int);
    CREATE TABLE

And insert some data with generate_series:
    
    postgres=# insert into gokhanadev values(generate_series(1,1000000), generate_series(1,1000000));
    INSERT 0 1000000

check the all visible tuples. **f** means visibility is false for the pages. **t**  means visibility true for the pages. :    

    postgres=# select count(blkno), all_visible from pg_visibility_map('gokhanadev') group by all_visible;
    count | all_visible
    -------+-------------
    4425 | f
    (1 row)

  
Now we can update the data to see dead tuples:

    postgres=# Update gokhanadev set id = id + 1 where id > 50000;
    UPDATE 950000
Let's check the dead tuples via pg_visibility_map:
    
    postgres=# select count(blkno), all_visible from pg_visibility_map('gokhanadev') group by all_visible;
     count | all_visible 
    -------+-------------
      8408 | f
       221 | t
    (2 rows)


You can wait a few minutes for auto-vacuum or execute the Vacuum comment

    postgres=# vacuum gokhanadev;
    VACUUM

Take a look what's happened after vacuum: 

    postgres=# select count(blkno), all_visible from pg_visibility_map('gokhanadev') group by all_visible;
     count | all_visible 
    -------+-------------
      8629 | t
    (1 row)