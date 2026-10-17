# CinnaDock

**CinnaDock** is a standalone macOS-style dock for **Linux Mint Cinnamon on X11**.

It is written in Python/GTK 3 and integrates directly with Cinnamon for window
tracking, window previews, activation, workspaces, running applications and
visibility behavior.

Current release: **55.0**

---

## Highlights

- Smooth macOS-style icon magnification
- Centered translucent glass dock
- Pinned and running applications
- Running-window indicators and multi-window count badges
- Selectable live window previews
- Exact activation of the selected window
- Preview support for minimized windows and windows on other workspaces
- Visual hover/focus state for individual preview cards
- Rich right-click application and window menus
- Drag/reorder edit mode
- Always Visible, Auto-hide and Dodge Windows modes
- Bottom-edge reveal area
- Configurable dock size, margins, offset and hover zoom
- Full-color, adaptive monochrome and symbolic icon modes
- Live settings reload
- Automatic startup integration
- Built-in runtime/performance profiler
- No extra Python packages installed by the installer

## Window previews

Applications with multiple windows can be handled directly from the dock.

Hover over an application to open its preview panel. Every open window gets its
own selectable preview card. Clicking a preview activates that exact window.

CinnaDock also handles:

- minimized windows;
- windows on another Cinnamon workspace;
- multiple windows belonging to the same application;
- pointer movement from the dock icon to the floating preview;
- diagonal pointer movement without neighbouring icons stealing the preview;
- a clear visual selected state for the preview currently under the pointer.

## Installation

CinnaDock is intended for **Linux Mint Cinnamon running an X11 session**.

The installer expects the standard GTK 3 Python bindings normally included with
Linux Mint:

- Python 3
- PyGObject / `python3-gi`
- GTK 3 / `gir1.2-gtk-3.0`

Clone or download the repository, open a terminal in the CinnaDock source
directory, and run:

```bash
./install.sh
```

The installer:

- checks the required system Python/GTK bindings;
- installs CinnaDock into `~/CinnaDock`;
- creates `~/.local/bin/cinnadock`;
- adds an autostart entry;
- adds a desktop application entry;
- starts CinnaDock.

It does **not** install system packages.

Verify the installation with:

```bash
cinnadock --version
```

Expected output:

```text
CinnaDock 55.0
```

## Commands

Open settings:

```bash
cinnadock --settings
```

Show the installed version:

```bash
cinnadock --version
```

Stop CinnaDock:

```bash
cinnadock --quit
```

Start it manually:

```bash
cinnadock
```

## Uninstall

From the source directory:

```bash
./uninstall.sh
```

The uninstaller removes the installed application, launcher and autostart
entries.

Your configuration is intentionally kept in:

```text
~/.config/cinnadock/
```

Remove that directory manually if you also want to reset all settings.

## Main features

### Dock behavior

CinnaDock supports three visibility modes:

| Mode | Behavior |
| --- | --- |
| **Always Visible** | Dock remains visible |
| **Auto-hide** | Dock hides when the pointer leaves |
| **Dodge Windows** | Dock hides when windows overlap its area |

The hidden dock can be revealed again from a configurable bottom-edge trigger
area.

### Magnification

Hover magnification is smoothly animated and configurable from the settings
window.

The rendering path uses quantized icon sizes and caching so that already
rendered icon variants can be reused rather than recreated every frame.

### Running applications

CinnaDock can show:

- pinned applications;
- unpinned running applications;
- running-state dots;
- configurable running-indicator color;
- window-count badges when an application owns multiple windows.

### Context menus

Application context menus include actions such as:

- show/open application;
- launch another instance;
- desktop-file actions;
- select individual windows;
- minimize/restore;
- maximize/unmaximize;
- workspace actions;
- close other windows;
- close all windows;
- pin/unpin;
- reorder;
- configure autostart where available.

Available actions depend on the application and window state.

### Icon rendering

CinnaDock supports multiple icon styles:

- normal full-color application icons;
- adaptive tonal monochrome rendering;
- native symbolic icons;
- generated symbolic fallback icons.

Generated symbolic icons have separate controls for stroke/weight, rounded
border width and border inset.

## Performance design

CinnaDock is still a Python application, but its active rendering path is
designed to avoid unnecessary GTK work.

Important optimizations include:

