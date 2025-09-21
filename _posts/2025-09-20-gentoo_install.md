---
layout: post
title:  "Установка Gentoo"
category: unix
---

#### **Устанавливал с помощью Live CD/DVD Linux Mint 22.1**

##### Узнаем название тома и запускуем программу разметки диска(cfdisk).

Для примера я буду использовать /dev/sda/

_**Не забываем про `sudo su`**!_

- sudo su 

- lsblk

- cfdisk /dev/sda -z

##### Разобьём диск на нужные разделы. 

*Я всегда использую следующую разметку*:

    /dev/sda1 (512MB) — /boot/efi
    /dev/sda2 (всё остальное) — /
    
Как вариант можно добавить /home.

##### Создаём файловые системы на нашем диске.

- mkfs.fat -F 32 /dev/sda1 # раздел для Grub

- mkfs.ext4 /dev/sda2 # раздел для корня в ext4

##### Монтируем

Разработчики Gentoo рекомендуют монтировать разделы не в /mnt (как происходит, например, в Arch Linux), а именно в /mnt/gentoo.

- mkdir /mnt/gentoo # создаём общую директорию монтирования

- mount /dev/sda2 /mnt/gentoo

- mkdir -p /mnt/gentoo/efi # создаём директорию для раздела загрузки

- mount /dev/sda1 /mnt/gentoo/efi

##### Хватаем stage3 архив и скачиваем к нам в папку.

На сайте с загрузкой мы видим следующие виды архивов:
```
Stage 3 (openrc) — базовая система с системой инициализации OpenRC.
Stage3 (systemd) — базовая система с systemd.
Stage 3 (desktop profile | openrc) — система уже с собранным desktop-профилем и OpenRC.
Stage 3 (desktop profile | systemd | merged usr) — думаю понятно.
```
Я использую Stage3 с OpenRC и готовым desktop профилем.Вы можете выбрать любой архив под ваши задачи и нужды. ПКМ по профилю и скопируем ссылку для скачивания. 

Загрузим архив в /mnt/gentoo:

- cd /mnt/gentoo

- wget https://distfiles.gentoo.org/releases/amd64/autobuilds/20250315T023326Z/stage3-amd64-desktop-openrc-20250315T023326Z.tar.xz # OpenRC + Desktop Profile

##### Распакуем архив.

- tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner

#### Конфигурируем portage.

Зададим MAKEOPTS. Тут всё зависит от ресурсов вашего компьютера. Одна задача занимает около 2 гигабайт оперативной памяти. Следовательно, флаг "-j2" будет занимать 4 ГБ ОЗУ и так далее. В моем случае я возьму "-j10" как самый средний вариант.

- nano /mnt/gentoo/etc/portage/make.conf
```
MAKEOPTS="-j10"

GRUB_PLATFORMS="efi-64"
```
##### Сразу зададим зеркало для репозиториев. 

Зеркала можно посмотреть здесь:

https://www.gentoo.org/downloads/mirrors/

По аналогии с прошлым - скопируем ссылку и добавим следующее в: 

/mnt/gentoo/etc/portage/make.conf 
```
GENTOO_MIRRORS="http://mirror.yandex.ru/gentoo-distfiles/"
```
##### Примем необходимые лицензии. 

Я принимаю все, потому пропишу:
```
ACCEPT_LICENSE="*"
```
##### Пример базового make.conf(Будьте внимательны,данные приведены для моей машины-здесь только для примера!!!)
```
CFLAGS="-march=native -O2 -pipe"
CXXFLAGS="${CFLAGS}"
CHOST="x86_64-pc-linux-gnu"
USE_ARCHIVES="lzo rar zip"
CPU_FLAGS_X86="aes avx avx2 fma3 mmx mmxext popcnt sse sse2 sse3 sse4_1 sse4_2 ssse3"
USE_CPUFLAGS_LEGACY="aes avx avx2 fma3 mmx mmxext popcnt sse sse2 sse3 sse4_1 sse4_2 ssse3"
USE="dri X xorg dbus nls ude pulseaudio pavucontrol alsa session libnotify policykit svg jpeg gconf gtk gtk3 $USE_ARCHIVES $USE_CPUFLAGS_LEGACY -wayland -telemetry -gnome -kde -minimal -qt"
MAKEOPTS="-j10"
GRUB_PLATFORMS="pc"
FEATURES="parallel-fetch metadata-transfer candy sandbox -xattr"
GENTOO_MIRRORS="http://mirror.yandex.ru/gentoo-distfiles/"
ABI_X86="64"
ACCEPT_LICENSE="*"
PORTDIR="/var/db/repos/"
REPOSDIR="/var/db/repos"
DISTDIR="${REPOSDIR}/distfiles"
PKGDIR="${REPOSDIR}/packages"
DARKELF_FEATURES="postmerge_distclean"
LINGUAS="ru ru_RU"
L10N="$LINGUAS"
GRUB_PLATFORMS="efi-64"
VIDEO_CARDS="intel"
ALSA_CARDS="hda-intel intel8x0"
INPUT_DEVICES="evdev libinput amdgpu"
```
##### "Чрутнемся" в нашу Gentoo.

