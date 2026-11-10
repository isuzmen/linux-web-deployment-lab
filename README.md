# Linux Web Deployment Lab

A hands-on DevOps project demonstrating how to deploy a web application on an Ubuntu Linux server using Apache, systemd, and UFW.

## Architecture

Windows Client
    |
    | HTTP :80
    v
UFW Firewall
    |
    v
Apache Reverse Proxy
    |
    | /app/ -> http://127.0.0.1:3000/
    v
Python HTTP Server
    |
    v
index.html

## Technologies

- Ubuntu Linux
- Apache HTTP Server
- Python HTTP Server
- systemd
- UFW Firewall
- SSH
- Git and GitHub (repository management)

## Project Structure

- `app/index.html` - Sample web page
- `deploy/devops-web.service` - systemd service definition
- `deploy/devops-app-proxy.conf` - Apache reverse proxy configuration
- `RUNBOOK.md` - Operations and troubleshooting guide

## How It Works

1. Apache listens for incoming HTTP connections on port 80.
2. Requests to `/app/` are forwarded to `127.0.0.1:3000`.
3. A Python HTTP server serves the application.
4. systemd manages the Python server and starts it during boot.
5. UFW allows incoming SSH and HTTP traffic.

## Verification

Check the application service:

    systemctl status devops-web

Test the backend directly from Ubuntu:

    curl http://127.0.0.1:3000/

Test the reverse proxy:

    curl http://127.0.0.1/app/

Test from another computer:

    curl http://SERVER_IP/app/

Expected response:

    <h1>Hello from DevOps App!</h1>

## Notes

- This project is intended for learning and demonstration.
- Python's built-in HTTP server is not intended for production use.
- The included systemd service uses an environment-specific home directory.
- The backend listens on localhost and is not directly exposed to the network.

## Installation on Ubuntu

### 1. Install dependencies

    sudo apt update
    sudo apt install apache2 python3 ufw

### 2. Configure the systemd service

Before installing the service, edit `deploy/devops-web.service` to match your system:

- Set `User=` to your Ubuntu username.
- Update `WorkingDirectory=` to your project directory.
- Update the directory path in `ExecStart=`.

Copy the service definition:

    sudo cp deploy/devops-web.service /etc/systemd/system/
    sudo systemctl daemon-reload
    sudo systemctl enable --now devops-web

### 3. Configure Apache

    sudo cp deploy/devops-app-proxy.conf /etc/apache2/conf-available/
    sudo a2enmod proxy proxy_http
    sudo a2enconf devops-app-proxy
    sudo apache2ctl configtest
    sudo systemctl restart apache2

### 4. Configure the firewall

Ensure your SSH access is allowed before enabling UFW.

For this lab, SSH uses TCP 22 and HTTP uses TCP 80:

    sudo ufw allow 22/tcp
    sudo ufw allow 80/tcp
    sudo ufw enable
    sudo ufw status verbose

Do not blindly enable UFW on a remote server without checking its SSH configuration and ensuring recovery access.

### 5. Verify the deployment

On Ubuntu:

    systemctl status devops-web --no-pager
    curl http://127.0.0.1:3000/
    curl http://127.0.0.1/app/

From another computer:

    curl http://SERVER_IP/app/

Expected response:

    <h1>Hello from DevOps App!</h1>

For troubleshooting, see RUNBOOK.md.

## Docker Deployment

This project uses Docker Compose to run two services:

- **web:** NGINX serves the HTML page and acts as a reverse proxy.
- **api:** Python HTTP server providing a `/health` endpoint.

### Architecture

Client -> NGINX (web:80) -> Python API (api:5000)

NGINX forwards `/api/health` requests to `/health` on the API.

The web service is available at `127.0.0.1:8080`.
The API port is not published to the host.

### Start

```bash
sudo docker compose up -d --build
```

### Verify

```bash
sudo docker compose ps
curl http://127.0.0.1:8080/
curl http://127.0.0.1:8080/api/health
```

### Logs

```bash
sudo docker compose logs --tail=20 web
sudo docker compose logs --tail=20 api
```

### Stop

```bash
sudo docker compose down
```

### Healthcheck

Docker periodically checks the API's `/health` endpoint.

During startup, Docker Compose waits for the API to become healthy before starting the web service.
