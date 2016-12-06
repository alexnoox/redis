# Maestrano MySQL
Docker image for Redis with support for authentication and replication

# Supported tags and respective `Dockerfile` links

-	[`3.2.1`, `3.2`, `3`, `latest` (*3.2/Dockerfile*)](https://github.com/maestrano/redis/blob/master/3.2/Dockerfile)

# What is Redis?

Redis is an open-source, networked, in-memory, key-value data store with optional durability. It is written in ANSI C. The development of Redis has been sponsored by Pivotal since May 2013; before that, it was sponsored by VMware. According to the monthly ranking by DB-Engines.com, Redis is the most popular key-value store. The name Redis means REmote DIctionary Server.

> [wikipedia.org/wiki/Redis](https://en.wikipedia.org/wiki/Redis)

![logo](https://raw.githubusercontent.com/docker-library/docs/01c12653951b2fe592c1f93a13b4e289ada0e3a1/redis/logo.png)

# How to use this image

## start a redis instance

```console
$ docker run --name some-redis -d maestrano/redis
```

This image includes `EXPOSE 6379` (the redis port), so standard container linking will make it automatically available to the linked containers (as the following examples illustrate).

## start with persistent storage

```console
$ docker run --name some-redis -d maestrano/redis redis-server --appendonly yes
```

If persistence is enabled, data is stored in the `VOLUME /data`, which can be used with `--volumes-from some-volume-container` or `-v /docker/host/dir:/data` (see [docs.docker volumes](http://docs.docker.com/userguide/dockervolumes/)).

For more about Redis Persistence, see [http://redis.io/topics/persistence](http://redis.io/topics/persistence).

## connect to it from an application

```console
$ docker run --name some-app --link some-redis:maestrano/redis -d application-that-uses-redis
```

## ... or via `redis-cli`

```console
$ docker run -it --link some-redis:redis --rm maestrano/redis redis-cli -h redis -p 6379
```

## Environment Variables

### `REDIS_PASS`

Specify authentication password for connecting to redis (set `requirepass` directive in redis conf). Setting this environment variable will require clients to issue AUTH <PASSWORD> before processing any other commands.

Warning: since Redis is pretty fast an outside user can try up to 150k passwords per second against a good box. This means that you should
use a very strong password otherwise it will be very easy to break.

### `REDIS_MASTER`

Specify a redis master to replicate data from. Format must follow <host>:<port>. If a `REDIS_PASS` is specified then this pass will be used to set the master-slave replication.

**Example**
```sh
# Container setup
HOST_PUB_IP=`ifconfig | grep en0 -A 5 | grep "inet " | cut -d' ' -f2`
PORT_NODE_1=33001
PORT_NODE_2=33002

docker run -d -p $PORT_NODE_1:6379 \
  -e SELF_HOST=$HOST_PUB_IP \
  -e SELF_PORT=$PORT_NODE_1 \
  -e REDIS_MASTER=$HOST_PUB_IP:$PORT_NODE_1 \
  -e REDIS_PASS=somepass \
  --name r1 \
  alachaum/redis

docker run -d -p $PORT_NODE_2:6379 \
  -e SELF_HOST=$HOST_PUB_IP \
  -e SELF_PORT=$PORT_NODE_2 \
  -e REDIS_MASTER=$HOST_PUB_IP:$PORT_NODE_1 \
  -e REDIS_PASS=somepass \
  --name r2 \
  alachaum/redis

# Set key
while true; do
  key=$(openssl rand -base64 16)
  redis-cli -h $HOST_PUB_IP -p $PORT_NODE_1 -a somepass set $key somevalue
done

# Count keys
redis-cli -h $HOST_PUB_IP -p $PORT_NODE_1 -a somepass info | grep 'db0' | cut -d',' -f1 | cut -d'=' -f2
redis-cli -h $HOST_PUB_IP -p $PORT_NODE_2 -a somepass info | grep 'db0' | cut -d',' -f1 | cut -d'=' -f2
```

# License

View [license information](http://redis.io/topics/license) for the software contained in this image.

# Supported Docker versions

This image is officially supported on Docker version 1.11.2.

Support for older versions (down to 1.6) is provided on a best-effort basis.

Please see [the Docker installation documentation](https://docs.docker.com/installation/) for details on how to upgrade your Docker daemon.
