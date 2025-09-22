# Arr Stack Installation and Configuration Guide

This guide will help you deploy a complete media automation stack using Docker. This setup is based on the configuration from the [mediaStack GitHub repository](https://github.com/frandevaux/mediaStack).

### Useful Links:
- [Servarr Wiki](https://wiki.servarr.com/)
- [Trash Guides](https://trash-guides.info/)
- [Ascii ART](https://patorjk.com/software/taag/#p=display&f=ANSI%20Shadow)

---
## 1. Preparation ⚙️

### Download Files
First, download the necessary configuration files from the GitHub repository. You can do this by cloning the repository or downloading the ZIP file.

```bash
# Example of cloning the repository
git clone [https://github.com/frandevaux/mediaStack.git](https://github.com/frandevaux/mediaStack.git)
cd mediaStack
````

### Environment Setup (`.env`)

Before you start, it is **crucial** to configure the `.env` file. This file is located in the same folder as `docker-compose.yml`.

1.  **Open the `.env` file** with a text editor.
2.  **Define your `ARRPATH`**: This is the main path on your host machine where all configurations and media files will be stored.
      * **Linux Example**: `ARRPATH=/media/Arr`
      * **Windows Example**: `ARRPATH=C:\Docker\Arr`
3.  **Define your `PUID` and `PGID`**: These are the User and Group IDs. On Linux, `1000:1000` is usually correct for the main user. For Windows, you can leave them as `1000:1000` since Docker Desktop handles permissions differently.

-----

## 2\. Docker Installation 🐳

Make sure you are in the same folder as your `docker-compose.yml` and `.env` files.

  * **To bring up all services**:
    ```bash
    docker-compose up -d 
    ```
  * **To stop the services**:
    ```bash
    docker-compose stop
    ```
  * **To stop and remove the containers**:
    ```bash
    docker-compose down
    ```

### Permission Management (Critical Step)

#### For Linux Users:

The containers need permission to write to the folder you defined in `ARRPATH`. Run the `chown` command to assign ownership of the folder to the user and group specified in your `.env` file.

```bash
# Replace 1000:1000 and /media/Arr with the values from your .env
sudo chown -R 1000:1000 /media/Arr
```

#### For Windows Users:

You do not need to use `chown`. Instead, ensure that **your Windows user account** has "Full control" over the `ARRPATH` folder.

1.  Right-click the folder (e.g., `C:\Docker\Arr`) -\> **Properties**.
2.  Go to the **Security** tab.
3.  Select your user, click **Edit...**, and check the **Full control** box.

-----

## 3\. Services Configuration 🛠️

You can now access and configure each service from your web browser.

### **qBittorrent (Download Client)**

1.  **Find the temporary password** in the container logs:
    ```bash
    docker logs qbittorrent
    ```
2.  **Access the web UI**: http://localhost:8080.
3.  Log in with the username `admin` and the temporary password.
4.  Go to **Tools -\> Options -\> WebUI**.
5.  **Change the username and password** to permanent ones.
6.  (Optional) Check "Bypass authentication for clients on localhost".

### **Prowlarr (Indexer Manager)**

1.  **Access the web UI**: http://localhost:9696 (will require creating a username and password).
2.  Go to **Settings -\> Download Clients** and click `+`.
3.  Select **qBittorrent**.
4.  Configure it as follows:
      * **Host**: `qbittorrent` (this is the service name in Docker, more reliable than an IP).
      * **Port**: `8080`.
      * **Username/Password**: The ones you just configured in qBittorrent.
5.  Click **Test** to verify the connection, then **Save**.
6.  Go to **Indexers**, add your preferred ones (e.g., yts, rarbg), and then click **Sync App Indexers**.

### **Sonarr (TV Show Manager)**

1.  **Access the web UI**: http://localhost:8989.
2.  **Set up the Root Folder**: Go to **Settings -\> Media Management -\> Add Root Folder**. The path you must add is `/tv`.
3.  **Connect the Download Client**: Go to **Settings -\> Download Clients**, click `+`, and configure qBittorrent (same as in Prowlarr, using `qbittorrent` as the host).
4.  **Connect with Prowlarr**: Go to **Settings -\> General**, and copy the **API Key**. Then, in Prowlarr, go to **Settings -\> Apps**, click `+`, select Sonarr, and paste the API Key. Use `sonarr` as the hostname.

### **Radarr (Movie Manager)**

1.  **Access the web UI**: http://localhost:7878.
2.  **Set up the Root Folder**: Go to **Settings -\> Media Management -\> Add Root Folder**. The path is `/movies`.
3.  **Connect the download client** and **sync with Prowlarr** following the same steps as for Sonarr, using `radarr` as the hostname and its own API Key.

### **Lidarr (Music Manager)**

1.  **Access the web UI**: http://localhost:8686.
2.  Follow the same steps as for Sonarr/Radarr. The root media folder is `/music`.

### **Bazarr (Subtitle Manager)**

1.  **Access the web UI**: http://localhost:6767.
2.  **Connect with Sonarr and Radarr**:
      * Go to **Settings -\> Sonarr**. Enable it.
      * **Host**: `sonarr`.
      * **Port**: `8989`.
      * Paste the **Sonarr API Key**.
      * Test the connection and save.
      * Repeat the process in **Settings -\> Radarr** (Host: `radarr`, Port: `7878`, Radarr API Key).

### **Jellyfin (Media Server)**

1.  **Access the web UI**: http://localhost:8096.
2.  During the initial setup wizard, add your media libraries.
3.  When asked for the folder path, use the **standardized paths** that the containers see:
      * **Movies**: `/movies`
      * **TV Shows**: `/tv`
      * **Music**: `/music`

### **Homarr (Dashboard)**

  * **Access the web UI**: http://localhost:7575.
  * Configure your services to have a centralized dashboard.

-----

## 4\. (Optional) Remote Access with Tailscale 🌐

To securely access your services from outside your home, you can use Tailscale.

### Prerequisite

1.  **Generate an Auth Key** in the [Tailscale admin console](https://login.tailscale.com/admin/settings/keys). Disable expiration for convenience.
2.  **Add the key to your `.env` file**:
    ```
    TS_AUTHKEY=tskey-auth-your-key-here...
    ```

### Installation

The `docker-compose.yml` file already includes the Tailscale service. Simply bring up the stack with `docker-compose up -d`.

### Post-Installation Steps

1.  **Authorize Routes**: In your [Tailscale admin console's Machines page](https://login.tailscale.com/admin/machines), find your new machine (`docker-gateway`). Click the `...` menu -\> **Review subnet routes...** and enable the routes it advertises.
2.  **Disable Key Expiry**: In the same menu, click **Disable key expiry...**.

### How to Use It?

To access a service, you need its **internal Docker IP**. To find Sonarr's IP, for example:

```bash
docker inspect sonarr | grep IPAddress
```

If the IP is `172.20.0.5`, you can access Sonarr from any device on your Tailnet at the URL: **`http://172.20.0.5:8989`**. Repeat this step for the other services.

That's it\! Your media automation stack is complete and accessible.

```
```
