---
title: Dev environment setup
date: 2024-12-07
draft: false
cascade:
  type: docs
---

This page details how to setup a development environment for Yeti so you can
start contributing.

## Clone the yeti-docker repo

Like for production deployments, the Yeti development environment is based on
Docker. The `yeti-docker` repository contains all the necessary files to build
the Docker images and run the containers.

```bash
git clone https://github.com/yeti-platform/yeti-docker
cd yeti-docker
```

Further instructions are available in the repo's
[README](https://github.com/yeti-platform/yeti-docker/blob/main/dev/README.md).

## Run the `init.sh` script

```bash
cd dev
./init.sh
```

This will clone the frontend and the backend repos to your local filesystem, and
start the dev Docker containers that you need.

## Spin up some terminals

We recommend using a terminal multiplexer like `tmux` or `screen` to manage the
multiple terminals you'll need to run the frontend and backend servers.

You're going to run a couple of commands in different terminals:

### API server

This is the main Yeti backend server. We'll set it up to auto-reload whenever
changes are made to the Python code.

```bash
docker compose exec api poetry run uvicorn core.web.webapp:app --reload --host 0.0.0.0
```

### Frontend server

Even if you're not going to work on the frontend, you'll need to run the server
to render the main Yeti web UI.

```bash
docker compose exec frontend npm run dev
```

### Celery workers

Celery is the task queue used by Yeti. If you plan to develop new feeds,
analyzers, or other background tasks, you'll need to run the Celery workers.

```bash
docker compose exec -it api poetry run celery -A core.taskscheduler worker --loglevel=INFO --purge -B -P threads
```

### Events tasks

If you wanna work with events tasks, you need to run one or several events
consumers.

```bash
docker compose exec -it api poetry run python -m core.events.consumers events
```

You can adjust concurrency with `--concurrency <number_of_worker>` and enable
debugging output with `--debug`.

## Start hacking!

The docker containers use volumes to read your local directory so you can make
changes to the code locally, with your favorite editor, and see them reflected
in the running containers.

If you kill the containers, the code changes will still exist on your local
filesystem, and re-creating them will pick up where you left off.
