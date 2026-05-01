# 🍋 LimeTorrent (Binary Release)

LimeTorrent is a **REST API-based torrent manager** compiled into a standalone **Linux (Ubuntu) binary**.

No Python or external dependencies required — just run the binary.

---

# 🚀 Features

- Add torrents via:
  - Magnet link
  - `.torrent` file
  - Direct URL
- Real-time monitoring
- Pause / Resume / Stop
- Remove torrent (optional file deletion)
- Global & per-torrent speed limits
- Persistent resume data (no reset on restart)
- Built-in Web UI
- Authentication + view-only mode
- Super-seeding (global & per torrent)
- Create & seed torrents
- Recheck & reannounce
- All-time statistics tracking

---

# 📦 Installation (Binary)

```bash
git clone https://github.com/lurma813/LimeTorrent
sudo cp LimeTorrent/LimeTorrent /usr/local/bin
sudo chmod u+rwx /usr/local/bin/LimeTorrent
rm -r LimeTorrent/
```

---

# ⚙️ CLI Options

| Flag | Description |
|------|------------|
| --host | Bind address (default: 127.0.0.1) |
| --port | Server port (default: 5000) |
| --download-dir | Download directory |
| --torrent-dir | Output directory for created torrents |
| --resume-dir | Resume data directory |
| --upload-limit | Global upload limit (bytes/sec) |
| --download-limit | Global download limit |
| --connections | Max connections |
| --upload-slots | Upload slots per torrent |
| --listen | Libtorrent interface |
| --webui-user | WebUI username |
| --webui-pass | WebUI password |
| --super-seeding | Enable super seeding |
| --debug | Enable verbose logging |

---

# 🌐 Web UI

Access:

```
http://localhost:5000/webui
```

Default credentials:
- Username: `admin`
- Password: auto-generated (printed in console)

---

# 📁 Default Paths

- Downloads: `~/Downloads`
- Resume data: `~/.limetorrent/resume`
- Created torrents: `~/.limetorrent/created`

---

# 🔌 REST API (Full Examples)

## ➕ ADD TORRENT

### Magnet

```bash
curl -X POST http://localhost:5000/add/magnet \
-H "Content-Type: application/json" \
-d '{"magnet":"magnet:?xt=urn:btih:..."}'
```

### File

```bash
curl -X POST http://localhost:5000/add/file \
-F "torrent=@file.torrent"
```

### URL

```bash
curl -X POST http://localhost:5000/add/url \
-H "Content-Type: application/json" \
-d '{"url":"https://example.com/file.torrent"}'
```

---

## 📊 LIST & STATUS

### List all torrents

```bash
curl http://localhost:5000/list
```

### Get torrent status

```bash
curl http://localhost:5000/status/<hash>
```

---

## 📁 FILE MANAGEMENT

### List files

```bash
curl http://localhost:5000/files/<hash>
```

### Set file priorities

```bash
curl -X POST http://localhost:5000/files/<hash> \
-H "Content-Type: application/json" \
-d '{"priorities":[1,1,0,0]}'
```

---

## 🎛️ CONTROL

### Pause

```bash
curl -X POST http://localhost:5000/pause/<hash>
```

### Resume

```bash
curl -X POST http://localhost:5000/resume/<hash>
```

### Stop

```bash
curl -X POST http://localhost:5000/stop/<hash>
```

---

## ❌ REMOVE

### Remove torrent

```bash
curl -X DELETE http://localhost:5000/remove/<hash>
```

### Remove + delete files

```bash
curl -X DELETE "http://localhost:5000/remove/<hash>?delete_files=1"
```

---

## ⚡ SPEED LIMIT

### Per torrent

```bash
curl -X POST http://localhost:5000/limit/<hash> \
-H "Content-Type: application/json" \
-d '{"download":1048576,"upload":524288}'
```

### Global

```bash
curl -X POST http://localhost:5000/limit/global \
-H "Content-Type: application/json" \
-d '{"download":0,"upload":0}'
```

---

## 🔧 TOOLS

### Recheck

```bash
curl -X POST http://localhost:5000/recheck/<hash>
```

### Reannounce

```bash
curl -X POST http://localhost:5000/announce/<hash>
```

### Trackers

```bash
curl http://localhost:5000/trackers/<hash>
```

---

## 🌱 SEEDING

### Seed existing data

```bash
curl -X POST http://localhost:5000/seed \
-F "torrent=@file.torrent"
```

### Enable super seeding (per torrent)

```bash
curl -X POST http://localhost:5000/super_seed/<hash>
```

### Enable super seeding (global)

```bash
curl -X POST http://localhost:5000/super_seed/global \
-H "Content-Type: application/json" \
-d '{"enabled":true}'
```

---

## 💾 SAVE

```bash
curl -X POST http://localhost:5000/save
```

---

## ❤️ HEALTH CHECK

```bash
curl http://localhost:5000/health
```

---

# 🔄 Resume Behavior

- Torrents automatically restore on startup
- No restart from 0%
- Recheck only when needed

---

# 📈 Statistics

- Session upload/download
- All-time stats
- Stored in `_stats.json`

---

# 🔐 Security

- Session-based authentication
- Lockout after 3 failed attempts
- Anonymous users are view-only

---

# 🧠 Use Cases

- Seedbox
- NAS server
- VPS automation
- Remote torrent server
- Headless environments

---

# ⚠️ Disclaimer

Use responsibly and comply with local laws.

---
