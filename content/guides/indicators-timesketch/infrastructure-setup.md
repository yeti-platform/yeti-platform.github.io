---
title: Infrastructure setup
date: 2024-02-02
draft: false
cascade:
  type: docs
---

## Before we start

Yeti offers two installation methods: [Kubernetes](#k8s-installation) (K8s) or
[Docker](#docker-installation) (through `docker-compose`). Choose your preferred
method and follow the instructions in this guide.

## Using Kubernetes / minikube

Kubernetes (k8s) / minikube will get you started in no time, thanks to the
OSDFIR Helm chart (you can read more about it
[here](https://osdfir.blogspot.com/2024/04/welcoming-yeti-to-osdfir-infrastructure.html)).
If you are new to Kubernetes, consider reviewing the OSDFIR Infrastructure
[getting started guide](https://github.com/google/osdfir-infrastructure/blob/main/docs/getting-started.md).

### Prerequisites

{{< tabs items="macOS,Windows" >}}

{{< tab >}}

Use Homebrew:

```bash
brew install helm minikube kubectl
```

{{< /tab >}}

{{< tab >}}

Use Chocolatey to install mikibube and helm:

```bash
choco install minikube
```

```bash
choco install kubernetes-helm
```

{{< /tab >}}

{{< /tabs >}}

To get started, ensure you have [Helm](https://helm.sh/docs/intro/install/) and
[Kubectl](https://kubernetes.io/docs/tasks/tools/) installed and are
authenticated to your Kubernetes cluster.

{{< callout type="info" >}}

**Note**: If you don't have a remote k8s setup, you can still use
[Minikube](https://minikube.sigs.k8s.io/docs/start/) or
[KIND](https://kind.sigs.k8s.io/docs/user/quick-start/) to install Yeti and
Timesketch locally.

For minikube, you might need to run:

```bash
kubectl config use-context minikube
```

{{< /callout >}}

### Pull chart and install release

Once complete, add the repo containing the Helm charts as follows:

```console
helm repo add osdfir-charts https://google.github.io/osdfir-infrastructure
```

If you had already added this repo earlier, run `helm repo update` to retrieve
the latest versions of the packages. You can then run
`helm search repo osdfir-charts` to see the available charts.

To install the Yeti and Timesketch chart, pick a release name of your choice,
for example, using a release name of `my-release`, run the following:

```console
helm install my-release osdfir-charts/osdfir-infrastructure \
--set global.turbinia.enabled=false
```

{{< callout type="info" >}}

**Note:** Using `--set global.turbinia.enabled` disables the Turbinia deployment
from being installed given it is not being used as part of this guide and is
enabled by default otherwise.

{{< /callout >}}

To uninstall the chart:

```console
helm uninstall my-release
```

For instructions on installing Yeti along with other integrated DFIR tools,
refer to the main
[OSDFIR Infrastructure](https://github.com/google/osdfir-infrastructure)
repository. Additionally, refer to the Yeti Helm chart
[README](https://github.com/google/osdfir-infrastructure/tree/main/charts/yeti)
for a comprehensive list of configuration options.

### That's it!

You're now ready to start your investigation with Timesketch and Yeti. Head to
[the investigation steps](/guides/indicators-timesketch/investigation) to follow
the rest of the guide.

## Using Docker and `docker compose`

- You'll be running two set of docker compose "projects". One for Yeti, and one
  for Timesketch;
- You'll connect the Timesketch and Yeti containers to the same network;
- **ALL of the `docker compose` commands need to be run in the directory that
  have a `docker-compose.yaml` file.**

You can cleanup everything at the end of the workshop by doing
`docker compose down` on each project's respective docker compose directory.

To stay organized, we recommend you create a directory called
`yeti_platform_guide` and run all these commands from there.

### Installing Yeti

Get a release docker image started:

```bash
git clone https://github.com/yeti-platform/yeti-docker
cd yeti-docker/prod
docker compose -p yeti up
```

You can use the `--no-cache` option in the `docker compose up` command to
rebuild the images from scratch.

```bash
# If you're running an outdated version of Yeti
# pull the latest images:
docker pull yetiplatform/yeti-frontend:latest
docker pull yetiplatform/yeti:latest
```

This will create and run the latest release Yeti containers, and start a web
service running on [http://localhost:80/](http://localhost:80/). You can check
the service is up now, but you'll need to create a user to continue:

```console
docker compose -p yeti exec -it api /docker-entrypoint.sh create-user yeti yeti --admin
```

The output should contain the line:

```console
User yeti successfully created! API key: yeti:<APIKEY>
```

Head to [http://localhost:80/](http://localhost:80/) and log in with the
credentials of the user you just created. Take note of that API key, as it will
be used by Timesketch in the next step.

### Installing Timesketch

We're going to be using a development version of Timesketch, start by cloning
the Timesketch repository:

```console
git clone https://github.com/google/timesketch
```

Edit `timesketch/data/timesketch.conf` to point to our deployed Yeti instance

```
[...]

# Threatintel Yeti analyzer-specific configuration
# URI root to Yeti's API, e.g. 'https://localhost:8080/api'
YETI_API_ROOT = 'http://yeti-frontend/api/v2'

# API key to authenticate requests
YETI_API_KEY = 'c90811fd5cd0978e2c2672332cfc712503725863c63885e90db4afe1070c5b21'  # API key from previous step

# Labels to narrow down indicator selection
YETI_INDICATOR_LABELS = ['domain']  # unused

[...]

```

Go to the docker directory, and run docker compose up:

```console
cd timesketch/docker/dev
docker compose -p timesketch up
```

This can take a moment. After a few minutes, you should see:

```
timesketch-dev  | Timesketch development server is ready!
```

Then open two terminals (it's a good idea to use tmux or something similar), and
run the following commands:

_Shell 1:_

```console
cd timesketch/docker/dev
docker compose -p timesketch exec timesketch gunicorn --reload -b 0.0.0.0:5000 --log-file - --timeout 120 timesketch.wsgi:application
```

This will start the Timesketch web server.

_Shell 2:_

```console
cd timesketch/docker/dev
docker compose -p timesketch exec timesketch celery -A timesketch.lib.tasks.celery worker --loglevel=info
```

This will start the Timesketch Celery workers, which are used to import plaso
files and run analyzers.

Open [http://localhost:5000](http://localhost:5000) or
[http://127.0.0.1:5000](http://127.0.0.1:5000) and login with `dev` / `dev`

### Connecting Yeti and Timesketch

#### Docker network connectivity

List networks in your Docker environment:

```console
$ docker network ls
NETWORK ID     NAME                 DRIVER    SCOPE
e0e2da34fe54   bridge               bridge    local
aff34be18958   host                 host      local
4ef6dd47f376   none                 null      local
d5c1f8727703   timesketch_default   bridge    local
82ac0824ce2a   yeti_network         bridge    local
```

- `timesketch_default` → Docker compose network for our Timesketch compose
  project. The network name prefix was specified in the `-p` flag when running
  the `docker compose up` command.
- `yeti_network` → Docker compose network for Yeti. The name of the network was
  specified in the Yeti `docker-compose.yaml` file.

Inspect the Yeti network:

```console
docker network inspect yeti_network
```

This section should be somewhere in the output of the above command:

```json
{
 "Containers": {
            "e5ffa7aa1929d407f69422ef0a4d6e6fc52317907f2cee32420f1326402c2084": {
                "Name": "yeti-frontend",
                "EndpointID": "3ec358652ed8d96111d44326770e8d2f73a4462fdbe7552d5e57cb894ed886cc",
                "MacAddress": "02:42:ac:1e:00:06",
                "IPv4Address": "172.30.0.6/16",
                "IPv6Address": ""
            },
            "f6f096d206d94c257d353904e15ffa914c542a9e3688ebd49bb8cdd67358f2a3": {
                "Name": "yeti-tasks",
                "EndpointID": "58380dc3312692970f3421bb1663e5f4cf7982bfbbf793092634013d947eaedc",
                "MacAddress": "02:42:ac:1e:00:05",
                "IPv4Address": "172.30.0.5/16",
                "IPv6Address": ""
            }
}
```

Connect the `yeti-tasks` and `yeti-frontend` to `timesketch_default` (the
Timesketch network). We need this so that:

- The Timesketch server can query the Yeti API server (running on
  `yeti-frontend`);
- The Yeti task service (running on `yeti-tasks`) can feed off the Timesketch
  API.

```console
docker network connect timesketch_default yeti-tasks
docker network connect timesketch_default yeti-frontend
```

You should see these two containers in the result of
`docker network inspect timesketch_default`.

{{< callout type="info" >}}

**Optional**: You can test that the hosts can ping each other by doing

```console
docker exec -it yeti-frontend /bin/bash
apt update && apt install inetutils-ping -y
ping timesketch-dev
```

{{< /callout >}}

{{< callout type="info" >}}

**Note**: In Docker, you can refer to hosts on the network by their container
name (e.g. `yeti-frontend`, what you see in the result of `docker ps -a`) or by
"service name" in the respective compose file (e.g. `frontend`)

{{< /callout >}}

### That's it!

You're now ready to start your investigation with Timesketch and Yeti. Head to
[the investigation steps](/guides/indicators-timesketch/investigation) to follow
the rest of the guide.

## Getting GRR set up (optional)

Good docs at
[https://grr-doc.readthedocs.io/en/latest/installing-grr-server/via-docker.html](https://grr-doc.readthedocs.io/en/latest/installing-grr-server/via-docker.html)

### Installing the GRR server

```console
docker run \
  --name grr-server \
  -e EXTERNAL_HOSTNAME=localhost \
  -e ADMIN_PASSWORD=demo \
  -p 0.0.0.0:8000:8000 -p 0.0.0.0:8080:8080 \
  ghcr.io/google/grr:v3.4.6.7
```

Wait a few minutes, and you should be good to go (this takes a while)

### Installing GRR clients

You can either install GRR clients on the docker container itself, or any host
you want, provided that they can reach the server through the
`EXTERNAL_HOSTNAME` variable you provided above.

```console
docker exec -it grr-server /bin/bash
```

```console
root@b7d5a20b496e:/usr/share/grr-server/executables/installers# ls -la
total 117152
drwxr-xr-x 2 root root     4096 Oct 18 12:00 .
drwxr-xr-x 1 root root     4096 Oct 18 11:56 ..
-rw-r--r-- 1 root root 34287616 Oct 18 11:58 GRR_3.4.6.7_amd64.msi
-rw-r--r-- 1 root root 34287616 Oct 18 11:59 dbg_GRR_3.4.6.7_amd64.msi
-rw-r--r-- 1 root root      914 Oct 18 11:57 grr_3.4.6.7_amd64.changes
-rw-r--r-- 1 root root 17655240 Oct 18 11:57 grr_3.4.6.7_amd64.deb
-rw-r--r-- 1 root root 16904836 Oct 18 11:57 grr_3.4.6.7_amd64.pkg
-rw-r--r-- 1 root root 16805741 Oct 18 12:00 grr_3.4.6.7_amd64.rpm
```

You can either:

- Install one of these on the server (the already running docker container)

If the Docker install doesn't make the client appear on your client list, launch
it manually:

```console
/usr/sbin/grrd --config=/usr/lib/grr/grr_3.4.6.7_amd64/grrd.yaml
```

- or just copy one of the clients and run it on your host.

```console
docker cp grr-server:/usr/share/grr-server/executables/installers/grr_3.4.6.7_amd64.pkg /tmp
```

- or head to
  [http://localhost:8000/#/manage-binaries](http://localhost:8100/#/manage-binaries)
  and pick the one for your platform.

Then connect the `grr-server` to the rest of your network:

```console
docker network connect dev_default grr-server
```

## Troubleshooting

### `Error response from daemon: Ports are not available: exposing port TCP 127.0.0.1:5001 -> 0.0.0.0:0: listen tcp 127.0.0.1:5001: bind: address already in use`

This means you have a service listening on a port required by the container.
It's common that this happens if you have other docker containers running, or
SSH tunneling with port forwarding going on. VSCode will sometimes forward ports
for you if you're doing remote development.

### `ModuleNotFoundError: No module named 'timesketch'`

Just retry the command (the installation happens after the docker container is
launched, so it may take a few seconds before the python scripts are done
installing)

### `"Service "tasks" is not running" when running a Docker compose command`

It probably means you're running the command from a directory that doesn't
contain a docker compose file (or that contains the wrong docker compose file).
Double-check the directories:

- `yeti-docker/prod/` for all things Yeti related
- `timesketch/docker/dev/` for all things Timesketch related
