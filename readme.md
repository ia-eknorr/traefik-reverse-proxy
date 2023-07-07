# Global Reverse Proxy Stack

Local reverse proxy instance for use with docker and other dev stacks. Functionally, using Traefik allows you to assign "common sense" labels to your other docker services (e.g. Ignition development containers) instead of choosing an arbitrary port number. Out with `http://localhost:9088` and in with `http://my-dev-gw.localtest.me`. Traefik binds to Port 80 on your host - it must be free in order to work!

## How to use

1. Clone this repository into your project directory

    ```bash
    cd /path/to/project
    git clone https://inductive-git.ia.local/dev-saleseng/demo-project-dev.git .
    ```

2. Spin up the Traefik Reverse Proxy:

    ```bash
    cd /path/to/project
    docker-compose -f docker-compose.yml -p proxy up -d
    ```

> :memo: **_Note_**: It is recommended that `path/to/project` mentioned below is separate from active projects, in `projects/infrastructure/traefik-proxy`.

## How to stop use

1. Spin down and stop the Reverse Proxy:

    ```bash
    cd /path/to/project
    docker-compose down
    ```
