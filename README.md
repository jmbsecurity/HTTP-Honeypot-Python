# HTTP-Honeypot-Python
Basic HTTP Honeypot written in Python.

# Simple HTTP Honeypot

A lightweight Python honeypot that logs all incoming HTTP requests to a JSONL file for threat intelligence and monitoring.

## Features

- Captures all HTTP methods (GET, POST, PUT, DELETE, HEAD, OPTIONS)
- Logs request details: IP, headers, user agent, path, POST data, timestamps
- JSONL output format for easy parsing
- No external dependencies (uses Python standard library)
- Fake server header to appear as Apache

## Requirements

- Python 3.6+

## Usage

```bash
# Basic usage (listens on port 8080)
python http_honeypot.py

# Custom port
python http_honeypot.py --port 9000

# Custom log file location
python http_honeypot.py --logfile /var/log/honeypot.jsonl

# Name your honeypot (useful for multiple instances)
python http_honeypot.py --name "vps-01"

# All options
python http_honeypot.py --port 8080 --logfile /var/log/honeypot.jsonl --name "my-honeypot"
```

## Options

| Flag | Default | Description |
|------|---------|-------------|
| `-p, --port` | 80,8080, whatever port you would like | Port to listen on |
| `-l, --logfile` | honeypot.jsonl | Path to log file |
| `-n, --name` | default | Honeypot identifier for logs |

## Log Format

Each request is logged as a JSON object on its own line:

```json
{
  "timestamp": "2025-02-05T14:30:00+00:00",
  "unix_timestamp": 1738765800,
  "honeypot_name": "my-honeypot",
  "remote_addr": "192.168.1.100",
  "remote_port": 54321,
  "method": "GET",
  "path": "/admin/login.php",
  "request_version": "HTTP/1.1",
  "headers": {
    "Host": "example.com",
    "User-Agent": "Mozilla/5.0...",
    "Accept": "*/*"
  },
  "user_agent": "Mozilla/5.0...",
  "host": "example.com",
  "post_data": null
}
```

## Analyzing Logs

```bash
# Watch logs in real-time
tail -f honeypot.jsonl

# Pretty print with jq
tail -f honeypot.jsonl | jq .

# Count requests by IP
jq -r '.remote_addr' honeypot.jsonl | sort | uniq -c | sort -rn

# Find top requested paths
jq -r '.path' honeypot.jsonl | sort | uniq -c | sort -rn | head -20

# Filter POST requests
jq 'select(.method == "POST")' honeypot.jsonl

# Requests from a specific IP
jq 'select(.remote_addr == "45.33.12.5")' honeypot.jsonl
```

## Running as a Service

Create `/etc/systemd/system/honeypot.service`:

```ini
[Unit]
Description=HTTP Honeypot
After=network.target

[Service]
Type=simple
User=nobody
ExecStart=/usr/bin/python3 /opt/honeypot/http_honeypot.py --port 8080 --logfile /var/log/honeypot.jsonl --name "my-vps"
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Then enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable honeypot
sudo systemctl start honeypot
```

## Firewall

Make sure your port is open:
Can set to whatever port you would like

```bash
# UFW
sudo ufw allow 8080/tcp

# iptables
sudo iptables -A INPUT -p tcp --dport 8080 -j ACCEPT
```

## License

MIT