Для начала скопируем информацию про DNS, чтобы сеть продолжала работать внутри новой системы.

- cp --dereference /etc/resolv.conf /mnt/gentoo/etc/

```
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
mount --bind /run /mnt/gentoo/run
mount --make-slave /mnt/gentoo/run
```
Обязательно! Если используете не LiveCD от Gentoo, то проверим отсутствие симлинка с /dev/shm на /run/shm и сделаем /dev/shm временной файловой системой.
```
test -L /dev/shm && rm /dev/shm && mkdir /dev/shm
mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm
chmod 1777 /dev/shm /run/shm
```
##### Зайдем в нашу систему.
```
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
```
##### Обновляем репозитории.

- emerge-webrsync

#### Выбираем профиль.

Профиль - это набор пакетов, собранных для определенных целей или задач. Например, Desktop Profile представляет собой набор программного обеспечения, нацеленный на использование системы в качестве десктопа. Desktop Gnome предназначен для удобства работы с окружением GNOME на операционной системе и так далее. 

Посмотрим какие профили есть.

- eselect profile list

##### Available profile symlink targets:

  [1]   default/linux/amd64/17.1 (stable)
  [2]   default/linux/amd64/17.1/selinux (stable)
  [3]   default/linux/amd64/17.1/hardened (stable)
  [4]   default/linux/amd64/17.1/hardened/selinux (stable)
  [5]   default/linux/amd64/17.1/desktop (stable) *
  [6]   default/linux/amd64/17.1/desktop/gnome (stable)
  [7]   default/linux/amd64/17.1/desktop/gnome/systemd (stable)
  [8]   default/linux/amd64/17.1/desktop/gnome/systemd/merged-usr (stable)
  [9]   default/linux/amd64/17.1/desktop/plasma (stable)
  [10]  default/linux/amd64/17.1/desktop/plasma/systemd (stable)
  [11]  default/linux/amd64/17.1/desktop/plasma/systemd/merged-usr (stable)
  [12]  default/linux/amd64/17.1/desktop/systemd (stable)
  [13]  default/linux/amd64/17.1/desktop/systemd/merged-usr (stable)
  ........

Здесь мы можем увидеть, какие профили есть в нашей системе. Если вы собираетесь использовать оконные менеджеры, или там Xfce или Mate, я советую выбрать профиль "default/linux/amd64/23.0/desktop (stable)". Если же вы планируете использовать Plasma или GNOME, то следует выбрать соответствующие профили.

- eselect profile set 'numbe_profile'

##### USE-флаги и обновление "миров".

Маленькая справочка:

USE-флаги позволяют очень гибко конфигурировать ПО, включая или выключая те или иные возможности. Тут всё индивидуально,но для базовой установки их можно не настраивать.

##### Обновить "миры", установить пакеты из профиля и обновить систему.

- emerge-webrsync && emerge -avuDN @world

Если вы взяли Stage3 без десктоп-профиля, но выбрали его позднее - ждите, просто ждите.

##### Настраиваем время и локали.

Если вы используйте OpenRC, то пропишите данную команду.

- echo "Asia/Kamchatka" > /etc/timezone

Локали, как и в арче, находятся в /etc/locale.gen. Однако вместо списка там надо будет написать свои.

- nano /etc/locale.gen
```
en_US.UTF-8 UTF-8 # английский
ru_RU.UTF-8 UTF-8 # русский
```
- locale-gen

##### Ставим ядро.

Первое что делаем,устанавливаем linux-firmware.

- emerge --ask sys-kernel/linux-firmware

Теперь, для установки ядра, установим пакет installkernel (не ядро). Однако перед установкой, добавим поддержку Dracut.

- nano /etc/portage/package.use/installkernel
```
sys-kernel/installkernel dracut
```
##### Установим сам пакет.

- emerge --ask sys-kernel/installkernel

Gentoo предлагает два варианта - gentoo-kernel и gentoo-kernel-bin. 

Если хотите сами собрать ядро, сконфигурировать его под своё железо и получить опыт сборки ядер, то берите стандартный gentoo-kernel.

- emerge --ask sys-kernel/gentoo-kernel-bin

##### Заполняем fstab

##### my_fstab

