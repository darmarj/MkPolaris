---
date: 2023-12-22
authors: [darmarj]
description: >
  TLS/SSL cert via Let's Encrypt with Caddy reverse proxy
categories:
  - vaultWarden
  - Proxy
  - Proxmox
---

# Autommatic HTTPS Certifications for Services on Internal Home Network
For internal home network **without opening a port on your firewall**, this article explains how to setup automatic HPPTS certifications via Let's Encrypt for services.

## Requirements
Today, encryption of data in transit via HTTPS is considered mandatory and is often easy to implement via the excellent free service Let’s Encrypt. However, it typically requires that the service in question be reachable from the internet for domain verification purposes. Exposing internal services to the internet is often not desirable, however, as this makes the services attackable. Therefore looking for a way to obtain Let’s Encrypt certificates without creating a security risk by opening firewall ports. In a nutshell:

-   Internal services on a home network are to be made accessible via HTTPS without certificate warnings.
-   No access to internal services from the internet.
    *   No open ports. No open ports.
    *   No cloud-based tunneling service

## DNS Challenge + Caddy to the Rescue
Luckily, there is an alternative to the [Let’s Encrypt HTTP challenge: DNS](https://letsencrypt.org/docs/challenge-types/). There’s also an excellent free reverse proxy that happily handles everything related to HTTPS, including the Let’s Encrypt DNS challenge: [Caddy]. This is how the setup works:

-   Use a subdomain of a public DNS domain for the hostnames of our services.
    *   For security reasons, the subdomain should be used exclusively for the purposes described in this article, not for email (see below).
-   HTTPS certificates are handled by a Caddy instance that acts as a reverse proxy.
-   Caddy is configured to auto-manage Let’s Encrypt certificates via the DNS challenge, which uses TXT records for verification.

[Caddy]: ../posts/Caddy.md

??? pied-piper "Let’s Encrypt DNS Challenge Explained"
    Here’s what happens when a certificate is requested via the Let’s Encrypt DNS challenge:
    -   The Let’s Encrypt client creates a special _acme-challenge DNS TXT record.
    -   Let’s Encrypt queries DNS for that record. If the record’s data is in order, Let’s Encrypt issues the certificate.
    -   The Let’s Encrypt client deletes the _acme-challenge DNS TXT record as it’s not needed any more.

??? pied-piper "Let’s Encrypt DNS Challenge & DNS Zone Security"
    -   API write access to the DNS record _acme-challenge is required for automatic renewal.
    -   **Problem**: most DNS providers don’t have granular access control. API tokens are often valid for the entire zone or even account (Cloudflare). This would allow modification of existing MX records (redirecting your email).
    -   [DNS alias mode](https://github.com/acmesh-official/acme.sh/wiki/DNS-alias-mode) could be a solution (but isn’t).
        *   In DNS alias mode, we’d set up a second DNS zone used exclusively for ACME (Let’s Encrypt) validation. API access is only required for this validation domain.
        *   We’d set up a CNAME record to point from the main domain to the validation domain.
        *   Unfortunately, this doesn’t work with Caddy’s Cloudflare DNS module.
    -   Instead, we use a **DNS domain exclusively** for the purposes described in this article (i.e., ACME challenges and, optionally, name resolution).

## Preparation
<span class="vividgreen">Cloudflare API Token</span>

A free Cloudflare account to manage the DNS domain for the hostnames of the services. Alternatively, you can use any DNS provider that’s supported by Caddy (search the list of [modules](https://caddyserver.com/docs/modules/) for <span class="carrot">dns.providers</span>).

In Cloudflare account, create an API token with the following properties for own Domain:

-   Required permissions:
    *   Zone - Zone -read
    *   Zone - DNS - edit
-   Scope: Include Specific zone
    *   The own public domain is required for the zone resources.

<span class="vividgreen">DNS & Name Resolution</span>

**Host Name Resolution**

Let’s Encrypt certificates are for hostname exclusively, IP addresses are not included (which would technically be possible). It is, therefore, not possible to access the services via HTTPS by IP address, e.g. `https://192.168.0.4`. Instead, to access the services by fully-qualified domain name (FQDN) and need a way to resolve those names into IP addresses. On this example, we need to add one A record for the whoami service that testing with the following:

```bash title="💁‍♂️    DNS Resolve"
#Replace with the IP Address of the host that Docker running
subdomain.domain.com     IP Address

e.g.
whoami.abc.com    192.168.0.4
```

## Caddy Container on Proxmos VE
Run through the preparatory steps to set up <ins>[Docker on the Proxmox host](../posts/Proxmox-Docker.md)</ins>

<span class="vividgreen">Preparation: Increase UDP Buffer Sizes for QUIC</span>

The QUIC protocol (implemented by Caddy) requires larger buffers than normally available in Linux (source). Add the following to <span class="cyangray">/etc/sysctl.conf</span>:
```yaml
net.core.rmem_max=2500000
net.core.wmem_max=2500000
```

Reboot and check the values with the following commands:
```yaml
sysctl net.core.rmem_max
sysctl net.core.wmem_max
```

<span class="vividgreen">Dockerized Caddy Directory Structure</span>

This is what the directory structure will look like when it's done:
```bash
/rpool/
 └── data/
     └── docker/
         └── caddy/
             ├── config/
             ├── data/
             ├── dockerfile-dns/
                 └── Dockerfile
             ├── container-vars.env
             ├── Caddyfile
             └── docker-compose.yml
```
!!! warning "Docker convention"
    Create the `caddy` directory. The subdirectories `config` and `data` are created by docker compose on the first run.

<span class="vividgreen">Customized Caddy Docker Image</span>

We need a custom Caddy Docker image that includes the Cloudflare DNS plugin, which is required for the Let’s Encrypt DNS challenge.
Create the file <span class="cyangray">dockerfile-dns/Dockerfile</span> with the following content:
```docker title="💁‍♂️ Dockfile" hl_lines="8"
ARG VERSION=2

FROM caddy:${VERSION}-builder AS builder

RUN xcaddy build \
    --with github.com/caddy-dns/cloudflare

FROM caddy:${VERSION}

COPY --from=builder /usr/bin/caddy /usr/bin/caddy
```
!!! abstract "Comments"
    The second <span class="vividskype">**FROM**</span> instruction – means produce a much smaller image by simply overlaying the newly-built binary on top of the [regular caddy image](https://hub.docker.com/_/caddy)

<span class="vividgreen">Caddy container-vars.env File</span>

Everything that is specific to the deployment goes into the <span class="cyangray">container-vars.env</span> file. This includes domain names, IP addresses, API keys, email addresses, and so on.

Create with the following content:
```yaml
MY_DOMAIN=home.yourdomain.com               # replace with your domain
MY_HOST_IP=192.168.0.4                      # replace with your Docker host's IP address
CLOUDFLARE_API_TOKEN=<YOUR_TOKEN>           # add your token
```

<span class="vividgreen">Caddy Docker Compose File</span>

Create <span class="cyangray">docker-compose.yml</span> with the following content:
```yaml hl_lines="18 23"
version: "3.9"

services:

  caddy:
    build: ./dockerfile-dns
    container_name: caddy
    hostname: caddy
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    networks:
      - caddynet
    env_file:
      - container-vars.env
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile:ro
      - ./data:/data
      - ./config:/config

  whoami:
    image: "containous/whoami"
    container_name: "whoami"
    hostname: "whoami"
    networks:
      - caddynet

networks:

  caddynet:
    attachable: true
    driver: bridge
```

!!! tip
    The `file` and `directories` in the section <span class="vividskype">**volumes**</span> are <span class="red">bind-mounted</span> from the Docker host. Their content persists when the container is destroyed. `Caddyfile` is mounted read-only (`:ro`).

    The <span class="vividskype">**whoami**</span> service is created strictly for testing purposes. You can remove it once all are working as expected.

<span class="vividgreen">Caddyfile for the whoami Container</span>

Create Caddy’s configuration file <span class="cyangray">Caddyfile</span> in the same directory as the <span class="cyangray">docker-compose.yml</span> file and paste the following content:
```yaml
whoami.{$MY_DOMAIN} {
	reverse_proxy whoami:80
	tls {
		dns cloudflare {env.CLOUDFLARE_API_TOKEN}
	}
}
```

<span class="vividgreen">Start the Containers</span>

Navigate into the directory with the <span class="cyangray">docker-compose.yml</span> file and run:
```docker
docker compose up -d
```

!!! abstract "Docker logs"
    Inspect the container <span class="vividskype">**logs**</span> for errors with the command.
    ```bash
    docker compose logs --tail 30 --timestamps
    ```

<span class="vividgreen">Test</span>

Open `https://whoami.abc.com` in your browser. It should display without certificate warnings or errors.

## Proxmox Let’s Encrypt Certificate
Proxmox is accessible via HTTPS exclusively but comes, understandably, only with a self-signed certificate. Proxmox’s built-in support for Let’s Encrypt does not include the DNS challenge, but now have everything in place to use Caddy container to proxy access to the host’s web interface, too.

<span class="vividgreen">Proxmox Caddyfile</span>

Add the following to `Caddyfile`:
```yaml
proxmox.{$MY_DOMAIN} {
	reverse_proxy {$MY_HOST_IP}:8006 {
      transport http {
         tls_insecure_skip_verify
      }
   }
	tls {
		dns cloudflare {env.CLOUDFLARE_API_TOKEN}
	}
}
```
<span class="vividgreen">DNS A Record</span>

Add the following A record to DNS domain provider:
```bash title="💁‍♀️ DNS Resolve"
#Replace with the IP address of the host that Docker runnning
e.g
proxmox.abc.com 192.168.0.4
```

<span class="vividgreen">Reload Caddy’s Configuration</span>

Instruct Caddy to reload its configuration by running:
```docker
docker exec -w /etc/caddy caddy caddy reload
```

Now be able to access the Proxmox web interface at `https://proxmox.abc.com` without getting a certificate warning from your browser.

## Adjust Proxmox Proxy Settings
Right now, it should actually be possible to reach the Proxmox web interface with HTTPS with valid SSL keys via port 443. Unfortunately the original Port 8006 is also still open. So let's going to fix that now.

Edit the <span class="cyangray">/etc/default/pveproxy</span> configuration file with the following contents:
```yaml title="💁‍♂️ Pveproxy Config" hl_lines="1"
ALLOW_FROM="172.19.0.0/16"
DENY_FROM="all"
POLICY="allow"
```

!!! warning
    This configuration file tells the native Proxmox webserver to only accept requests from inside the same machine <span class="vividskype">**(172.19.0.0/16 -> localhost**</span>), so only requests coming from Caddy, running as a neighbor so to speak, are actually processed.

Finally make sure to restart pveproxy service:
```bash title="💁‍♀️ Pveproxy service"
service pveproxy restart
```

Congratulations! Now the Proxmox web interface should run on HTTPS Port 443 exclusively.
