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
