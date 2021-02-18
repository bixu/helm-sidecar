# The Helm Sidecar

This repo implements the Sidecar pattern for containers using Helm in a
minimalist fashion.

## Setup

Run the following commands to set up a local Kubernetes and install the pattern
using Helm (the package manager for Kubernetes).

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew bundle
kind create cluster
helm install --wait sidecar .
```

(Note that this worfklow assume macOS, although there _is_ support for Brew on
Linux ;)

## Manually Testing the Pattern

To manually interact with sidecar containers inside the Kubernetes pods created
by Helm, we can do the following:

```shell
FIRST_POD=$(kubectl get pods | grep Running | cut -d' ' -f1 | head -n1)
kubectl exec -it $FIRST_POD --container=ubuntu -- bash
```

Now we are inside the Ubuntu container we are going to use as a "client" for
both Redis and Memcached.

Let's install Netcat to do some low-level connectivity testing:

```shell
apt update && apt install --yes netcat
```

Let's verify that neither Redis nor Memcached are running in this container:

```shell
ps -ef
```

Now we can send a "ping" to the Redis sidecare over local networking in the Pod:

```shell
echo -e '*1\r\n$4\r\nPING\r\n' \
  | nc -q 0 localhost 6379

+PONG
```

...and a `stats` command to the Memcached sidecar:

```shell
echo -e 'stats' \
  | nc -q 0 localhost 11211

STAT pid 1
STAT uptime 1685
STAT time 1613674569

... truncated output ...

STAT moves_within_lru 0
STAT direct_reclaims 0
STAT lru_bumps_dropped 0
END
```
