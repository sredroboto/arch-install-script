# sred-box install (EM DESENVOLVIMENTO)

## Texto Guia de como fazer toda a minha instalação do Arch Linux.

Caso não queira seguir o guia manualmente tem o script bash que já faz tudo automatizado.
(O objetivo desse texto é ter em um só lugar tudo o que eu faço manualmente para ter o meu Setup Up and Running e me dar um norte de como vou fazer o meu script Bash.)

```bash
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
  <tbody>
    <tr>
      <th>Mount point on the installed system</th>
      <th>Partition</th>
      <th>
        <a
          href="https://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_type_GUIDs"
          class="extiw"
          title="wikipedia:GUID Partition Table"
          >Partition type GUID</a
        >
      </th>
      <th>
        <a
          href="https://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_entries_.28LBA_2.E2.80.9333.29"
          class="extiw"
          title="wikipedia:GUID Partition Table"
          >Partition attributes</a
        >
      </th>
      <th>Suggested size</th>
    </tr>
    <tr>
      <td><code>/boot</code> or <code>/efi</code></td>
      <td><code>/dev/sda1</code></td>
      <td>
        <code>C12A7328-F81F-11D2-BA4B-00A0C93EC93B</code>:
        <a href="/index.php/EFI_system_partition" title="EFI system partition"
          >EFI system partition</a
        >
      </td>
      <td></td>
      <td>260 MiB</td>
    </tr>
    <tr>
      <td><code>/</code></td>
      <td><code>/dev/sda2</code></td>
      <td>
        <code>4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709</code>: Linux x86-64 root (/)
      </td>
      <td></td>
      <td>23–32 GiB</td>
    </tr>
    <tr>
      <td><code>[SWAP]</code></td>
      <td><code>/dev/sda3</code></td>
      <td>
        <code>0657FD6D-A4AB-43C4-84E5-0933C84B4F4F</code>: Linux
        <a href="/index.php/Swap" title="Swap">swap</a>
      </td>
      <td></td>
      <td>More than 512 MiB</td>
    </tr>
    <tr>
      <td><code>/home</code></td>
      <td><code>/dev/sda4</code></td>
      <td><code>933AC7E1-2EB4-4F13-B844-0E14E2AEF915</code>: Linux /home</td>
      <td></td>
      <td>Remainder of the device</td>
    </tr>
  </tbody>
</table>

### BIOS/GPT example layout

<table>
  <tbody>
    <tr>
      <th>Mount point on the installed system</th>
      <th>Partition</th>
      <th>
        <a
          href="https://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_type_GUIDs"
          class="extiw"
          title="wikipedia:GUID Partition Table"
          >Partition type GUID</a
        >
      </th>
      <th>
        <a
          href="https://en.wikipedia.org/wiki/GUID_Partition_Table#Partition_entries_.28LBA_2.E2.80.9333.29"
          class="extiw"
          title="wikipedia:GUID Partition Table"
          >Partition attributes</a
        >
      </th>
      <th>Suggested size</th>
    </tr>
    <tr>
      <td>None</td>
      <td><code>/dev/sda1</code></td>
      <td>
        <code>21686148-6449-6E6F-744E-656564454649</code>:
        <a
          href="/index.php/BIOS_boot_partition"
          class="mw-redirect"
          title="BIOS boot partition"
          >BIOS boot partition</a
        ><sup>1</sup>
      </td>
      <td></td>
      <td>1 MiB</td>
    </tr>
    <tr>
      <td><code>/</code></td>
      <td><code>/dev/sda2</code></td>
      <td>
        <code>4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709</code>: Linux x86-64 root (/)
      </td>
      <td><code>2</code>: Legacy BIOS bootable</td>
      <td>23–32 GiB</td>
    </tr>
    <tr>
      <td><code>[SWAP]</code></td>
      <td><code>/dev/sda3</code></td>
      <td>
        <code>0657FD6D-A4AB-43C4-84E5-0933C84B4F4F</code>: Linux
        <a href="/index.php/Swap" title="Swap">swap</a>
      </td>
      <td></td>
      <td>More than 512 MiB</td>
    </tr>
    <tr>
      <td><code>/home</code></td>
      <td><code>/dev/sda4</code></td>
      <td><code>933AC7E1-2EB4-4F13-B844-0E14E2AEF915</code>: Linux /home</td>
      <td></td>
      <td>Remainder of the device</td>
    </tr>
  </tbody>
</table>

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
pacstrap /mnt base base-devel vim linux-hardened linux-firmware grub **efibootmgr** networkmanager

genfstab -U /mnt >> /mnt/etc/fstab
```

