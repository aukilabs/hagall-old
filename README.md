# The MIT License (MIT)
Copyright © 2022 Auki Labs Limitd

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

# Hagall

This repository only contains Hagall releases as Hagall is not open source at this moment.

![](hagall.png)

_Hagall means Hail in Old Norse._

Hagall is a Real Time Networking Server responsible for processing, responding to and broadcasting networking messages to connected clients (participants) in a session similar to how a multiplayer networking engine handles message passing in an first-person-shooter game.

Hagall is through its module system extensible, but in its core a simple networking engine that manages 3 types of abstractions:

- **Session** - A session facilitates the communication and in-memory persistence of participants, entities and actions inside an OpenGL coordinate system in unit meters. A session is similar to an FPS game session. Participants' positional data and actions are sent and broadcast as quickly as possible and only the messages that are required to retrieve the current state of the session are stored in-memory to support late joiners. Multiple sessions can exist in the same Hagall server and each session is identified by a string ID in the format `<hagall_id>x<session_id>` e.g. `5fx3a`.
- **Participant** - Represents a connected client e.g. a mobile device or other hardware that wishes to interact with entities and other participants in a session.
- **Entity** - An entity is an object in a session with a _Pose_ and an ID and it is owned by a specific participant. Hagall does not care about what an entity represents, it could be a 3D asset, an audio source or a particle system. It's up to the application implementer to map their game objects to corresponding entities.

The core responsibilities of Hagall are:

- Creation and deletion of sessions; participant authentication and participant joining/leaving sessions.
- Addition and deletion of entities.
- Broadcasting of messages to participants.

# Server Operator's Manual

While we at Auki Labs run several Hagall servers in multiple regions on AWS, we allow for anyone to become a Hagall server operator and help us run part of our infrastructure.

At a later stage we plan to launch a system that rewards Hagall server operators with tokens based on traffic served.

If you have spare compute resources, enough bandwidth and wish to become a Hagall server operator, please see a deployment method below for instructions on how to get started.

## Running a Hagall server

### Minimum requirements

- x86 or ARM processor
- At least 64 MiB of RAM
- A stable Internet connection, we recommend at least 10 Mbps downstream and upstream
- A supported operating system, we currently provide pre-compiled binaries for Windows, macOS, Linux, FreeBSD, Solaris as well as Docker
- An HTTPS compatible web server or reverse proxy with an SSL certificate installed, alternatively use our Docker Compose or Kubernetes set-up that includes it

Auki's Hagall Discovery Service will perform regular checks on the health of your server to determine if it's fit to serve traffic. Make sure that you have enough spare compute capacity and bandwidth for hosting sessions or your server might be delisted from the network.

### Configuration

Configuration parameters can be passed to Hagall either as environment variables or flags.

`./hagall --public-endpoint https://hagall.example.com` (where `https://hagall.example.com` is the public, external address where this Hagall server is reachable) will launch Hagall with sane defaults, but if you for some reason want to modify the default configuration of Hagall you can run `./hagall -h` for a full list of parameters.

Here are some example parameters:

| Flag             | Environment variable   | Default | Sample                     | Description                                                                                                                |
| ---------------- | ---------------------- | ------- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| --wallet-addr     | HAGALL_WALLET_ADDR     | _N/A_   | 0x0                        | The crypto wallet address to be rewarded. (currently N/A)                                                                  |
| --public-endpoint | HAGALL_PUBLIC_ENDPOINT | _N/A_   | https://hagall.example.com | The public endpoint where this Hagall server is reachable. This endpoint will be registered with Hagall Discovery Service. |
| --log-level       | HAGALL_LOG_LEVEL       | info    | debug                      | The log level (debug, info, warning or error)                                                                              |
## Binary

> **_NOTE:_** If you choose to use this deployment method, you need to set up your own HTTPS web server or reverse proxy with an SSL certificate.

### Currently supported platforms

We build pre-compiled binaries for these operating systems and architectures:

* Windows x86, x86_64
* macOS x86_64, ARM64 (M1)
* Linux x86, x86_64, ARM, ARM64
* FreeBSD x86, x86_64
* Solaris x86_64

1. Download the latest Hagall from [GitHub](https://github.com/aukilabs/hagall/releases)
2. Run it with `./hagall --public-endpoint=https://hagall.example.com`

You can optionally use a daemon manager such as systemd, launchd, SysV Init, runit or Supervisord.

## Docker

Hagall is available on [Docker Hub](https://hub.docker.com/r/aukilabs/hagall).

Here's an example of how to run it:

```
docker run -e HAGALL_PUBLIC_ENDPOINT=https://hagall.example.com aukilabs/hagall:stable
```

### Supported tags
_See the full list on [Docker Hub](https://hub.docker.com/r/aukilabs/hagall)._

* `latest` (bleeding edge, not recommended)
* `stable` (latest stable version, recommended)
* `v0` (specific major version)
* `v0.4` (specific minor version)
* `v0.4.14` (specific patch version)

### Docker Compose

Since Hagall needs to be exposed with an HTTPS address and Hagall itself doesn't terminate HTTPS, we recommend you to use our Docker Compose file that sets up an nginx proxy container that terminates HTTPS and a letsencrypt container that obtains a free Let's Encrypt SSL certificate alongside Hagall.

1. Download the latest Docker Compose YAML file from [GitHub](https://github.com/aukilabs/hagall/blob/main/docker-compose.yml)
2. Configure the environment variables to your liking (you must at least set `VIRTUAL_HOST`, `LETSENCRYPT_HOST` and `HAGALL_PUBLIC_ENDPOINT`)
3. With the YAML file in the same folder, start the containers using Docker Compose: `docker-compose up -d`

## Kubernetes

Auki provides a Helm chart for running Hagall in Kubernetes. We recommend that you use this Helm chart rather than writing your own Kubernetes manifests. For more information about [what Helm is](https://helm.sh/docs/topics/architecture/) and how to [install](https://helm.sh/docs/intro/install/) it, see Helm's official website.

### Requirements

* Kubernetes 1.14+
* Helm 3
* An HTTPS compatible ingress controller

### Installing

The chart can be deployed by CI/CD tools such as ArgoCD or Flux or it can be deployed using Helm on the command line like this:

```
helm repo add aukilabs https://charts.aukiverse.com
helm install hagall aukilabs/hagall --set config.HAGALL_PUBLIC_ENDPOINT=https://hagall.example.com
```

### Uninstalling

To uninstall (delete) the `hagall` deployment:

```
helm delete hagall
```

### Values

Please see [values.yaml](https://github.com/aukilabs/helm-charts/blob/main/charts/hagall/values.yaml) for the available values and their defaults.

Values can be overridden either by using a values file (the `-f` or `--values` flags) or by setting them on the command line using the `--set` flag. For more information, see the official [documentation](https://helm.sh/docs/helm/helm_install/).

You must at least set the `config.HAGALL_PUBLIC_ENDPOINT` key for server registration to work.


