# Hagall

## The MIT License (MIT)
Copyright © 2022 Auki Labs Limited

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Introduction

_Hagall means Hail in Old Norse._

Hagall is a Real Time Networking Server responsible for processing, responding to and broadcasting networking messages to connected clients (participants) in a session similar to how a multiplayer networking engine handles message passing in an first-person-shooter game.

Hagall is through its module system and Entity Component System extensible, but in its core a simple networking engine that manages 3 types of abstractions:

- **Session** - A session facilitates the communication and in-memory persistence of participants, entities and actions inside an OpenGL coordinate system in unit meters. A session is similar to an FPS game session. Participants' positional data and actions are sent and broadcast as quickly as possible and only the messages that are required to retrieve the current state of the session are stored in-memory to support late joiners. Multiple sessions can exist in the same Hagall server and each session is identified by a string ID in the format `<hagall_id>x<session_id>` e.g. `5fx3a`.
- **Participant** - Represents a connected client e.g. a mobile device or other hardware that wishes to interact with entities and other participants in a session.
- **Entity** - An entity is an object in a session with a _Pose_ and an ID and it is owned by a specific participant. Hagall does not care about what an entity represents, it could be a 3D asset, an audio source or a particle system. It's up to the application implementer to map their game objects to corresponding entities.

The core responsibilities of Hagall are:

- Creation and deletion of sessions; participant authentication and participant joining/leaving sessions.
- Addition and deletion of entities.
- Broadcasting of messages to participants.

## Server Operator's Manual

While we at Auki Labs run several Hagall servers in multiple regions on AWS, we allow for anyone to become a Hagall server operator and help us run part of our infrastructure.

At a later stage we plan to launch a system that rewards Hagall server operators with tokens based on traffic served.

If you have spare compute resources, enough bandwidth and wish to become a Hagall server operator, please see a deployment method below for instructions on how to get started.

