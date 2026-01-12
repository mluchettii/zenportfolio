---
tags:
    - Cloud
    - DigitalOcean
    - Homelab
    - Network
    - VPS
---

# DigitalOcean

DigitalOcean is a cloud computing platform offering IaaS and PaaS solutions for developers. Their most popular IaaS product, Droplets, provides scalable virtual machines that are ideal for web hosting and VPS deployments.

<figure markdown>
![alt text](../../screenshots/digitalocean-01.png#center){.shadowed-image style="width: 100%;"}
<figcaption markdown class="annotate">DigitalOcean Droplet dashboard</figcaption>
</figure>

## Choosing a plan

I opted for the Basic Premium Intel plan that includes 1 vCPU, 1 GB memory, 35 GB SSD, and 1 TB storage. While being economical, this VPS has enough resources to host a Pangolin stack, but not more than that due to memory constraints.

## Configuration

### VPS setup

I chose Debian as the host OS and then proceeded to configure the VPS:

**SSH hardening**

```sh title="Non-root user creation"
sudo useradd mluchet
sudo passwd mluchet            # Set mluchet's password

su mluchet                     # Log in to non-root user account
```

```sh title="Create SSH keypair for passwordless access"
ssh-keygen                     # Generate a 2048-bit RSA public/private key pair
                               #    in the ~/.ssh directory
```

Then, I manually copied my workstation's public key to the VPS for passwordless SSH authentication:

```sh title="SSH public key transfer"
# (1) On the local machine:

cat .ssh/id_ed25519.pub                     # Show the local machine's public key

ssh-ed25519 <secret> example@email.com      # Copy this public key to the clipboard



# (2) On the VPS, logged in as the non-root user:

mkdir -p ~/.ssh         # Create the ~/.ssh directory
                        #   if it doesn't already exist

echo "<clipboard-contents>" >> ~/.ssh/authorized_keys

# (3) Append the local machine's public key
#       to the droplet's list of authorized SSH keys
```

```sh title="SSH hardening (1/2)"
chmod -R go= ~/.ssh            # Prevent other users on the system from reading
                               #   private keys

chown -R $USER:$USER ~/.ssh    # Set the user as the owner of the directory and
                               #   its contents
```

```sh title="SSH hardening (2/2)"
PasswordAuthentication no      # Disable password authentication

UsePAM no                      # Prevent PAM from bypassing the above control

PermitRootLogin no             # Disable root login
```

```sh title="Restart SSH"
sudo systemctl restart ssh     # Restart SSH service for changes to take effect
```

**Docker**

The next step was to install the Docker Engine using the official instructions for a [Debian install](https://docs.docker.com/engine/install/debian/). After installation, I added my user to the `docker` group:

```sh
sudo usermod -aG docker $USER
```

### Pangolin setup

Pangolin combines the reverse proxy capabilities of Traefik and WireGuard tunneling for secure access to private services without opening firewall ports. To get started, I ran the official installation script and followed the [documentation](https://docs.pangolin.net/self-host/quick-install), configuring Pangolin to use my homelab's FQDN, and enabling CrowdSec for banning malicious IPs.

**Sites**

After deploying the Pangolin compose stack, I created a **Newt Site** for my Pi server. Newt is Pangolin's custom user-space WireGuard agent that is deployed on clients that receive proxied traffic from Pangolin. I created a Newt Site on my Pi server by deploying the Docker Compose stack that was generated for my site:

<figure markdown>
![alt text](../../screenshots/pangolin-01.png#center){.shadowed-image style="width: 75%;"}
<figcaption markdown class="annotate">Creating a Newt Site</figcaption>
</figure>

<figure markdown>
![alt text](../../screenshots/pangolin-02.png#center){.shadowed-image style="width: 75%;"}
<figcaption markdown class="annotate">Site is online: VPS and Pi are communicating over the Newt tunnel</figcaption>
</figure>

**Resources**

With the Site deployed, I then created my first public **Resource**. In Pangolin, you can create HTTPS or raw TCP/UDP Resources. I set up Authentik as my first Public HTTPS Resource:

<figure markdown>
![alt text](../../screenshots/pangolin-03.png#center){.shadowed-image style="width: 75%;"}
<figcaption markdown class="annotate">Configuring Public Resource for Authentik</figcaption>
</figure>

With this configuration, the HTTPS request for `authentik.mydomain.com` is forwarded over the Newt tunnel to my Pi server that has Nginx Proxy Manager (NPM) listening on port 443. Since the Custom Host Header was set to `authentik.mydomain.com`, NPM reads this and routes the traffic to the Authentik host's internal port.

**Authentication and ACL Rules**

Pangolin also supports platform SSO as an authentication mechanism, as well as ACL rule configuration. For my Authentik resource, I made sure to enable platform SSO for Members, and configured the rules such that US-based requests are passed to the authentication portal and all other countries are blocked.

<figure markdown>
![alt text](../../screenshots/pangolin-04.png#center){.shadowed-image style="width: 75%;"}
<figcaption markdown class="annotate">Pangolin Resource rules</figcaption>
</figure>

For anyone connecting from the US (or a US VPN), this is what they will see:

<figure markdown>
![alt text](../../screenshots/pangolin-05.png#center){.shadowed-image style="width: 75%;"}
<figcaption markdown class="annotate">Pangolin Authentication portal</figcaption>
</figure>