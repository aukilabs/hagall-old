# Hagall

## The MIT License (MIT)
Copyright © 2022 Auki Labs Limited

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the “Software”), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Introduction

_Hagall means Hail in Old Norse._

Hagall is a Real-Time Networking Server responsible for processing, responding to and broadcasting networking messages to connected clients (participants) in a session similar to how a multiplayer networking engine handles message passing in a first-person-shooter game.

Hagall is, through its module system and Entity Component System, extensible, but at its core, a simple networking engine that manages 3 types of abstractions:

- **Session** - A session facilitates the communication and in-memory persistence of participants, entities and actions inside an OpenGL coordinate system in unit meters. A session is similar to an FPS game session. Participants' positional data and actions are sent and broadcast as quickly as possible, and only the messages that are required to retrieve the current state of the session are stored in memory to support late joiners. Multiple sessions can exist in the same Hagall server, and each session is identified by a string ID in the format `<hagall_id>x<session_id>` e.g. `5fx3a`.
- **Participant** - Represents a connected client e.g., a mobile device or other hardware that wishes to interact with entities and other participants in a session.
- **Entity** - An entity is an object in a session with a _Pose_ and an ID, and it is owned by a specific participant. Hagall does not care about what an entity represents. It could be a 3D asset, an audio source, or a particle system. It's up to the application implementer to map their game objects to corresponding entities.

The core responsibilities of Hagall are:

- Creation and deletion of sessions; participant authentication and participant joining/leaving sessions.
- Addition and deletion of entities.
- Broadcasting of messages to participants.

