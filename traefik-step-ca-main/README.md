# Traefik + step-ca HTTPS Reverse Proxy

This solution provides a reverse proxy via [Traefik](https://docs.traefik.io) and local Certificate Authority via [step-ca](https://smallstep.com/docs/step-ca).

Features include:

- step-ca preconfigured with ACME provisioner for automatic certificate generation
- Traefik preconfigured to target step-ca as HTTPS certificate resolver

## Setup

Prerequisites:

- Install [step-cli](https://smallstep.com/docs/step-cli/installation/index.html) on your computer.

Configuration:

1. First, create two Docker networks, one for our Reverse Proxy, and another for internal use (no egress):

```bash
docker network create proxy
docker network create --internal internal
```

2. Next, bring up the Compose stack:

```bash
docker compose up -d
```

3. View the container logs of the `stepca` service:

```bash
docker compose logs stepca
```

Note the following from the output (the username/password will not be shown again after this initial launch):
- CA administrative username
- CA administrative password
- X.509 Root Fingerprint

4. Setup your `step` CLI (substituting the fingerprint gathered above accordingly):

```bash
step ca bootstrap --ca-url https://stepca.localtest.me:9000 --fingerprint xxxxxxxxx
```

5. [Optional] If desired, extend the max TLS certificate duration (for manually generated certs):

```bash
# First, open the editor (add the snippet in the next section to the `authority` object)
docker compose exec stepca vi config/ca.json
# Once edits are complete, restart stepca
docker compose restart stepca
```

This is the snippet (specifically, the `claims` section, but don't forget to add a comma at the end of the `enableAdmin` line) that needs edited to extend the default and max certificate durations (past the 24hr default max):

```json
	"authority": {
		"enableAdmin": true,
		"claims": {
			"defaultTLSCertDuration": "720h0m0s",
			"maxTLSCertDuration": "2160h0m0s"
		}
	}
```

6. Finally, locate the `~/.step/certs/root_ca.crt` Root CA Certificate and place in your OS keystore.  This will establish the root trust for certificates generated via Traefik by step-ca.

7. Verify that you can reach the Traefik Dashboard via https at:

- https://proxy.localtest.me

## Creating Docker Containers with HTTPS

You can apply the following labels to enable HTTPS:

- `traefik.enable=true`
- `traefik.hostname=<hostname>`

Setting the above will make the following container:

```bash
docker run -d -l traefik.enable=true -l traefik.hostname=ia-ignition \
    --network proxy --no-healthcheck \
    inductiveautomation/ignition:8.1.37 \
    -n ia-ignition \
    -a ia-ignition.localtest.me \
    -h 80 \
    -s 443 -- gateway.useProxyForwardedHeader=true
```

... resolve via https://ia-ignition.localtest.me

## Generating a custom key+cert

You can also use the `step` CLI (in concert with your provisioned step-ca) to generate standalone certificates using the "admin" provisioner:

```bash
step ca certificate \*.localtest.me \
    localtest.me.crt localtest.me.key \
    --san=\*.localtest.me --not-after 2160h \
    --provisioner=admin
```

The above will create a certificate and key file that you can use to load into an Ignition Gateway (not running behind Traefik, for example).  Keep in mind that you may also need the root and/or intermediate certificate from step-ca.
