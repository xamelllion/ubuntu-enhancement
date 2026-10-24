# First to install

What I install first on a fresh Ubuntu: base packages, virtualization and everyday apps.

## Base packages

```bash
sudo apt install git zsh curl wget vim vlc htop xclip build-essential
```

| Package | What it's for |
| --- | --- |
| `git` | Version control |
| `zsh` | Shell, configured in [zshrc.md](zshrc.md) |
| `curl`, `wget` | Downloading files and install scripts |
| `vim` | Editor in the terminal |
| `vlc` | Video and audio player |
| `htop` | Process and resource monitor |
| `xclip` | Clipboard from the terminal, used by the `xc` alias |
| `build-essential` | `gcc`, `make` and headers for building from source |

## Virtualization

QEMU/KVM with the virt-manager GUI. Runs Linux and Windows 11 guests.

### 1. Install the packages

```bash
sudo apt install \
  qemu-system-x86 qemu-utils \
  libvirt-daemon-system libvirt-clients virt-manager \
  virtiofsd ovmf swtpm swtpm-tools \
  dnsmasq-base bridge-utils
```

| Package | What it's for |
| --- | --- |
| `qemu-system-x86`, `qemu-utils` | The hypervisor and `qemu-img` for disk images |
| `libvirt-daemon-system`, `libvirt-clients` | The `libvirtd` service and `virsh` |
| `virt-manager` | GUI for creating and running VMs |
| `virtiofsd` | Shared folders between the host and a guest |
| `ovmf` | UEFI firmware for guests |
| `swtpm`, `swtpm-tools` | Virtual TPM, required by Windows 11 |
| `dnsmasq-base`, `bridge-utils` | NAT and bridged networking for guests |

### 2. Give your user access

```bash
sudo usermod -aG libvirt,kvm "$USER"
```

Log out and back in for the new groups to apply.

### 3. Start libvirt

```bash
sudo systemctl enable --now libvirtd
```

### 4. Check

```bash
virsh -c qemu:///system list --all
```

It should print a table of VMs (empty on a fresh install) without errors. After that, open **Virtual Machine Manager**.

## Additional tools

| Tool | What it's for | How to install |
| --- | --- | --- |
| [uv](https://docs.astral.sh/uv/) | Python versions, venvs and packages | `curl -LsSf https://astral.sh/uv/install.sh \| sh` |
| [conda](https://github.com/conda-forge/miniforge) | Conda environments, via Miniforge | See below |
| [Docker](https://docs.docker.com/engine/install/ubuntu/) | Containers | Official apt repo, see the link |
| Spotify | Music | `sudo snap install spotify` |
| Telegram | Messenger | `sudo snap install telegram-desktop` |
| qBittorrent | Torrent client | `sudo apt install qbittorrent` |

**Miniforge:**

```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

**Docker without sudo** (after installing it from the official repo):

```bash
sudo usermod -aG docker "$USER"
```

Log out and back in, then check with `docker run hello-world`.
