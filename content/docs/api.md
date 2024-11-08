---
title: API docs
date: 2023-12-29T10:23:05Z
draft: false
weight: 3
---

Yeti has a fully capable REST API.

## Auto-generated documentation

The auto-generated documentation can be found at `http://<YETI_HOSTNAME>/docs`.

## Authentication

Every user in Yeti has an API key that they can use to interact with the API.
The API key acts as a refresh token which is used to obtain a JWT access token.

Making authenticated requests to the API is a three step process:

1. Get an API key for the user you want to authenticate as.
2. Do a POST request to `/api/v2/auth/api-token` using the API key in the
   `x-yeti-apikey` header.
3. Grab the access token from the response, and reuse it in the `Authorization`
   header of subsequent requests.

This can be accomplished with the following Python code:

```python
import requests
apikey = "5902c3f2e63a172e0da2a8e9162771b3c7e0d98b813804f44149c1cd15dbcc6e"

# Add your API key to the x-yeti-apikey header
# Write a requests POST call with the api key in the header
response = requests.post(
    "http://localhost:8000/api/v2/auth/api-token",
    headers={"x-yeti-apikey": apikey},
)

access_token = response.json().get("access_token")

response = requests.get(
    "http://localhost:8000/api/v2/auth/me",
    headers={"authorization": f"Bearer {access_token}"},
)

print("response:", response.json())

# Or using a requests Session object, so you don't have to pass it every time
yeti_session = requests.Session()
yeti_session.headers.update({"authorization": f"Bearer {access_token}"})
response = yeti_session.get("http://localhost:8000/api/v2/auth/me")
print("response:", response.json())
```

## Import Observables

Observables endpoints have been implemented to easily import them into Yeti.

### Import observables from text

Save observables from a text containing one observable per line. Tag with defined tags.

* Endpoint `POST api/v2/observables/import/text`
* Usage:
```python
import requests

observables = """1.1.1[.]1
8.8.8.8
tomchop[.]me
google.com
http://google.com/
http://tomchop[.]me/
d41d8cd98f00b204e9800998ecf8427e
da39a3ee5e6b4b0d3255bfef95601890afd80709
e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"""

requests.post("api/v2/observables/import/text", json={"text": observables, "tags": ["tag1", "tag2"]})
```

### Import observables from file

Save observables from a file containing one observable per line. Tag with defined tags.

* Endpoint `POST api/v2/observables/import/file`
* Usage:
```python
import requests

with open(path_to_file, "rb") as file:
   requests.post("/api/v2/observables/import/file", files={"file": file}, data={"tags": ["tag1", "tag2"]})
```

### Import observables from URL

Save observables from a text containing one observable per line. Tag with defined tags.

* Endpoint `POST api/v2/observables/import/file`
* Usage:
```python
import requests

requests.post("api/v2/observables/import/text",  json={"url": "https://example.com", "tags": ["tag1", "tag2"]})
```

## Technical details

### Concurrency

Concurrency is handled both by uvicorn workers and FastAPI. By default, FastAPI supports coroutines through async / await usage. However, I/O bound path operation functions are not async when doing ArangoDB calls since the backend does not support async calls. Switching to non-async definitions lets FastAPI run blocking calls in awaited thread pool as explained in [documentation](https://fastapi.tiangolo.com/async/#in-a-hurry):

> If you are using a third party library that communicates with something (a database, an API, the file system, etc.) and doesn't have support for using await, (this is currently the case for most database libraries), then declare your path operation functions as normally, with just def.

Regarding technical details, this is how [FastAPI handles concurrency](https://fastapi.tiangolo.com/async/#path-operation-functions) with non-async path operation functions:

> When you declare a path operation function with normal def instead of async def, it is run in an external threadpool that is then awaited, instead of being called directly (as it would block the server).

Some path operation functions are still async because:
* they call others async functions and make non time-consuming calls to ArangoDB.
* they do not perform any ArangoDB calls.