### chroot

```bash
arch-chroot /mnt
pacman -Syu
pacman -S apparmor
```

### Fuso Horario

```bash

ln -sf /usr/share/zoneinfo/America/Fortaleza /etc/localtime
hwclock --systohc

```

### Localization

Uncomment en_US.UTF-8 pt_BR.UTF-8 and other needed locales in /etc/locale.gen, and generate them with:

```bash
locale-gen
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

### Network configuration

Create the hostname file.

```bash
#/etc/hostname
myhostname (nome que você queira para a maquina)
```

Add matching entries to hosts(5)

```bash
#/etc/hosts

127.0.0.1   localhost
::1         localhost
127.0.1.1   myhostname.localdomain  myhostname
```

### Preparing the boot partition

```bash
# mkfs.ext4 /dev/sdb1 or mkfs.fat -F32 /dev/sdb1
# mkdir /mnt/boot
# mount /dev/sdb1 /mnt/boot

```

### Configuring mkinitcpio

```bash
#Entre com o vim nesse arquivo /etc/mkinitcpio.conf e adicione o que está **marcado** ao HOOKS.

HOOKS=(base **udev** autodetect **keyboard** keymap consolefont modconf block **encrypt** **lvm2** filesystems fsck)

# depois salve e execute o comando.

mkinitcpio -p linux-hardened
```

### Configuring the boot loader

```bash
#Edite o /etc/default/grub

#Adicione a GRUB_CMDLINE_LINUX_DEFAULT
cryptdevice=UUID=device-UUID:cryptlvm root=/dev/MyVolGroup/root

#Adicione ao GRUB_CMDLINE_LINUX (se não estiver lá adicione um)
apparmor=1 security=apparmor audit=1

# E descomente a linha (se ela não existir adicione)
GRUB_ENABLE_CRYPTODISK=y
```

### Ativar hardening e instalação do GRUB

#### Execute ativa o apparmor auditd

```bash

systemctl enable apparmor
systemctl enable auditd
```

#### Caso UEFI

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB --recheck
```

#### Caso BIOS

```bash
grub-install --target=i386-pc /dev/sdX --recheck
```

#### Finalizando configuração do GRUB com o Kernel

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Depois disso tudo de `exit` umount todos os discos e reinicie, se inciar o GRUB com o sistema operacional deu tudo certo.
E ao inciar de o comando `sudo apparmor_status` caso mostre 44 profiles deu tudo certo.

## Firejail

