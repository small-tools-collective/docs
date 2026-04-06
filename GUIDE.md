# //small tools// — guide

Digital tools for journalists and activists under threat.  
Two tools. Simple to use. No technical knowledge required.

---

## What this is

### Version A — soft clean
Press **Ctrl+Alt+S** at any time to:
- Close Telegram, Signal, Discord, Slack
- Clear Firefox and Chrome history, sessions, and cookies
- Wipe the clipboard
- Clear recently opened files
- Lock the screen

Nothing is permanently deleted. Use this whenever you feel unsafe or before handing over your device.

### Version B — panic wipe
Insert your panic USB drive to arm the system.  
Then press **Ctrl+Alt+Del** to trigger the wipe.

What happens:
- Shell history, SSH keys, GPG keys deleted
- Documents, Downloads, Desktop folders deleted
- All browser data deleted
- Messenger data deleted
- A fake kernel panic screen appears — the computer looks broken to anyone watching

The screen stays frozen. To anyone looking, the machine crashed.

---

## How it works

```
VERSION A

  Ctrl+Alt+S
       │
       ├── close messengers
       ├── clear browser history
       ├── wipe clipboard
       ├── clear recent files
       └── lock screen ✓


VERSION B

  USB inserted ──→ ARMED ──→ Ctrl+Alt+Del ──→ wipe ──→ kernel panic screen
                     │
             USB removed ──→ DISARMED (nothing happens)
```

The wipe only triggers if **both** conditions are met: USB inserted AND Ctrl+Alt+Del pressed. Removing the USB cancels everything.

---

## Install

**Requires:** Linux (X11), bash, systemd

**One command:**

```bash
bash <(curl -fsSL https://raw.githubusercontent.com/small-tools-collective/scripts/main/install.sh)
```

Or if you have the files locally:

```bash
bash install.sh
```

The installer will:
1. Check and list any missing dependencies
2. Install the background daemon (for wipe signal handling)
3. Set the keyboard shortcuts
4. Walk you through USB detection step by step (for Version B)

**The wipe runs in simulation mode by default.** No files are actually deleted until you enable real mode. Test everything first.

---

## Keyboard shortcuts

| Keys | Action |
|---|---|
| `Ctrl+Alt+S` | Soft clean + lock screen |
| `Ctrl+Alt+Del` | Confirm wipe (only works if USB is inserted) |
| `Ctrl+Alt+A` | Disarm (cancel wipe without removing USB) |

---

## Setting up your panic USB

You need one dedicated USB drive for Version B. Any drive works.

**During install:**
1. When prompted, make sure the USB is NOT plugged in
2. Then plug it in and press Enter
3. The installer detects it automatically and configures the rule

That USB is now your kill switch. Keep it on you.

**The USB does NOT store any data.** It's identified only by its hardware ID. The USB is not destroyed or modified when used.

---

## Your first test

After install:

**Test Version A (safe, always works):**
1. Open a browser, visit a few pages
2. Press `Ctrl+Alt+S`
3. Screen should lock within a few seconds
4. Check that browser history is cleared

**Test Version B (simulation — no real deletion):**
1. Insert your panic USB
2. Wait a moment (system is now armed)
3. Press `Ctrl+Alt+Del`
4. A kernel panic screen should fill the screen
5. The log at `logs/wipe.log` shows what *would* have been deleted

**Enable real wipe** only when you're confident it works. Edit `bin/wipe-execute.sh` and change:
```bash
WIPE_MODE="${WIPE_MODE:-simulation}"
```
to:
```bash
WIPE_MODE="${WIPE_MODE:-real}"
```

---

## Check system status

```bash
bash bin/status.sh
```

Shows whether the system is armed, daemon is running, and recent log activity.

---

## Troubleshooting

**Keyboard shortcut doesn't work**

```bash
# Check that xbindkeys is running
pgrep xbindkeys

# If not, start it
xbindkeys

# To make it start automatically, add 'xbindkeys &' to your ~/.xprofile
```

**Daemon isn't running**

```bash
systemctl --user status wipe-daemon.service

# Restart it
systemctl --user restart wipe-daemon.service

# View logs
journalctl --user -u wipe-daemon.service -f
```

**USB isn't triggering the arm**

```bash
# Watch udev events — plug your USB in while this is running
udevadm monitor --udev --subsystem-match=usb

# Check the rule is installed
ls /etc/udev/rules.d/ | grep panic

# Check your USB IDs match the rule
lsusb
```

**Screen didn't lock after soft clean**

The screen locker is auto-detected. If it's not found:
```bash
# Install one — loginctl is usually enough on systemd systems
sudo apt install gnome-screensaver
# or
sudo apt install i3lock
```

---

## What gets wiped (Version B)

| Category | What's deleted |
|---|---|
| Shell | `.bash_history`, `.zsh_history` |
| SSH | `known_hosts` |
| GPG | `trustdb.gpg`, key rings |
| Files | `Documents/`, `Downloads/`, `Desktop/` |
| Firefox | history, sessions, cookies, cache |
| Chrome/Brave/Edge | history, sessions, cache |
| Messengers | Telegram, Signal, Discord (app data) |
| Misc | clipboard, thumbnail cache, system journal |

You can customize what gets deleted by editing the arrays in `bin/wipe-execute.sh`.

---

## Security notes

- The wipe uses `shred` on individual files where possible. On SSDs, shred is less effective due to wear leveling — consider full-disk encryption as a complementary layer.
- The kernel panic screen is a deterrent, not a lockout. Anyone who reboots the machine will see a normal login screen.
- The armed flag lives in `/tmp` (RAM). A reboot clears it automatically — the system is disarmed after any restart.
- Wipe logs are stored in `logs/wipe.log`. These may be forensically recoverable. Consider clearing them after a test.

---

## Files

```
small-tools/
├── bin/
│   ├── arm-wipe.sh        triggered by USB insertion
│   ├── confirm-wipe.sh    triggered by Ctrl+Alt+Del
│   ├── disarm.sh          triggered by USB removal
│   ├── wipe-daemon.sh     background signal listener
│   ├── wipe-execute.sh    the actual wipe
│   ├── soft-clean.sh      session cleanup
│   ├── panic-screen.sh    kernel panic screensaver
│   └── status.sh          show system state
├── config/
│   ├── udev-rule.rules    USB hardware trigger
│   ├── wipe-daemon.service  systemd service
│   └── xbindkeysrc        keyboard shortcuts
├── install.sh             run this once to set everything up
└── logs/wipe.log          activity log (created at runtime)
```

---

*//small tools// — hfk bremen, digital media masters*  
*distributed through trusted networks*
