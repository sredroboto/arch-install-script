# sred-box install (EM DESENVOLVIMENTO)
## Texto Guia de como fazer toda a minha instalação do Arch Linux.
Caso não queira seguir o guia manualmente tem o script bash que já faz tudo automatizado.
(O objetivo desse texto é ter em um só lugar tudo o que eu faço manualmente para ter o meu Setup Up and Running e me dar um norte de como vou fazer o meu script Bash.)

``` bash
loadkeys br-abnt2
ls /sys/firmware/efi/efivars
timedatectl set-ntp true
fdisk -l
```

## Conectar a internet Internet

### Ethernet 
Se o cabo já está conectado ao PC ao iniciar o live-usb, então já está funcionando! 
A archiso já é configurada para automaticamente reconhecer o cabo e conectar a internet.

### Wifi

AINDA VOU ESCREVER


## Partição do disco
### UEFI/GPT example layout

<table>
<tbody><tr>
<th>Mount point on the installed system
</th>
<th>Partition
</th>
<th><a href="https://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_type_GUIDs" class="extiw" title="wikipedia:GUID Partition Table">Partition type GUID</a>
</th>
<th><a href="https://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_entries_.28LBA_2.E2.80.9333.29" class="extiw" title="wikipedia:GUID Partition Table">Partition attributes</a>
</th>
<th>Suggested size
</th></tr>
<tr>
<td><code>/boot</code> or <code>/efi</code>
</td>
<td><code>/dev/sda1</code>
</td>
<td><code>C12A7328-F81F-11D2-BA4B-00A0C93EC93B</code>: <a href="/index.php/EFI_system_partition" title="EFI system partition">EFI system partition</a>
</td>
<td>
</td>
<td>260 MiB
</td></tr>
<tr>
<td><code>/</code>
</td>
<td><code>/dev/sda2</code>
</td>
<td><code>4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709</code>: Linux x86-64 root (/)
</td>
<td>
</td>
<td>23–32 GiB
</td></tr>
<tr>
<td><code>[SWAP]</code>
</td>
<td><code>/dev/sda3</code>
</td>
<td><code>0657FD6D-A4AB-43C4-84E5-0933C84B4F4F</code>: Linux <a href="/index.php/Swap" title="Swap">swap</a>
</td>
<td>
</td>
<td>More than 512 MiB
</td></tr>
<tr>
<td><code>/home</code>
</td>
<td><code>/dev/sda4</code>
</td>
<td><code>933AC7E1-2EB4-4F13-B844-0E14E2AEF915</code>: Linux /home
</td>
<td>
</td>
<td>Remainder of the device
</td></tr></tbody></table>

### BIOS/GPT example layout
<table>
<tbody><tr>
<th>Mount point on the installed system
</th>
<th>Partition
</th>
<th><a href="https://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_type_GUIDs" class="extiw" title="wikipedia:GUID Partition Table">Partition type GUID</a>
</th>
<th><a href="https://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_entries_.28LBA_2.E2.80.9333.29" class="extiw" title="wikipedia:GUID Partition Table">Partition attributes</a>
</th>
<th>Suggested size
</th></tr>
<tr>
<td>None
</td>
<td><code>/dev/sda1</code>
</td>
<td><code>21686148-6449-6E6F-744E-656564454649</code>: <a href="/index.php/BIOS_boot_partition" class="mw-redirect" title="BIOS boot partition">BIOS boot partition</a><sup>1</sup>
</td>
<td>
</td>
<td>1 MiB
</td></tr>
<tr>
<td><code>/</code>
</td>
<td><code>/dev/sda2</code>
</td>
<td><code>4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709</code>: Linux x86-64 root (/)
</td>
<td><code>2</code>: Legacy BIOS bootable
</td>
<td>23–32 GiB
</td></tr>
<tr>
<td><code>[SWAP]</code>
</td>
<td><code>/dev/sda3</code>
</td>
<td><code>0657FD6D-A4AB-43C4-84E5-0933C84B4F4F</code>: Linux <a href="/index.php/Swap" title="Swap">swap</a>
</td>
<td>
</td>
<td>More than 512 MiB
</td></tr>
<tr>
<td><code>/home</code>
</td>
<td><code>/dev/sda4</code>
</td>
<td><code>933AC7E1-2EB4-4F13-B844-0E14E2AEF915</code>: Linux /home
</td>
<td>
</td>
<td>Remainder of the device
</td></tr></tbody></table>

## LVM on LUKS

```
+-----------------------------------------------------------------------+ +----------------+
| Logical volume 1      | Logical volume 2      | Logical volume 3      | | Boot partition |
|                       |                       |                       | |                |
| [SWAP]                | /                     | /home                 | | /boot          |
|                       |                       |                       | |                |
| /dev/MyVolGroup/swap  | /dev/MyVolGroup/root  | /dev/MyVolGroup/home  | |                |
|_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _|_ _ _ _ _ _ _ _ _ _ _ _| | (may be on     |
|                                                                       | | other device)  |
|                         LUKS2 encrypted partition                     | |                |
|                           /dev/sda1                                   | | /dev/sdb1      |
+-----------------------------------------------------------------------+ +----------------+
```
### Preparing the disk
```bash
# cryptsetup luksFormat /dev/sda1
# cryptsetup open /dev/sda1 cryptlvm

# pvcreate /dev/mapper/cryptlvm

# vgcreate MyVolGroup /dev/mapper/cryptlvm

# lvcreate -L 8G MyVolGroup -n swap
# lvcreate -L 32G MyVolGroup -n root
# lvcreate -l 100%FREE MyVolGroup -n home

# mkfs.ext4 /dev/MyVolGroup/root
# mkfs.ext4 /dev/MyVolGroup/home
# mkswap /dev/MyVolGroup/swap

# mount /dev/MyVolGroup/root /mnt
# mkdir /mnt/home
# mount /dev/MyVolGroup/home /mnt/home
# swapon /dev/MyVolGroup/swap


```
## Instalação

