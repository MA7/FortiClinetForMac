# 💼 VPN Setup Guide with SwiftBar and OpenFortiVPN on macOS

## 🧰 Prerequisites
- macOS Ventura or Sonoma
- Terminal access with admin privileges (`sudo`)
- Valid Fortinet VPN credentials

---

## 1️⃣ Install Homebrew (if not installed)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Add Brew to your shell environment:
```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
source ~/.zshrc  # or source ~/.bash_profile
```

---

## 2️⃣ Install OpenFortiVPN
```bash
brew install openfortivpn
```

Create a config file:
```bash
nano ~/vpn.conf
```

Sample content:
```
host = vpn.example.com
port = 443
username = your_username
password = your_password
```

> Tip: Avoid storing the password in plaintext, or secure the file with `chmod 600`.

---

## 3️⃣ Install SwiftBar

- Download from: [https://swiftbar.app](https://swiftbar.app)
- Install and run the app
- Set the plugin directory, e.g., `~/SwiftBar/Plugins`

---

## 4️⃣ Create SwiftBar Plugin to Control VPN

### Create the file:

```bash
mkdir -p ~/SwiftBar/Plugins
nano ~/SwiftBar/Plugins/vpn_status.5s.sh
```

### Paste the following content:

\`\`\`bash
#!/bin/bash
refresh=true

START_FILE='/tmp/vpn_start_time'
CONNECT_CMD='sudo openfortivpn &'
DISCONNECT_CMD='sudo killall openfortivpn'

if pgrep -x openfortivpn > /dev/null; then
  STATUS="connected"
else
  STATUS="disconnected"
fi

case "$1" in
  connect)
    eval "$CONNECT_CMD"
    date +%s > "$START_FILE"
    exit
    ;;
  disconnect)
    eval "$DISCONNECT_CMD"
    rm -f "$START_FILE"
    exit
    ;;
esac

if [ "$STATUS" = "connected" ]; then
  if [ -f "$START_FILE" ]; then
    start_time=$(cat "$START_FILE")
    now=$(date +%s)
    duration=$((now - start_time))

    hours=$((duration / 3600))
    minutes=$(((duration % 3600) / 60))
    seconds=$((duration % 60))

    duration_str=""
    [ "$hours" -gt 0 ] && duration_str="${duration_str}${hours}h "
    [ "$minutes" -gt 0 ] && duration_str="${duration_str}${minutes}m "
    duration_str="${duration_str}${seconds}s"

    echo "🔒 $duration_str | color=green"
  else
    echo "🔒 ? | color=green"
  fi
else
  echo "🔓 | color=red"
fi

echo "---"

if [ "$STATUS" = "connected" ]; then
  echo "⛔ Disconnect VPN | bash='$0' param1=disconnect terminal=false refresh=true color=red"
  echo "✅ VPN Connected: $duration_str | color=green"
else
  echo "⚡ Connect VPN | bash='$0' param1=connect terminal=false refresh=true color=green"
  echo "❌ VPN Disconnected | color=red"
fi

echo "---"
echo "🔄 Refresh | refresh=true"
\`\`\`

### Make it executable:

```bash
chmod +x ~/SwiftBar/Plugins/vpn_status.5s.sh
```

---

## 5️⃣ Enable Passwordless Execution (Optional)

```bash
sudo visudo
```

Add the following line at the end:

```
your_username ALL=(ALL) NOPASSWD: /usr/local/bin/openfortivpn, /usr/bin/killall openfortivpn
```

---

## ✅ All Done!

SwiftBar will now monitor your VPN status every 5 seconds:

- Green/red icon in the menu bar
- Session duration is shown
- One-click connect/disconnect

---

## 📌 Tips

- Add macOS notification like this:
```bash
osascript -e 'display notification "VPN connected" with title "VPN Status"'
```

- Check path for openfortivpn:
```bash
which openfortivpn
```

---

Created with ❤️
