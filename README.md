## Руководство по установке Arch Linux с окружением Hyprland рядом с Windows 11

## Подготовка к установке

### Подключение и проверка сети
Подключите флешку с загруженным образом Arch Linux к компьютеру. Проверьте подключение к сети:
```
ping -c 4 google.com
```

### Просмотр текущей разметки
```
lsblk
fdisk -l
```
Определите диск, на который будет устанавливаться Arch Linux (например, `/dev/sda` или `/dev/sdc`).

### Создание разделов с cfdisk
```
cfdisk /dev/sda  # замените на ваш диск
```

**Схема разделов:**
- ➕ **Новый EFI раздел для Arch** - 1ГиБ (тип: EFI System)
- ➕ **Корневой раздел** - оставшееся место (тип: Linux filesystem)

### Форматирование разделов
```
# Форматирование EFI раздела
mkfs.fat -F32 /dev/sda1

# Форматирование корневого раздела
mkfs.ext4 /dev/sda2
```

### Монтирование разделов
```
mount /dev/sda2 /mnt
mount --mkdir /dev/sda1 /mnt/boot
```

## 📦 Установка базовой системы

```
# Установка базовых пакетов
pacstrap /mnt base base-devel linux linux-headers linux-firmware nano dhcpcd networkmanager intel-ucode

# Генерация fstab
genfstab -U /mnt >> /mnt/etc/fstab
```

## 🔧 Настройка системы

### Переход в установленную систему
```
arch-chroot /mnt
```

### Настройка времени и локали
```
# Имя хоста
echo "archlinux" > /etc/hostname

# Временная зона
ln -sf /usr/share/zoneinfo/Europe/Moscow /etc/localtime
hwclock --systohc

# Локализация
nano /etc/locale.gen # Раскомментируйте строчки en_US.UTF-8 UTF-8 и ru_RU.UTF-8 UTF-8
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf

# Раскладка клавиатуры
echo "KEYMAP=ru" > /etc/vconsole.conf
echo "FONT=cyr-sun16" >> /etc/vconsole.conf
```

### Настройка сети
```
# Настройка hosts
cat > /etc/hosts << EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   archlinux.localdomain archlinux
EOF

# Включение сетевого сервиса
systemctl enable NetworkManager
systemctl enable dhcpcd
```

### Создание пользователя
```
# Пароль root
passwd

# Создание обычного пользователя
useradd -m -G wheel -s /bin/bash username
passwd username

# Настройка sudo
pacman -S sudo
EDITOR=nano visudo
# Раскомментируйте строку: %wheel ALL=(ALL) ALL
```

## 🎯 Установка загрузчика

```
# Установка GRUB и утилит
pacman -S grub efibootmgr os-prober

# Включите поддержку Windows в GRUB:
echo "GRUB_DISABLE_OS_PROBER=false" >> /etc/default/grub
os-prober

# Установка GRUB
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch --recheck

# При установке Arch Linux рядом с Windows 11 может возникнуть проблема, что загрузчик Arch Linux не будет отображаться в boot меню (появляется при запуске пк после нажатия F11 в моем случае) - данные команды помогли мне решить данную проблему
grub-install --efi-directory=/boot --removable
grub-install --efi-directory=/boot --target=x86_64-efi --bootloader-id=Arch --recheck

# Финальная конфигурация
grub-mkconfig -o /boot/grub/grub.cfg
```

## 🖥️ Установка графического окружения

### Hyprland и Wayland компоненты
```
pacman -S hyprland wayland wayland-protocols xdg-desktop-portal-hyprland \
    waybar rofi alacritty thunar polkit-gnome \
    hyprpaper hyprlock hypridle hyprpolkitagent \
    git curl wget swaync
```

### Дисплейный менеджер
```
pacman -S sddm sddm-kcm qt6-svg qt6-virtualkeyboard qt6-multimedia-ffmpeg
systemctl enable sddm
```

### Основные приложения
```
pacman -S alacritty nautilus rofi-wayland firefox
```

