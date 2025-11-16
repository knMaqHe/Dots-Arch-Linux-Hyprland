## Руководство по установке Arch Linux с окружением Hyprland рядом с Windows 11

## Зависимости:
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
pacman -S --needed hyprland wayland wayland-protocols xdg-desktop-portal-hyprland \
    waybar polkit-gnome \
    hyprpaper hyprlock hypridle hyprpolkitagent \
    mesa vulkan-icd-loader pipewire
```

### Дисплейный менеджер
```
pacman -S --needed sddm sddm-kcm qt6-svg qt6-virtualkeyboard qt6-multimedia-ffmpeg
systemctl enable sddm
```

### Основные приложения
```
pacman -S --needed alacritty nautilus rofi-wayland firefox swaync
```

### Системные утилиты
```
pacman -S grim slurp wl-clipboard cliphist
pacman -S brightnessctl playerctl wireplumber
pacman -S udiskie udisks2 file-roller
pacman -S git curl wget dbus
```

### Звуковая система
```
pacman -S pulseaudio pulseaudio-alsa pavucontrol
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

## 🎨 Настройка окружения

### Создание конфигурационных директорий
```
mkdir -p /home/username/.config/hypr
mkdir -p /home/username/.config/waybar
mkdir -p /home/username/Images/Wallpaper
```

### Загрузка конфигов с репозитория GitHub
```
git clone https://github.com/knMaqHe/Dots-Arch-Linux-Hyprland.git
```

### Установка конфигов
Переместите соответствующие папки с конфигурациями для Hyprland, rofi, waybar, alacritty в директорию `~/.config`. Папку `Images` переместите в домашнюю директорию пользователя

### Установка кастомной темы для SDDM
Переместите папку `sddm-astronaut-theme` в директорию `/usr/share/sddm/themes` и выполните:
```
echo "[Theme]
Current=sddm-astronaut-theme" | sudo tee /etc/sddm.conf
echo "[General]
InputMethod=qtvirtualkeyboard" | sudo tee /etc/sddm.conf.d/virtualkbd.conf
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
chmod 777 yay 
cd yay
makepkg -si
cd .. && rm -rf yay
```

### Установка курсора Bibata
```
yay -S bibata-cursor-theme
echo "exec-once = hyprctl setcursor Bibata-Modern-Ice 24" >> ~/.config/hypr/source/autostart.conf
echo "env = XCURSOR_THEME,Bibata-Modern-Ice
env = XCURSOR_SIZE,24" >> ~/.config/hypr/source/environment.conf
```

### Настройка звука
```
# Основные пакеты PipeWire
sudo pacman -S pipewire pipewire-pulse pipewire-audio pipewire-alsa
sudo pacman -S wireplumber

# Дополнительные кодекки и поддержка
sudo pacman -S gst-plugin-pipewire gst-plugins-good gst-plugins-bad gst-plugins-ugly

# GUI для управления звуком
sudo pacman -S helvum pavucontrol
```

### Включение сервисов
```
# Для пользовательского уровня
systemctl --user enable pipewire pipewire-pulse wireplumber
systemctl --user start pipewire pipewire-pulse wireplumber

# Проверка статуса
systemctl --user status pipewire
systemctl --user status wireplumber
```

---

**В процессе настройки конфигураций для различных приложений я изучал готовые решения других пользователей. Я брал наиболее удачные элементы из разных конфигураций, адаптировал их под свои потребности и интегрировал в свою систему.**

---
