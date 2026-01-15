### Goal

On Debian (MacBook Air 2013), **close the lid to turn off the backlight** while keeping the server running (no suspend).

### 1) Prevent suspend on lid close (systemd-logind)

Edit:

```bash
sudo vi /etc/systemd/logind.conf
```

Set:

```ini
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

Apply:

```bash
sudo systemctl restart systemd-logind
```

### 2) Install and enable ACPI daemon (acpid)

```bash
sudo apt update
sudo apt install -y acpid
sudo systemctl enable --now acpid
```

Optional test to confirm lid events exist:

```bash
sudo acpi_listen
# Expected:
# button/lid LID open
# button/lid LID close
```

### 3) Identify the backlight device

```bash
ls /sys/class/backlight
# Example output:
# acpi_video0
```

### 4) Create ACPI event rules for lid open/close

**Lid close rule**

```bash
sudo tee /etc/acpi/events/lid-close >/dev/null <<'EOF'
event=button/lid LID close
action=/etc/acpi/lid-backlight.sh close
EOF
```

**Lid open rule**

```bash
sudo tee /etc/acpi/events/lid-open >/dev/null <<'EOF'
event=button/lid LID open
action=/etc/acpi/lid-backlight.sh open
EOF
```

### 5) Create the backlight control script

```bash
sudo tee /etc/acpi/lid-backlight.sh >/dev/null <<'EOF'
#!/bin/sh
set -eu

BL="/sys/class/backlight/acpi_video0"
LOG="/tmp/lid-backlight.log"

echo "$(date) action=$1" >> "$LOG"
[ -d "$BL" ] || exit 0

if [ "$1" = "close" ]; then
  # save current brightness
  cat "$BL/brightness" > /run/last_brightness 2>/dev/null || true

  # prefer bl_power if present
  if [ -f "$BL/bl_power" ]; then
    echo 4 > "$BL/bl_power" 2>/dev/null || true
    echo "$(date) set bl_power=4" >> "$LOG"
  fi

  # also try brightness=0 (some drivers ignore it)
  echo 0 > "$BL/brightness" 2>/dev/null || echo 1 > "$BL/brightness" 2>/dev/null || true
  echo "$(date) set brightness low" >> "$LOG"

elif [ "$1" = "open" ]; then
  if [ -f "$BL/bl_power" ]; then
    echo 0 > "$BL/bl_power" 2>/dev/null || true
    echo "$(date) set bl_power=0" >> "$LOG"
  fi

  if [ -f /run/last_brightness ]; then
    cat /run/last_brightness > "$BL/brightness" 2>/dev/null || true
    echo "$(date) restored brightness" >> "$LOG"
  else
    # fallback to a reasonable value (half of max)
    MAX="$(cat "$BL/max_brightness" 2>/dev/null || echo 10)"
    echo $((MAX/2)) > "$BL/brightness" 2>/dev/null || true
    echo "$(date) fallback brightness" >> "$LOG"
  fi
fi
EOF

sudo chmod +x /etc/acpi/lid-backlight.sh
sudo systemctl restart acpid
```

### 7) Verification

- Close lid → backlight off, server continues running
    
- Open lid → brightness restored
    

Debug/logging:

```bash
cat /tmp/lid-backlight.log
journalctl -u acpid --no-pager | tail -n 50
```

---
## Troubleshooting

### 1) Closing the lid still suspends the system

**Symptoms**

- Services stop responding when lid is closed.
    
- `ping`/SSH fails after lid close, and resumes only after opening.
    

**Fix**  
Verify `systemd-logind` is configured to ignore lid close:

```bash
sudo grep -E 'HandleLidSwitch' /etc/systemd/logind.conf
```

Ensure these are set (not commented out with `#`):

