# Wayland screen time tracker

Per-app and per-website screen time tracking for KDE Plasma (Wayland), with a CLI viewer.

## How it works

- [screentime_daemon.py](screentime_daemon.py) runs as a systemd user service. It
  injects a small KWin script (via the `org.kde.kwin.Scripting` D-Bus interface)
  that reports every window activation back to the daemon. Every 5 seconds the
  daemon credits elapsed time to the focused app in a SQLite database at
  `~/.local/share/screentime/screentime.db`.
- For browser windows (Zen, Firefox, Chromium, etc.) the daemon also watches the
  window title and credits time to a per-website table. The site name is parsed
  from the title (e.g. "Video - YouTube — Zen Browser" → "YouTube") — it's a
  heuristic, not a real URL; pages whose titles omit the site name get odd labels.
- Tracking pauses while the screen is locked, and suspend gaps are not counted.
- If KWin restarts, the daemon re-injects the tracker script automatically.
- History is kept forever (the DB grows by a few KB per day).

## Usage

```
screentime                today's total and per-app breakdown
screentime yesterday      same, for yesterday
screentime week           per-day totals for the last 7 days
screentime history [N]    per-day totals for the last N days (default 14)
screentime apps [N]       per-app totals over the last N days (default 7)
screentime sites [N]      per-website totals over the last N days (default: today)
screentime 2026-07-01     breakdown for a specific date
```

## Install layout

- `~/.local/bin/screentime` → symlink to [screentime](screentime) (the CLI)
- `~/.config/systemd/user/screentime.service` → symlink to
  [screentime.service](screentime.service), enabled via
  `systemctl --user enable --now screentime`

## Service management

```
systemctl --user status screentime     # check the daemon
journalctl --user -u screentime -f     # follow its logs
systemctl --user restart screentime    # restart after editing the daemon
```
