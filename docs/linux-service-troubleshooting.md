# Linux Service Troubleshooting

A practical guide for investigating Linux services, processes, ports, packages, and logs on Ubuntu.

## 1. Check Service Status

Use `systemctl` to inspect a service managed by systemd.

```bash
systemctl status apache2.service
```

Important fields:

- `Loaded`: Shows whether the service unit is loaded and whether it is enabled.
- `Active`: Indicates the current service state.
- `Main PID`: Identifies the main process.
- `TriggeredBy`: Shows related triggering units, when present.

A service can be active even when it is disabled. For example, it may have been started manually or activated by a socket unit.

## 2. Inspect Processes and Ports

Check the main process of a service:

```bash
ps -p 4511 -o pid,ppid,comm,stat
```

Find processes listening on TCP port 80:

```bash
sudo ss -ltnp '( sport = :80 )'
```

A listening port indicates that a process or socket is accepting connection attempts. It does not guarantee that the application is functioning correctly.

## 3. Inspect Service Configuration

Use `systemctl cat` to inspect the definition of a systemd unit:

```bash
systemctl cat ssh.service
```

The `ExecStart` directive specifies the command used to start the service.

For example, the SSH server service can start the `sshd` executable.

## 4. Identify the Software Package

On Ubuntu, use `dpkg -S` to identify which installed package owns a file:

```bash
dpkg -S /usr/sbin/sshd
```

Inspect installed and candidate package versions:

```bash
apt-cache policy openssh-server
```

This helps connect a running service to its executable and installed software package.

## 5. Examine Logs

Inspect systemd journal records for a service:

```bash
journalctl -u apache2.service -n 20
```

Inspect recent Apache HTTP access records:

```bash
tail -n 20 /var/log/apache2/access.log
```

Inspect Apache error messages:

```bash
tail -n 20 /var/log/apache2/error.log
```

The systemd journal and application log files serve different purposes. Not every HTTP request will appear in the systemd journal.

## 6. Diagnose HTTP Responses

Send an HTTP request and display response headers:

```bash
curl -I http://127.0.0.1/
```

Common status codes:

- `200 OK`: The request succeeded.
- `404 Not Found`: The requested resource was not found.
- `500 Internal Server Error`: The server encountered an error while processing the request.

An HTTP 404 response does not automatically mean the web server service is broken.

## 7. Basic Troubleshooting Workflow

1. Verify the service state.
2. Identify the running process and listening port.
3. Inspect the service and application configuration.
4. Check application files and permissions when relevant.
5. Examine journal entries and application logs.
6. Test the application and compare the results with the collected evidence.

The goal is to identify the failing component before changing the system configuration.
