# ns8-patchmon

This is a template module for [NethServer 8](https://github.com/dz00te/ns8-core).

## Install

Instantiate the module with:

```bash
add-module ghcr.io/dz00te/patchmon:latest 1
```

The output of the command will return the instance name.
Output example:

```json
{"module_id": "patchmon1", "image_name": "patchmon", "image_url": "ghcr.io/dz00te/patchmon:latest"}
```

## Configure

Let's assume that the patchmon instance is named `patchmon1`.

Launch `configure-module`, by setting the following parameters:

- `host`: a fully qualified domain name for the application
- `http2https`: enable or disable HTTP to HTTPS redirection (true/false)
- `lets_encrypt`: enable or disable Let's Encrypt certificate (true/false)

Example:

```bash
api-cli run configure-module --agent module/patchmon1 --data - <<EOF
{
  "host": "patchmon.domain.com",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

The above command will:

- start and configure the patchmon instance
- configure a virtual host for trafik to access the instance

## Get the configuration

You can retrieve the configuration with:

```bash
api-cli run get-configuration --agent module/patchmon1
```

## Update

```bash
api-cli run update-module --data '{"module_url":"ghcr.io/dt00te/patchmon:latest","instances":["patchmon1"],"force":true}'
```

## Uninstall

To uninstall the instance:

```bash
remove-module --no-preserve patchmon1
```

## Smarthost setting discovery

Some configuration settings, like the smarthost setup, are not part of the
`configure-module` action input: they are discovered by looking at some
Redis keys. To ensure the module is always up-to-date with the
centralized [smarthost
setup](https://dz00te.github.io/ns8-core/core/smarthost/) every time
patchmon starts, the command `bin/discover-smarthost` runs and refreshes
the `state/smarthost.env` file with fresh values from Redis.

Furthermore if smarthost setup is changed when patchmon is already
running, the event handler `events/smarthost-changed/10reload_services`
restarts the main module service.

See also the `imageroot/systemd/user/patchmon.service` file.

This setting discovery is just an example to understand how the module is
expected to work: it can be rewritten or discarded completely.

## Debug

Some CLI commands are needed to debug:

- The module runs under an agent that initiates a lot of environment variables (in `/home/patchmon1/.config/state`), it could be nice to verify them on the root terminal:

  ```bash
  runagent -m patchmon1 env
  ```

- You can become runagent for testing scripts and initiate all environment variables:

  ```bash
  runagent -m patchmon1
  ```

  The path becomes:

  ```bash
  echo $PATH
  /home/patchmon1/.config/bin:/usr/local/agent/pyenv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/
  ```

- If you want to debug a container or see environment inside:

  ```bash
  runagent -m patchmon1
  podman ps
  ```

  Example output:

  ```
  CONTAINER ID  IMAGE                                      COMMAND               CREATED        STATUS        PORTS                    NAMES
  d292c6ff28e9  localhost/podman-pause:4.6.1-1702418000                          9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  80b8de25945f-infra
  d8df02bf6f4a  docker.io/library/mariadb:10.11.5          --character-set-s...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  mariadb-app
  9e58e5bd676f  docker.io/library/nginx:stable-alpine3.17  nginx -g daemon o...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  patchmon-app
  ```

- You can see what environment variables are inside the container:

  ```bash
  podman exec patchmon-app env
  ```

  Example output:

  ```
  PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
  TERM=xterm
  PKG_RELEASE=1
  MARIADB_DB_HOST=127.0.0.1
  MARIADB_DB_NAME=patchmon
  MARIADB_IMAGE=docker.io/mariadb:10.11.5
  MARIADB_DB_TYPE=mysql
  container=podman
  NGINX_VERSION=1.24.0
  NJS_VERSION=0.7.12
  MARIADB_DB_USER=patchmon
  MARIADB_DB_PASSWORD=patchmon
  MARIADB_DB_PORT=3306
  HOME=/root
  ```

- You can run a shell inside the container:

  ```bash
  podman exec -ti patchmon-app sh
  / #
  ```

## Testing

Test the module using the `test-module.sh` script:

```bash
./test-module.sh <NODE_ADDR> ghcr.io/dz00te/patchmon:latest
```

The tests are made using [Robot Framework](https://robotframework.org/).

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- add your repository to [hosted.weblate.org](https://hosted.weblate.org) or ask a NethServer developer to add it to ns8 Weblate project
