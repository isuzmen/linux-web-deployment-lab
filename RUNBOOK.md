# Operations Runbook

This runbook documents how to operate and troubleshoot the Linux Web Deployment Lab.

## 1. Architecture

Client -> Apache (:80) -> Python HTTP Server (127.0.0.1:3000)

- Apache handles incoming HTTP requests.
- Requests to /app/ are forwarded to the Python server.
- systemd manages the Python application.
- UFW controls incoming network traffic.

## 2. Where to Run Commands

Windows PowerShell:
- Connect to the Ubuntu server using SSH.
- Test the application from outside Ubuntu.

Ubuntu terminal:
- Manage services.
- Inspect listening ports.
- Check logs and firewall rules.
- Test the backend locally.

## 3. Check the Application Service

    systemctl status devops-web --no-pager

Look for:
- active (running): the service is running.
- inactive: the service is stopped.
- failed: the service encountered a failure.

Manage the service:

    sudo systemctl start devops-web
    sudo systemctl stop devops-web
    sudo systemctl restart devops-web

Check whether it starts automatically:

    systemctl is-enabled devops-web

## 4. Check Listening Ports

    sudo ss -tlnp

Expected:
- Apache listens on port 80.
- Python listens on 127.0.0.1:3000.

## 5. Test HTTP Connectivity

Run on Ubuntu to test Python directly:

    curl http://127.0.0.1:3000/

Run on Ubuntu to test Apache and the reverse proxy:

    curl http://127.0.0.1/app/

Run on Windows to test external access:

    curl.exe http://SERVER_IP/app/

Expected application response:

    <h1>Hello from DevOps App!</h1>

## 6. Check Logs

Python application logs:

    journalctl -u devops-web -n 50 --no-pager

Current boot logs:

    journalctl -b -u devops-web --no-pager

Apache error logs:

    sudo tail -n 50 /var/log/apache2/error.log

## 7. Check Apache Reverse Proxy

Proxy configuration:

    /etc/apache2/conf-available/devops-app-proxy.conf

Inspect the active configuration:

    sudo cat /etc/apache2/conf-enabled/devops-app-proxy.conf

Check configuration syntax:

    sudo apache2ctl configtest

After a valid configuration change:

    sudo systemctl reload apache2

## 8. Check Firewall Rules

    sudo ufw status verbose

Required incoming ports for this lab:
- TCP 22: SSH
- TCP 80: HTTP

The Python server listens only on localhost.
Port 3000 does not need an incoming UFW allow rule.

Always preserve SSH access when changing firewall rules.

## 9. Troubleshooting: /app/ Returns HTTP 503

1. Check the Python service using systemctl.
2. Inspect listening ports using ss.
3. Test the backend directly using curl on port 3000.
4. If the backend works, inspect Apache proxy configuration.
5. Read Apache error logs for the specific failure.
6. Fix the identified cause and verify HTTP access again.

A 503 response does not prove that the backend received the request.
Apache may generate 503 when the backend is unavailable.

## 10. After Changing a systemd Unit

    sudo systemctl daemon-reload
    sudo systemctl restart devops-web

Use daemon-reload when the systemd unit definition changes.
Do not use it unnecessarily for ordinary application file changes.

## 11. Security Notes

- Do not commit SSH private keys or passwords to GitHub.
- The Python built-in HTTP server is for laboratory use only.
- Review hardcoded paths and usernames before deploying elsewhere.
- Do not expose port 3000 to the external network.
