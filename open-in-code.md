# "Open in VS Code" in the Nautilus context menu (Ubuntu)

Adds an **Open in VS Code** entry to the context menu of the Files (Nautilus) file manager:

- on right-click in an empty area of a folder (opens the current folder);
- on right-click on a file or folder (opens the selected item).

Works through the [nautilus-python](https://wiki.gnome.org/Projects/NautilusPython) extension.

## Requirements

- Ubuntu with Nautilus (Files)
- VS Code installed, with the `code` command available in the terminal:
  ```bash
  code --version
  ```

## Installation

### 1. Install nautilus-python

```bash
sudo apt install python3-nautilus
```

### 2. Find out the Nautilus API version

```bash
nautilus --version
ls /usr/lib/x86_64-linux-gnu/girepository-1.0/ | grep -i nautilus
```

The second command prints a file like `Nautilus-4.1.typelib`. The number in the name is the API version.
It changes between Nautilus releases (for example, `3.0` on Ubuntu 22.04 and older, `4.0` on Ubuntu 22.10–24.04, `4.1` on newer versions), so the script below picks the version automatically.

### 3. Create the extension

```bash
mkdir -p ~/.local/share/nautilus-python/extensions
nano ~/.local/share/nautilus-python/extensions/vscode_open.py
```

File contents:

```python
from subprocess import Popen
from gi import require_version

# Pick whichever API version is available on the system
for _v in ('4.1', '4.0', '3.0'):
    try:
        require_version('Nautilus', _v)
        break
    except ValueError:
        continue

from gi.repository import Nautilus, GObject


class VSCodeExtension(GObject.GObject, Nautilus.MenuProvider):
    def _open(self, menu, path):
        Popen(['code', path])

    # Right-click on an empty area inside a folder.
    # *args is needed because API 3.0 passes `window` as the first argument.
    def get_background_items(self, *args):
        current_folder = args[-1]
        item = Nautilus.MenuItem(
            name='VSCodeExtension::OpenBackground',
            label='Open in Code',
        )
        item.connect('activate', self._open, current_folder.get_location().get_path())
        return [item]

    # Right-click on a file or folder
    def get_file_items(self, *args):
        files = args[-1]
        if len(files) != 1:
            return []
        item = Nautilus.MenuItem(
            name='VSCodeExtension::OpenFile',
            label='Open in Code',
        )
        item.connect('activate', self._open, files[0].get_location().get_path())
        return [item]
```

### 4. Restart Nautilus

```bash
nautilus -q
nautilus
```

Open any folder and right-click an empty area or a file. The **Open in VS Code** entry should appear.

## Troubleshooting

Check the steps in order and stop at the first problem you find.

**1. Is the extension installed?**

```bash
ls /usr/lib/x86_64-linux-gnu/nautilus/extensions-*/ | grep python
```

You should see `libnautilus-python.so`. If it's missing, run `sudo apt install python3-nautilus` again.

**2. Is the file in the right place?**

```bash
ls -l ~/.local/share/nautilus-python/extensions/
```

`vscode_open.py` must be in exactly this folder.

**3. Does the script run without errors?**

```bash
python3 ~/.local/share/nautilus-python/extensions/vscode_open.py
```

It should print nothing. Common errors:

| Error | Cause and fix |
| --- | --- |
| `ValueError: Namespace Nautilus not available for version X` | The API version doesn't match the installed one. Add the version from step 2 of the installation to the `for _v in (...)` loop. |
| `SyntaxError` / `IndentationError` | Indentation got mangled when copying. |

**4. Run Nautilus with debug output**

```bash
nautilus -q
NAUTILUS_PYTHON_DEBUG=misc nautilus
```

The terminal output shows whether the extension file was loaded and what errors occurred. If you skip `nautilus -q`, the old process keeps running without the extension.

**5. The entry is there, but VS Code doesn't open**

```bash
which code
```

Use the resulting path instead of `'code'` in the script, for example `'/usr/bin/code'` or `'/snap/bin/code'`.

## Uninstall

```bash
rm ~/.local/share/nautilus-python/extensions/vscode_open.py
nautilus -q
```
