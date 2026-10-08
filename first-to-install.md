### First to install
sudo apt install git zsh curl wget vim vlc htop xclip build-essential 

### Virtualization
sudo apt install qemu-system-x86 qemu-utils libvirt-daemon-system libvirt-clients virt-manager virtiofsd ovmf swtpm swtpm-tools dnsmasq-base bridge-utils && sudo usermod -aG libvirt,kvm "$USER" && sudo systemctl enable --now libvirtd

### Additional tools
uv, conda, docker, spotify, telegram, qbittorrent
