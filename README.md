# nautilus-copypath-claude

A GNOME **Nautilus** extension that adds a right-click **"Copy Path"** to files and
folders — patched so the copied path plays nicely with **[Claude Code](https://www.anthropic.com/claude-code)**
and with GNOME's virtual locations (Recent / Trash / Search).

This is a fork of [**ronen25/nautilus-copypath**](https://git.sr.ht/~ronenk17/nautilus-copypath)
by Ronen Lapushner (and Fynn Freyer), distributed under the **GPL-3.0-or-later** license.
All credit for the original extension goes to them — see [Credits](#credits).

## Why this fork

Two problems with plain "copy path" when you live in a terminal running Claude Code:

1. **Claude Code auto-attaches image-file paths.** Paste an absolute path to a
   `.png` / `.jpg` / `.gif` into the Claude Code prompt and it reads the file and
   attaches it as an *image* instead of inserting the path as text. Animated GIFs
   then fail (unsupported by the vision API). There is currently **no setting to
   disable this** (see claude-code issues
   [#36391](https://github.com/anthropics/claude-code/issues/36391),
   [#50866](https://github.com/anthropics/claude-code/issues/50866)).
2. **The "Recent" / "Trash" / "Search" views** expose files via virtual URIs
   (`recent://`, `trash://`, `search://`), where upstream's `get_location().get_path()`
   returns `None` — so "Copy Path" copied nothing.

## What this fork changes

- **`~`-home paths (the key trick).** Copied paths use `~/…` instead of `/home/you/…`.
  A `~`-path is **not absolute**, so Claude Code does **not** auto-attach it — it
  lands as plain text you can reference. `~` is still understood by Claude and by
  your shell.
- **Virtual locations resolved.** `recent://`, `trash://` and `search://` items are
  followed to their real file via the `standard::target-uri` Gio attribute, so
  "Copy Path" works there too.
- **Single quotes by default** — `'~/Pictures/shot.png'` — safe for paths with spaces.
- **One menu item** (`Copy Path`) instead of extra clutter, plus a fix for an
  upstream bug where "Copy Directory Path" set a *list* on the clipboard instead of
  a string.

> Note: `'~/…'` inside quotes is **not** tilde-expanded by a shell (`cd '~/x'` won't
> work). This is fine for pasting into Claude Code, Telegram, editors, etc. If you
> mostly paste into a shell, set `NAUTILUS_COPYPATH_QUOTE_PATHS=0`.

## Install

Tested on **Ubuntu 24.04, GNOME Shell 46, X11**.

```bash
# 1. runtime for Nautilus Python extensions
sudo apt install -y python3-nautilus

# 2. drop the extension in place
mkdir -p ~/.local/share/nautilus-python/extensions
cp nautilus-copypath.py ~/.local/share/nautilus-python/extensions/

# 3. restart Nautilus
nautilus -q
```

Then right-click any file or folder → **Copy Path**.

## Configuration

Behaviour is controlled by environment variables the running Nautilus process sees
(the defaults in this fork differ from upstream):

| Variable | This fork's default | Effect |
|---|---|---|
| `NAUTILUS_COPYPATH_QUOTE_PATHS` | `1` (on) | wrap the path in single quotes |
| `NAUTILUS_COPYPATH_SANITIZE_PATHS` | `0` (off) | escape spaces/parens with `\` |
| `NAUTILUS_COPYPATH_PATH_SEPARATOR` | newline | separator for multiple selections |
| `NAUTILUS_COPYPATH_WINPATH` | off | Windows-style backslash paths |

The `~`-home rewrite is always on.

## Credits

- Original extension: **[nautilus-copypath](https://git.sr.ht/~ronenk17/nautilus-copypath)**
  © Ronen Lapushner 2017–2025, © Fynn Freyer 2023.
- Fork patches (`~`-paths for Claude Code, virtual-location resolution, single-quote
  default, single-item menu): 2026.

## License

**GPL-3.0-or-later**, same as upstream. See [`LICENSE`](LICENSE).
