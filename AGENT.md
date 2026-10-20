# 🛰️ ZiuroDB Local Agent & Background Daemon Guide

The **ZiuroDB Local Agent** is a secure, lightweight tunnel that connects the ZiuroDB cloud/web dashboard directly to databases running on your private network or `localhost` (MongoDB, MySQL, PostgreSQL, Redis) with zero open firewall ports and zero manual configuration.

---

## 🛠️ How it Works

The agent uses an **Outbound Encrypted WebSocket Tunnel** (`wss://` over TLS) to bridge your local environment and your ZiuroDB browser session:

1. **Authentication**: The agent authenticates to the ZiuroDB server using a cryptographically generated `agentId` and `agentToken`.
2. **Persistent Tunnel**: Once connected, an encrypted WebSocket connection stays active in the background.
3. **Local Command Execution**: When you query or explore databases in the web dashboard, requests are routed through the tunnel and executed locally in process memory against MongoDB (`27017`), MySQL (`3306`), or PostgreSQL (`5432`).
4. **100% Privacy & Security**: Your database passwords and records never persist on cloud servers; data is only passed in memory during active query execution.

---

## 🚀 Choose Your Setup Method

ZiuroDB provides 3 simple ways to link your local databases:

### Option 1: ⚡ 1-Click Desktop App (Recommended — Zero Terminal)

The easiest method for non-technical users. Requires **no Node.js** and **no terminal commands**.

1. **Download & Install**:
   - In ZiuroDB, navigate to **Connections → Download Local Agent**.
   - Select your operating system:
     - **macOS**: `.dmg` (Apple Silicon M1/M2/M3/M4 & Intel)
     - **Windows**: `.exe` (Windows 10 / 11 64-bit)
     - **Linux**: `.AppImage` or `.deb`
   - Open the application once.
2. **Auto-Pair with Browser**:
   - In your browser on ZiuroDB, click **"Auto-Link Local Agent"**.
   - **If the Desktop App is running**: ZiuroDB auto-detects the background daemon on `127.0.0.1:19876` and connects **instantly in <500ms (0 clicks)**!
   - **If the Desktop App is closed**: Click **"Open in ZiuroDB Desktop App"** in the popup to launch via deep link (`ziurodb://pair-agent`) and pair automatically.
3. **Background Tray**:
   - The app stays quietly in your system tray / menu bar with minimal CPU/RAM usage.

---

### Option 2: 📦 Standalone 1-Click ZIP (No Node.js Required)

If you prefer a lightweight portable folder without installing the desktop app:

1. **Download Pre-Configured Package**:
   - On the **Connections** page, click **"Auto-Link Local Agent"**.
   - Switch to the **"📦 1-Click ZIP"** tab and click **"Download Configured Agent (.zip)"**.
2. **Extract & Double-Click Launcher**:
   - Extract the downloaded ZIP file.
   - Double-click the launcher for your system:
     - **macOS**: Double-click `start-mac.command`
     - **Windows**: Double-click `start-windows.bat`
     - **Linux**: Run `./start.sh`
3. **Online Status**:
   - The runner executes the pre-bundled JavaScript runtime (`index.js`). The agent immediately displays `✅ Tunnel Online` and pairs with your browser session.

---

### Option 3: 💻 Developer Terminal CLI (`npx`)

If you have **Node.js (v18+)** installed and prefer terminal commands:

1. In ZiuroDB on the **Connections** page, click **"Auto-Link Local Agent"**.
2. Switch to the **"💻 Terminal CLI"** tab.
3. Copy and run the generated one-liner in your terminal:
   ```bash
   npx ziurodb tunnel --id=agent-xxxxxx --token=pat-xxxxxx --url=http://localhost:5001
   ```

---

## 🗄️ Auto-Discovered Databases

Once your agent connects, ZiuroDB automatically links default local database endpoints:
- 🍃 **MongoDB**: `127.0.0.1:27017`
- 🐬 **MySQL**: `127.0.0.1:3306`
- 🐘 **PostgreSQL**: `127.0.0.1:5432`
- 🔴 **Redis**: `127.0.0.1:6379`

You can also add custom database ports and credentials under **Connections → Add Connection** and select your active Local Agent.

---

## 🌙 Running in Background (Production Linux Servers)

For headless production Linux servers or remote VPCs, use **PM2** to keep the agent running permanently across server reboots:

```bash
# Install PM2 globally
npm install -g pm2

# Start agent in background
pm2 start npm --name ziurodb-agent -- start

# Configure system startup
pm2 startup
pm2 save
```

### Managing PM2 Background Process
- **Status**: `pm2 status`
- **View Live Logs**: `pm2 logs ziurodb-agent`
- **Restart**: `pm2 restart ziurodb-agent`
- **Stop**: `pm2 stop ziurodb-agent`

---

## 🔒 Security Summary

| Feature | Guarantee |
| :--- | :--- |
| **Connection Direction** | Outbound only (no open ports or firewall changes) |
| **Encryption** | TLS / WSS encrypted WebSocket tunnel |
| **Authentication** | Cryptographic Personal Access Tokens (`pat-...`) |
| **Data Privacy** | 100% local memory execution; 0 credentials or rows stored on cloud servers |

