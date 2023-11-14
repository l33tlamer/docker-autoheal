# Docker Autoheal

Monitor and restart unhealthy docker containers. 
This functionality was proposed to be included with the addition of `HEALTHCHECK`, however didn't make the cut.
This container is a stand-in till there is native support for `--exit-on-unhealthy` https://github.com/docker/docker/pull/22719.

# This is a fork of https://github.com/willfarrell/docker-autoheal

Features that have been added:

* Support for **[Pushover](https://pushover.net/)**, **[ntfy](https://ntfy.sh/)** and **[Gotify](https://gotify.net/)** notifications.

In addition to the existing, the following new **environment variables** can be used:

| `Variable`  | *Default* | `Example` | *Explanation* |
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

Example of a complete docker-compose.yml

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

