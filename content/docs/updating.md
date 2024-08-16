---
title: Updating Yeti
date: 2024-07-04T12:00:00
draft: false
weight: 1
---

If you update yeti, follow these instructions:

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

```bash
git  pull
sudo docker compose pull
```

4. Start the containers

```bash
sudo docker compose up -d
```

5. Restore database

```bash
sudo docker compose run --rm -v $(pwd)/backup:/backup arangodb arangorestore --server.endpoint tcp://arangodb:8529 --input-directory /backup --server.database yeti --overwrite true
```

6. Restore configuration file

```bash
sudo docker ps yeti.conf api:/app/
sudo docker ps yeti.conf tasks:/app/
```

7. Restart contenairs

```bash
sudo docker compose restarts tasks api
```
