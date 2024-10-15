---
title: File Storage Clients & Export Configuration
date: 2024-09-23T9:14:05Z
draft: false
weight: 1
---

This section describes what file storage clients are, how to configure them, and
how they relate to exports in `yeti`.

## What does a File Storage Client Look Like?

File Storage Clients are pluggable clients and use the following API:

```py
class FileStorageClient:
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

Implementing a class with the above spec and then including it in the
`core/clients/file_storage/classes/` directory `yeti` will immediately begin
using your new client.

## How Does Yeti Decide What File Storage Clients to Use?

Decision making for File Storage Client are done via the `PREFIX` you provide
for your class.

Thus far there are only two support clients: local storage (default) and S3.

```py
from core.clients.file_storage import get_client

s3_client = get_client("s3://bucketname/prefix")
local_storage_client = get_client("/opt/yeti/exports")
```

In the above example two clients are created: The first implemented S3 Client
matches the prefix `s3://` and will initialize a `boto` client to interact with
S3 compatible bucket. The second client setups up a local storage client which
will interact with the local disk of the system. All paths that do not match one
an existing prefix of one the client will default to using the local storage
client.

### How to Configure a File Storage Client?

Currently, the only use for File Storage Client is for storage of export tasks
as configured through the `system.export_path` config value.

In order to provide flexible support for clients like `boto3`, environment
variable are used for configuration and permissioning.

These environment variables can be injected several different ways, but the
easiest is just including them in the `docker-compose.yaml`

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

\*\* Note: For S3 there are several different environment variables to configure
everything from different providers to how authentication to the buckets are
done and much more. Shown above is just a common pattern, not an
all-encompassing one.

The you can include your secret credentials in a `.env` file.

## Clients

The following are a list of file storage clients and general configuration
required for them:

### Local Storage

`local_storage` is the default client used. If none of the loaded Client
Prefixes match the path you provide, then `local_storage` will be used.

Example:

```python
from core.clients.file_storage import get_client

local_storage_client = get_client("/opt/yeti/exports")
```

### S3

Amazon S3 (Simple Storage Service) is a scalable object storage service used for
storing and retrieving any amount of data at any time and is a common spec used
by other cloud providers and file storage services.

If you configured path starts with: `s3://` then the `S3StorageClient` will be
loaded:

```python
from core.clients.file_storage import get_client

local_storage_client = get_client("s3://bucket-name/path/")
```

#### S3 Bucket Configuration

Add respective configuration for accessing your bucket as an environment
variable; this is easiest done in `docker-compose.yaml`:

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

#### Install the `s3` Extra

As `s3` is an optional feature; you will need to included installation of it's
dependencies.

This can be done by adding `--extras s3` to your existing poetry install:

```bash
poetry install --extras s3
```
