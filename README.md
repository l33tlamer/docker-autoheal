# Docker Autoheal

*Monitor and restart unhealthy docker containers and get notified about it.*

Docker in Swarm Mode and kubernetes can automatically restart containers that become *unhealthy*. However "stand-alone" Docker still does not have this feature. There the **[healthcheck](https://docs.docker.com/engine/reference/builder/#healthcheck)** is a purely informative status and nothing more. This container allows you to automatically restart these troublesome containers and receive a notification about it. This is a fork of **[willfarrell/docker-autoheal](https://github.com/willfarrell/docker-autoheal/)**, all credit to the original creator. I have only added a few features to it and provide Docker images to use.

# Features that have been added:

* Native support for **[Pushover](https://pushover.net/)**, **[ntfy](https://ntfy.sh/)** and **[Gotify](https://gotify.net/)** notifications (without having to host a Apprise instance).

Especially **ntfy** and **Gotify** can be useful for **selfhosted** setups when either no internet connection is available and/or when use of thirdparty services should be minimized.

# Example docker-compose.yml:

````
version: "3.3"

services:
  autoheal:
    container_name: autoheal
    image: l33tlamer/docker-autoheal:latest
    restart: unless-stopped
    environment:
      - "PUSHOVER_USER_KEY=abcd"
      - "PUSHOVER_APP_TOKEN=1234"
      - "PUSHOVER_TITLE=Autoheal @ Raspberry Basement"
      - "NTFY_URL=http://192.168.20.50:8113/autoheal"
      - "NTFY_TOKEN=ABCD"
      - "NTFY_TITLE=Autoheal @ Raspberry Basement"
      - "GOTIFY_URL=http://192.168.20.50:8114/message?token=ABCD"
      - "GOTIFY_TITLE=Autoheal @ Raspberry Basement"
      - "GOTIFY_PRIORITY=2"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /etc/localtime:/etc/localtime:ro
````

# Environment Variables

The following **new** variables can be used:

| Variable | Default | Example | Explanation |
|:-------------:|:-------------:|:-------------:|:-------------:|
| `PUSHOVER_USER_KEY` | *none* | `ABCD` | *Your User Key* |
| `PUSHOVER_APP_TOKEN` | *none* | `1234` | *Your App Token* |
| `PUSHOVER_TITLE` | `Autoheal` | `Autoheal @ Raspberry Basement` | *Message Title to be used* |
| `NTFY_URL` | *none* | `http://192.168.20.50:8113/autoheal` | *Full URL to the ntfy instance, incl. the topic* |
| `NTFY_TOKEN` | *none* | `ABCD` | *Authorization Token* |
| `NTFY_TITLE` | `Autoheal` | `Autoheal @ Raspberry Basement` | *Message Title to be used* |
| `NTFY_PRIORITY` | `default` | `5` | *Priority of message (1-5, default=3, highest=5, lowest=1)*  |
| `NTFY_TAGS` | `repeat` | `repeat,heavy_check_mark` | *Tags/Emoji of message, comma-separated list* |
| `GOTIFY_URL` | *none* | `http://192.168.20.50:8114/message?token=ABCD` | *Full URL to the Gotify instance incl. token parameter* |
| `GOTIFY_TITLE` | `Autoheal` | `Autoheal @ Raspberry Basement` | *Message Title to be used* |
| `GOTIFY_PRIORITY` | `default` | `2` | *Priority of message (1-10, default=0, highest=10, lowest=1)*  |

These are the **original** variables offered by autoheal:

| Variable | Default | Example | Explanation |
|:-------------:|:-------------:|:-------------:|:-------------:|
| `AUTOHEAL_CONTAINER_LABEL` | `autoheal` | `healme` | *Label on containers to watch for (use `all` to watch all containers without using any label)* |
| `AUTOHEAL_INTERVAL` | `5` | `15` | *How often to check on health status (seconds)* |
| `AUTOHEAL_START_PERIOD` | `0` | `90` | *Grace period to wait before first check (seconds)* |
| `AUTOHEAL_DEFAULT_STOP_TIMEOUT` | `10` | `30` | *Grace period to wait before killing a stopped container (seconds)* |
| `DOCKER_SOCK` | `/var/run/docker.sock` | `tcp://localhost:2375` | *Either UNIX socket or TCP address for Docker host access* |
| `CURL_TIMEOUT` | `30` | `60` | *Timeout for curl connections to the Docker API* |
| `POST_RESTART_SCRIPT` | *none* | `/opt/script.sh` | *Script to execute after a restart (must be mounted into the container)* |
| `WEBHOOK_URL` | *none* | `http://example.com/webhook?ABCD` | *Webhook URL to trigger when a container has been restarted* |
| `APPRISE_URL` | *none* | `http://localhost:8000/notify` | *Apprise URL to trigger when a container has been restarted* |

**NOTE**: **[Apprise](https://github.com/caronc/apprise)** support appears to exist but is **currently not tested**.

# Optional container label

You can apply the label `autoheal.stop.timeout=20` to individual containers to override the the stop timeout on them. This can be useful if you have some that take much longer to properly stop. To avoid having them killed, increase the timeout accordingly.

# Docker UNIX Socket

Be aware that providing any container with direct access to the Docker UNIX socket is almost equal to providing that container (and any potential exploits) root access to the Docker host machine. Only give socket access (especially read/write) to container images you trust. The advantage is that this is the easiest setup.

To use autoheal with the Docker socket, you need to map the socket path on the host to the container.

Example: `/var/run/docker.sock:/var/run/docker.sock`

# Docker TCP

As a alternative, you can use the Docker host over TCP. Refer to the Docker documentation on **[how to enable it](https://docs.docker.com/config/daemon/remote-access/)** and **[how to secure it](https://docs.docker.com/engine/security/protect-access/)**.  

To use autoheal with Docker over TCP, you do not need to map a volume. Instead set the environment variable `DOCKER_SOCK`.

Example: `DOCKER_SOCK=tcp://HOST:PORT`

# Docker TCP with mTLS

If you protect the **[Docker TCP port with mTLS](https://docs.docker.com/engine/security/https/)**, you must specify `tcps://` as the protocol for `DOCKER_SOCK` and you need to provide the certificates and key to the autoheal container.

They need to be mounted to `/certs/ca.pem` `/certs/client-cert.pem` and `/certs/client-key.pem`

# Docker-Socket-Proxy

For increased security you can use something like **[Tecnativa/docker-socket-proxy](https://github.com/Tecnativa/docker-socket-proxy)** to restrict access to only specific parts of the Docker API over TCP. This may also be easier to setup and restrict to localhost than enabling the Docker host API mentioned above.

To use autoheal with Docker-Socket-Proxy, proceed as above, by using the environment variable `DOCKER_SOCK`.

# Timezone

If you want to make use of a specific timezone inside the container you can map `/etc/localtime` into the container.

Example: `/etc/localtime:/etc/localtime:ro`

<br>
