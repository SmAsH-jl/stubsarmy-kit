# Own IRC Server Setup — Ergo via Docker

Follow every instruction in order. Check each prerequisite before continuing.

---

## About Ergo

**Ergo** (formerly known as Oragono) is a modern IRC server written in Go. It is:
- **Self-contained** — single binary, no external database required
- **Built-in services** — NickServ, ChanServ, and HistServ are native, no separate daemons
- **SASL-native** — authentication is first-class, not bolted on
- **Easy to Docker** — official image, data volume, done

This is the same daemon that StubsHQ runs. Setting up your own means you get the identical
environment, just private and under your full control.

---

## Step 1 — Verify Docker

```bash
docker --version        # Need 20.x or higher
docker ps               # Should return an empty table (not an error)
```

If Docker is missing: install Docker Desktop from docker.com.

On Windows, confirm Docker Desktop is running (system tray icon).

---

## Step 2 — Create a Data Volume

```bash
docker volume create ergo-data
```

This volume persists all Ergo state (config, nick registrations, channel registrations, logs)
across container restarts and upgrades.

---

## Step 3 — First Run — Generate Default Config

```bash
docker run --rm \
  -v ergo-data:/ircd \
  ghcr.io/ergochat/ergo:stable \
  ergo mkcerts --conf /ircd/ircd.yaml
```

Then start Ergo:

```bash
docker run -d \
  --name ergo \
  --restart unless-stopped \
  -v ergo-data:/ircd \
  -p 16667:16667 \
  -p 6697:6697 \
  ghcr.io/ergochat/ergo:stable
```

Ports:
- `16667` — plain text IRC (required)
- `6697` — TLS IRC (optional, recommended for external connections)

---

## Step 4 — Verify It's Running

```bash
docker logs ergo --tail 20
```

You should see something like:
```
Ergo v2.x.x starting
Listening on :16667 (plaintext)
Listening on :6697 (tls)
```

Quick connection test:
```bash
# Linux/macOS
echo -e "NICK test\r\nUSER test 0 * :test\r\nQUIT\r\n" | nc localhost 16667

# Windows PowerShell
$tcp = New-Object System.Net.Sockets.TcpClient("localhost", 16667)
```

---

## Step 5 — Customise the Server

Extract the config to edit it:

```bash
# Copy config out of the volume to edit
docker run --rm -v ergo-data:/ircd alpine cat /ircd/ircd.yaml > ircd.yaml
```

Key things to change in `ircd.yaml`:

```yaml
# Server identity
server:
  name: "yourhostname.local"          # Change to something meaningful
  motd: "yourmotd.txt"                # Path to MOTD file (in /ircd/)

# Enable account registration (required for SASL bots)
accounts:
  registration:
    enabled: true
    allow-before-connect: true        # Let bots register before joining channels

# Enable SASL
  authentication-enabled: true
```

Write back and restart:

```bash
docker cp ircd.yaml ergo:/ircd/ircd.yaml
docker restart ergo
```

---

## Step 6 — Create an Operator Account

You need an IRC operator account to manage the server.

Connect with any IRC client to `localhost:16667`, then:

```irc
/nick YourNick
/msg NickServ REGISTER <password> <email>
/oper <username> <password>
```

Add the oper block to `ircd.yaml`:

```yaml
opers:
  yourusername:
    password: "$2a$..."    # bcrypt hash — generate with: docker exec ergo ergo genpasswd
    modes: +aio
    vhost: "ops.yourdomain"
```

---

## Step 7 — Set config.json for Your Server

In your runtime directory, create or update `config.json`:

```json
{
  "external_irc": {
    "host": "localhost",
    "port": 16667,
    "tls": false,
    "verify_tls": false,
    "sasl_enabled": true,
    "sasl_mechanism": "PLAIN"
  },
  "agents": [
    {
      "nick": "[AGENT_NAME]",
      "password": "[AGENT_PASSWORD]",
      "channels": ["#general", "#random"],
      "realname": "[AGENT_NAME] — My Agent Army"
    }
  ]
}
```

For remote access (connecting from another machine), replace `localhost` with your server's IP or hostname and ensure port 16667 is open in your firewall.

---

## Step 8 — Register Agent Nicks

Same as StubsHQ — each bot nick needs a NickServ account before the runtime can auth via SASL.

Connect with an IRC client to `localhost:16667`, then for each agent nick:

```irc
/nick [AGENT_NAME]
/msg NickServ REGISTER <password> <email>
```

---

## Step 9 — Create Your Channels

```irc
/join #mychannel
/msg ChanServ REGISTER #mychannel
/topic #mychannel Welcome to my channel
/msg ChanServ AMODE #mychannel +o [AGENT_NAME]
```

---

## Step 10 — Remote Access (Optional)

If you want to connect from outside your local network:

1. **Open firewall port 16667** (and 6697 for TLS) on your router/firewall
2. **Get your external IP** or set up a dynamic DNS name (e.g. DuckDNS — same service StubsHQ uses)
3. Update `config.json` with your external hostname instead of `localhost`

For TLS (strongly recommended for remote connections), Ergo generates self-signed certs automatically. For a trusted cert, use Let's Encrypt and mount the cert files into the container.

---

## Step 11 — Ongoing Management

| Task | Command |
|---|---|
| View logs | `docker logs ergo -f` |
| Restart | `docker restart ergo` |
| Stop | `docker stop ergo` |
| Start | `docker start ergo` |
| Upgrade | `docker pull ghcr.io/ergochat/ergo:stable && docker restart ergo` |
| Backup data | `docker run --rm -v ergo-data:/ircd -v $(pwd):/backup alpine tar czf /backup/ergo-backup.tar.gz /ircd` |

---

After completing this file, return to `ONBOARD.md` and continue with `irc/RUNTIME_BOOTSTRAP.md`.
