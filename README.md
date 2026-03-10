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
git clone https://github.com/newman2x/xfce4-docklike-plugin-patched.git
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
Complete reference for styling via `~/.config/xfce4-docklike-plugin/gtk.css`

---

### Theme Variables

Defined at runtime from the current GTK theme:

| Variable | Description |
|---|---|
| `@menu_bgcolor` | Popup menu background color |
| `@menu_item_color` | Menu item text color |
| `@menu_item_color_hover` | Menu item text color on hover |
| `@menu_item_bgcolor_hover` | Menu item background color on hover |
| `@active_indicator_color` | Active window indicator color |
| `@inactive_indicator_color` | Inactive window indicator color |

---

### Selector Tree

```
#docklike-plugin                          the entire plugin widget
└── .group                                each app button in the dock
    ├── .open_group                       has at least 1 open window
    ├── .active_group                     currently focused app
    ├── .hover_group                      mouse hovering over button
    ├── .drop_target                      drag-and-drop highlight
    └── .window_count                     badge showing window count
                                          only visible when > 1 window
                                          position: bottom-right (GTK_ALIGN_END)

.xfce-docklike-window                     the floating popup window
└── .menu                                 menu container box
                                          background: @menu_bgcolor
    └── .menu_item                        one row per open window
        ├── .active_menu_item             row of currently focused window
        ├── .hover_menu_item              row being hovered
        └── grid                          3-column layout inside the row
            │
            ├── col 0, row 0  .icon       app icon (from .desktop file)
            │                             size controlled in updateIcon()
            │                             fallback: window's own icon
            │
            ├── col 1, row 0  .title      window title text
            │                             ellipsized at end if too long
            │                             width: gtk_label_set_width_chars()
            │                             active window = bold markup <b>
            │                             minimized window = italic markup <i>
            │
            ├── col 2, row 0  button      close button ✕
            │                             icon: "window-close"
            │                             no border (GTK_RELIEF_NONE)
            │
            └── col 0-2, row 1  .preview  window thumbnail screenshot
                                          spans all 3 columns (full width)
                                          hidden when window is minimized
                                          scale: Settings::previewScale
                                          refresh: Settings::previewSleep ms
                                          X11 only — not available on Wayland
```

---

### Grid Layout

```
┌──────────┬────────────────────────┬──────────┐  ← row 0
│  .icon   │        .title          │  button  │
├──────────┴────────────────────────┴──────────┤  ← row 1
│                  .preview                    │
└──────────────────────────────────────────────┘
  col 0          col 1               col 2
```

---

### Full CSS Example

```css
/* ── Dock Buttons ─────────────────────────────────── */

#docklike-plugin {
  /* the whole plugin */
}

.group {
  border-radius: 6px;
}

.open_group {
  box-shadow: inset 0px -3px 0px 0px @inactive_indicator_color;
}

.active_group {
  box-shadow: inset 0px -3px 0px 0px @active_indicator_color;
  background-color: alpha(@active_indicator_color, 0.1);
}

.hover_group {
  background-color: alpha(@menu_item_bgcolor_hover, 0.2);
}

.drop_target {
  box-shadow: inset 0px -3px 0px 0px @active_indicator_color;
}

/* Window count badge */
.window_count {
  background-color: alpha(@menu_item_bgcolor_hover, 0.85);
  color: @menu_item_color;
  border-radius: 999px;
  border: 1px solid alpha(@menu_item_color, 0.5);
  font-size: 10px;
  min-width: 16px;
  min-height: 16px;
  padding: 0px;
  margin: 2px;
}


/* ── Popup Menu ───────────────────────────────────── */

.xfce-docklike-window .menu {
  margin: 0px;
  padding: 0px;
  border: 0px;
  background-color: @menu_bgcolor;
}

.xfce-docklike-window .menu_item {
  padding: 4px 6px;
}

.xfce-docklike-window .menu_item grid {
  margin: 0px;
  padding: 0px;
}

/* Hovered row */
.xfce-docklike-window .hover_menu_item {
  background-color: alpha(@menu_item_bgcolor_hover, 0.2);
}

/* Focused window row */
.xfce-docklike-window .active_menu_item {
  background-color: alpha(@active_indicator_color, 0.15);
}

/* App icon */
.xfce-docklike-window .menu_item .icon {
  margin-right: 4px;
}

/* Window title */
.xfce-docklike-window .menu_item .title {
  font-size: 11px;
  color: @menu_item_color;
}

/* Close button */
.xfce-docklike-window .menu_item button {
  padding: 0px;
  margin: 0px;
}

/* Preview thumbnail */
.xfce-docklike-window .menu_item .preview {
  margin: 4px 2px 2px 2px;
  border-radius: 6px;
  border: 1px solid alpha(@menu_item_color, 0.15);
}
```

---

### Notes

- CSS file: `~/.config/xfce4-docklike-plugin/gtk.css`
- Reload: `xfce4-panel -r`
- The `.xfce-docklike-window` prefix is **required** for popup menu selectors
- Dock button selectors (`.group`, `.window_count`) do **not** need the prefix
- The indicator bar/dot/circle is drawn with **Cairo** — not styleable via CSS
- Preview thumbnails are **X11 only** — Wayland does not support foreign window capture


## Upstream

This is a patch on top of the official [xfce4-docklike-plugin](https://gitlab.xfce.org/panel-plugins/xfce4-docklike-plugin) v0.5.1 by Nicolas Szabo and David Keogh, licensed under GPL-3.0-or-later.
