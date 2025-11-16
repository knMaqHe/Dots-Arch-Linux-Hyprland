## Arch Linux Installation Guide with Hyprland alongside Windows 11

## If you already have Arch Linux installed and want to skip to configuration setup, install these required dependencies:
```
pacman -S --needed hyprland wayland wayland-protocols xdg-desktop-portal-hyprland \
    waybar polkit-gnome \
    hyprpaper hyprlock hypridle hyprpolkitagent \
    mesa vulkan-icd-loader pipewire
pacman -S --needed alacritty nautilus rofi-wayland firefox swaync
pacman -S --needed sddm sddm-kcm qt6-svg qt6-virtualkeyboard qt6-multimedia-ffmpeg
pacman -S --needed ttf-firacode-nerd noto-fonts noto-fonts-emoji ttf-dejavu
pacman -S --needed ttf-nerd-fonts-symbols-mono ttf-ibm-plex
pacman -S --needed grim slurp wl-clipboard cliphist
pacman -S --needed brightnessctl playerctl wireplumber
pacman -S --needed udiskie udisks2 file-roller
pacman -S --needed git curl wget dbus
pacman -S --needed pulseaudio pulseaudio-alsa pavucontrol
pacman -S --needed papirus-icon-theme lxappearance kvantum nwg-look
```

## Preparation for Installation

### Network Connection and Verification
Connect the USB drive with the downloaded Arch Linux image to your computer. Check network connectivity:
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
- ➕ **New EFI Partition for Arch** - 1GB (type: EFI System)
- ➕ **Root Partition** - remaining space (type: Linux filesystem)

### Formatting Partitions
```
# Formatting EFI Partition
mkfs.fat -F32 /dev/sda1

# Formatting Root Partition
mkfs.ext4 /dev/sda2
```

### Mounting Partitions
```
mount /dev/sda2 /mnt
mount --mkdir /dev/sda1 /mnt/boot
```

## 📦 Installing Base System

```
# Installing Base Packages
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

# Timezone
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc

# Localization
nano /etc/locale.gen # Uncomment lines en_US.UTF-8 UTF-8 and ru_RU.UTF-8 UTF-8
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Keyboard Layout
echo "KEYMAP=ru" > /etc/vconsole.conf
echo "FONT=cyr-sun16" >> /etc/vconsole.conf
```

### Network Configuration
```
# Hosts Configuration
cat > /etc/hosts << EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   archlinux.localdomain archlinux
EOF

# Enabling Network Services
systemctl enable NetworkManager
systemctl enable dhcpcd
```

### User Creation
```
# Root Password
passwd

# Creating Regular User
useradd -m -G wheel -s /bin/bash username
passwd username

# sudo Configuration
pacman -S sudo
EDITOR=nano visudo
# Uncomment the line: %wheel ALL=(ALL) ALL
```

## 🎯 Bootloader Installation

```
# Installing GRUB and Utilities
pacman -S grub efibootmgr os-prober

# Enable Windows Support in GRUB:
echo "GRUB_DISABLE_OS_PROBER=false" >> /etc/default/grub
os-prober

# GRUB Installation
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch --recheck

# When installing Arch Linux alongside Windows 11, you might encounter an issue where the Arch Linux bootloader doesn't appear in the boot menu (appears when starting PC after pressing F11 in my case) - these commands helped me solve this problem
grub-install --efi-directory=/boot --removable
grub-install --efi-directory=/boot --target=x86_64-efi --bootloader-id=Arch --recheck

# Final Configuration
grub-mkconfig -o /boot/grub/grub.cfg
```

## 🖥️ Graphical Environment Installation

### Hyprland and Wayland Components
```
pacman -S --needed hyprland wayland wayland-protocols xdg-desktop-portal-hyprland \
    waybar polkit-gnome \
    hyprpaper hyprlock hypridle hyprpolkitagent \
    mesa vulkan-icd-loader pipewire
```

### Display Manager
```
pacman -S --needed sddm sddm-kcm qt6-svg qt6-virtualkeyboard qt6-multimedia-ffmpeg
systemctl enable sddm
```

### Main Applications
```
pacman -S --needed alacritty nautilus rofi-wayland firefox swaync
```

### System Utilities
```
pacman -S grim slurp wl-clipboard cliphist
pacman -S brightnessctl playerctl wireplumber
pacman -S udiskie udisks2 file-roller
pacman -S git curl wget dbus
```

### Audio System
```
pacman -S pulseaudio pulseaudio-alsa pavucontrol
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

## 🎨 Environment Configuration

### Creating Configuration Directories
```
mkdir -p /home/username/.config/hypr
mkdir -p /home/username/.config/waybar
mkdir -p /home/username/Images/Wallpaper
```

### Downloading Configs from GitHub Repository
```
git clone https://github.com/knMaqHe/Dots-Arch-Linux-Hyprland.git
```

### Installing Configs
Move the corresponding configuration folders for Hyprland, rofi, waybar, alacritty to the `~/.config` directory. Move the `Images` folder to the user's home directory.

### Installing Custom Theme for SDDM
Move the `sddm-astronaut-theme` folder to the `/usr/share/sddm/themes` directory and execute:
```
echo "[Theme]
Current=sddm-astronaut-theme" | sudo tee /etc/sddm.conf
echo "[General]
InputMethod=qtvirtualkeyboard" | sudo tee /etc/sddm.conf.d/virtualkbd.conf
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

### Installing AUR Helper (yay)
```
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
chmod 777 yay 
cd yay
makepkg -si
cd .. && rm -rf yay
```

### Installing Bibata Cursor
```
yay -S bibata-cursor-theme
echo "exec-once = hyprctl setcursor Bibata-Modern-Ice 24" >> ~/.config/hypr/source/autostart.conf
echo "env = XCURSOR_THEME,Bibata-Modern-Ice
env = XCURSOR_SIZE,24" >> ~/.config/hypr/source/environment.conf
```

---

**During the configuration setup for various applications, I studied ready-made solutions from other users. I took the most successful elements from different configurations, adapted them to my needs, and integrated them into my system.**

---
