---
alias: si4aef8hee
description: Overview
---

TODO N: polish

# Local Cluster

## Database Access (SQL)

Follow this guide to connect directly to the MySQL database powering your local Prisma cluster.

If you used `prisma local start` to start your local Prisma cluster, you will have two containers running:

`prisma` is running the main Prisma service
`prisma-db` is running the MySQL server that stores your data

This guide explains how to connect to your local MySQL server in order to query and update data directly.

> In the following, we assume that you have a local Prisma service running (as described in the [previous chapter](!alias-ohs4asd0pe)).

#### 1. Get the name of the Docker container that runs the MySQL database

list your running Prisma docker containers:

```sh
docker ps --filter name=prisma
```

Verify that there is a container with the name `prisma-db` image `mysql:5.7`.

#### 2. Open MySQL client

To open the MySQL client in your terminal, run the following command. Note that prisma-db is the container name from the list above:

```sh
 docker exec -it prisma-db mysql -u root --host 127.0.0.1 --port 3306 --password=prisma
```

#### 3. Send SQL queries to the database

Once the MySQL client is open, you can ask for the currently available databases:

```mysql
show databases;
```

This will print the following output:

```
+---------------------------+
| Database                  |
+---------------------------+
| information_schema        |
| prisma                    |
| logs                      |
| mysql                     |
| performance_schema        |
| sys                       |
| <service-id>              |
+---------------------------+
```

From the available databases, the following three are relevant for you:

* `<service-id>`: The name of this database is a combination of the name and stage of your service. It looks like this: `service@stage`. It contains meta information about your service used to generate the GraphQL schema.
* `prisma`: This database contains meta-information about the Prisma service definition.

You can now select one of these databases with the `use` command, for example if your `<service-id>` is `my-service@dev`, then run:

```mysql
use my-service@dev;
```

You can list tables like this:

```sh
show tables;
```

```
+----------------------+
| Tables_in_my-app@dev |
+----------------------+
| Post                 |
| _RelayId             |
+----------------------+
```

Now you can start writing queries:

```sh
SELECT * FROM Post;
```

```
+---------------------------+---------------------+---------------------+-------------+----------------------+-------------------------------------------------+
| id                        | createdAt           | updatedAt           | isPublished | title                | text                                            |
+---------------------------+---------------------+---------------------+-------------+----------------------+-------------------------------------------------+
| cjc82o6cg000b0135wpxgybf6 | 2018-01-09 20:12:02 | 2018-01-09 20:12:02 |           1 | Hello World          | This is my first blog post ever!                |
| cjc82o6mo000d013599yzlwls | 2018-01-09 20:12:02 | 2018-01-09 20:12:02 |           1 | My Second Post       | My first post was good, but this one is better! |
| cjc82o6n4000f01350jortmv2 | 2018-01-09 20:12:02 | 2018-01-09 20:12:02 |           0 | Solving World Hunger | This is a draft...                              |
+---------------------------+---------------------+---------------------+-------------+----------------------+-------------------------------------------------+
```

You can quit MySQL like this:

```sh
exit;
```

## Debugging

You can view logs from your local Prisma cluster to debug issues.

### Logs

you can view normal debug logs:

```sh
prisma local logs
```

### Docker logs

If you need more extensive logs you can view the raw logs from the containers running MySQL and Prisma:

```sh
docker logs prisma

docker logs prisma-db
```

### Verify Docker containers

If you get an error message saying `Error response from daemon: No such container` you can verify that the containers are running:

```sh
docker ps
```

You should see output similar to this:

```
❯ docker ps
CONTAINER ID  IMAGE                       COMMAND                 CREATED            STATUS            PORTS                   NAMES
7210106b6650  prismagraphql/prisma:1.0.0  "/app/bin/single-ser…"  About an hour ago  Up About an hour  0.0.0.0:4466->4466/tcp  prisma
1c15922e15ba  mysql:5.7                   "docker-entrypoint.s…"  About an hour ago  Up About an hour  0.0.0.0:3306->3306/tcp  prisma-db
```

### Nuke

If your local prisma cluster is in an unrecoverable state, the easiest option might be to completely reset it. Be careful as this command will reset all data in your local cluster.

```sh
❯ prisma local nuke
Nuking local cluster 10.9s
Booting fresh local development cluster 18.4s
```