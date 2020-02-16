# sred-box install
## Texto Guia de como fazer toda a minha instalação do Arch Linux.
Caso não queira seguir o guia manualmente tem o script bash que já faz tudo automatizado.
(O objetivo desse texto é ter em um só lugar tudo o que eu faço manualmente para ter o meu Setup Up and Running e me dar um norte de como vou fazer o meu script Bash.)

(EM DESENVOVIMENTO)


``` bash
loadkeys br-abnt2
ls /sys/firmware/efi/efivars
timedatectl set-ntp true
fdisk -l

```

UEFI/GPT layout

Mount point on the installed system |	Partition |	Partition type GUID  | Partition attributes | Suggested size 
| ------------- |:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|-----:|
/boot or /efi |	/dev/sda1 |	C12A7328-F81F-11D2-BA4B-00A0C93EC93B: EFI system partition || 260 MiB 
/             | /dev/sda2 | 4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709: Linux x86-64 root (/) || 23–32 GiB 
[SWAP]        |	/dev/sda3 |	0657FD6D-A4AB-43C4-84E5-0933C84B4F4F: Linux swap 		    || More than 512 MiB 
/home         | /dev/sda4 |	933AC7E1-2EB4-4F13-B844-0E14E2AEF915: Linux /home 		    || Remainder of the device 


BIOS/GPT example layout
Mount point on the installed system |	Partition |	Partition type GUID  | Partition attributes | Suggested size 
| ------------- |:-------------:|:-------------:|:-------------:|:-------------:|:-------------:|-----:|
None   | /dev/sda1  | 21686148-6449-6E6F-744E-656564454649: BIOS boot partition1  	|                         |1 MiB 
/ 	   | /dev/sda2  | 4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709: Linux x86-64 root (/)   | 2: Legacy BIOS bootable |	23–32 GiB 
[SWAP] | /dev/sda3 	| 0657FD6D-A4AB-43C4-84E5-0933C84B4F4F: Linux swap 		        |                         |More than  512 MiB 
/home  | /dev/sda4 	| 933AC7E1-2EB4-4F13-B844-0E14E2AEF915: Linux /home 	        |                         |Remainder of the device 


``` bash

```


Fontes:
https://wiki.archlinux.org/index.php/Installation_guide
https://wiki.archlinux.org/index.php/General_recommendations
https://www.reddit.com/r/linux/comments/9galhz/creating_a_hardened_arch_linux_installation_with/