---
tags:
    - Containerization
    - Docker
    - Linux
    - Remote Desktop
    - RustDesk
    - Windows
---

# RustDesk

## Description

[**RustDesk**](https://rustdesk.com/) is an open-source remote desktop software written primarily in Rust. It allows remote access and control of computers and devices across multiple operating systems such as Windows, macOS, Linux, iOS, and Android. It aims to be a secure, private, and customizable alternative to proprietary software like TeamViewer or AnyDesk. It supports essential features including end-to-end encryption, file transfer, chat, and TCP tunneling. A notable advantage of RustDesk is the ability to use it with a self-hosted server, giving users full control over their data and enhanced privacy without relying on third-party servers. It offers a lightweight and user-friendly experience without mandatory account creation.

## Server setup (Docker)

Self-hosting your own RustDesk server ensures a higher quality connection for your clients.

1. Open these ports on your server's firewall:

    ```sh
    ufw allow 21114:21119/tcp
    ufw allow 21116/udp
    sudo ufw enable
    ```

2. Run these commands to pull and deploy the official RustDesk server Docker compose file:

    ```sh
    bash <(wget -qO- https://get.docker.com)  # remove this line if Docker is already installed, 
    wget rustdesk.com/oss.yml -O compose.yml  # or press Ctrl + C when prompted by the terminal
    sudo docker compose up -d
    ```

3. Check the Docker logs for the **hbbs** container to get your **public key**:

    ```sh
    sudo docker logs hbbs
    ```

    Log:

    ```
    INFO [src/rendezvous_server.rs:1205] Key: xxxxx
    ```

## Client setup

1. Download and install the latest stable release for your system:

    <https://github.com/rustdesk/rustdesk/releases>

2. Launch the application. Navigate to **Network -> ID/Relay server**.

3. Provide the following information:


    * **ID server**: (SERVER_IP):21116

    * **Key**: (KEY)

    * The **Relay server** and **API server** will be recognized automatically.

4. Verify that the client is connected to your server. Go back to the main window. The server status should say **Ready**.

5. To control another host, enter the client's ID into the **Control Remote Desktop** field and click **Connect**.

6. When prompted, enter the password set on the client. If using a one-time password, get it from the client before connecting.