```ini
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

Restart logind:

```bash
sudo systemctl restart systemd-logind
```

Also check if any desktop/power manager is overriding lid behavior (rare on a headless install).
### 2) Lid open/close events are not detected

**Symptoms**

- Script never runs; `/tmp/lid-backlight.log` doesn’t change.
    
- No lid events appear.
    

**Checks**  
Confirm the OS sees lid events:

```bash
sudo acpi_listen
```

Expected output when closing/opening:

```
button/lid LID close
button/lid LID open
```

If you see nothing:

- Ensure `acpid` is installed/running:
    
    ```bash
    systemctl status acpid --no-pager
    ```
    
- Check kernel ACPI is enabled (usually is). If you booted with unusual ACPI parameters, revert them.
    
### 3) `acpid` is running, but the script is not executed

**Symptoms**

- `acpi_listen` shows events, but `/tmp/lid-backlight.log` never updates.
    

**Fix**

1. Confirm the event files exist:
    

```bash
ls -l /etc/acpi/events/
```

2. Validate exact event match (must match your acpi_listen output):
    

```bash
cat /etc/acpi/events/lid-close
cat /etc/acpi/events/lid-open
```

They should match:

- `event=button/lid LID close`
    
- `event=button/lid LID open`
    

3. Confirm script is executable:
    

```bash
ls -l /etc/acpi/lid-backlight.sh
```

If needed:

```bash
sudo chmod +x /etc/acpi/lid-backlight.sh
sudo systemctl restart acpid
```

4. Inspect logs:
    

```bash
journalctl -u acpid --no-pager | tail -n 80
```
### 4) Script runs but screen/backlight doesn’t turn off

**Symptoms**

- `/tmp/lid-backlight.log` shows “close/open” actions, but display stays lit.
    

**Fix**  
Most common cause: wrong backlight device or driver ignores brightness=0.

1. Confirm available backlight devices:
    

```bash
ls /sys/class/backlight
```

2. Verify your chosen one exists:
    

```bash
ls -l /sys/class/backlight/acpi_video0/
```

3. Test manual control:
    

```bash
cat /sys/class/backlight/acpi_video0/brightness
echo 0 | sudo tee /sys/class/backlight/acpi_video0/brightness
```

If brightness changes but screen stays on, try `bl_power` (more reliable when available):

```bash
cat /sys/class/backlight/acpi_video0/bl_power 2>/dev/null || echo "no bl_power"
echo 4 | sudo tee /sys/class/backlight/acpi_video0/bl_power   # off
echo 0 | sudo tee /sys/class/backlight/acpi_video0/bl_power   # on
```

If you don’t have `bl_power` or it doesn’t work, you may be using the wrong backlight interface (some systems use `intel_backlight`).
### 5) Screen turns off on lid close, but brightness doesn’t restore

**Symptoms**

- Lid close turns off backlight correctly.
    
- Lid open keeps screen very dim or black.
    

**Fix**

1. Ensure the “open” event is firing:
    

```bash
tail -n 50 /tmp/lid-backlight.log
```

2. Confirm brightness was saved:
    

```bash
ls -l /run/last_brightness
cat /run/last_brightness
```

3. If `/run/last_brightness` is missing, set a safe default in the script (already included fallback logic), or hardcode a known good value.
    
### 6) `/proc/acpi/button/lid/*/state` always shows “closed”

**Symptoms**

- `cat /proc/acpi/button/lid/*/state` returns `closed` even when lid is open.
    

**Why**  
On some hardware/BIOS/ACPI implementations this state file can be unreliable.

**Fix**  
Do **not** rely on `/proc/.../state`. Use the lid events from `acpi_listen` and the explicit acpid rules (open/close) as the source of truth.
### 7) Backlight toggles but the Apple logo is still visible

**Explanation**  
That’s normal: the Apple logo is physically printed / reflective. The real check is whether the LCD **backlight glow** is off.

---

If you want, paste your final files (`/etc/acpi/events/lid-open`, `lid-close`, and the script) and I’ll format them neatly as a “Config Listing” section for your README.