If you are running into problems, you can report them as [issues](https://github.com/aukilabs/hagall/issues) here on GitHub or talk to us on [Discord](https://discord.gg/aukiverse).

**It is important that your server is running 24/7 without interruption. When a server with active players is shut down, the players will lose their game session which is bad experience. Thus servers that misbehave will be delisted and not receive any more incoming traffic.**

### Minimum requirements

Most modern computers will be able to run Hagall. We have tested on desktops, laptops, servers and Raspberry Pis.

- An x86 or ARMv6+ processor
- At least 64 MiB of RAM
- At least 20 MiB of disk space
- A supported operating system, we currently provide pre-compiled binaries for Windows, macOS, Linux, FreeBSD, Solaris as well as Docker images

Additionally you need this in order to expose Hagall to the Internet:

- A web server or reverse proxy which
  - is compatible with HTTPS and WebSockets (HTTP/1.1 or later)
  - has an SSL certificate installed
- A stable Internet connection with
  - an externally accessible, public IP address for your reverse proxy to listen to
  - at least 10 Mbps downstream and upstream
- A domain name configured to point to your IP address

You can use our Docker Compose or Kubernetes setup that includes a basic nginx reverse proxy with a Let's Encrypt issued SSL certificate.

Auki's Hagall Discovery Service will perform regular checks on the health of your server to determine if it's fit to serve traffic. Make sure that you have enough spare compute capacity and bandwidth for hosting sessions or your server might be delisted from the network.

### Configuration

Configuration parameters can be passed to Hagall either as environment variables or flags.

`./hagall --public-endpoint https://hagall.example.com` (where `https://hagall.example.com` is the public, external address where this Hagall server is reachable) will launch Hagall with sane defaults, but if you for some reason want to modify the default configuration of Hagall you can run `./hagall -h` for a full list of parameters.

Here are some example parameters:

| Flag              | Environment variable   | Default | Example                    | Description                                                                                                                |
| ----------------- | ---------------------- | ------- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| --addr            | HAGALL_ADDR            | :4000   | :4000                      | Listening address for client connections. This is the port you want your reverse proxy to forward traffic to.              |
| --log-indent      | HAGALL_LOG_INDENT      | false   | true                       | Indent logs                                                                                                                |
| --public-endpoint | HAGALL_PUBLIC_ENDPOINT | _N/A_   | https://hagall.example.com | The public endpoint where this Hagall server is reachable. This endpoint will be registered with Hagall Discovery Service. |
| --log-level       | HAGALL_LOG_LEVEL       | info    | debug                      | The log level (debug, info, warning or error)                                                                              |

### Binary

> **_NOTE:_** If you choose to use this deployment method, you need to set up your own HTTPS web server or reverse proxy with an SSL certificate. Hagall listens for incoming connections on port 4000 by default, but this is changeable, see the configuration options above.

#### Currently supported platforms

We build pre-compiled binaries for these operating systems and architectures:

* Windows x86, x86_64
* macOS x86_64, ARM64 (M1)
* Linux x86, x86_64, ARM, ARM64
* FreeBSD x86, x86_64
* Solaris x86_64

> **_NOTE:_** Auki Labs doesn't test all of these platforms actively. Windows, FreeBSD and Solaris builds are currently experimental. We don't guarantee that everything works, but feel free to reach out with your test results.

1. Download the latest Hagall from [GitHub](https://github.com/aukilabs/hagall/releases)
2. Run it with `./hagall --public-endpoint=https://hagall.example.com`
3. Expose it using your own reverse proxy with an SSL certificate.

We comment you to use a daemon manager such as systemd, launchd, SysV Init, runit or Supervisord, to make sure Hagall stays running at all times.

Make sure that your web server or reverse proxy has support for WebSockets. If you choose to use nginx, this configuration is needed to enable WebSocket support for the Hagall upstream:
```
proxy_http_version 1.1;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

### Docker

Hagall is available on [Docker Hub](https://hub.docker.com/r/aukilabs/hagall).

Here's an example of how to run it:

```
docker run -e HAGALL_PUBLIC_ENDPOINT=https://hagall.example.com aukilabs/hagall:stable
```

Hagall listens for incoming traffic on port 4000 by default.

#### Supported tags
_See the full list on [Docker Hub](https://hub.docker.com/r/aukilabs/hagall)._

* `latest` (bleeding edge, not recommended)
* `stable` (latest stable version, recommended)
* `v0` (specific major version)
* `v0.4` (specific minor version)
* `v0.4.27` (specific patch version)

#### Upgrading

If you're using a non-version specific tag (`stable` or `latest`) or if the version tag you use matches the new version of Hagall you want to upgrade to, simply run `docker pull aukilabs/hagall:stable` (where `stable` is the tag you use) and then restart your container with `docker restart hagall` (if `hagall` is the name of your container).

If you're using a version specific tag and the new version of Hagall you want to upgrade to doesn't match the tag you use, you need to first change the tag you use and then restart your container. (`v0` matches any v0.x.x version, `v0.4` matches any v0.4.x version and so on.)

### Docker Compose

Since Hagall needs to be exposed with an HTTPS address and Hagall itself doesn't terminate HTTPS, we recommend you to use our Docker Compose file that sets up an `nginx-proxy` container that terminates HTTPS and a `letsencrypt` container that obtains a free Let's Encrypt SSL certificate alongside Hagall.

1. Configure your domain name to point to your externally exposed public IP address and configure any firewalls and port forwarding rules to allow incoming traffic to port 80 and 443.
2. Download the latest Docker Compose YAML file from [GitHub](https://github.com/aukilabs/hagall/blob/main/docker-compose.yml).
3. Configure the environment variables to your liking (you must at least set `VIRTUAL_HOST`, `LETSENCRYPT_HOST` and `HAGALL_PUBLIC_ENDPOINT`, set these to the domain name you configured in step 1).
4. With the YAML file in the same folder, start the containers using Docker Compose: `docker-compose up -d`

#### Upgrading

You can do the same steps as for Docker, but if you're not already running Hagall or you have modified the `docker-compose.yml` file recently and want to deploy the changes, you can navigate to the folder where you have your `docker-compose.yml` file and then run `docker-compose pull` followed by `docker-compose down hagall` and `docker-compose up -d hagall`.

### Kubernetes

Auki provides a Helm chart for running Hagall in Kubernetes. We recommend that you use this Helm chart rather than writing your own Kubernetes manifests. For more information about [what Helm is](https://helm.sh/docs/topics/architecture/) and how to [install](https://helm.sh/docs/intro/install/) it, see Helm's official website.

#### Requirements

* Kubernetes 1.14+
* Helm 3
* An HTTPS and WebSocket compatible ingress controller with an SSL certificate that has already been configured

#### Installing

The chart can be deployed by CI/CD tools such as ArgoCD or Flux or it can be deployed using Helm on the command line like this:

```
helm repo add aukilabs https://charts.aukiverse.com
helm install hagall aukilabs/hagall --set config.HAGALL_PUBLIC_ENDPOINT=https://hagall.example.com
```

#### Uninstalling

To uninstall (delete) the `hagall` deployment:

```
helm delete hagall
```

#### Values

Please see [values.yaml](https://github.com/aukilabs/helm-charts/blob/main/charts/hagall/values.yaml) for the available values and their defaults.

Values can be overridden either by using a values file (the `-f` or `--values` flags) or by setting them on the command line using the `--set` flag. For more information, see the official [documentation](https://helm.sh/docs/helm/helm_install/).

You must at least set the `config.HAGALL_PUBLIC_ENDPOINT` key for server registration to work. But depending on which ingress controller you use, you need to set `ingress.enabled=true`, `ingress.hosts[0].host=hagall.example.com` and so on.

#### Upgrading

We recommend you to change to use `image.pullPolicy: Always` if you use a non-specific version tag like `stable`/`v0`/`v0.4` (configured by changing the `image.tag` value of the Helm chart), or choose to use a specific version tag like `v0.4.33`. Check *Supported tags* or the *Tags* tab on [Docker Hub](https://hub.docker.com/r/aukilabs/hagall) for the tags you can use.
