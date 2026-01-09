---
tags:
    - DNS
    - Docker
    - Homelab
    - Raspberry Pi
    - Security
    - SSH
    - Systems
---

# Raspberry Pi

The Raspberry Pi serves as a great entry-level homelab server. It's a proven, quality SBC that fits a multitude of use cases. For my server, I wanted to use something that has a large community knowledge base, making it easier to seek guidance and troubleshoot problems. Lastly, since the server is meant to be always on, I wanted the server to be energy efficient without sacrificing performance. So far, the Pi has met all the demands that I ask of it with some resources to spare.

## Initial setup

### SSH server

On a fresh install of Pi OS (which is just Debian), I added a non-root user account, and used the `ssh-keygen` command to generate an SSH key pair for secure remote access. This creates a public and private key that I can use to authenticate from a separate host. This host will have to use the following command to obtain the public key from the server. Assume `ras` is the username, `pi` is the hostname, and its IP address is `10.0.20.100`:

```
ssh-copy-id ras@10.0.20.100

The authenticity of host '10.0.20.100 (10.0.20.100)' can't be established.
ECDSA key fingerprint is <...>.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
ras@10.0.20.100's password:
```

After password authentication, the key will be added to the host:

```
Number of key(s) added: 1

Now try logging in to the machine, with:   "ssh 'ras@10.0.20.100'"
and check to make sure that only the key(s) you wanted were added.
```

This allows me to log in without using a password. To further harden the system, I made the following changes to the SSH server configuration file:

``` title="/etc/ssh/sshd_config"
PasswordAuthentication no      # Disables password authentication
UsePAM no                      # Prevents PAM from bypassing the above control
```

```sh title="Restart SSH"
sudo systemctl restart ssh     # Restart SSH service for changes to take effect
```

### Docker

The next step was to install the Docker Engine using the official instructions for a [Debian install](https://docs.docker.com/engine/install/debian/). After installation, I added the user `ras` to the `docker` group to bypass the `sudo` requirement to interact with Docker:

```sh
sudo usermod -aG docker $USER
```

The server currently runs 40+ containers across 10 stacks. A stack can be composed of one container or many. Containers that interact with each other are often stacked together. I have a stack for my server dashboard, another for my media server, and another for event notifications. The easiest way to stack containers is by writing Docker Compose YAML files.

**Docker Compose**

In a nutshell, a Docker compose file tells the engine what container images to pull, what environment variables to apply, the ports to be forwarded, where to map storage volumes, etc. In the following example, my idea was to create a Vaultwarden server for password management that is only accessible via the accompanying "sidecar" container's Tailscale IP address. And, for the Vaultwarden server to be operational, it depends on the TS sidecar satisfying the healthcheck (status: "healthy"). Here is the Docker compose file that I made for this use case:

```yml title="docker-compose.yml"
services:
  vaultwarden:
    image: vaultwarden/server:latest
    network_mode: service:vaultwarden-ts
    container_name: vaultwarden
    environment:
      DOMAIN: "https://vault.ts.${FQDN}"
    volumes:
      - ./vw-data/:/data/
    ports:
      - ${TS_IP}:8020:80
    depends_on:
      vaultwarden-ts:
        condition: service_healthy
    healthcheck:
      test: ["CMD", "pgrep", "-f", "vaultwarden-ts"]
      interval: 1m
      timeout: 10s
      retries: 3
      start_period: 30s
    restart: unless-stopped

  vaultwarden-ts:
    image: tailscale/tailscale:latest
    container_name: vaultwarden-ts
    hostname: vaultwarden
    environment:
      - TS_AUTHKEY=${TS_AUTHKEY}
      - TS_STATE_DIR=/var/lib/tailscale
      - TS_USERSPACE=false
      - TS_ENABLE_HEALTH_CHECK=true
      - TS_LOCAL_ADDR_PORT=127.0.0.1:41234
      - TS_ACCEPT_DNS=true
    volumes:
      - ./ts/config:/config
      - ./ts/state:/var/lib/tailscale
    devices:
      - /dev/net/tun:/dev/net/tun
    cap_add:
      - net_admin
    healthcheck:
      test: ["CMD", "wget", "--spider", "-q", "http://127.0.0.1:41234/healthz"]
      interval: 1m
      timeout: 10s
      retries: 3
      start_period: 10s
    restart: always
```

Once ready, I can bring up the stack by running the following command:

```sh
docker compose up -d    # This deploys the stack in "detached" mode, meaning it is running in the background
```

Once the containers are up, I can monitor and control them using the following commands:

```sh title="View container logs"
docker logs <container>
```

```sh title="Access container's interactive terminal"
docker exec -it <container> sh
```

Or safely bring the whole stack down with one command:

```sh
docker compose down
```

### Self-hosting

With Docker ready, I then started configuring and deploying self-hosted services in containers. This includes web applications, databases, LLMs, media streaming, etc. These are the most impactful ones for me:

**AdGuard Home**

I use AdGuard Home as my network's default DNS server. It filters out unwanted traffic, whether it be ads, trackers, malware, blacklisted websites, etc. It also supports setting DNS rewrites to form your own custom, local domain records.

**Authentik**

Authentik is an OIDC/OAuth2 provider that I use to log in to my applications that support it. It supports SSO, MFA, user and group management, customizable flows and stages, and more. It adds an extra layer of security to applications, allowing me to disable password sign-in to them entirely.

**ntfy**

ntfy is a push notification server that can be used in various ways. I set it up to automatically send Authentik notifications containing context like login fail/success with username and location information. When also paired with LoggiFly, I can receive notifications based on Docker log events that match a specific regular expression. For example, on my file server, when a user downloads a file, I will immediately receive a formatted, readable notification telling me what user downloaded what file and when.

**Nginx Proxy Manager**

I use NPM as a reverse proxy server. It routes external traffic to internal services and handles SSL certificates automatically via Let's Encrypt and Cloudflare DNS.

**OpenCloud**

OpenCloud is a minimal, lightweight fork of NextCloud. I use it to sync my documents across all my devices.

**SearXNG**

This is classified as a private metasearch engine that aggregates results from multiple search engines. I use it to replace the likes of Google and DuckDuckGo.

**Vaultwarden**

This one is impactful because I had a bad habit of using the same complex password for every account. With this, I can generate complex passwords for my accounts and save them to the vault. The accompanying app and browser extension also support autofill, which has saved me a lot of time and headache remembering and typing long passwords.