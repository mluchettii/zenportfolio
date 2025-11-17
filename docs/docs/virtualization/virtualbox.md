---
tags:
    - VirtualBox
    - Virtualization
---

# Oracle VirtualBox

## Description

[**Oracle VirtualBox**](https://www.virtualbox.org/) is a virtualization solution that allows you to run multiple guest operating systems on a single physical host. You can use this to create isolated environments for testing in my homelab, using Windows and Linux operating systems. With this, it's possible to replicate real-world scenarios for vulnerability assessments, log analysis, and threat detection.

## Network adapter types

VirtualBox's networking modes that I have set up include the NAT Network, host-only network, and bridged network.

### NAT network

The NAT network is the default networking mode in VirtualBox. It allows virtual machines to access external networks like the internet through the host machine without requiring any configuration on the host network or guest OS.

### Bridged network

The bridged network mode connects the virtual machine directly to your physical network. It will have its own IP address and can be accessed by other devices in the network without forwarding ports.

### Host-only network

Lastly, the host-only network mode is used creates a private network exclusively between the host machine and the virtual machines connected to it, without any connection to external networks or the internet.

## How to forward a VM's ports within the NAT network

1. In VirtualBox, open the Network preferences menu and select the **NAT Networks** tab.
2. Select the corresponding network and click the **Port Forwarding** tab in the lower pane.
3. To forward HTTPS and SSH ports, create entries that look like the following, where **Protocol**, **Host IP**, and **Host Port** are unchanged:  

    | Name | Protocol | Host IP | Host Port | Guest IP | Guest Port |
    | :---------: | :----------: | :----------: | :----------: | :----------: | :----------: |
    |HTTPS|TCP|127.0.0.1|443|192.168.14.0|10443|
    |SSH|TCP|127.0.0.1|22|192.168.14.0|10022|

<figure markdown>
![alt text](images/vbox-portforward.png#center){.shadowed-image style="width: 100%;"}
<figcaption markdown class="annotate">Example</figcaption>
</figure>