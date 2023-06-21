# Global Reverse Proxy Stack

Local reverse proxy instance for use with docker and other dev stacks. Binds to Port 80 on your host, must be free in order to work!

## How to use

Bring up the Traefik Reverse Proxy:

```bash
  cd /path/to/local/folder
  docker compose up -d
```

Bring down and stop the Reverse Proxy:

```bash
  docker compose down
```