- a single Cairo/GTK drawing canvas instead of one GTK widget per icon;
- lazy icon-size caching;
- cached Cairo surfaces;
- a cached transparent dock shell;
- dirty-region redraws;
- redraw coalescing to at most one GTK invalidation per display frame;
- event-driven pointer tracking while visible;
- a low-rate hidden edge-reveal watcher;
- an on-demand GTK frame-clock callback that stops when animation settles;
- cached Cinnamon window/application state for hover operations.

When the dock is idle, the animation frame clock goes to sleep instead of
running continuously.

### Profiler

**About CinnaDock** contains a built-in animation profiler with measurements for
frame timing, pointer handling, geometry, rendering, redraws and caches.

Profiler instrumentation is disabled during normal use. It is enabled only
while the About window is open and disabled again immediately when the window
closes.

Runtime CPU/RSS information refreshes once per second, while profiler values are
published in completed 5-second measurement windows.

## Settings

Settings are stored as JSON in:

```text
~/.config/cinnadock/config.json
```

The settings application runs separately from the dock. Changes are detected by
the running dock and applied without requiring a full reinstall.

Common settings include:

- icon size;
- hover zoom;
- dock height;
- side margin;
- bottom offset;
- preview delay;
- visibility mode;
- auto-hide delay;
- reveal-area behavior;
- running-indicator appearance;
- icon rendering mode;
- generated symbolic icon controls.

## Installed files

| Purpose | Location |
| --- | --- |
| Application code | `~/CinnaDock/` |
| Command launcher | `~/.local/bin/cinnadock` |
| Configuration | `~/.config/cinnadock/config.json` |
| Runtime log | `~/.config/cinnadock/cinnadock.log` |
| Cache | `~/.cache/cinnadock/` |
| Autostart entry | `~/.config/autostart/cinnadock.desktop` |
| Desktop entry | `~/.local/share/applications/cinnadock.desktop` |

The launcher explicitly uses:

```text
/usr/bin/python3
```

This avoids accidentally launching the dock through an unrelated virtual
environment or Conda Python installation.

## Architecture

CinnaDock is split into focused modules:

| File | Responsibility |
| --- | --- |
| `cinnadock.py` | Small executable entry point |
| `cinnadock_cli.py` | CLI, lifecycle and diagnostics |
| `cinnadock_common.py` | Shared constants, configuration and helpers |
| `cinnadock_apps.py` | `.desktop` discovery and application launching |
| `cinnadock_cinnamon.py` | Cinnamon D-Bus / MetaWindow integration |
| `cinnadock_widgets.py` | Preview, reveal and supporting GTK widgets |
| `cinnadock_settings.py` | Standalone settings application |
| `cinnadock_dock.py` | Main dock window, rendering and runtime state |

## Cinnamon integration

CinnaDock is intentionally Cinnamon-specific.

It uses Cinnamon's window/application information and MetaWindow behavior for
features such as:

- running-window discovery;
- application matching;
- window activation;
- workspace switching;
- minimizing/restoring;
- compositor-backed window previews.

This is why CinnaDock targets Linux Mint Cinnamon/X11 rather than trying to be a
generic cross-desktop dock.

## Upgrade from GlassDock

CinnaDock was previously developed under the name **GlassDock**.

The installer contains one-time migration support. If CinnaDock has no existing
configuration but finds:

```text
~/.config/glassdock/config.json
```

it copies that configuration to:

```text
~/.config/cinnadock/config.json
```

The old GlassDock launcher, autostart entry and installed code are removed, but
the old GlassDock configuration directory is left in place as a backup.

## Troubleshooting

If CinnaDock does not start, check:

```text
~/.config/cinnadock/cinnadock.log
```

You can also verify that the GTK 3 Python bindings are available to the same
system Python used by CinnaDock:

```bash
/usr/bin/python3 -c 'import gi; gi.require_version("Gtk", "3.0"); from gi.repository import Gtk; print(Gtk.get_major_version())'
```

To restart CinnaDock manually:

```bash
cinnadock --quit
cinnadock
```

## Platform scope

CinnaDock is built for:

- Linux Mint
- Cinnamon
- GTK 3
- X11

Its Cinnamon and X11 integration is deliberate; portability to other desktop
environments is not currently a design goal.
