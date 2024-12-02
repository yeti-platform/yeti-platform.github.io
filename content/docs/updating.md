---
title: Updating Yeti
date: 2024-07-04T12:00:00
draft: false
weight: 1
---

{{< callout type="warning" >}} **Make sure to look at the release notes for any
breaking changes, and adjust steps accordingly!** {{< /callout >}}

1. Backup your database

```bash
docker compose run --rm -v $(pwd)/backup:/backup arangodb arangodump --server.endpoint tcp://a
rangodb:8529 --server.database yeti --output-directory /backup --overwrite true
```

2. Backup your configuration file

```bash
docker cp api:/app/yeti.conf /path/to/backup
```

3. Stop the containers

```bash
cd yeti-docker/prod && docker compose down
```

4. Update

- `git pull` will pull in the latest changes to the Docker compose setup
  (effectively changing your infrastructure, so be mindful of any breaking
  changes).
- `docker compose pull` will pull in the latest images for the Yeti services,
  this should be safer to update but we still recommend to read the release
  notes before updating.

```bash
git pull
docker compose pull
```

5. Run migrations

Sometimes the database schema changes between versions, and you need to run a
migration command to sync the database schema and the code that Yeti is running.

To do so:

```bash
docker compose run --rm api /docker-entrypoint.sh migrate-arangodb
```

6. Start the containers

```bash
docker compose up -d
```

7. _(optional)_ Restore database

```bash
docker compose run --rm -v $(pwd)/backup:/backup arangodb arangorestore --server.endpoint tcp://arangodb:8529 --input-directory /backup --server.database yeti --overwrite true
```

8. Restore configuration file

```bash
docker ps yeti.conf api:/app/
docker ps yeti.conf tasks:/app/
```

9. Restart containers

```bash
docker compose restarts tasks api
```