- nano /etc/fstab
```
/dev/sda1	/boot/efi	vfat	noatime	0 0
/dev/sda2	/	ext4	noatime	0 1
# /dev/sda3       none            swap        sw                      0 0
# /dev/sda4       /home           ext4        defaults                1 2
```
##### Зададим hostname.

- echo gentoo > /etc/hostname

Вместо "gentoo" можно любое.

##### Заполняем /etc/hosts:

- nano /etc/hosts

К уже существующим строкам добавим(если нет):

- 127.0.0.1  gentoo

##### Ставим важный дополнительный софт.

##### Установим DHCP и Grub командой ниже:

- emerge --ask net-misc/dhcpcd sys-boot/grub sys-boot/efibootmgr

##### Конфигурируем GRUB.

- emerge --ask sys-boot/grub

- emerge --ask --update --newuse --verbose sys-boot/grub

**GRUB теперь установлен в системе, но еще не активирован**

#### Установка

###### Для систем UEFI:

- grub-install --efi-directory=/efi
```
Установка для платформы x86_64-EFI.
Установка завершена. Ошибка не сообщается.
```
##### Для создания окончательной конфигурации GRUB, запустите команду grub-mkconfig:

- grub-mkconfig -o /boot/grub/grub.cfg
```
Генерируя grub.cfg ...
Нашел изображение Linux: /boot/vmlinuz-6.6.21-gentoo
Найдено INITRD Изображение: /BOOT/INITRAMFS-GENKERNEL-AMD64-6.6.21-GENTOO
сделанный
```
##### Задаём пароль и делаем юзера.

- passwd # пароль для рута

- useradd -m -G users,wheel,audio,video -s /bin/bash jenit   # добавляем пользователя

- passwd jenit # даём пароль пользователю

_______________________________________________________________


##### Eix - это инструмент для поиска и работы с пакетами в Gentoo

Установите его и обновите его базу данных:

- emerge --ask app-portage/eix

- eix-update

_______________________________________________________________
##### Установка графической оболочки

###  Иксы

- emerge --ask  x11-base/xorg-server  # установка xorg

- emerge --ask x11-base/xorg-drivers

                     
_____________________________________________________________

***Дисплейный менеджер (display manager, DM, иногда login manager) обеспечивает пользователю графический экран входа для запуска графического сеанса X или Wayland***

https://wiki.gentoo.org/wiki/Display_manager/ru

##### XFCE

- emerge --ask xfce-base/xfce4-meta   # это xfce4

- emerge --ask x11-drivers/xf86-video-intel # моя видеокарта

##### Terminal(необязательно,чего нет xfce-base/xfce4-meta можно установить после установки системы)...

- emerge --ask xfce-extra/xfce4-pulseaudio-plugin xfce-extra/xfce4-taskmanager x11-themes/xfwm4-themes app-editors/mousepad xfce-base/xfce4-power-manager x11-terms/xfce4-terminal xfce-base/thunar xfce-extra/xfce4-notifyd 

##### Обновите переменные среды системы:

- env-update && . /etc/profile

https://wiki.gentoo.org/wiki/Xfce/Guide/en

##### Установить LightDM:

- emerge --ask x11-misc/lightdm             # это lightdm

- emerge --ask gui-libs/display-manager-init

- rc-update add display-manager default

- rc-update add dbus default

- rc-update add lightdm default

- rc-service dbus start

- rc-service display-manager start

##### Для запуска LightDM

- /etc/init.d/dbus start

- /etc/init.d/lightdm start

### Вносим наш дисплейменеджер

- nano -w /etc/conf.d/display-manager

### Пример:

- CHECKVT=7

- DISPLAYMANAGER="lightdm" # XFCE

https://wiki.gentoo.org/wiki/LightDM/ru
_______________________________________________________________


### Запуск Xfce в консоли

Выйдите из оболочки root и войдите в систему как обычный пользователь.

- echo "exec startxfce4" > ~/.xinitrc

- echo "exec dbus-launch --exit-with-session xfce4-session" > ~/.xinitrc

Теперь запустите графическую среду, введя startx

- startx

https://wiki.gentoo.org/wiki/Xfce/ru

_______________________________________________________________

## Перезагружаемся и получаем нашу Gentoo!

- exit

- cd

- umount -l /mnt/gentoo/dev{/shm,/pts,}

- umount -R /mnt/gentoo

- reboot
_______________________________________________________________

## About.Focus!!!

Иногда,при установке программ,может выскочить предупреждение.

Лечится так,смотреть ниже:

 * IMPORTANT: config file '/etc/portage/package.use/installkernel' needs updating.
 * See the CONFIGURATION FILES and CONFIGURATION FILES UPDATE TOOLS
 * sections of the emerge man page to learn how to update config files.


- etc-update

- -3

- yes

- Enter



