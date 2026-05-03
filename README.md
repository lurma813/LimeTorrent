<div align="center">

<img src="https://i.kek.sh/YZnRLgK4dJX.png" width="80" alt="LimeTorrent logo">

# 🍋 LimeTorrent

**A lightweight, self-hosted torrent manager with a REST API and Web UI.**  
Built on [libtorrent 2.0.x](https://libtorrent.org) and [Flask](https://flask.palletsprojects.com).

[![Release](https://img.shields.io/github/v/release/lurma813/limetorrent?color=a3e635&label=release)](https://github.com/lurma813/limetorrent/releases)
[![Platform](https://img.shields.io/badge/platform-Ubuntu%20%7C%20Linux-a3e635)](#installation)
[![License](https://img.shields.io/github/license/lurma813/limetorrent?color=a3e635)](LICENSE)

</div>

---

## Table of Contents

- [Features](#features)
- [Screenshots](#screenshots)
- [Installation](#installation)
  - [Pre-built Binary (Ubuntu)](#pre-built-binary-ubuntu)
  - [Running from Source](#running-from-source)
- [Usage](#usage)
  - [CLI Options](#cli-options)
  - [Environment Variables](#environment-variables)
  - [Examples](#examples)
- [Authentication](#authentication)
- [REST API Overview](#rest-api-overview)
- [State Model](#state-model)
- [Resume Persistence](#resume-persistence)
- [Running as a Service (systemd)](#running-as-a-service-systemd)
- [Running with Docker](#running-with-docker)
- [Updating](#updating)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)

---

## Features

- **Multiple add methods** — magnet links, `.torrent` file upload, or direct `.torrent` URL
- **Full torrent control** — pause, resume, stop, remove (with optional data deletion), recheck, re-announce
- **Per-file priority** — select which files to download and at what priority
- **Speed limits** — global and per-torrent upload/download caps
- **Super-seeding mode** — per-torrent and global, ideal for initial seeding to a swarm
- **Resume persistence** — torrent state survives restarts; on-disk resume data written atomically
- **All-time statistics** — cumulative upload/download totals persisted across restarts
- **REST API** — full JSON API with API-key authentication for automation and scripting
- **Web UI** — dark/light theme, live progress, drag-and-drop upload, multi-select bulk actions
- **View-only mode** — unauthenticated users can monitor torrents but cannot modify anything
- **Login rate limiting** — IP lockout after 3 failed attempts (5-minute cooldown)
- **Graceful shutdown** — SIGTERM/SIGINT handler saves all resume data before exit
- **Verbose logging** — optional debug mode; minimal output by default

---

## Screenshots

> _Web UI — dark theme_

![WebUI dark](https://i.kek.sh/YZnRLgK4dJX.png)

---

## Installation

### Pre-built Binary (Ubuntu)

LimeTorrent ships as a **single self-contained binary** compiled for Ubuntu (64-bit). No Python installation is required.

**Requirements:**
- Ubuntu 20.04 / 22.04 / 24.04 (or compatible Debian-based distro)
- `libtorrent-rasterbar` shared library (usually pre-installed or available via apt)

**Step 1 — Install the libtorrent runtime library**

```bash
sudo apt-get update
sudo apt-get install -y libboost-system1.74.0 libtorrent-rasterbar2.0
```

> If your Ubuntu version ships a different `libtorrent-rasterbar` version, install whichever is available (`apt-cache search libtorrent`).

**Step 2 — Download the binary**

```bash
# Replace X.Y.Z with the latest release version
wget https://github.com/lurma813/limetorrent/releases/download/vX.Y.Z/limetorrent-ubuntu-amd64
chmod +x limetorrent-ubuntu-amd64
```

**Step 3 — Run**

```bash
./limetorrent-ubuntu-amd64
```

On first launch you will see:

```
LimeTorrent running on http://127.0.0.1:5000
  - WebUI available at http://127.0.0.1:5000/webui
  - API Documentation at http://127.0.0.1:5000/doc
  - WebUI user: admin  |  password: aB3xKz  (AUTO-GENERATED)
  - API Key (for automation): a1b2c3d4e5f6...
  - libtorrent 2.0.9 | listen: 0.0.0.0:6881 | conns: 500
```

The auto-generated password is shown **once** at startup. Note it down or set a custom password via `--webui-pass`.

---

### Running from Source

**Requirements:**
- Python 3.10+
- `libtorrent` Python bindings (`python3-libtorrent` or via pip)
- `Flask` 3.x

```bash
# 1. Clone the repo
git clone https://github.com/lurma813/limetorrent.git
cd limetorrent

# 2. Install dependencies
pip install flask libtorrent

# 3. Run
python main.py
```

---

## Usage

### CLI Options

```
Usage: limetorrent [OPTIONS]

Options:
  --host HOST             Bind address (default: 127.0.0.1)
  --port PORT             Bind port   (default: 5000)
  --download-dir DIR      Download directory (default: ~/Downloads)
  --torrent-dir DIR       Output directory for created .torrent files
  --resume-dir DIR        Resume data storage directory
  --upload-limit BPS      Global upload limit in bytes/s  (0 = unlimited)
  --download-limit BPS    Global download limit in bytes/s (0 = unlimited)
  --upload-slots N        Max upload slots per torrent (default: 8)
  --connections N         Max total connections (default: 500)
  --listen IFACE:PORT     libtorrent listen interface (default: 0.0.0.0:6881)
  --webui-user USER       WebUI username (default: admin)
  --webui-pass PASS       WebUI password (default: random 6-char)
  --super-seeding         Enable global super-seeding mode on startup
  --debug                 Enable full debug logging (includes HTTP request log)
  --help                  Show this help message and exit
```

### Environment Variables

All CLI options can also be set via environment variables. CLI arguments take priority over environment variables.

| Variable | CLI equivalent | Default |
|---|---|---|
| `HOST` | `--host` | `127.0.0.1` |
| `PORT` | `--port` | `5000` |
| `DOWNLOAD_DIR` | `--download-dir` | `~/Downloads` |
| `TORRENT_DIR` | `--torrent-dir` | `~/Downloads/.limetorrent/created` |
| `RESUME_DIR` | `--resume-dir` | `~/Downloads/.limetorrent/resume` |
| `GLOBAL_UPLOAD_LIMIT` | `--upload-limit` | `0` |
| `GLOBAL_DOWNLOAD_LIMIT` | `--download-limit` | `0` |
| `UPLOAD_SLOTS` | `--upload-slots` | `8` |
| `CONNECTIONS_LIMIT` | `--connections` | `500` |
| `LISTEN_INTERFACES` | `--listen` | `0.0.0.0:6881` |
| `WEBUI_USER` | `--webui-user` | `admin` |
| `WEBUI_PASS` | `--webui-pass` | _(random)_ |
| `SUPER_SEEDING` | `--super-seeding` | `false` |
| `DEBUG` | `--debug` | `false` |

### Examples

```bash
# Listen on all interfaces, custom port
./limetorrent --host 0.0.0.0 --port 8080

# Set credentials and download directory
./limetorrent --webui-user admin --webui-pass mysecret --download-dir /data/torrents

# Limit upload to 2 MB/s, download to 10 MB/s
./limetorrent --upload-limit 2097152 --download-limit 10485760

# Enable super-seeding and verbose logging
./limetorrent --super-seeding --debug

# Using environment variables
HOST=0.0.0.0 PORT=8080 WEBUI_PASS=secret ./limetorrent
```

---

## Authentication

LimeTorrent supports two authentication methods:

### 1. API Key (recommended for automation)

Pass the `Lime-API-Key` header with every request:

```bash
curl -H "Lime-API-Key: YOUR_API_KEY" http://127.0.0.1:5000/list
```

The API key is printed to stdout on startup and is also visible in **Settings → API Key** (requires session login).

### 2. Session Cookie (WebUI login)

Log in via the Web UI at `/webui` or via the API:

```bash
curl -s -c cookies.txt -X POST http://127.0.0.1:5000/webui/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"YOUR_PASS"}'

# Use the cookie for subsequent requests
curl -b cookies.txt http://127.0.0.1:5000/list
```

> **Rate limiting:** After 3 failed login attempts, the IP is locked out for 5 minutes.

### View-only Access

Unauthenticated users who open the Web UI can **view** torrent status but cannot add, pause, stop, or remove torrents.

---

## REST API Overview

Full interactive documentation is available at **`http://127.0.0.1:5000/doc`** when the server is running.

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/add/magnet` | Add torrent via magnet link |
| `POST` | `/add/file` | Add torrent via `.torrent` file upload |
| `POST` | `/add/url` | Add torrent via direct `.torrent` URL |
| `GET` | `/list` | List all torrents |
| `GET` | `/status/{hash}` | Status of a single torrent |
| `GET` | `/files/{hash}` | List files and priorities |
| `POST` | `/files/{hash}` | Set per-file download priorities |
| `GET` | `/peers/{hash}` | List connected peers |
| `GET` | `/trackers/{hash}` | List tracker URLs and status |
| `GET` | `/magnet/{hash}` | Get magnet URI |
| `POST` | `/pause/{hash}` | Pause torrent |
| `POST` | `/resume/{hash}` | Resume torrent |
| `POST` | `/stop/{hash}` | Stop torrent |
| `DELETE` | `/remove/{hash}` | Remove torrent (`?delete_files=1` to wipe data) |
| `POST` | `/recheck/{hash}` | Force piece recheck |
| `POST` | `/announce/{hash}` | Force re-announce to trackers |
| `POST` | `/limit/{hash}` | Per-torrent speed limits |
| `POST` | `/limit/global` | Global speed limits |
| `POST` | `/super_seed/{hash}` | Toggle super-seeding per torrent |
| `POST` | `/super_seed/global` | Toggle global super-seeding |
| `POST` | `/create` | Create `.torrent` from local path |
| `POST` | `/seed` | Seed existing local data |
| `GET` | `/stats/global` | Session + all-time stats |
| `POST` | `/save` | Persist resume data to disk now |
| `GET` | `/health` | Health check |
| `GET` | `/api/key` | Get current API key _(session required)_ |
| `POST` | `/api/key/toggle` | Enable/disable API key auth |
| `GET` | `/webui` | Web UI (browser) |
| `GET` | `/doc` | API documentation |

---

## State Model

| State | Description |
|-------|-------------|
| `downloading` | Actively downloading; progress < 100% |
| `seeding` | Download complete; actively uploading to peers |
| `paused` | Temporarily paused; can auto-resume |
| `stopped` | Explicitly stopped (`auto_managed=off`); progress < 100% |
| `completed` | Explicitly stopped after finishing download |
| `checking` | Running piece verification |
| `metadata` | Fetching torrent metadata (magnet link, no info-hash yet) |

---

## Resume Persistence

LimeTorrent saves **resume data** to disk so torrents survive server restarts.

- Resume files are stored in `RESUME_DIR` (default: `~/Downloads/.limetorrent/resume/`)
- Each torrent gets a `<infohash>.resume` file and, if completed, a `<infohash>.completed` marker
- On startup, torrents in `stopped`, `paused`, or `downloading` state are automatically **rechecked** to verify on-disk data integrity
- Torrents already `seeding` or `completed` skip recheck (already verified)

**Transferring torrents to another machine:**

1. Copy the downloaded data folder to the same path on the new machine
2. Copy the corresponding `.resume` file from `RESUME_DIR` to the new instance's `RESUME_DIR`
3. Start LimeTorrent — it will recheck and resume from where it left off

---

## Running as a Service (systemd)

To run LimeTorrent automatically on boot:

**1. Create a systemd unit file**

```bash
sudo nano /etc/systemd/system/limetorrent.service
```

```ini
[Unit]
Description=LimeTorrent - Torrent Manager
After=network.target

[Service]
Type=simple
User=YOUR_USERNAME
WorkingDirectory=/opt/limetorrent
ExecStart=/opt/limetorrent/limetorrent-ubuntu-amd64 \
  --host 0.0.0.0 \
  --port 5000 \
  --download-dir /data/torrents \
  --webui-pass YOUR_SECURE_PASSWORD
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

**2. Enable and start**

```bash
sudo systemctl daemon-reload
sudo systemctl enable limetorrent
sudo systemctl start limetorrent

# Check status
sudo systemctl status limetorrent

# View logs
sudo journalctl -u limetorrent -f
```

**3. (Optional) Open the firewall port**

```bash
sudo ufw allow 5000/tcp
```

---

## Running with Docker

> A Dockerfile is not yet included in the release. The following example shows how to run the binary inside an Ubuntu container.

```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y \
    libboost-system1.74.0 \
    libtorrent-rasterbar2.0 \
    && rm -rf /var/lib/apt/lists/*

COPY limetorrent-ubuntu-amd64 /usr/local/bin/limetorrent
RUN chmod +x /usr/local/bin/limetorrent

VOLUME ["/data/downloads", "/data/resume"]
EXPOSE 5000 6881 6881/udp

CMD ["limetorrent", \
     "--host", "0.0.0.0", \
     "--download-dir", "/data/downloads", \
     "--resume-dir", "/data/resume"]
```

```bash
docker build -t limetorrent .

docker run -d \
  --name limetorrent \
  -p 5000:5000 \
  -p 6881:6881 \
  -p 6881:6881/udp \
  -v /my/downloads:/data/downloads \
  -v /my/resume:/data/resume \
  -e WEBUI_PASS=mysecret \
  limetorrent
```

---

## Updating

1. Stop the running instance:
   ```bash
   sudo systemctl stop limetorrent
   # or: kill -SIGTERM <pid>
   ```
   LimeTorrent will save all resume data before exiting.

2. Replace the binary with the new version:
   ```bash
   wget -O /opt/limetorrent/limetorrent-ubuntu-amd64 \
     https://github.com/lurma813/limetorrent/releases/download/vX.Y.Z/limetorrent-ubuntu-amd64
   chmod +x /opt/limetorrent/limetorrent-ubuntu-amd64
   ```

3. Start again:
   ```bash
   sudo systemctl start limetorrent
   ```

All torrents and their state will be restored automatically from the resume directory.

---

## Troubleshooting

**Port already in use**

```
OSError: [Errno 98] Address already in use
```

Another process is using port 5000. Either stop it or choose a different port:
```bash
./limetorrent --port 8080
```

**`libtorrent` shared library not found**

```
error while loading shared libraries: libtorrent-rasterbar.so.2.0
```

Install the missing runtime library:
```bash
sudo apt-get install libtorrent-rasterbar2.0
# or for older Ubuntu:
sudo apt-get install libtorrent-rasterbar9
```

**Torrent stuck at 0% / metadata**

This is normal for magnet links — the client must first fetch the torrent metadata from peers. It may take a few minutes if the swarm is small. Use `/trackers/{hash}` to check tracker status.

**Resume data lost after restart**

Ensure you send `SIGTERM` (not `SIGKILL`) when stopping the process. `SIGKILL` bypasses the shutdown handler and does not save resume data. If using systemd, `systemctl stop` sends SIGTERM by default.

**Web UI shows "View only" after login**

Clear your browser cookies for the LimeTorrent domain and log in again.

---

## FAQ

**Can I run LimeTorrent on Windows?**  
The pre-built binary targets Ubuntu/Linux. You can run it from source on Windows if you install Python and the `libtorrent` Windows bindings, though this is not officially supported.

**Is there a mobile app?**  
No native app exists. The Web UI is responsive and works in mobile browsers.

**Where is the configuration file?**  
LimeTorrent is configured entirely via CLI arguments or environment variables. There is no config file by design — use a systemd unit or shell script to persist your preferred settings.

**How do I change the API key?**  
The API key is randomly generated at each startup. If you need a stable key, set a custom `WEBUI_PASS` and use session authentication, or manage the API key toggle via the Settings page in the Web UI.

**Can I run multiple instances?**  
Yes — use different `--port`, `--download-dir`, and `--resume-dir` values for each instance.

---

<div align="center">

Built with ❤️ using [libtorrent](https://libtorrent.org) + [Flask](https://flask.palletsprojects.com)

[GitHub](https://github.com/lurma813/limetorrent) · [Issues](https://github.com/lurma813/limetorrent/issues) · [Releases](https://github.com/lurma813/limetorrent/releases)

</div>
