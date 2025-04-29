# FortiClinetForMac

# 💼 مستند راه‌اندازی VPN با SwiftBar و OpenFortiVPN روی macOS

## 🧰 پیش‌نیازها
- macOS Ventura یا Sonoma
- دسترسی به ترمینال با اکانت ادمین (برای `sudo`)
- اتصال به Fortinet VPN با تنظیمات معتبر

---

## 1️⃣ نصب Homebrew (اگر نصب نیست)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

افزودن مسیر Brew به محیط ترمینال:
```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
source ~/.zshrc  # یا source ~/.bash_profile
```

---

## 2️⃣ نصب OpenFortiVPN
```bash
brew install openfortivpn
```

ساخت فایل کانفیگ:
```bash
nano ~/vpn.conf
```

نمونه:
```
host = vpn.example.com
port = 443
username = your_username
password = your_password
```

> توصیه: از ذخیره پسورد در فایل پرهیز کن یا با `chmod 600` ایمنش کن.

---

## 3️⃣ نصب SwiftBar

- دریافت از: [https://swiftbar.app](https://swiftbar.app)
- نصب اپلیکیشن
- تنظیم مسیر پلاگین‌ها، مثلاً: `~/SwiftBar/Plugins`

---

## 4️⃣ ساخت پلاگین SwiftBar برای VPN

### ساخت فایل:

```bash
mkdir -p ~/SwiftBar/Plugins
nano ~/SwiftBar/Plugins/vpn_status.5s.sh
```

### محتوای فایل:

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
  echo "⛔ قطع VPN | bash='$0' param1=disconnect terminal=false refresh=true color=red"
  echo "✅ VPN وصل است: $duration_str | color=green"
else
  echo "⚡ وصل VPN | bash='$0' param1=connect terminal=false refresh=true color=green"
  echo "❌ VPN قطع است | color=red"
fi

echo "---"
echo "🔄 رفرش | refresh=true"
\`\`\`

### اجرای دستور دسترسی:

```bash
chmod +x ~/SwiftBar/Plugins/vpn_status.5s.sh
```

---

## 5️⃣ اجرای بدون نیاز به پسورد (اختیاری)

```bash
sudo visudo
```

افزودن خط زیر به انتهای فایل:

```
your_username ALL=(ALL) NOPASSWD: /usr/local/bin/openfortivpn, /usr/bin/killall openfortivpn
```

---

## ✅ تمام شد!

SwiftBar وضعیت VPN را هر ۵ ثانیه بررسی می‌کند:

- آیکون سبز/قرمز در نوار وضعیت
- زمان اتصال نمایش داده می‌شود
- دکمه برای وصل یا قطع با یک کلیک

---

## 📌 نکات تکمیلی:

- اضافه کردن نوتیفیکیشن macOS:
```bash
osascript -e 'display notification "VPN وصل شد" with title "VPN Status"'
```

- بررسی مسیر اجرای openfortivpn:
```bash
which openfortivpn
```

---

تهیه‌شده با ❤️
