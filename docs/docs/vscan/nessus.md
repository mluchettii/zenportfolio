---
tags:
    - Tenable Nessus
    - Vulnerability Scanners
    - Security
    - Docker
    - Containerization
    - Linux
---

# Tenable Nessus

## Description

[**Tenable Nessus**](https://www.tenable.com/products/nessus) is a leading vulnerability scanner developed by Tenable, widely regarded as the gold standard for identifying security weaknesses across networks, systems, applications, and cloud environments. Nessus performs comprehensive scans to detect software flaws, missing patches, misconfigurations, default credentials, and insecure protocols that could be exploited by attackers.

## Docker setup

Before starting the setup, make sure Docker is installed on your machine, as it will be used to deploy and run the Nessus container.

1. Create a working directory for Nessus:
    ```sh
    sudo mkdir nessus && cd nessus
    ```

2. Make the following Docker Compose YAML file:
    ```yml title="docker-compose.yml"
    services:
        nessus:
            image: 'tenable/nessus:latest-oracle'
            volumes:
                - './nessus_data:/data'
                - '/var/run/docker.sock:/var/run/docker.sock'
            network_mode: "host"
            restart: always
            container_name: nessus
    ```
3. Deploy the container:

    ```sh
    sudo docker compose up -d
    ```

## Application setup

1. Access the HTTPS web console via the host machine's LAN IP address: `https://192.168.1.x:8834`.

2. Wait for Nessus to initialize. Then, select which Nessus product you want to install:

    === ":lucide-infinity: Essentials"

        * Free vulnerability scanner for up to 16 IP addresses per scanner.
        * Same core scanning engine as paid versions.
        * Ideal for students, personal labs, and those evaluating vulnerability management solutions.
        * Community support and basic documentation.

    === ":lucide-dollar-sign: Professional"

        * Try for free
        * Unlimited asset scanning, advanced reporting, and professional support.
        * Suitable for security teams, consultants, and businesses needing regular assessments across many devices.
        * Includes features like extensive policy templates and integration support.

    === ":lucide-dollar-sign: Expert"

        * Try for free
        * Adds external attack surface scanning (e.g., domains, cloud infrastructure), domain monitoring, and advanced cloud configuration assessment.
        * Designed for enterprises with advanced security and compliance needs; offers clustering and large-scale automation capabilities.

3. Create a username and password that you will use to log in to Nessus.

4. Wait for Nessus to download and initialize the remaining plugins.

## Basic vulnerability scan setup

Log in to Nessus. A new **Welcome to Nessus** page will appear, prompting you to launch a host discovery scan to identify what hosts on your network are available to scan. 

1. In **Targets**, type in the gateway address of your local network followed by `/24`. Click **Submit**.

    Example: `192.168.1.1/24`

    This will scan all the devices on your network.

    !!! info "Gateway address"

        To find the gateway address, enter this command in your Linux terminal:

        ```sh
        ip route | grep default
        ```

        It will be the first address (after `default via`).

2. Wait for the discovery scan to complete. Then, select the hosts you want to scan for vulnerabilities and click **Run Scan**.

    !!! info "Nessus Essentials host limit"

        The selected hosts count towards the 16 host limit on the Nessus Essentials license.

3. Now wait for the **Basic Network Scan** to complete. Once finished, click on the host with the most vulnerabilities and you will see a list of vulnerabilities ranked from **Low** to **Critical** severity. Some tests are marked as **Info** for 'informational'.

These are my scan results:

??? example "Basic scan results"
    <figure markdown>
    ![alt text](images/nessus-scan-01.png#center){.shadowed-image style="width: 90%;"}
    </figure>

## Vulnerability analysis (Blog)

This is more of a blog entry/demo than proper documentation. I will keep it here until the Zensical team adds the blogging feature.

### Vulnerability A: Outdated Apache version

??? example "Vulnerability A details"
    
    **Vulnerability**: Apache 2.4.x < 2.4.64 Multiple Vulnerabilities
    
    **Severity**: High  

    **Description**  

    The version of Apache httpd installed on the remote host is prior to 2.4.64. It is, therefore, affected by multiple vulnerabilities as referenced in the 2.4.64 advisory.

    - In certain proxy configurations, a denial of service attack against Apache HTTP Server versions 2.4.26 through to 2.4.63 can be triggered by untrusted clients causing an assertion in mod_proxy_http2.
    Configurations affected are a reverse proxy is configured for an HTTP/2 backend, with ProxyPreserveHost set to on. (CVE-2025-49630)

    - HTTP response splitting in the core of Apache HTTP Server allows an attacker who can manipulate the Content-Type response headers of applications hosted or proxied by the server can split the HTTP response.
    This vulnerability was described as CVE-2023-38709 but the patch included in Apache HTTP Server 2.4.59 did not address the issue. Users are recommended to upgrade to version 2.4.64, which fixes this issue.
    (CVE-2024-42516)

    - SSRF in Apache HTTP Server with mod_proxy loaded allows an attacker to send outbound proxy requests to a URL controlled by the attacker. Requires an unlikely configuration where mod_headers is configured to modify the Content-Type request or response header with a value provided in the HTTP request. Users are recommended to upgrade to version 2.4.64 which fixes this issue. (CVE-2024-43204)

    - Server-Side Request Forgery (SSRF) in Apache HTTP Server on Windows allows to potentially leak NTLM hashes to a malicious server via mod_rewrite or apache expressions that pass unvalidated request input. This issue affects Apache HTTP Server: from 2.4.0 through 2.4.63. Note: The Apache HTTP Server Project will be setting a higher bar for accepting vulnerability reports regarding SSRF via UNC paths. The server offers limited protection against administrators directing the server to open UNC paths. Windows servers should limit the hosts they will connect over via SMB based on the nature of NTLM authentication.
    (CVE-2024-43394)

    - Insufficient escaping of user-supplied data in mod_ssl in Apache HTTP Server 2.4.63 and earlier allows an untrusted SSL/TLS client to insert escape characters into log files in some configurations. In a logging configuration where CustomLog is used with %{varname}x or %{varname}c to log variables provided by mod_ssl such as SSL_TLS_SNI, no escaping is performed by either mod_log_config or mod_ssl and unsanitized data provided by the client may appear in log files. (CVE-2024-47252)

    Note that Nessus has not tested for these issues but has instead relied only on the application's self-reported version number.  

    **Solution**  

    Upgrade to Apache version 2.4.64 or later.  

    **Output**  


        URL               : http://192.168.1.14:8749/
        Installed version : 2.4.62
        Fixed version     : 2.4.64

        To see debug logs, please visit individual host

    | **Port**           | **Hosts**        |
    | :--------------| :----------- |
    | 8749 / tcp / www| 192.168.1.14|

From the vulnerability details, it seems like there is an application using an outdated version of Apache. For this vulnerability, the attack vectors include HTTP response splitting and Server-Side Request Forgery (SSRF). To investigate this further, I checked what application uses TCP port `8749` on the host:

```sh
$ sudo ss -tulnp | grep :8749  

tcp   LISTEN 0      4096                       0.0.0.0:8749       0.0.0.0:*    users:(("docker-proxy",pid=5966,fd=7))
tcp   LISTEN 0      4096                          [::]:8749          [::]:*    users:(("docker-proxy",pid=5991,fd=7))
```

The port points to a Docker proxy, which means that there is a container using an outdated version of Apache. I ran the following command to find what container is serving data on port `8749`:

```sh
$ sudo docker ps --format '{{.ID}} {{.Names}} {{.Ports}}' | grep 8749  

4e4ac50ae257 freshrss 0.0.0.0:8749->80/tcp, [::]:8749->80/tcp
```

The FreshRSS container was the culprit. Upon further research, I found out that the installed version (`1.26.0`) was behind the current version (`1.27.1`). I then updated the container and rescanned the host to confirm the remediation.

??? example "Vulnerability A rescan results"
    <figure markdown>
    ![alt text](images/nessus-scan-02.png#center){.shadowed-image style="width: 90%;"}
    </figure>

As expected, the vulnerability was remediated.

My vulnerability report:

``` title="Vulnerability A Report"
Host IP: 192.168.1.14  
Time Detected: Nov. 15, 2025 @ 15:51  
Scan Policy: Basic Network Scan  
Vulnerability: Apache 2.4.x < 2.4.64 Multiple Vulnerabilities  
Description: The version of Apache httpd installed on the remote host is prior to  
2.4.64. It is, therefore, affected by multiple vulnerabilities as referenced in the  
2.4.64 advisory.  

Analysis: Outdated version of Apache detected listening on TCP port 8749 through a  
Docker proxy. Vulnerable Docker container found out to be 'freshrss'. The container  
was updated to latest version, remediating the vulnerability.

Status: CLOSED | TRUE POSITIVE | REMEDIATED
```

### Vulnerability B: Outdated JQuery library

??? example "Vulnerability B details"
    
    **Vulnerability**: JQuery 1.2 < 3.5.0 Multiple XSS
    
    **Severity**: Medium  

    **Description**  

    According to the self-reported version in the script, the version of JQuery hosted on the remote web server is greater than or equal to 1.2 and prior to 3.5.0. It is, therefore, affected by multiple cross site scripting vulnerabilities.  

    Note, the vulnerabilities referenced in this plugin have no security impact on PAN-OS, and/or the scenarios required for successful exploitation do not exist on devices running a PAN-OS release.  

    **Solution**  

    Upgrade to JQuery version 3.5.0 or later.  

    **See Also**

    https://blog.jquery.com/2020/04/10/jquery-3-5-0-released/  

    https://security.paloaltonetworks.com/PAN-SA-2020-0007


    **Output**

        URL               : https://192.168.1.14:8384/vendor/jquery/jquery-2.2.2.js
        Installed version : 2.2.2
        Fixed version     : 3.5.0

        To see debug logs, please visit individual host

    | **Port**           | **Hosts**        |
    | :--------------| :----------- |
    | 8384 / tcp / www| 192.168.1.14|

This time, a JavaScript library called JQuery was detected as outdated and vulnerable to a XSS attack. This library is used by an application on port `8384`.  

I performed the same steps as the previous vulnerability to find the root cause:

```sh
$ sudo ss -tulnp | grep :8384  

tcp   LISTEN 0      4096                       0.0.0.0:8384       0.0.0.0:*    users:(("docker-proxy",pid=7338,fd=7))                                                                     
tcp   LISTEN 0      4096                          [::]:8384          [::]:*    users:(("docker-proxy",pid=7345,fd=7))
```

Again, the problem originated from a Docker container:

```sh
$ sudo docker ps --format '{{.ID}} {{.Names}} {{.Ports}}' | grep 8384  

3e08c688b427 syncthing 0.0.0.0:8384->8384/tcp, 0.0.0.0:21027->21027/udp, [::]:8384->8384/tcp, [::]:21027->21027/udp, 0.0.0.0:22000->22000/tcp, [::]:22000->22000/tcp, 0.0.0.0:22000->22000/udp, [::]:22000->22000/udp
```

Since I found the Syncthing Docker container as the root cause, I manually updated it (referring to the latest version listed on its Docker Hub repository), and then rescanned to confirm the remediation.

??? note "Manual Docker container update (Portainer method)"

    I am running the LinuxServer.io version of Syncthing. The [documentation](https://hub.docker.com/r/linuxserver/syncthing) recommends pulling the latest version: `linuxserver/syncthing:latest`. The easiest way to do this is is through **Portainer**. 

    1. Log in to Portainer and navigate to your containers.

    2. Click on the container you want to update. Then, click on **Duplicate/Edit**.

    3. Under the **Image Configuration** section, change the image's release tag from what is currently installed to a more updated version. If you are not specific on the version that you want, type in `:latest`. If the image is already using the `:latest` tag, then proceed to step 4.

    4. Scroll down and click **Deploy the container**. This will redeploy the container using the version of the Docker image you have set, keeping the configuration intact.

Surprisingly, the same vulnerability was detected again, meaning the JQuery library was not updated. I tried a different approach, this time changing the Docker image repo from the LinuxServer.io one to the official one offered by Syncthing (`syncthing/syncthing:latest`), and deployed it as a brand new container.

The result was the same, so I did further research on the project's GitHub Issues page and came across [this](https://github.com/syncthing/syncthing/issues/10051). It turns out that Syncthing has multiple outdated JavaScript libraries that need updating. If I want this vulnerability to disappear, the only thing I can do is wait for the library to be updated, or switch to an alternative file synchronization tool.  

All in all, the risk of this XSS vulnerability is only moderate, and only exploitable in specific attack scenarios requiring prior access. To mitigate this risk, I enabled HTTPS, set a complex password for the web console, and ensured that no sensitive files will be used by this application.

My vulnerability report:

``` title="Vulnerability B Report"
Host IP: 192.168.1.14  
Time Detected: Nov. 15, 2025 @ 15:51  
Scan Policy: Basic Network Scan  
Vulnerability: JQuery 1.2 < 3.5.0 Multiple XSS  
Description: According to the self-reported version in the script, the version of JQuery  
hosted on the remote web server is greater than or equal to 1.2 and prior to 3.5.0. It  
is, therefore, affected by multiple cross site scripting vulnerabilities.  

Analysis: Outdated version of JQuery detected, used by 'syncthing' Docker container  
listening on TCP port 8384 through a Docker proxy. The container was updated to the  
latest official version. Vulnerability still remains, but is negligible. Awaiting  
update of vulnerable JavaScript library 'jquery-2.2.2' -> 'jquery-3.5.0'.

Status: CLOSED | FALSE POSITIVE | AWAITING REMEDIATION
```

!!! tip "Reminder"
    Always keep your operating systems, applications, and dependencies up to date by regularly applying security patches and updates. When working in a production environment, test updates in a staging environment before deployment. Where feasible, automate the update process to maintain consistent protection against known vulnerabilities and monitor systems closely after updates.

### Vulnerability C: SMB Signing not required

??? example "Vulnerability C details"

    **Vulnerability**: SMB Signing not required
    
    **Severity**: Medium  

    **Description**  

    Signing is not required on the remote SMB server. An unauthenticated, remote attacker can exploit this to conduct man-in-the-middle attacks against the SMB server.  

    **Solution**  

    Enforce message signing in the host's configuration. On Windows, this is found in the policy setting 'Microsoft network server: Digitally sign communications (always)'. On Samba, the setting is called 'server signing'. See the 'see also' links for further details.  

    **See Also**

    http://www.nessus.org/u?df39b8b3  
    http://technet.microsoft.com/en-us/library/cc731957.aspx  
    http://www.nessus.org/u?74b80723  
    https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html  
    http://www.nessus.org/u?a3cac4ea  

    **Output**

    No output recorded.

    To see debug logs, please visit individual host

    | **Port**           | **Hosts**        |
    | :--------------| :----------- |
    | 445 / tcp / cifs| 192.168.1.14|

The last notable vulnerability that was discovered on this host was related to SMB signing. SMB signing is important because it ensures the integrity and authenticity of the data exchanged over the SMB protocol between clients and servers. This is a crucial security feature to protect network communications and shared data from interception and tampering.

The remediation of this vulnerability involves enabling SMB signing on the Linux host's Samba share. I can do this by adding two lines under the `[global]` section of the host's `smb.conf` file:

```sh title="Open the Samba configuration file"
sudo nano /etc/samba/smb.conf
```

``` title="Change smb.conf"
[global]
## smb signing
   client signing = mandatory
   server signing = mandatory
```

```sh title="Restart the Samba services"
sudo systemctl restart smbd nmbd
```

After another scan, the result showed a successful remediation:

??? example "Vulnerability C rescan results"
    <figure markdown>
    ![alt text](images/nessus-scan-03.png#center){.shadowed-image style="width: 90%;"}
    </figure>

My vulnerability report:

``` title="Vulnerability C Report"
Host IP: 192.168.1.14  
Time Detected: Nov. 15, 2025 @ 15:51  
Scan Policy: Basic Network Scan  
Vulnerability: SMB Signing not required  
Description: Signing is not required on the remote SMB server. An unauthenticated,  
remote attacker can exploit this to conduct man-in-the-middle attacks against the  
SMB server.  

Analysis: Mandatory SMB signing was detected as disabled on Linux Samba share.  
Samba configuration file was changed to require SMB signing from clients. The  
vulnerability was remediated.

Status: CLOSED | TRUE POSITIVE | REMEDIATED
```