> Este es un manual para la instalación de **ArchLinux** base, este comprende configuraciones, comandos, y paquetes instalados para un buen funcionamiento. Se puede agregar otros configuraciones desde la ***Wiki de Archlinux***
# Comprobando si el modo del Boot utiliza EFI (si no da erro, el computador utiliza EFI)

``` Shell
ls /sys/firmware/efi/efivars
```

# Realizando al particionado del disco sda
``` Shell
cfdisk
# 512M -> EFI
# 4GB -> SWAP
# Resto de espacio para -> ROOT
```
## Verifica las particiones creadas
``` Shell
lsblk
```

## Formateando y montado de particiones creadas

``` Shell
mkfs.fat -F32 /dev/sda1
mkfs.btrfs /dev/sda3

# Creando SWAP
mkswap /dev/sda2
swapon /dev/sda2

# Montar root
mount /dev/sda3 /mnt

# Crear y Mountar EFI
mkdir /mnt/boot
mkdir /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```
# Verificamos los mirrorlist y agregar nuevos enlaces si lo deceamos
```Shell
nano /etc/pacman.d/mirrorlist
```

# instalamos el sistema base para la particion montada a traves de pacstrap

``` Shell
pacstrap /mnt base base-devel linux-zen linux-firmware nano

#generamos un fstab
genfstab -U /mnt >> /mnt/etc/fstab
#verificamos el fstab creada
nano /mnt/etc/fstab
```
# iniciando arch, el sistema montado en /mnt (sda3)
``` Shell
arch-chroot /mnt

#definimos zona horaria
ln -sf /usr/share/zoneinfo/America/Lima /etc/localtime

#iniciamos hwclock
hwclock --systohc

#editamos /etc/locale.gen y descomentamos para idioma en ingles seria en_US.UTF-8 UTF-8
nano /etc/locale.gen

#luego de configurar en el anterior comando invoca el siguiente comando
locale-gen

#configura los archivos directo con los comando o manualmente si no hay archivo los creas
echo LANG=en_US.UTF-8 > /etc/locale.conf
echo KEYMAP=la-latin1 > /etc/vconsole.conf
echo NameHostname > /etc/hostname #identificacione de la maquina en una red

#creamos un passwd para root
passwd

#creando los host
nano /etc/hosts

#configuraciones para el comando anterior
127.0.0.1		localhost
::1					localhost
127.0.0.1   HostName.fqdomain.exmple Hostname #example Archlinux.localdomain Archlinux

#instalamos el paquete de red y lo activamos para cada inicio de sesion
pacman -S networkmanager
systemctl enable NetworkManager

#instalamos grub
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=boot/efi
grub-mkconfig -o /boot/grub/grub.cfg

#salimos de arch-chroot /mnt
exit

#desmontamos sda3
umount -R /mnt

reboot
```

# Creando PostInstall
```Shell
# Instalando Xorg
pacman -S xorg xterm xorg-xinit

# agregando usuarios
useradd -m nameuser

#creando contraseña para nameuser
passwd nameuser

#configurando permiso
nano /etc/sudoers

#agregamos el nameuser para darles permisos al igual que roots "User privilege specification"
username ALL(ALL:ALL) ALL

#instalando gestor de arranque de la parte del login
pacman -S lightdm lightdm-gtk-greeter

#configurando lightdm
nano /etc/lightdm/lightdm.conf

#buscamos la linea "greeter-session=example-gtk-gnome" y la remplazamos por
greeter-session=lightdm-gtk-greeter

#habilitar servicios
systemctl enable lightdm.service
```