Instale o git e de clone nesse repositorio
[Firejail](https://github.com/netblue30/firejail)
depois entre dentro da pasta e execute

```bash
./configure --prefix=/usr --enable-apparmor
make
sudo make install-strip
```

To enable its AppArmor profile, execute :
`sudo aa-enforce firejail-default` If apparmor throws error about duplicate lines in a specific directory, simply go to that directory and comment the duplicate lines.

For some reason it throws an error about some "/ or variable", but when you restart and run sudo apparmor_status it should show firejail-default in the profiles.

## X Window System

```bash
sudo pacman -S xorg
```

## Instalação do Desktop environment KDE

```bash
sudo pacman -S plasma kde-applications
```

## Instalação Display Manager sddm

```bash
sudo pacman -S sddm sddm-kcm
sudo systemctl enable sddm.service
```

## Modulos da ucode

### Intel

```bash
sudo pacman -S intel-ucode
```

### AMD

```bash
sudo pacman -S amd-ucode
```

Depois reconfigure o grub.

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```


## Modulos da Placa de Vídeo

### Escolha um e instale

<table class="wikitable" style="text-align: center;">
  <tbody>
    <tr>
      <th>Brand</th>
      <th>Type</th>
      <th>Driver</th>
      <th>OpenGL</th>
      <th>
        OpenGL (<a
          href="/index.php/Multilib"
          class="mw-redirect"
          title="Multilib"
          >multilib</a
        >)
      </th>
      <th>Documentation</th>
    </tr>
    <tr>
      <th rowspan="2">AMD / ATI</th>
      <td rowspan="2">Open source</td>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=xf86-video-amdgpu"
            >xf86-video-amdgpu</a
          ></span
        >
      </td>
      <td rowspan="2">
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=mesa"
            >mesa</a
          ></span
        >
      </td>
      <td rowspan="2">
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=lib32-mesa"
            >lib32-mesa</a
          ></span
        >
      </td>
      <td><a href="/index.php/AMDGPU" title="AMDGPU">AMDGPU</a></td>
    </tr>
    <tr>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=xf86-video-ati"
            >xf86-video-ati</a
          ></span
        >
      </td>
      <td><a href="/index.php/ATI" title="ATI">ATI</a></td>
    </tr>
    <tr>
      <th>Intel</th>
      <td>Open source</td>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=xf86-video-intel"
            >xf86-video-intel</a
          ></span
        >
      </td>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=mesa"
            >mesa</a
          ></span
        >
      </td>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=lib32-mesa"
            >lib32-mesa</a
          ></span
        >
      </td>
      <td>
        <a href="/index.php/Intel_graphics" title="Intel graphics"
          >Intel graphics</a
        >
      </td>
    </tr>
    <tr>
      <th rowspan="3">NVIDIA</th>
      <td>Open source</td>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=xf86-video-nouveau"
            >xf86-video-nouveau</a
          ></span
        >
      </td>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=mesa"
            >mesa</a
          ></span
        >
      </td>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=lib32-mesa"
            >lib32-mesa</a
          ></span
        >
      </td>
      <td><a href="/index.php/Nouveau" title="Nouveau">Nouveau</a></td>
    </tr>
    <tr>
      <td rowspan="2">Proprietary</td>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=nvidia"
            >nvidia</a
          ></span
        >
      </td>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=nvidia-utils"
            >nvidia-utils</a
          ></span
        >
      </td>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://www.archlinux.org/packages/?name=lib32-nvidia-utils"
            >lib32-nvidia-utils</a
          ></span
        >
      </td>
      <td rowspan="2"><a href="/index.php/NVIDIA" title="NVIDIA">NVIDIA</a></td>
    </tr>
    <tr>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://aur.archlinux.org/packages/nvidia-390xx/"
            >nvidia-390xx</a
          ></span
        ><sup><small>AUR</small></sup>
      </td>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://aur.archlinux.org/packages/nvidia-390xx-utils/"
            >nvidia-390xx-utils</a
          ></span
        ><sup><small>AUR</small></sup>
      </td>
      <td>
        <span class="plainlinks archwiki-template-pkg"
          ><a
            rel="nofollow"
            class="external text"
            href="https://aur.archlinux.org/packages/lib32-nvidia-390xx-utils/"
            >lib32-nvidia-390xx-utils</a
          ></span
        ><sup><small>AUR</small></sup>
      </td>
    </tr>
  </tbody>
</table>

### MESA

```bash
sudo pacman -S mesa lib32-mesa
```

### Extra ATI

```bash
sudo pacman -S xf86-video-ati mesa-vdpau lib32-mesa-vdpau
```

### Extra Intel

```bash
sudo pacman -S vulkan-intel
```

### Nvidia

[Leia a wiki(complicado demais)](https://wiki.archlinux.org/index.php/NVIDIA)

## Instalação de audio

```bash
sudo pacman -S alsa-utils pulseaudio-alsa pulseaudio-bluetooth
```

## Instalação de bluethooth

```bash
sudo pacman -S bluez bluez-utils
# Ativar serviço
sudo systemctl enable bluetooth.service
sudo systemctl start bluetooth.service
```

## Mirrorlist por velocidade

### Faça um backup

```bash
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```

### Instale

```bash
sudo pacman -S reflector
```

### Execute

```bash
 reflector --latest 200 --protocol https --sort rate --save /etc/pacman.d/mirrorlist

```

## JAVA

```bash
sudo pacman -S jre-openjdk-headless jre-openjdk jdk-openjdk openjdk-doc openjdk-src jre11-openjdk-headless jre11-openjdk jdk11-openjdk openjdk11-doc openjdk11-src
```

## Packages sortidos

### Web browsers

```bash
sudo pacman -S lynx firefox
```

### File Sharing

```bash
sudo pacman -S youtube-dl
```

### Torrent

```bash
sudo pacman -S qbittorrent
```

### Email Client

```bash
sudo pacman -S thunderbird
```

### IRC Client

```bash
sudo pacman -S hexchat
```

### file explorer

```bash
sudo pacman -S ranger
```

### Video Player

```bash
sudo pacman -S vlc mpv
```

### Image Viwer

```bash
sudo pacman -S feh
```

### Raster graphics editors

```bash
sudo pacman -S gimp krita
```

### Vector graphics editors

```bash
sudo pacman -S inkscape
```

### 3D computer graphics

```bash
sudo pacman -S blender
```

### Audio players

```bash
sudo pacman -S cmus rhythmbox
```

### Audio editors

```bash
sudo pacman -S audacity
```

### Digital audio workstations

```bash
sudo pacman -S lmms
```

### Audio synthesis environments

```bash
sudo pacman -S supercollider csound
```

### Sound generators

```bash
sudo pacman -S hydrogen
```

### Video editors

```bash
sudo pacman -S  kdenlive
```

### Subtitle editors

Ainda em busca

### Screencast

```bash
sudo pacman -S obs-studio
```

### Disk usage display

```bash
sudo pacman -S ncdu
```

### Task managers

```bash
sudo pacman -S htop
```

### Code Editor

```bash
sudo pacman -S code
```

### Office

```bash
sudo pacman -S libreoffice-still-pt-br
```

### Latex - Text Converter - Markdown

```bash
sudo pacman -S texlive-most pandoc
```

## O QUE AINDA FALTA ADICIONAR/FAZER

- Adicionar os usuários necessarios e o tipo de root (Script ou deixar o proprio usuário fazer?).
- Organizar a mirrorlist de acordo com a velocidade e tipo. (Ver Arch Wiki sobre isso)
- Olhar o [Category:Laptops](https://wiki.archlinux.org/index.php/Category:Laptops) para instalar o que é necessario para o notebook
- Sistemas de arquivos
- Setting up a firewall
- Instalação de Fontes necessárias
- Listas de aplicativos necessários para o meu workflow.
- Instalação dos apps necessários ao meu dia a dia.
- O que é necessário ao pacman.conf
- Criar um script/service para roda o reflector.
- Adicionar forma de editar o pacman.conf
- Instalar o yay
- Instalar pacotes do AUR

## Fontes

[Arch Wiki: Installation_guide](https://wiki.archlinux.org/index.php/Installation_guide)

[Arch Wiki: General Recommendations](https://wiki.archlinux.org/index.php/General_recommendations)

[Reddit: creating_a_hardened_arch_linux](https://www.reddit.com/r/linux/comments/9galhz/creating_a_hardened_arch_linux_installation_with/)

[Arch Wiki: Encrypting_an_entire_system#LVM_on_LUKS](https://wiki.archlinux.org/index.php/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS)

[Arch Wiki: GRUB](https://wiki.archlinux.org/index.php/GRUB)

[Arch Wiki: Microcode](https://wiki.archlinux.org/index.php/Microcode)

[Arch Wiki: Reflector](https://wiki.archlinux.org/index.php/Reflector)

[Arch Wiki: Intel Graphics](https://wiki.archlinux.org/index.php/Intel_graphics)

[Arch Wiki: ATI](https://wiki.archlinux.org/index.php/ATI)

[Arch Wiki: List_of_applications](https://wiki.archlinux.org/index.php/List_of_applications#Network_connection)