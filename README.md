<div align="center">

<img src="https://i.kek.sh/YZnRLgK4dJX.png" width="80" alt="LimeTorrent logo">

# LimeTorrent

**A lightweight, self-hosted torrent manager with a REST API and Web UI.**  
Built on [libtorrent 2.0.x](https://libtorrent.org) and [Flask](https://flask.palletsprojects.com).

[![Platform](https://img.shields.io/badge/platform-Ubuntu%20%7C%20Linux%20%7C%20Windows-a3e635)](#installation)
[![License](https://img.shields.io/github/license/lurma813/LimeTorrent?color=a3e635)](LICENSE)

</div>

---

## Table of Contents

- [Features](#features)
- [Screenshots](#screenshots)
- [Installation](#installation)
- [Usage](#usage)
  - [CLI Options](#cli-options)
  - [Environment Variables](#environment-variables)
  - [Examples](#examples)
- [Authentication](#authentication)
- [REST API Overview](#rest-api-overview)
- [State Model](#state-model)
- [Resume Persistence](#resume-persistence)
- [Running as a Service (systemd)](#running-as-a-service-systemd)
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

<p align="center">
  <span style="display: inline-flex; gap: 20px;">
    <img src="https://i.kek.sh/krUEExxQ1tK.png" width="300" alt="Light-Theme">
    <img src="https://i.kek.sh/zrjcjd0xGPS.png" width="300" alt="Dark-Theme">
  </span>
</p>

---

## Installation
**Ubuntu/Debian**
```bash
git clone https://github.com/lurma813/LimeTorrent
sudo cp LimeTorrent/LimeTorrent /usr/local/bin
sudo chmod u+rwx /usr/local/bin/LimeTorrent
rm -r LimeTorrent/
```

**Run**

```bash
LimeTorrent 
```

On first launch you will see:

```
LimeTorrent running on http://127.0.0.1:5000
  - WebUI available at http://127.0.0.1:5000/webui
  - API Documentation at http://127.0.0.1:5000/doc
  - WebUI user: admin  |  password: lAQ90OeO...  (AUTO-GENERATED)
  - API Key : cc69b715bbc953e69127697ab63...
  - libtorrent 2.1.0.0 | listen: 127.0.0.1:6881 | conns: 500
```

The auto-generated password is shown **once** at startup. Note it down or set a custom password via `--webui-pass`.

---

## Usage

### CLI Options

```
usage: LimeTorrent [-h] [--host HOST] [--port PORT]
                           [--download-dir DIR] [--torrent-dir DIR]
                           [--resume-dir DIR] [--upload-limit BPS]
                           [--download-limit BPS] [--upload-slots N]
                           [--connections N] [--listen IFACE:PORT]
                           [--webui-user USER] [--webui-pass PASS]
                           [--super-seeding] [--debug]

LimeTorrent — libtorrent 2.0.x REST API server.
Manages torrents via HTTP: add (magnet/file/URL), pause, stop,
delete (by hash or .torrent file), seed, create, monitor, and more.

options:
  -h, --help            show this help message and exit
  --host HOST           Bind address (default: 127.0.0.1, env: HOST)
  --port PORT           Bind port (default: 5000, env: PORT)
  --download-dir DIR    Directory for downloaded files (env: DOWNLOAD_DIR)
  --torrent-dir DIR     Directory for created .torrent files (env:
                        TORRENT_DIR)
  --resume-dir DIR      Directory for resume data (env: RESUME_DIR)
  --upload-limit BPS    Global upload limit in bytes/s, 0=unlimited (env:
                        GLOBAL_UPLOAD_LIMIT)
  --download-limit BPS  Global download limit in bytes/s, 0=unlimited (env:
                        GLOBAL_DOWNLOAD_LIMIT)
  --upload-slots N      Max upload slots per torrent (default: 8, env:
                        UPLOAD_SLOTS)
  --connections N       Max connections limit (default: 500, env:
                        CONNECTIONS_LIMIT)
  --listen IFACE:PORT   libtorrent listen interface (default: 127.0.0.1:6881,
                        env: LISTEN_INTERFACES)
  --webui-user USER     WebUI username (default: admin, env: WEBUI_USER)
  --webui-pass PASS     WebUI password (default: 6-char random, env:
                        WEBUI_PASS)
  --super-seeding       Enable global super-seeding mode on startup (env:
                        SUPER_SEEDING)
  --debug               Enable full debug logging incl. HTTP requests (env:
                        DEBUG)

API Endpoints (quick reference):
  POST   /add/magnet            Add torrent via magnet link
  POST   /add/file              Add torrent via .torrent file upload
  POST   /add/url               Add torrent via direct .torrent URL
  GET    /list                  List all torrents
  GET    /status/<hash>         Status of a single torrent
  GET    /files/<hash>          List files in torrent with priorities
  POST   /files/<hash>          Set per-file priorities
  GET    /monitor               Live streaming monitor
  POST   /pause/<hash>          Pause torrent
  POST   /stop/<hash>           Stop torrent (pause + save resume)
  POST   /stop/file             Stop torrent identified by .torrent file
  POST   /resume/<hash>         Resume torrent
  DELETE /remove/<hash>         Remove torrent (add ?delete_files=1 to wipe data)
  DELETE /remove/file           Remove torrent identified by .torrent file
  POST   /limit/<hash>          Set per-torrent speed limits (JSON body)
  POST   /limit/global          Set global speed limits
  POST   /recheck/<hash>        Force recheck
  POST   /announce/<hash>       Force re-announce
  GET    /trackers/<hash>       List trackers
  POST   /create                Create .torrent from local path
  POST   /seed                  Seed local data with .torrent file
  GET    /magnet/<hash>         Get magnet URI
  POST   /save                  Persist resume data for all torrents
  GET    /health                Health check
  POST   /super_seed/<hash>     Toggle super-seeding mode for a torrent
  POST   /super_seed/global     Set global super-seeding mode on/off
  GET    /webui                 Web UI (browser)
  GET    /webui/login           Login page
  POST   /webui/login           Authenticate
  POST   /webui/logout          Logout
  GET    /doc                   API documentation

Examples:
  LimeTorrent --host 127.0.0.1 --port 8080
  LimeTorrent --upload-limit 1048576 --download-limit 5242880
  LimeTorrent --webui-user admin --webui-pass mysecret
  curl -X POST http://127.0.0.1:5000/add/magnet -d '{"magnet":"magnet:?xt=..."}'
  curl -X DELETE http://127.0.0.1:5000/remove/<hash>?delete_files=1
  curl -X POST http://127.0.0.1:5000/stop/<hash>
  curl -X POST -F torrent=@file.torrent http://127.0.0.1:5000/stop/file
  curl -X DELETE -F torrent=@file.torrent http://127.0.0.1:5000/remove/file
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
LimeTorrent --host 0.0.0.0 --port 8080

# Set credentials and download directory
LimeTorrent --webui-user admin --webui-pass mysecret --download-dir /data/torrents

# Limit upload to 2 MB/s, download to 10 MB/s
LimeTorrent --upload-limit 2097152 --download-limit 10485760

# Enable super-seeding and verbose logging
LimeTorrent --super-seeding --debug

# Using environment variables
HOST=0.0.0.0 PORT=8080 WEBUI_PASS=secret LimeTorrent
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
sudo nano /etc/systemd/system/LimeTorrent.service
```

```ini
[Unit]
Description=LimeTorrent - Torrent Server
After=network.target

[Service]
Type=simple
User=YOUR_USERNAME
WorkingDirectory=/opt/LimeTorrent
ExecStart=/opt/LimeTorrent/LimeTorrent \
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
sudo systemctl enable LimeTorrent
sudo systemctl start LimeTorrent

# Check status
sudo systemctl status LimeTorrent

# View logs
sudo journalctl -u LimeTorrent -f
```

**3. (Optional) Open the firewall port**

```bash
sudo ufw allow 5000/tcp
```

---

## FAQ

**Can I run LimeTorrent on Windows?**  
Yes (tested on windows 11 23h2).

**Can I run multiple instances?**  
Yes — use different `--port`, `--download-dir`, and `--resume-dir` values for each instance.

---

<div align="center">

Built with ❤️ using [libtorrent](https://libtorrent.org) + [Flask](https://flask.palletsprojects.com)

[GitHub](https://github.com/lurma813/LimeTorrent) · [Issues](https://github.com/lurma813/LimeTorrent/issues) · [Releases](https://github.com/lurma813/LimeTorrent/releases)

</div>