Every Hagall server needs a unique wallet to participate in the [posemesh economy](https://www.posemesh.org/#economy).

## Video tutorial

This tutorial will walk you through the process of setting up a Hagall node using Docker Compose. Scroll down for other setup options and detailed guidance.

[![Getting started with Hagall](https://img.youtube.com/vi/sb0Qwe413_k/0.jpg)](https://www.youtube.com/watch?v=sb0Qwe413_k)

## Server Operator's Manual

While we at Auki Labs run several Hagall servers in multiple regions on AWS, we allow anyone to become a Hagall server operator and help us run part of the posemesh infrastructure.

After TGE, server operators will be rewarded with tokens for participating. At a later stage, we plan to reward operators based on traffic served.

If you have spare compute resources, enough bandwidth and wish to become a Hagall server operator, please see the deployment methods below for instructions on how to get started.

If you are running into problems, you can report them as [issues](https://github.com/aukilabs/hagall/issues) here on GitHub or talk to us on [Discord](https://discord.gg/aukiverse).

**It is important that your server is running 24/7 without interruption. When a server with active participants is shut down, the participants will lose their session, which give them a bad experience. Thus servers that misbehave will be delisted and not receive any more incoming traffic.**

### Minimum requirements

Most modern computers will be able to run Hagall. We have tested on desktops, laptops, servers and Raspberry Pis.

- An x86 or ARMv6+ processor
- At least 64 MiB of RAM
- At least 20 MiB of disk space
- A supported operating system, we currently provide pre-compiled binaries for Windows, macOS, Linux, FreeBSD, Solaris as well as Docker images

Additionally, you need this in order to expose Hagall to the Internet:

- A web server or reverse proxy which
  - is compatible with HTTPS and WebSockets (HTTP/1.1 or later)
  - has an SSL certificate installed
- A stable Internet connection with
  - an externally accessible, public IP address for your reverse proxy to listen to
  - at least 10 Mbps downstream and upstream
- A domain name configured to point to your IP address

You can use our Docker Compose or Kubernetes setup that includes a basic nginx reverse proxy with a Let's Encrypt issued SSL certificate.

Auki's Hagall Discovery Service will perform regular checks on the health of your server to determine if it's fit to serve traffic. Make sure that you have enough spare compute capacity and bandwidth for hosting sessions, or your server might be delisted from the network.

### Configuration

Configuration parameters can be passed to Hagall either as environment variables or flags.

`./hagall --public-endpoint https://hagall.example.com` (where `https://hagall.example.com` is the public, external address where this Hagall server is reachable) will launch Hagall with sane defaults, but if you for some reason want to modify the default configuration of Hagall you can run `./hagall -h` for a full list of parameters.

Here are some example parameters:

| Flag               | Environment variable    | Default | Example                    | Description                                                                                                                |
| ------------------ | ----------------------- | ------- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| --addr             | HAGALL_ADDR             | :4000   | :4000                      | Listening address for client connections. This is the port you want your reverse proxy to forward traffic to.              |
| --log-indent       | HAGALL_LOG_INDENT       | false   | true                       | Indent logs                                                                                                                |
| --public-endpoint  | HAGALL_PUBLIC_ENDPOINT  | _N/A_   | https://hagall.example.com | The public endpoint where this Hagall server is reachable. This endpoint will be registered with Hagall Discovery Service. |
| --log-level        | HAGALL_LOG_LEVEL        | info    | debug                      | The log level (debug, info, warning or error)                                                                              |
| --private-key-file | HAGALL_PRIVATE_KEY_FILE | _N/A_   | hagall-private.key         | The file that contains the private key of a Hagall server-unique Ethereum-compatible wallet                              |
| --private-key      | HAGALL_PRIVATE_KEY      | _N/A_   | 0x0                        | The private key of a Hagall server-unique Ethereum-compatible wallet                              |


Every Hagall server needs a unique wallet. You can generate it in a wallet app of your choice (such as MetaMask) and copy its private key to a file called `hagall-private.key`.
If you wish to generate a wallet on the command-line, make sure that you back up your private key (for example by adding it to your wallet app) to not lose your stake or rewards. Here is an example command to generate a wallet and save its private key to a file so Hagall can use it:
```
pip3 install web3
python3 -c "from web3 import Web3; w3 = Web3(); acc = w3.eth.account.create(); print(f'{w3.to_hex(acc.key)}')" > hagall-private.key
chmod 400 hagall-private.key
```

We recommend that the private key is supplied as a file (`HAGALL_PRIVATE_KEY_FILE`) rather than directly on the command line or through environment variables (`HAGALL_PRIVATE_KEY`) as files are more secure.

**DO NOT CONFIGURE A WALLET WITH EXISTING ASSETS**, instead generate a new wallet for every Hagall server you operate.
The private key of your wallet is only used by Hagall for authentication and verification of your reputation deposit and will stay on your machine. But if someone gains access to the private key file on your server, they will get access to your wallet, so please take appropriate precautions.

### Binary

> **_NOTE:_** If you choose to use this deployment method, you need to set up your own HTTPS web server or reverse proxy with an SSL certificate. Hagall listens for incoming connections on port 4000 by default, but this is changeable, see the configuration options above.

#### Currently supported platforms

We build pre-compiled binaries for these operating systems and architectures:

* Windows x86, x86_64
* macOS x86_64, ARM64 (M1)
* Linux x86, x86_64, ARM, ARM64
* FreeBSD x86, x86_64
* Solaris x86_64

> **_NOTE:_** Auki Labs doesn't test all of these platforms actively. Windows, FreeBSD and Solaris builds are currently experimental. We don't guarantee that everything works but feel free to reach out with your test results.

1. Download the latest Hagall from [GitHub](https://github.com/aukilabs/hagall/releases).
2. Generate an Ethereum-compatible wallet and put its private key in a file called `hagall-private.key` like the example above.
3. Run it with `./hagall --public-endpoint=https://hagall.example.com --private-key-file hagall-private.key`
4. Expose it using your own reverse proxy with an SSL certificate.

We recommend you use a daemon manager such as systemd, launchd, SysV Init, runit or Supervisord, to make sure Hagall stays running at all times.

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
docker run --name=hagall --restart=unless-stopped --detach --mount "type=bind,source=$(pwd)/hagall-private.key,target=/hagall-private.key,readonly" -e HAGALL_PUBLIC_ENDPOINT=https://hagall.example.com -e HAGALL_PRIVATE_KEY_FILE=/hagall-private.key -p 4000:4000 aukilabs/hagall:stable
```

Hagall listens for incoming traffic on port 4000 by default. The port can be changed by
changing Hagall's [configuration](#Configuration) or by simply changing the publish
(`-p`) argument in the `docker run` command.

We also recommend you to configure Docker to start automatically with your operating system. Using `--restart=unless-stopped` in your `docker run` command will start Hagall automatically after the Docker daemon has started.

#### Supported tags
_See the full list on [Docker Hub](https://hub.docker.com/r/aukilabs/hagall)._

* `latest` (bleeding edge, not recommended)
* `stable` (latest stable version, recommended)
* `v0` (specific major version)
* `v0.5` (specific minor version)
* `v0.5.0` (specific patch version)

#### Upgrading

If you're using a non-version specific tag (`stable` or `latest`) or if the version tag you use matches the new version of Hagall you want to upgrade to, simply run `docker pull aukilabs/hagall:stable` (where `stable` is the tag you use) and then restart your container with `docker restart hagall` (if `hagall` is the name of your container).

If you're using a version-specific tag and the new version of Hagall you want to upgrade to doesn't match the tag you use, you need to first change the tag you use and then restart your container. (`v0` matches any v0.x.x version, `v0.5` matches any v0.5.x version, and so on.)

### Docker Compose

Since Hagall needs to be exposed with an HTTPS address and Hagall itself doesn't terminate HTTPS, instead of using the pure Docker setup as described above, we recommend you to use our Docker Compose file that sets up an `nginx-proxy` container that terminates HTTPS and a `letsencrypt` container that obtains a free Let's Encrypt SSL certificate alongside Hagall.

1. Configure your domain name to point to your externally exposed public IP address and configure any firewalls and port forwarding rules to allow incoming traffic to ports 80 and 443.
2. Download the latest Docker Compose YAML file from [GitHub](https://github.com/aukilabs/hagall/blob/main/docker-compose.yml).
3. Configure the environment variables to your liking (you must at least set `VIRTUAL_HOST`, `LETSENCRYPT_HOST` and `HAGALL_PUBLIC_ENDPOINT`, set these to the domain name you configured in step 1).
4. Generate an Ethereum-compatible wallet and put its private key in a file called `hagall-private.key` like in the example above; alternatively, see this guide on [how to generate a wallet with MetaMask](https://www.posemesh.org/hagall-upgrade-guide).
5. With the YAML file in the same folder, start the containers using Docker Compose: `docker-compose up -d`

Just as with the pure Docker setup, we recommend you configure Docker to start automatically with your operating system. If you use our standard Docker Compose YAML file, the containers will start automatically after the Docker daemon has started.

#### Upgrading

You can do the same steps as for Docker, but if you're not already running Hagall or you have modified the `docker-compose.yml` file recently and want to deploy the changes, you can navigate to the folder where you have your `docker-compose.yml` file and then run `docker-compose pull` followed by `docker-compose down` and `docker-compose up -d`.

Note that the `docker-compose pull` command will also upgrade the other containers defined in `docker-compose.yml` such as the nginx proxy and the Let's Encrypt helper.

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
helm install hagall aukilabs/hagall --set config.HAGALL_PUBLIC_ENDPOINT=https://hagall.example.com --set-file secrets.privateKey=hagall-private.key
```

#### Uninstalling

To uninstall (delete) the `hagall` deployment:

```
helm delete hagall
```

#### Values

Please see [values.yaml](https://github.com/aukilabs/helm-charts/blob/main/charts/hagall/values.yaml) for the available values and their defaults.

Values can be overridden either by using a values file (the `-f` or `--values` flags) or by setting them on the command line using the `--set` flag. For more information, see the official [documentation](https://helm.sh/docs/helm/helm_install/).

You must at least set the `config.HAGALL_PUBLIC_ENDPOINT` key for server registration to work. But depending on which ingress controller you use, you need to set `ingress.enabled=true`, `ingress.hosts[0].host=hagall.example.com` and so on. You also need to configure a secret containing the private key of your Hagall-exclusive wallet, one per Hagall server, either using an existing secret inside Kubernetes or by passing the wallet as a file, letting the chart create the Kubernetes secret for you.

#### Upgrading

We recommend you change to use `image.pullPolicy: Always` if you use a non-specific version tag like `stable`/`v0`/`v0.5` (configured by changing the `image.tag` value of the Helm chart) or choose to use a specific version tag like `v0.5.0`. Check *Supported tags* or the *Tags* tab on [Docker Hub](https://hub.docker.com/r/aukilabs/hagall) for the tags you can use.

### Testing

After launching Hagall, it's a good idea to take a look at the logs to make sure your server is registered, working and accessible. There are a few things you can look for:

* "hagall is successfully registered to hds" should show up in the log
* `"message":"new client is connected","tags":{"app-key":"0xSMOKE"}}` should show up in the log. These are health checks (also known as smoke tests) running from the central Hagall Discovery Service (HDS) to test that the server is working
* You can also check metrics like `ws_connected_clients`, see more [details](#Metrics) below

### Troubleshooting and metrics

If registration to HDS fails, check the status code in the log message. The status code is the response from HDS when it tries to call your Hagall server.

* `"status":"504 Gateway Timeout"` or `403 Forbidden` could mean that HDS couldn't reach your Hagall instance because the connection to your public endpoint (URL) timed out. It could happen because you didn't do port forwarding in your router or didn't allow your web server / reverse proxy (such as nginx) or port 443 in your firewall. We have also seen cases where Internet Service Providers blocked common service ports like 80 and 443. Here's an example for [Orcon](https://help.orcon.net.nz/hc/en-us/articles/360005168154-Port-filtering-in-My-Orcon).
* You can test that your Hagall is reachable from the public Internet using [reqbin.com](https://reqbin.com/). Write your Hagall URL (the "public endpoint" address you configured) and press the Send button. If you get a status 400 (Bad Request) back, the text is green and the content says "not websocket protocol", everything is working as it should.
* `403 Forbidden` could also happen if your clock is out of sync. Sync it and try again. It could also happen if you run two Hagall servers at the same time with the same public endpoint (URL). Make sure you only run one Hagall with one unique public endpoint.
* `503 Service Unavailable` means that HDS reaches a reverse proxy (such as nginx), but the backend server (Hagall) is unavailable. Check the reverse proxy logs to verify that HDS actually reaches the reverse proxy you think it reaches. And make sure Hagall is running and that the reverse proxy points to Hagall using the address it listens to (port 4000 by default). You can also try to restart your reverse proxy.
* `400 Bad Request` could mean that the format of your public endpoint is invalid. It needs to be prefixed with `https://` and not `http://`. Follow the same format as in the [configuration](#Configuration) section. Note that if you use the Docker Compose setup, `VIRTUAL_HOST` and `LETSENCRYPT_HOST` should be without the `https://` prefix and `HAGALL_PUBLIC_ENDPOINT` should have the `https://` prefix.

#### Admin port

Other than the ordinary port used for WebSocket traffic, Hagall is also listening on a separate port for administrative purposes.
This port is not exposed externally by default and should not be, so you would have to connect to it from the same machine as you're running Hagall on or from your internal network only.

```
    --admin-addr                string               Admin listening address.
                                                     Env:     HAGALL_ADMIN_ADDR
                                                     Default: ":18190"
```

For example, if Hagall is running on your local machine and you visit http://localhost:18190/debug/pprof/ you can get profiling data.
If you set up Prometheus, you can scrape metrics from the /metrics endpoint.

##### Endpoints

* `/metrics` - Prometheus-formatted metrics
* `/health` - Health check endpoint, returns 200 OK if service is running
* `/debug/pprof/` - Index page of Go's [pprof](https://pkg.go.dev/net/http/pprof) package
  * There are many sub-endpoints useful for profiling. Please refer to the index page

#### Metrics

Other than the standard Golang metrics coming from [prometheus/client_golang](https://github.com/prometheus/client_golang) with `go_*`, `process_*` prefixes etc., these additional metrics are available:

* `ws_*` - WebSocket related metrics
  * A very useful metric to look at is `ws_connected_clients`, which is a gauge that represents the number of connected WebSocket clients. Note that the smoke tests are also WebSocket clients, so it will flip between 0 and 1 during these tests.