```bash
# pacstrap /mnt base base-devel vim linux-hardened grub **efibootmgr**

# genfstab -U /mnt >> /mnt/etc/fstab
```

### chroot

```bash
# arch-chroot /mnt
# pacman -Syu
# pacman -S apparmor
```

Fuso Horario
``` bash

# ln -sf /usr/share/zoneinfo/America/Fortaleza /etc/localtime
# hwclock --systohc

```


Localization
Uncomment en_US.UTF-8 pt_BR.UTF-8 and other needed locales in /etc/locale.gen, and generate them with:

```bash
# locale-gen
```
Create the locale.conf(5) file, and set the LANG variable accordingly:
```bash
#/etc/locale.conf

LANG=en_US.UTF-8
```

If you set the keyboard layout, make the changes persistent in vconsole.conf(5): 

```bash
#/etc/vconsole.conf

KEYMAP=de-latin1
```

Network configuration
Create the hostname file: 
```bash
#/etc/hostname

myhostname (nome que você queira para a maquina)

```
Add matching entries to hosts(5): 

```
#/etc/hosts

127.0.0.1	localhost
::1		    localhost
127.0.1.1	myhostname.localdomain	myhostname
```

### Preparing the boot partition
```bash
# mkfs.ext4 /dev/sdb1 or mkfs.fat -F32 /dev/sdb1
# mkdir /mnt/boot
# mount /dev/sdb1 /mnt/boot

```
### Configuring mkinitcpio

```
Entre com o vim nesse arquivo /etc/mkinitcpio.conf
e adicione o que está **marcado** ao HOOKS.

# HOOKS=(base **udev** autodetect **keyboard** keymap consolefont modconf block **encrypt** **lvm2** filesystems fsck)

# depois salve e execute o comando.

mkinitcpio -p linux-hardened
```
### Configuring the boot loader

```
#Edite o /etc/default/grub.
#Adicione a GRUB_CMDLINE_LINUX_DEFAULT
cryptdevice=UUID=device-UUID:cryptlvm root=/dev/MyVolGroup/root
#Depois
#Adicione ao GRUB_CMDLINE_LINUX (se não estiver lá adicione um)
apparmor=1 security=apparmor audit=1

# E descomente a linha:
GRUB_ENABLE_CRYPTODISK=y 
#se ela não existir adicione
```
### Depois
```bash
#Execute 

systemctl enable apparmor
systemctl enable auditd

#Caso UEFI
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB --recheck

#Caso BIOS
grub-install --target=i386-pc /dev/sdX --recheck


#Então
grub-mkconfig -o /boot/grub/grub.cfg
```

Depois disso tudo de ```exit``` umount todos os discos e reinicie, se inciar o GRUB com o sistema operacional deu tudo certo.

E ao inciar de o comando ```sudo apparmor_status``` caso mostre 44 profiles deu tudo certo.

## Firejail
Instale o git e de clone nesse repositorio
https://github.com/netblue30/firejail
depois entre dentro da pasta e execute

```bash
./configure --prefix=/usr --enable-apparmor
 
make

sudo make install-strip
```
To enable its AppArmor profile, execute :
```sudo aa-enforce firejail-default``` If apparmor throws error about duplicate lines in a specific directory, simply go to that directory and comment the duplicate lines.

For some reason it throws an error about some "/ or variable", but when you restart and run sudo apparmor_status it should show firejail-default in the profiles.

# X Window System

```bash
sudo pacman -S xorg
```


## O QUE AINDA FALTA ADICIONAR/FAZER
* Instalação do Xorg e modulos da placa de vídeo.
* Adicionar os usuários necessarios e o tipo de root.
* Organizar a mirrorlist de acordo com a velocidade e tipo.
* xdg-user-dirs-update
* Olhar o [Category:Laptops](https://wiki.archlinux.org/index.php/Category:Laptops) para instalar o que é necessario para o notebook
* Instalar Codecs 
* Sistemas de arquivos
* Setting up a firewall
* Instalação de Fontes necessárias
* Temas gtk,qt.
* Gerenciamento de sessão.
* Instalação do ALSA e PULSEAUDIO para pode ter audio no sistema.
* Instalação da WM (dwm) execução(dmenu), terminal(st) e dotfiles.
* Listas de aplicativos necessários para o meu workflow.
* Instalação dos apps necessários ao meu dia a dia.
* O que é necessário ao pacman.conf

Fontes:

https://wiki.archlinux.org/index.php/Installation_guide
https://wiki.archlinux.org/index.php/General_recommendations
https://www.reddit.com/r/linux/comments/9galhz/creating_a_hardened_arch_linux_installation_with/
https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS
https://wiki.archlinux.org/index.php/GRUB