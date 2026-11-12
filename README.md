# hypr-float-memory

Remember where floating windows were, **per app**, and put them back when they
reopen. Made for [Omarchy](https://omarchy.org) 4 on Hyprland 0.57, where a
floating window always comes back at a default size in the middle of the
screen.

Background: [omacom/omarchy discussion #17876](https://github.com/omacom/omarchy/discussions/17876).

## What it does

- Remembers the last floating geometry of each app (window class): the monitor,
  the position relative to that monitor, and the size.
- When that app opens a floating window again, or one of its windows is
  floated, it puts it back with that size and position, right after Omarchy
  has placed it.
- It stays on the monitor the window opens on, which is the one you are
  working on. A spot saved on the other monitor is carried over
  proportionally.
- Only the first floating window of an app is tracked, so dialogs (Save, Open)
  don't overwrite the main window's spot. A second window of the same app keeps
  Omarchy's default placement.
- The geometry is clamped to the part of the monitor clear of bars, so a
  window never lands off-screen or with its title bar under the top bar.
- Tiled windows are never touched, and neither are Omarchy shell plugin
  windows (`org.quickshell`), file dialogs (portals), Hyprland dialogs and
  Folio's preview.

State lives in `~/.local/state/omarchy/float-positions.json`.

## Install

Requires Python 3 (standard library only), `hyprctl`, and a systemd user
session.

```bash
git clone https://github.com/TinyWorkshopDesign/hypr-float-memory.git
cd hypr-float-memory
install -Dm755 hypr-float-memory ~/.local/bin/hypr-float-memory
install -Dm644 hypr-float-memory.service ~/.config/systemd/user/hypr-float-memory.service
systemctl --user daemon-reload
systemctl --user enable --now hypr-float-memory.service
```

## Remove

```bash
systemctl --user disable --now hypr-float-memory.service
rm ~/.local/bin/hypr-float-memory ~/.config/systemd/user/hypr-float-memory.service
rm -f ~/.local/state/omarchy/float-positions.json
```

## How it works

- Listens on Hyprland's event socket (`.socket2.sock`) for `openwindow` and
  `changefloatingmode`, waits ~100 ms so Omarchy's own placement runs first,
  then dispatches `hl.dsp.window.resize(...)` and `hl.dsp.window.move(...)`
  with absolute coordinates.
- Hyprland 0.57 has no event for a window being moved or resized, so the
  geometry is sampled every 2 seconds with `hyprctl -j clients`.

## Limits

- One remembered spot per app: apps whose windows differ a lot share it.
- A move made less than 2 seconds before closing a window can be lost.

Tested on a MacBook Pro M1 (Asahi Linux, ARM64) with an external monitor.

## License

MIT
