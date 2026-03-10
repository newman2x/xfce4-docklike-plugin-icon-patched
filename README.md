# xfce4-docklike-plugin — Patched

> A patched build of [xfce4-docklike-plugin](https://gitlab.xfce.org/panel-plugins/xfce4-docklike-plugin) for Linux Mint / Ubuntu (Noble 24.04, amd64) with icon consistency fixes and a styled window count badge.

---

## What's Changed

### 1. Consistent Icons (Dock ↔ Popup ↔ Whisker ↔ Desktop)

**Problem:** The popup menu showed the window's own icon (`xfw_window_get_icon`), while the dock button used the theme icon from the `.desktop` file. This caused mismatches — especially for apps like Software Manager (`mintinstall.py`).

**Fix:** `GroupMenuItem::updateIcon()` now uses the same `.desktop` theme icon as the dock button, with a fallback to the window icon if no `.desktop` match is found.

```
Before: dock button = 🟢 theme icon | popup = ❌ window icon (different)
After:  dock button = 🟢 theme icon | popup = 🟢 theme icon (same)
```

---

### 2. WM_CLASS Extension Stripping

**Problem:** Apps like `mintinstall.py`, `appname.sh`, or Wine `.exe` apps have file extensions in their `WM_CLASS`, which prevents matching against `.desktop` filenames (e.g. `mintinstall`).

**Fix:** `AppInfos::search()` now strips common scripting extensions (`.py`, `.sh`, `.rb`, `.pl`, `.exe`) before lookup — so `Mintinstall.py` → `mintinstall` → matches `mintinstall.desktop`.

Extensions stripped:
- `.py` — Python apps (e.g. Software Manager, Timeshift)
- `.sh` — Shell scripts
- `.rb` — Ruby apps
- `.pl` — Perl apps
- `.exe` — Wine apps

---

### 3. Window Count Badge (CSS Styled)

**Problem:** The window count label had no default styling — it appeared as plain text overlapping the icon.

**Fix:** Added default CSS for `.window_count` in `Theme.hpp`:
- Pill-shaped badge (border-radius: 999px)
- Semi-transparent background using GTK theme colors
- Hidden automatically when ≤ 2 windows (via `gtk_widget_hide`)
- Only visible when > 2 windows of the same app are open

```css
.window_count {
  background-color: alpha(@menu_item_bgcolor_hover, 0.85);
  color: @menu_item_color;
  border-radius: 999px;
  font-size: 9px;
  min-width: 10px;
  min-height: 10px;
  padding: 0px;
  margin: 0px;
}
```

You can override this in `~/.config/xfce4-docklike-plugin/gtk.css`.

---

## Installation

### Pre-built .deb (Linux Mint 22 / Ubuntu 24.04 Noble, amd64)

```bash
sudo apt install ./xfce4-docklike-plugin-patched_0.5.1-1_amd64.deb
xfce4-panel -r
```

> **Note:** Uses `Conflicts: xfce4-docklike-plugin` — will replace the system package. No prior installation needed.

---

## Build from Source

### Prerequisites

```bash
sudo apt install git meson ninja-build g++ \
  libgtk-3-dev libxfce4panel-2.0-dev libxfce4ui-2-dev \
  libxfce4util-dev libxfce4windowing-dev libxfce4windowingui-dev \
  libglib2.0-dev
```

### Clone & Build

```bash
git clone https://github.com/YOUR_USERNAME/xfce4-docklike-plugin-patched.git
cd xfce4-docklike-plugin-patched
meson setup build
cd build
ninja
```

### Install

```bash
sudo cp src/libdocklike.so \
  /usr/lib/x86_64-linux-gnu/xfce4/panel/plugins/libdocklike.so
xfce4-panel -r
```

---

## Files Changed

| File | Change |
|---|---|
| `src/GroupMenuItem.cpp` | `updateIcon()` — use `.desktop` theme icon instead of window icon |
| `src/AppInfos.cpp` | `search()` — strip `.py/.sh/.rb/.pl/.exe` from WM_CLASS before matching |
| `src/Group.cpp` | `updateStyle()` — hide label widget when ≤ 2 windows |
| `src/Theme.hpp` | `DEFAULT_THEME` — add `.window_count` CSS badge styling |

---

## CSS Customization

The popup menu and badge can be fully customized via:

```
~/.config/xfce4-docklike-plugin/gtk.css
```

Available CSS selectors:

```css
#docklike-plugin       /* the whole plugin */
.group                 /* each app icon button */
.open_group            /* app has open windows */
.active_group          /* currently focused app */
.hover_group           /* mouse hovering */
.menu                  /* popup menu container */
.menu_item             /* each window row in popup */
.active_menu_item      /* focused window row */
.hover_menu_item       /* hovered window row */
.window_count          /* window count badge */
.icon                  /* icon in popup menu item */
.title                 /* label in popup menu item */
.preview               /* thumbnail preview */
```

Available theme variables:

```css
@menu_bgcolor
@menu_item_color
@menu_item_color_hover
@menu_item_bgcolor_hover
@active_indicator_color
@inactive_indicator_color
```

---

## Upstream

This is a patch on top of the official [xfce4-docklike-plugin](https://gitlab.xfce.org/panel-plugins/xfce4-docklike-plugin) v0.5.1 by Nicolas Szabo and David Keogh, licensed under GPL-3.0-or-later.
