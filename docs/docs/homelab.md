---
tags:
    - Cloudflare
    - Containers
    - DigitalOcean
    - Docker
    - Firewall
    - Homelab
    - Linux
    - Networking
    - Nginx Proxy Manager
    - Pangolin
    - Reverse Proxy
    - Routers
    - Tailscale
    - VPS
    - VPN
---

# Homelab

## On-prem setup

I'll first go over the hardware that makes this possible. As my sole homelab server, I'm using a Raspberry Pi 5 with 16 GB of memory, a dedicated NVME drive for Pi OS, and an external HDD for file storage. It's connected to a MikroTik hEX S (2025) router that is running RouterOS. Also connected to the MikroTik is a NETGEAR Orbi RBR50 wireless router (set to AP mode) which provides wireless backhaul to an RBS50 mesh satellite on the first floor. Lastly, I have my cable modem connected to the WAN port on the MikroTik. Below is a simple topology diagram of my home network:

``` mermaid
graph TB
  %% ISP → Modem → Router (ether1)
  ISP((Internet))
  MODEM[Modem]

  %% MikroTik Router - Specific Ports
  MT[MikroTik Router]

  %% LAN Wired Devices (Specific Ethernet ports)
  RBR[Orbi RBR50<br/>AP1]
  PI[Raspberry Pi<br/>Server] 
  WS[Workstation]

  %% Orbi Wireless Mesh
  RBS[Orbi RBS50<br/>AP2]

  %% Wireless Clients (roaming)
  WIFI[Wireless Devices]

  %% Specific Port Connections
  ISP -->|cable| MODEM
  MODEM -->|ether1| MT

  MT -->|ether2| RBR
  MT -->|ether3| PI

  MT -->|ether4| WS

  RBR -.->|Wireless Backhaul| RBS

  %% Roaming Wireless
  RBR -.->|WiFi Roaming| WIFI
  RBS -.->|WiFi Roaming| WIFI

  %% Your exact styling scheme
  style PI stroke:#ff6666,stroke-width:2px,fill:none
  style MT stroke:#ff6666,stroke-width:2px,fill:none
  style RBR stroke:#ff6666,stroke-width:2px,fill:none
  style WS stroke:#ff6666,stroke-width:2px,fill:none
  style RBS stroke:#ff6666,stroke-width:2px,fill:none

  style ISP stroke:#3399ff,stroke-width:2px,fill:none
  style MODEM stroke:#3399ff,stroke-width:2px,fill:none
  style WIFI stroke:#ff6666,stroke-width:2px,fill:none
```

<div style="display: flex; justify-content: center; width: 100%;">
<figcaption markdown class="annotate">LAN topology</figcaption>
</div>

### Configuration

[**MikroTik Router**](router.md)

The backbone of the network. MikroTik routers run their proprietary RouterOS and are configurable via web interface, WinBox, or console. Here, I set up VLANs, firewall rules, DHCP scopes, and DNS, optimizing traffic segmentation and security for a multi-device home lab.

[**Raspberry Pi**](pi.md)

Another component of my on-prem setup is the Raspberry Pi, which is responsible for self-hosting the services I want to use and experiment with. The services are deployed in Docker containers, streamlining deployment and maintenance.

## Cloud setup

My homelab architecture is designed to secure access to some of my self-hosted services from the Internet, and have the rest be only accessible via my personal Tailscale network. In addition, I want to have my homelab server logically segmented from all the other devices on my home network, preventing lateral movement from potential attackers. The flowchart below illustrates this:

``` mermaid
graph TD
  %% External
  CLIENT[Client]
  NET((Internet))
  CF[Cloudflare DNS]

  %% Edge access
  VPS[DigitalOcean VPS<br/>Pangolin Proxy]
  TS[Tailscale Network<br/>Relay Servers]

  %% Core
  MT[MikroTik Router]
  LAN[LAN Devices]
  PI[Raspberry Pi<br/>Server]
  NPM[Nginx Proxy Manager]
  PUBLIC[Exposed Services<br/>*.mydomain.com]
  PRIVATE[Zero-Trust Services<br/>*.ts.mydomain.com]

  %% Flow
  CLIENT --> NET --> CF

  CF -->|*.mydomain.com| VPS
  CF -.->|*.ts.mydomain.com| TS

  VPS -.->|WireGuard Tunnel| MT
  TS -.->|WireGuard Mesh| MT

  MT -->|VLAN 0| LAN
  MT -->|VLAN 20| PI

  PI -->|HTTPS|NPM
  NPM --> PUBLIC
  NPM --> PRIVATE

  %% Styling
  style PI stroke:#ff6666,stroke-width:2px,fill:none
  style LAN stroke:#ff6666,stroke-width:2px,fill:none
  style NPM stroke:#ff6666,stroke-width:2px,fill:none
  style MT stroke:#ff6666,stroke-width:2px,fill:none

  style VPS stroke:#0063ff,stroke-width:2px,fill:none
  style PUBLIC stroke:#0063ff,stroke-width:2px,fill:none
  
  style PRIVATE stroke:#a0a0a0,stroke-width:2px,fill:none
  style TS stroke:#a0a0a0,stroke-width:2px,fill:none

  style NET stroke:#3399ff,stroke-width:2px,fill:none
  style CF stroke:#f38020,stroke-width:2px,fill:none
  style CLIENT stroke:#28a745,stroke-width:2px,fill:none
```

<div style="display: flex; justify-content: center; width: 100%;">
<figcaption markdown class="annotate">Hybrid access via cloud proxy and Tailscale mesh</figcaption>
</div>

### Configuration

Setting this up and maintaining it requires a solid understanding of cloud, network, and security concepts, in addition to skill with Linux and containers (and most importantly patience).

**Tailscale**

This served as a great starting point for personal remote access and zero-trust resource-sharing. By itself, Tailscale has all you need to get started with self-hosting. Built-in features include but are not limited to DNS, free HTTPS certificates, ACLs, and portforwarding.

**Cloudflare**

I eventually learned about reverse proxies and how my setup could benefit from using them, convincing me to acquire a domain. I now use Cloudflare to create and manage my DNS records, with one FQDN for homelab use and the other for my portfolio.

**DigitalOcean**

I use a basic 1vCPU, 1 GB VPS from DigitalOcean to serve as a dedicated external reverse proxy. It runs a Pangolin container stack capable of proxying requests to my Pi resources over a direct Newt (WireGuard) tunnel. Compared to using a Tailscale mesh VPN, this direct tunnel significantly reduces latency, especially over longer distances. In terms of convenience, users are not required to install and set up a VPN client to access the server's shared resources.