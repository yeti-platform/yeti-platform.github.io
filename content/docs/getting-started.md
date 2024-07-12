---
title: Getting started
date: 2023-11-05T17:40:05Z
draft: false
weight: 1
---

This section describes how to get an instance of Yeti up and running in no time.
Executed on a server, it will open a port on http://0.0.0.0:80/.

## Prerequisites

The supported install method for Yeti is using its dedicated Docker containers.
To install Yeti, you need to have the following tools installed on your system:

- git
- Docker
  - Use the official Docker install instructions:
    https://docs.docker.com/engine/install/debian/
  - Make sure you install the compose plugin as `docker compose` and not
    `docker-compose`.

```console
$ docker compose version
Docker Compose version v2.25.0
```

## Steps

{{% steps %}}

### 1. Start Docker containers

```bash
git clone https://github.com/yeti-platform/yeti-docker
cd yeti-docker/prod
docker compose up
```

This should get Yeti up and running on port http://localhost:80/. The docker
compose file starts up 4 containers:

- `api` - The API server, where most of the Yeti system runs; ()
- `frontend` - the web frontend, served by nginx; ()
- `tasks` - a Celery scheduler;
- `redis` - a Redis server;
- `arangodb` - an ArangoDB server.

It will also create a Docker network called `yeti_network` through which all
containers can talk to one another.

### 2. Create an admin user

You'll need to create an admin user to be able to log into the Yeti web
interface:

```bash
docker compose run --rm api create-user USERNAME PASSWORD --admin
```

### 3. Open the Yeti web UI

Head to [http://localhost:80/](http://localhost:80/) and log in with the
credentials you just created.

{{% /steps %}}
