## Полное руководство по установке Arch Linux с окружением Hyprland рядом с Windows 11.

```bash
### Просмотр текущей разметки
lsblk
fdisk -l

### Создание разделов с cfdisk
```bash
cfdisk /dev/sda1  # или /dev/vda - посмотрите что у вас
```

**Схема разделов:**
- ➕ **Новый EFI раздел для Arch** - 1ГиБ (тип: EFI System)
- ➕ **Корневой раздел** - оставшееся место (тип: Linux filesystem)

### Форматирование разделов
```bash
# Форматирование EFI раздела
mkfs.fat -F32 /dev/sda1

# Форматирование корневого раздела
mkfs.ext4 /dev/sda2
```

### Монтирование разделов
```bash
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot/efi
```

## 📦 Установка базовой системы

```bash
# Установка базовых пакетов
pacstrap /mnt base base-devel linux linux-headers linux-firmware nano dhcpcd networkmanager intel-ucode

# Генерация fstab
genfstab -U /mnt >> /mnt/etc/fstab
```

## 🔧 Настройка системы

### Переход в установленную систему
```bash
arch-chroot /mnt
```

### Настройка времени и локали
```bash
# Временная зона
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc

# Локализация
nano /etc/locale.gen # Раскоментируйте строчки en_US.UTF-8 UTF-8 и ru_RU.UTF-8 UTF-8
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Раскладка клавиатуры
/etc/vconsole.conf
KEYMAP=ru
FONT=cyr-sun16
```

### Настройка сети
```bash
echo "hostname" > /etc/hostname

# Настройка hosts - hostname задайте сами
/etc/hosts
127.0.0.1   localhost
::1         localhost
127.0.1.1   hostname.localdomain hostname

# Включение сетевого сервиса
systemctl enable dhcpcd
```

### Создание пользователя
```bash
# Пароль root
passwd

# Создание обычного пользователя
useradd -m -G wheel -s /bin/bash username
passwd username

# Настройка sudo
EDITOR=nano visudo
# Раскомментируйте строку: %wheel ALL=(ALL) ALL
```

## 🎯 Установка загрузчика

```bash
# Установка GRUB и утилит
pacman -S grub efibootmgr os-prober

# Обнаружение Windows
grub-mkconfig -o /boot/grub/grub.cfg

# Установка GRUB
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch --recheck

# Финальная конфигурация
grub-mkconfig -o /boot/grub/grub.cfg
```

## 🖥️ Установка графического окружения

### Hyprland и Wayland компоненты
```bash
pacman -S hyprland wayland wayland-protocols xdg-desktop-portal-hyprland
```

### Дисплейный менеджер
```bash
pacman -S sddm sddm-kcm
systemctl enable sddm
```

### Основные приложения
```bash
pacman -S alacritty nautilus rofi-wayland firefox
```

### Системные утилиты
```bash
pacman -S grim slurp wl-clipboard cliphist
pacman -S brightnessctl playerctl wireplumber
pacman -S udiskie polkit-gnome
pacman -S swaync hypridle hyprlock
pacman -S git wget curl
```

### Звуковая система
```bash
pacman -S pulseaudio pulseaudio-alsa pavucontrol
```

### Шрифты
```bash
pacman -S ttf-firacode-nerd noto-fonts noto-fonts-emoji ttf-dejavu
pacman -S ttf-nerd-fonts-symbols-mono ttf-ibm-plex
```

### Курсоры
```bash
pacman -S bibata-cursors bibata-extra-cursors
```

## 🎨 Настройка окружения

### Создание конфигурационных директорий
```bash
mkdir -p /home/username/.config/hypr
mkdir -p /home/username/.config/waybar
mkdir -p /home/username/Images/Wallpaper
```

### Настройка курсора
Добавьте в `/etc/environment`:
```bash
XCURSOR_THEME=Bibata-Modern-Ice
XCURSOR_SIZE=24
```

## 🚀 Завершение установки

```bash
# Выход из chroot
exit

# Размонтирование
umount -R /mnt/etc/vconsole.conf

# Перезагрузка
reboot
```

## 🛠️ Пост-установочная настройка

### Установка AUR помощника (yay)
```bash
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### Установка тем (опционально)
```bash
# Тема для Nautilus
yay -S gruvbox-gtk-theme

# Тема для SDDM
yay -S sddm-theme-astronaut
```

## Установка драйверов NVIDIA в Arch Linux

```markdown

## 📋 Быстрая установка (все команды)

```bash
# Установка драйверов
sudo pacman -S nvidia nvidia-utils nvidia-settings lib32-nvidia-utils linux-headers

# Настройка ядра
echo 'options nvidia_drm modeset=1' | sudo tee /etc/modprobe.d/nvidia.conf
sudo sed -i 's/GRUB_CMDLINE_LINUX=""/GRUB_CMDLINE_LINUX="nvidia_drm.modeset=1"/' /etc/default/grub
sudo grub-mkconfig -o /boot/grub/grub.cfg

# Черный список nouveau
echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf

# Обновление initramfs
sudo mkinitcpio -P

# Перезагрузка
reboot
```

## 📦 Установка драйверов

### Основные драйверы
```bash
sudo pacman -S nvidia nvidia-utils nvidia-settings
```

### Создание конфигурации modprobe
```bash
sudo nano /etc/modprobe.d/nvidia.conf
```

Добавьте:
```bash
options nvidia_drm modeset=1
```

## 🎯 Настройка для Wayland/Hyprland

### Добавление переменных окружения
```bash
sudo nano /etc/environment
```

Добавьте следующие строки:
```bash
LIBVA_DRIVER_NAME=nvidia
GBM_BACKEND=nvidia-drm
__GLX_VENDOR_LIBRARY_NAME=nvidia
WLR_NO_HARDWARE_CURSORS=1
```

### Конфигурация Hyprland
```bash
nano ~/.config/hypr/environment.conf
```

Добавьте:
```ini
env = LIBVA_DRIVER_NAME,nvidia
env = GBM_BACKEND,nvidia-drm
env = __GLX_VENDOR_LIBRARY_NAME,nvidia
env = WLR_NO_HARDWARE_CURSORS,1
```

## 🔄 Обновление системы

### Обновление initramfs
```bash
sudo mkinitcpio -P
```

### Перезагрузка системы
```bash
reboot
```

## 🧪 Проверка установки

### Проверка драйверов и карты
```bash
nvidia-smi
```

### Проверка OpenGL
```bash
glxinfo | grep "OpenGL renderer"
```

### Проверка Vulkan
```bash
vulkaninfo | grep "deviceName"
```

### Тест производительности
```bash
glxgears
```

### Настройка производительности
```bash
sudo nvidia-settings
```