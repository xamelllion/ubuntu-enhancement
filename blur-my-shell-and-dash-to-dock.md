# Blur my Shell and Dash to Dock (GNOME extensions)

Two GNOME Shell extensions that make the desktop look and feel better:

- [Dash to Dock](https://extensions.gnome.org/extension/307/dash-to-dock/) is the upstream dock that Ubuntu Dock is forked from. It has more settings and gets updates sooner.
- [Blur my Shell](https://extensions.gnome.org/extension/3193/blur-my-shell/) adds blur to the top panel, the dock, the overview, app folders and the lock screen.

Tested on Ubuntu 26.04 (GNOME Shell 50) with Dash to Dock v110 and Blur my Shell v76.

## Requirements

- Ubuntu with GNOME Shell:
  ```bash
  gnome-shell --version
  ```

## Installation

### 1. Install Extension Manager

```bash
sudo apt install gnome-shell-extension-manager
```

It installs, updates and configures extensions from [extensions.gnome.org](https://extensions.gnome.org) without a browser.

### 2. Install the extensions

**Option A: GUI.** Open **Extension Manager**, go to the **Browse** tab, find **Dash to Dock** and **Blur my Shell**, and click **Install** for each.

**Option B: terminal.** GNOME Shell downloads the version that matches your shell and asks for confirmation in a dialog:

```bash
for uuid in dash-to-dock@micxgx.gmail.com blur-my-shell@aunetx; do
  gdbus call --session \
    --dest org.gnome.Shell.Extensions \
    --object-path /org/gnome/Shell/Extensions \
    --method org.gnome.Shell.Extensions.InstallRemoteExtension "$uuid"
done
```

Each call should return `('successful',)`. Both ways enable the extension right away, no re-login needed.

### 3. Turn off Ubuntu Dock

Ubuntu Dock hides itself when Dash to Dock is enabled, but it's cleaner to disable it explicitly:

```bash
gnome-extensions disable ubuntu-dock@ubuntu.com
```

### 4. Check

```bash
gnome-extensions info dash-to-dock@micxgx.gmail.com | grep State
gnome-extensions info blur-my-shell@aunetx | grep State
gnome-extensions info ubuntu-dock@ubuntu.com | grep State
```

The first two should be `ACTIVE`, Ubuntu Dock should be `INITIALIZED` (installed but off).

## My settings

Settings live in dconf, so they can be applied with one command instead of clicking through the preferences. `dconf load` only overwrites the keys in the block, everything else stays as it is. Changes apply immediately.

### Dash to Dock

| Setting | Value |
| --- | --- |
| Position | Bottom, always visible |
| Size | Stretched to the full screen width, icons centered, max icon size 44 |
| Click on a running app | Minimize it, or show the overview if it has several windows |
| Show Apps button | At the right end, next to the icons |
| Trash and drives | Hidden |
| Monitor | Built-in laptop screen (`eDP-1`) |
| Theme | Default (the background is handled by Blur my Shell) |

```bash
dconf load /org/gnome/shell/extensions/dash-to-dock/ << 'EOF'
[/]
always-center-icons=true
apply-custom-theme=false
background-opacity=0.80000000000000004
click-action='minimize-or-overview'
dash-max-icon-size=44
dock-fixed=true
dock-position='BOTTOM'
extend-height=true
height-fraction=0.90000000000000002
hide-tooltip=false
isolate-monitors=false
preferred-monitor=-2
preferred-monitor-by-connector='eDP-1'
show-apps-always-in-the-edge=false
show-apps-at-top=false
show-mounts=false
show-mounts-only-mounted=false
show-show-apps-button=true
show-trash=false
EOF
```

Ubuntu Dock reads settings from the same dconf path, so these values also apply to it if you switch back.

### Blur my Shell

| Component | Blur |
| --- | --- |
| Dock | On, static, transparent dock style, rounded corners (radius 30, brightness 0.4) |
| Top panel | On, static (radius 30, brightness 0.6) |
| Overview, lock screen, screenshot UI | On (radius 30, brightness 0.6) |
| App folders, window list | On (radius 60, brightness 0.6) |
| Popups, menus, notifications | Off, quick settings corners not rounded |

```bash
dconf load /org/gnome/shell/extensions/blur-my-shell/ << 'EOF'
[/]
pipelines={'pipeline_default': {'name': <'Default'>, 'effects': <[<{'type': <'dual_kawase_blur'>, 'id': <'effect_000000000000'>, 'params': <{'unscaled_radius': <30>, 'brightness': <0.59999999999999998>}>}>]>}, 'pipeline_default_rounded': {'name': <'Default rounded'>, 'effects': <[<{'type': <'dual_kawase_blur'>, 'id': <'effect_000000000001'>, 'params': <{'unscaled_radius': <30>, 'brightness': <0.40000000000000002>}>}>]>}, 'pipeline_popup': {'name': <'Popup'>, 'effects': <[<{'type': <'dual_kawase_blur'>, 'id': <'effect_000000000003'>, 'params': <{'unscaled_radius': <30>, 'brightness': <0.40000000000000002>}>}>]>}, 'pipeline_migrated_dynamic_blur': {'name': <'Migrated dynamic blur'>, 'effects': <[<{'type': <'dual_kawase_blur'>, 'id': <'effect_migrated_dynamic_blur'>, 'params': <{'unscaled_radius': <60>, 'brightness': <0.59999999999999998>}>}>]>}}
rounded-blur-found=true
settings-version=7

[appfolder]
pipeline='pipeline_migrated_dynamic_blur'

[applications]
pipeline='pipeline_default'

[coverflow-alt-tab]
pipeline='pipeline_default'

[dash-to-dock]
blur=true
pipeline='pipeline_default_rounded'
static-blur=true
style-dash-to-dock=0

[lockscreen]
pipeline='pipeline_default'

[overview]
pipeline='pipeline_default'

[panel]
pipeline='pipeline_default'
static-blur=true

[popup]
blur=false
pipeline='pipeline_popup'
quick-settings-corner-radius=0
static-blur=false

[screenshot]
pipeline='pipeline_default'

[window-list]
pipeline='pipeline_migrated_dynamic_blur'
EOF
```

### Back up your own settings

After tweaking things in the preferences, save the current state in the same format:

```bash
dconf dump /org/gnome/shell/extensions/dash-to-dock/ > dash-to-dock.conf
dconf dump /org/gnome/shell/extensions/blur-my-shell/ > blur-my-shell.conf
```

Restore with `dconf load /org/gnome/shell/extensions/<name>/ < <name>.conf`.

## Troubleshooting

**1. Two docks, or the old Ubuntu Dock is still there**

```bash
gnome-extensions disable ubuntu-dock@ubuntu.com
gnome-extensions info dash-to-dock@micxgx.gmail.com | grep State
```

Dash to Dock must be `ACTIVE`. If it's `DISABLED`, turn it on with `gnome-extensions enable dash-to-dock@micxgx.gmail.com`.

**2. An extension is in the `ERROR` or `OUT OF DATE` state**

This usually happens after an Ubuntu release upgrade brings a new GNOME Shell. Open Extension Manager, go to the **Updates** tab and update the extension, then log out and back in. Check which GNOME versions the installed copy supports:

```bash
grep -A10 shell-version ~/.local/share/gnome-shell/extensions/blur-my-shell@aunetx/metadata.json
```

**3. The dock is not blurred**

- In Blur my Shell preferences, open **Dash to Dock** and check that blur is on. In dconf it's `[dash-to-dock] blur=true`.
- Blur my Shell attaches to the dock when it appears. If Dash to Dock was enabled after Blur my Shell, toggle Blur my Shell off and on:
  ```bash
  gnome-extensions disable blur-my-shell@aunetx && gnome-extensions enable blur-my-shell@aunetx
  ```

**4. Look at the GNOME Shell log**

```bash
journalctl -f -o cat /usr/bin/gnome-shell
```

Toggle the extension in another terminal and watch for `JS ERROR` lines that mention its uuid.

## Uninstall

```bash
gnome-extensions uninstall dash-to-dock@micxgx.gmail.com
gnome-extensions uninstall blur-my-shell@aunetx
gnome-extensions enable ubuntu-dock@ubuntu.com
```

The settings stay in dconf. To remove the Blur my Shell ones:

```bash
dconf reset -f /org/gnome/shell/extensions/blur-my-shell/
```

Don't reset `/org/gnome/shell/extensions/dash-to-dock/` unless you also want to reset Ubuntu Dock, since they share it.
