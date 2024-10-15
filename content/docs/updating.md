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
sudo docker compose run --rm -v $(pwd)/backup:/backup arangodb arangodump --server.endpoint tcp://a
rangodb:8529 --server.database yeti --output-directory /backup --overwrite true
```

2. Backup your configuration file

```bash
sudo docker cp api:/app/yeti.conf /path/to/backup
```

3. Stop the containers

```bash
cd yeti-docker/prod && sudo docker compose down
```

3. Update

- `git pull` will pull in the latest changes to the Docker compose setup
  (effectively changing your infrastructure, so be mindful of any breaking
  changes).
- `docker compose pull` will pull in the latest images for the Yeti services,
  this should be safer to update but we still recommend to read the release
  notes before updating.

```bash
git pull
sudo docker compose pull
```

4. Start the containers

```bash
sudo docker compose up -d
```

5. _(optional)_ Restore database

```bash
sudo docker compose run --rm -v $(pwd)/backup:/backup arangodb arangorestore --server.endpoint tcp://arangodb:8529 --input-directory /backup --server.database yeti --overwrite true
```

6. Restore configuration file

```bash
sudo docker ps yeti.conf api:/app/
sudo docker ps yeti.conf tasks:/app/
```

7. Restart containers

```bash
sudo docker compose restarts tasks api
```