### Системные утилиты
```
pacman -S grim slurp wl-clipboard cliphist
pacman -S brightnessctl playerctl wireplumber
pacman -S udiskie udisks2 file-roller polkit-gnome
```

### Звуковая система
```
pacman -S pulseaudio pulseaudio-alsa pavucontrol wireplumber
```

### Шрифты
```
pacman -S ttf-firacode-nerd noto-fonts noto-fonts-emoji ttf-dejavu
pacman -S ttf-nerd-fonts-symbols-mono ttf-ibm-plex
```

### Темы и иконки
```
pacman -S papirus-icon-theme lxappearance kvantum nwg-look
```

### Курсоры
```
pacman -S bibata-cursors bibata-extra-cursors
```

## 🎨 Настройка окружения

### Создание конфигурационных директорий
```
mkdir -p /home/username/.config/hypr
mkdir -p /home/username/.config/waybar
mkdir -p /home/username/Images/Wallpaper
```

### Настройка курсора
Добавьте в `/etc/environment`:
```
XCURSOR_THEME=Bibata-Modern-Ice
XCURSOR_SIZE=24
```

### Настройка темы SDDM
```
nano /etc/sddm.conf.d/theme.conf
```

Добавьте:
```
[Theme]
Current="sddm-astronaut-theme"
```

## 🚀 Завершение установки

```
# Выход из chroot
exit

# Размонтирование
umount -R /mnt

# Перезагрузка
reboot
```

## 🛠️ Пост-установочная настройка

### Установка AUR помощника (yay)
```
sudo pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si

# Или альтернативный вариант:
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
cd .. && rm -rf yay-bin
```

### Установка тем (опционально)
```
# Тема для Nautilus
yay -S gruvbox-gtk-theme

# Тема для SDDM
yay -S sddm-theme-astronaut
```

### Установка кастомной темы для SDDM
Переместите папку `sddm-astronaut-theme` в директорию `/usr/share/sddm/themes` и выполните:
```
echo "[Theme]
Current=sddm-astronaut-theme" | sudo tee /etc/sddm.conf
```

### Установка конфигов
Переместите соответствующие папки с конфигурациями для Hyprland, rofi, waybar, alacritty в директорию `~/.config`.

## Установка драйверов NVIDIA в Arch Linux

## 📋 Быстрая установка (все команды)

```
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
```
sudo pacman -S nvidia nvidia-utils nvidia-settings
```

### Создание конфигурации modprobe
```
sudo nano /etc/modprobe.d/nvidia.conf
```

Добавьте:
```
options nvidia_drm modeset=1
```

## 🎯 Настройка для Wayland/Hyprland

### Добавление переменных окружения
```
sudo nano /etc/environment
```

Добавьте следующие строки:
```
LIBVA_DRIVER_NAME=nvidia
GBM_BACKEND=nvidia-drm
__GLX_VENDOR_LIBRARY_NAME=nvidia
WLR_NO_HARDWARE_CURSORS=1
```

### Конфигурация Hyprland
```
nano ~/.config/hypr/environment.conf
```

Добавьте:
```
env = LIBVA_DRIVER_NAME,nvidia
env = GBM_BACKEND,nvidia-drm
env = __GLX_VENDOR_LIBRARY_NAME,nvidia
env = WLR_NO_HARDWARE_CURSORS,1
```

## 🔄 Обновление системы

### Обновление initramfs
```
sudo mkinitcpio -P
```

### Перезагрузка системы
```
reboot
```

## 🧪 Проверка установки

### Проверка драйверов и карты
```
nvidia-smi
```

### Проверка OpenGL
```
glxinfo | grep "OpenGL renderer"
```

### Проверка Vulkan
```
vulkaninfo | grep "deviceName"
```

### Тест производительности
```
glxgears
```

### Настройка производительности
```
sudo nvidia-settings
```

---

**В процессе настройки конфигураций для различных приложений я изучал готовые решения других пользователей. Я брал наиболее удачные элементы из разных конфигураций, адаптировал их под свои потребности и интегрировал в свою систему.**
