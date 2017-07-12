# Arch Linux installation & setup notes

## Install image

#### create + mount filesystem
- `gdisk /dev/[sda]`

#### uefi
- create 512MB partition with type `ef00`
- `mkfs.fat -F32 [partition]`
- `parted`:
  - toggle [partition #] boot
for each partition:
- `mkfs.[ext4] [partition]`
- `mount [partition] /mnt/[mount point]`

#### pacman
- edit */etc/pacman.d/mirrorlist*
- or: **reflector** `reflector --verbose --sort rate -c Canada -n 10 --save /etc/pacman.d/mirrorlist`

#### install
- `pacstrap /mnt base`
- `genfstab -p /mnt >> /mnt/etc/fstab`

#### inside chroot
- `arch-chroot /mnt`
  - `echo "[hostname]" > /etc/hostname`
  - `ln -s /usr/share/zoneinfo/[zone]/[subzone] /etc/localtime`
  - */etc/locale.gen*: uncomment locale
  - `locale-gen`
  - */etc/locale.conf*:
    - `LANG=[en_CA.utf-8]`
    - `LC_COLLATE=C`
  - */etc/mkinitcpio.conf*:
    - `HOOKS="[...] autodetect filesystems fsck block keyboard"`
  - `mkinitcpio -p linux`
  - `passwd`

#### grub
- **grub**
- edit */etc/default/grub*
- `grub-mkconfig -o /boot/grub/grub.cfg`
- mbr:
  - `grub-install --target=i386-pc --recheck /dev/[sda]`
- uefi:
  - **efibootmgr**
  - `mount [efi partition] [temp mount point]`
  - `grub-install --target=x86_64-efi --efi-directory=[mount point] --bootloader-id=grub --recheck`
- bugfix for multiple grub entries: `chmod 644 /etc/grub.d/10_linux`
- bugfix for grub error message: `ln -s /boot/grub/en\@quot.mo /boot/grub/en.mo`

#### refind
- **refind-efi**
- `mkdir /boot/efi`
- `mount /dev/[sda1] /boot/efi`
- `refind-install`
- `refind-mkrlconf`


## Installed system

#### uefi + grub + dual boot
- WINDOWS 7/8 COMMAND PROMPT (Administrator):
  - `powercfg -h off` (turn off hibernation. otherwise win7/8 overwrites grub's boot entry in efi on reboot)
- or:
  - `bcdedit /set {bootmgr} path \EFI\grub\grubx64.efi` (set windows boot manager to load grub's efi file)

#### network
- wired: `systemctl enable dhcpcd`
- wireless: **networkmanager** `systemctl enable NetworkManager`

#### pacman
- */etc/pacman.conf*:
  - `Color`
  - `TotalDownload`
  - `VerbosePkgLists`
  - `ILoveCandy`
  - uncomment `[multilib]` section

#### shell and users
- **zsh sudo**
- `useradd -m -G wheel -s /usr/bin/zsh marcus`
- `passwd marcus`
- `visudo`:
  - `%wheel ALL=(ALL) ALL`

#### yaourt
- **base-devel git**
- `git clone https://aur.archlinux.org/package-query`
- `git clone https://aur.archlinux.org/yaourt`
- (from each directory) `makepkg -si`

#### misc
- **mlocate** `updatedb`
- **rsyslog** `systemctl enable rsyslog`
- **terminus-font**
- */etc/vconsole.conf*:
  - `FONT=[ter-v16b]`

#### filesystem
- **ntfs-3g**
- */etc/fstab*:
  - `UUID=[uuid] [mountpoint] ntfs defaults,gid=users,umask=0000 0 0`
- **udevil** `systemctl enable devmon@[user]`

#### cups server
- **cups cups-filters aur:samsung-unified-driver**
- `systemctl enable cups`
- *http://localhost:631*:
  - Administration > Printers > Add Printer
  - Administration > Advanced > Share printers
- `systemctl enable avahi-daemon`
- `systemctl enable cups-browsed`
- */etc/cups/cupsd.conf*:
  - `Listen [ip]:631`

#### cups client
- **libcups**
- */etc/cups/client.conf*:
  - `ServerName`

## GUI
- **xorg-server xorg-xinit [ nvidia | catalyst | xf86-video-[vesa|intel|ati|nouveau] ]**

#### sddm
- **sddm aur:[archlinux-themes-sddm|sddm-archlinux-theme-git]**
- */etc/sddm.conf*:
  - `[Theme]`
  - `Current=[archlinux-simplyblack]`

#### lightdm
- **lightdm aur:lightdm-webkit2-greeter aur:lightdm-webkit2-theme-material2**
- */etc/lightdm/lightdm.conf*:
  - `greeter-session=lightdm-webkit2-greeter`
- */etc/lightdm/lightdm-webkit2-greeter.conf*:
  - `webkit-theme=material2`

#### openbox
- **openbox obconf openbox-themes**
- *~/.xprofile*:
  - `exec openbox-session`
- **menumaker nitrogen tint2 aur:compton**
- **rxvt-unicode**
- **ttf-[liberation|droid|roboto] aur:inconsolata-g**
- *~/.config/openbox/Autostart*:
  - `mmaker -f openbox`
  - `nitrogen --restore`
  - `tint2 &`
  - `compton &`

#### fancy gui
- **plasma-workspace qt5ct breeze-kde4**
- *~/.config/openbox/environment*:
  - `export QT_QPA_PLATFORMTHEME=qt5ct`
- `qt5ct`
- `qtconfig-qt4`
