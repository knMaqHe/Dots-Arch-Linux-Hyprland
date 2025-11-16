## Complete Guide to Installing Arch Linux with Hyprland alongside Windows 11

## Preparation for Installation

### Connection and Network Check
Connect the USB drive with the downloaded Arch Linux image to your computer. Check network connection:
```
ping -c 4 google.com
```

### View Current Partition Layout
```
lsblk
fdisk -l
```
Identify the disk where Arch Linux will be installed (e.g., `/dev/sda` or `/dev/sdc`).

### Creating Partitions with cfdisk
```
cfdisk /dev/sda  # replace with your disk
```

**Partition Scheme:**
- ➕ **New EFI partition for Arch** - 1GB (type: EFI System)
- ➕ **Root partition** - remaining space (type: Linux filesystem)

### Formatting Partitions
```
# Formatting EFI partition
mkfs.fat -F32 /dev/sda1

# Formatting root partition
mkfs.ext4 /dev/sda2
```

### Mounting Partitions
```
mount /dev/sda2 /mnt
mount --mkdir /dev/sda1 /mnt/boot
```

## 📦 Installing Base System

```
# Installing base packages
pacstrap /mnt base base-devel linux linux-headers linux-firmware nano dhcpcd networkmanager intel-ucode

# Generating fstab
genfstab -U /mnt >> /mnt/etc/fstab
```

## 🔧 System Configuration

### Entering the Installed System
```
arch-chroot /mnt
```

### Time and Locale Configuration
```
# Hostname
echo "archlinux" > /etc/hostname

# Time zone
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc

# Localization
nano /etc/locale.gen # Uncomment lines en_US.UTF-8 UTF-8 and ru_RU.UTF-8 UTF-8
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Keyboard layout
echo "KEYMAP=ru" > /etc/vconsole.conf
echo "FONT=cyr-sun16" >> /etc/vconsole.conf
```

### Network Configuration
```
# Hosts configuration
cat > /etc/hosts << EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   archlinux.localdomain archlinux
EOF

# Enabling network services
systemctl enable NetworkManager
systemctl enable dhcpcd
```

### User Creation
```
# Root password
passwd

# Creating regular user
useradd -m -G wheel -s /bin/bash username
passwd username

# sudo configuration
pacman -S sudo
EDITOR=nano visudo
# Uncomment the line: %wheel ALL=(ALL) ALL
```

## 🎯 Bootloader Installation

```
# Installing GRUB and utilities
pacman -S grub efibootmgr os-prober

# Enable Windows support in GRUB:
echo "GRUB_DISABLE_OS_PROBER=false" >> /etc/default/grub
os-prober

# GRUB installation
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch --recheck

# Additional commands for boot issues
grub-install --efi-directory=/boot --removable
grub-install --efi-directory=/boot --target=x86_64-efi --bootloader-id=arch --recheck

# Final configuration
grub-mkconfig -o /boot/grub/grub.cfg
```

## 🖥️ Graphical Environment Installation

### Hyprland and Wayland Components
```
pacman -S hyprland wayland wayland-protocols xdg-desktop-portal-hyprland \
    waybar rofi alacritty thunar polkit-gnome \
    hyprpaper hyprlock hypridle hyprpolkitagent \
    git curl wget swaync
```

### Display Manager
```
pacman -S sddm sddm-kcm qt6-svg qt6-virtualkeyboard qt6-multimedia-ffmpeg
systemctl enable sddm
```

### Main Applications
```
pacman -S alacritty nautilus rofi-wayland firefox
```

### System Utilities
```
pacman -S grim slurp wl-clipboard cliphist
pacman -S brightnessctl playerctl wireplumber
pacman -S udiskie udisks2 file-roller polkit-gnome
```

### Audio System
```
pacman -S pulseaudio pulseaudio-alsa pavucontrol wireplumber
```

### Fonts
```
pacman -S ttf-firacode-nerd noto-fonts noto-fonts-emoji ttf-dejavu
pacman -S ttf-nerd-fonts-symbols-mono ttf-ibm-plex
```

