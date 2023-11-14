## This is a fork of willfarrell/docker-autoheal

# Docker Autoheal

Monitor and restart unhealthy docker containers. 
This functionality was proposed to be included with the addition of `HEALTHCHECK`, however didn't make the cut.
This container is a stand-in till there is native support for `--exit-on-unhealthy` https://github.com/docker/docker/pull/22719.

# Features that have been added:

* Support for **[Pushover](https://pushover.net/)**, **[ntfy](https://ntfy.sh/)** and **[Gotify](https://gotify.net/)** notifications.

In addition to the existing, the following new **environment variables** can be used:

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

# Example of a complete `docker-compose.yml`

````
version: "3.3"

services:
  autoheal:
    container_name: autoheal
    image: l33tlamer/docker-autoheal:latest
    restart: unless-stopped
    environment:
      - "AUTOHEAL_CONTAINER_LABEL=autoheal"
      - "AUTOHEAL_INTERVAL=10"
      - "AUTOHEAL_START_PERIOD=60"
      - "AUTOHEAL_DEFAULT_STOP_TIMEOUT=30"
      - "CURL_TIMEOUT=30"
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

# Docker UNIX Socket

Be aware that providing any container direct access to the Docker UNIX socket is almost equal to providing that container (and any potential exploits) root access to the Docker host machine. Only give socket access (especially read/write) to container images you trust. The advantage is that this is the easiest setup.

To use autoheal with the Docker socket, you need to map the socket path on the host to the container.

Example: `/var/run/docker.sock:/var/run/docker.sock`

# Docker TCP

As a alternative, you can use the Docker host over TCP (refer to the Docker documentation on how to enable and how to secure it).

To use autoheal with Docker over TCP, you do not need to map a volume. Instead set the environment variable `DOCKER_SOCK`.

Example: `DOCKER_SOCK=tcp://HOST:PORT`

# Docker-Socket-Proxy

For increased security i highly recommend using something like https://github.com/Tecnativa/docker-socket-proxy to restrict access to only specific parts of the Docker API over TCP. This may also be easier to setup and restrict to localhost than enabling the Docker host API mentioned above.

To use autoheal with Docker-Socket-Proxy, same as above, set the environment variable `DOCKER_SOCK`.

Example: `DOCKER_SOCK=tcp://HOST:PORT`

# Environment Variables

These are the original variables offered by autoheal:

| Variable | Default | Example | Explanation |
|:-------------:|:-------------:|:-------------:|:-------------:|
| `AUTOHEAL_CONTAINER_LABEL` | `autoheal` | `healme` | *Label on containers to watch for* |
| `AUTOHEAL_INTERVAL` | `5` | `15` | *How often to check on health status (seconds)* |
| `AUTOHEAL_START_PERIOD` | `0` | `90` | *Grace period to wait before first check (seconds)* |
| `AUTOHEAL_DEFAULT_STOP_TIMEOUT` | `10` | `30` | *Grace period to wait before killing a stopped container (seconds)* |
| `DOCKER_SOCK` | `/var/run/docker.sock` | `tcp://localhost:2375` | *Either UNIX socket or TCP address for Docker host access* |
| `CURL_TIMEOUT` | `30` | `60` | *Timeout for curl connections to the Docker API* |
| `WEBHOOK_URL` | *none* | `http://discord.com/ABCD` | *Webhook URL to trigger when a container has been restarted* |

# Timezone

If you want to make use of a specific timezone inside the container you can map `/etc/localtime` into the container.

Example: `/etc/localtime:/etc/localtime:ro`


