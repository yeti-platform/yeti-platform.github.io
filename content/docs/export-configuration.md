---
title: Export Configuration
date: 2024-09-23T9:14:05Z
draft: true
weight: 1
---

This section describes what persistient storage clients are, how to configure them, and how they relate to exports in `yeti`.

## What Persistient Storage Clients Looks Like?

Persistient Storage Clients are pluggable clients and use the following API:

```py
class PersistientStorageClient:
    PREFIX: str

    def __init__(self, path: str):
        ...

    def file_path(self, file_name: str) -> str:
        ...

    def get_file(self, file_name: str) -> str:
        ...

    def put_file(self, file_name: str, contents: str) -> None:
        ...

    def delete_file(self, file_name: str) -> None:
        ...
```

Implementing a class with the above spec and then including it in the `core/clients/persistient_storage/classes/` directory `yeti` will immediately being using your new client.

## How Does Yeti Decide What Persistient Storage Clients to Use?

Descision making for Persistient Storage Client are done via the `PREFIX` you provide for your class.

Thus far there are only two support clients: local storage (default) and S3.

```py
from core.clients.persistient_storage import get_client

s3_client = get_client("s3://bucketname/prefix")
local_storage_client = get_client("/opt/yeti/exports")
```

In the above example two clients are created:  
The first implemented S3 Client matches the prefix `s3://` and will intialize a `boto` client to interact with S3 compatable bucket.
The second client setups up a local storage client which will interact with the local disk of the system. All paths that do not match one an existing prefix of one the client will default to using the local storage client.

### How to Configure a Persistient Storage Client?

Currently, the only use for persistient storage client is for storage of export tasks as configured through the `system.export_path` config value.

In order to provide flexible support for clients like `boto3`, environment variable are used for configuration and permissioning.

These envrionment variables can be injected several different ways, but the easiest is just including them in the `docker-compose.yaml`

```diff
services:
  api:
    env_file:
      - ${YETI_DOCKER_ENVFILE}
+    env:
+      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
+      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
    entrypoint:
      - /docker-entrypoint.sh

  tasks:
    env_file:
      - ${YETI_DOCKER_ENVFILE}
+    env:
+      - AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}
+      - AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY}
    command: ['tasks']
```

** Note: For S3 there are tons of different environment variables to configure everything from different providers to how authentication to the buckets are done. Shown above is just a common pattern, not an all-encompassing one.

The you can include your secret credentials in a `.env` file.
