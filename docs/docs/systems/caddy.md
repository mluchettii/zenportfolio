---
tags:
    - Caddy
    - Cloudflare
    - Containers
    - Docker
    - Reverse Proxy
---

# Caddy

## Description

[**Caddy**](https://caddyserver.com/) is a web server/reverse proxy/load balancer that serves sites over HTTPS by automatically obtaining and renewing TLS certificates. Written in Go, Caddy provides greater memory safety guarantees than C-based web servers. Caddy also has a plugin model that extends its functionality.

## Caddy installation and configuration

### Preface

For this demonstration, I am deploying [**Caddy Cloudflare**](https://github.com/CaddyBuilds/caddy-cloudflare), which is a custom version of the Caddy Docker image with built-in Cloudflare DNS-01 ACME validation. This is necessary for automatically obtaining SSL certificates for Cloudflare registered domains. Alternatively, for a bare metal install, one can use [**xcaddy**](https://github.com/caddyserver/xcaddy) to compile a custom Caddy binary with the Cloudflare DNS plugin.

### Caddy-cloudflare Docker compose

Create a working directory and a **Docker compose file** for Caddy-cloudflare:

```bash
mkdir caddy-cloudflare && cd caddy-cloudflare
sudo nano docker-compose.yml
```
Write the following configuration to the file:

```yml title="docker-compose.yml" 
services:
  caddy:
    image: ghcr.io/caddybuilds/caddy-cloudflare:latest
    container_name: caddy-cloudflare
    restart: unless-stopped
    cap_add:
      - NET_ADMIN
    ports:
      - "80:80"
      - "443:443"
      - "443:443/udp"
    volumes:
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./site:/srv
      - ./caddy_data:/data
      - ./caddy_config:/config
    environment:
      - CLOUDFLARE_API_TOKEN=your_cloudflare_api_token
volumes:
  caddy_data:
    external: true
  caddy_config:
```

Pull the Docker image and deploy the container:

```bash
docker compose up -d
```
### Caddyfile configuration

The **Caddyfile** is a configuration file that contains the information for creating reverse proxies and obtaining SSL certificates. This version of Caddy allows us to use **Cloudflare** as the ACME DNS challenge provider.

Here is an example Caddyfile for serving an Apache site that listens on port 8080:

```
apache.example.com {

    # Set this path to your site's directory.
    root * /usr/share/caddy

    file_server

    reverse_proxy localhost:8080

    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
}
```
Write changes to the file, then restart the Caddy Docker container:

```bash
docker compose down
docker compose up -d
```

!!! success "Result"
    With this configuration, Caddy will be able to serve the Apache site running on ```localhost:8080``` securely to anyone who visits ```apache.example.com```, given that a correct DNS record was configured on Cloudflare.