### Themes and Icons
```
pacman -S papirus-icon-theme lxappearance kvantum nwg-look
```

### Cursors
```
pacman -S bibata-cursors bibata-extra-cursors
```

## 🎨 Environment Configuration

### Creating Configuration Directories
```
mkdir -p /home/username/.config/hypr
mkdir -p /home/username/.config/waybar
mkdir -p /home/username/Images/Wallpaper
```

### Cursor Configuration
Add to `/etc/environment`:
```
XCURSOR_THEME=Bibata-Modern-Ice
XCURSOR_SIZE=24
```

### SDDM Theme Configuration
```
nano /etc/sddm.conf.d/theme.conf
```

Add:
```
[Theme]
Current="sddm-astronaut-theme"
```

## 🚀 Completing Installation

```
# Exit chroot
exit

# Unmounting
umount -R /mnt

# Reboot
reboot
```

## 🛠️ Post-Installation Setup

### AUR Helper Installation (yay)
```
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si

# Or alternative option:
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
cd .. && rm -rf yay-bin
```

### Theme Installation (Optional)
```
# Theme for Nautilus
yay -S gruvbox-gtk-theme

# Theme for SDDM
yay -S sddm-theme-astronaut
```

### Custom SDDM Theme Installation
Move the `sddm-astronaut-theme` folder to `/usr/share/sddm/themes` and run:
```
echo "[Theme]
Current=sddm-astronaut-theme" | sudo tee /etc/sddm.conf
```

### Configuration Files
Move corresponding configuration folders for Hyprland, rofi, waybar, alacritty to `~/.config` directory.

## NVIDIA Drivers Installation in Arch Linux

## 📋 Quick Installation (All Commands)

```
# Driver installation
sudo pacman -S nvidia nvidia-utils nvidia-settings lib32-nvidia-utils linux-headers

# Kernel configuration
echo 'options nvidia_drm modeset=1' | sudo tee /etc/modprobe.d/nvidia.conf
sudo sed -i 's/GRUB_CMDLINE_LINUX=""/GRUB_CMDLINE_LINUX="nvidia_drm.modeset=1"/' /etc/default/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg

# Blacklist nouveau
echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf

# Update initramfs
sudo mkinitcpio -P

# Reboot
reboot
```

## 📦 Driver Installation

### Main Drivers
```
sudo pacman -S nvidia nvidia-utils nvidia-settings
```

### Creating modprobe Configuration
```
sudo nano /etc/modprobe.d/nvidia.conf
```

Add:
```
options nvidia_drm modeset=1
```

## 🎯 Configuration for Wayland/Hyprland

### Adding Environment Variables
```
sudo nano /etc/environment
```

Add the following lines:
```
LIBVA_DRIVER_NAME=nvidia
GBM_BACKEND=nvidia-drm
__GLX_VENDOR_LIBRARY_NAME=nvidia
WLR_NO_HARDWARE_CURSORS=1
```

### Hyprland Configuration
```
nano ~/.config/hypr/environment.conf
```

Add:
```
env = LIBVA_DRIVER_NAME,nvidia
env = GBM_BACKEND,nvidia-drm
env = __GLX_VENDOR_LIBRARY_NAME,nvidia
env = WLR_NO_HARDWARE_CURSORS,1
```

## 🔄 System Update

### Update initramfs
```
sudo mkinitcpio -P
```

### System Reboot
```
reboot
```

## 🧪 Installation Verification

### Driver and Card Check
```
nvidia-smi
```

### OpenGL Check
```
glxinfo | grep "OpenGL renderer"
```

### Vulkan Check
```
vulkaninfo | grep "deviceName"
```

### Performance Test
```
glxgears
```

### Performance Configuration
```
sudo nvidia-settings
```

---

**When configuring various applications, I explored ready-made configurations from other users. I selected the most effective elements from different setups, adapted them to my needs, and integrated them into my system.**
