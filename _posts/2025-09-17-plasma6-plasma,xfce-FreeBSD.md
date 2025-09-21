---
title:  "Установка plasma6-plasma, xfce in FreeBSD"
layout: post
category: unix
---

#### Думаю сама установка FreeBSD больших трудностей не вызовет!

```
Перед установкой узнать свою видеокарту!!!
Здесь я делал установку со своей intel !!!
Здесь установка на "реальное железо" !!!
Смотреть: https://docs.freebsd.org/en/books/handbook/x11/
(установка XFCE без менеджера входа-DE
Если нужен менеджера входа в xfce то:
pkg ins -y lightdm lightdm-gtk-greeter
sysrc lightdm_enable="YES")
login: root
passwd: *******
pkg
pkg update
pkg ins -y xorg
pkg ins -y drm-kmod
pkg ins -y xf86-video-intel
pkg ins -y xfce

*****************************************************************

(установка минимальной Plasma6 с менеджером входа-DE)
login: root
passwd: *******
pkg
pkg update
pkg ins -y xorg
pkg ins -y drm-kmod
pkg ins -y xf86-video-intel
pkg ins -y plasma6-plasma
pkg ins -y sddm
sysrs sddm_enable=YES
sysrs sddm_lang="ru_RU"

******************************************************************

startx     # запуск без менеджера входа (DE)

XFCE
no root   # запуск без менеджера входа (DE)
$ edit .xinitrc
exec /usr/local/bin/startxfce4 --with-ck-launch
Esc --> a --> a

Plasma 6
$ edit .xinitrc
exec dbus-launch --exit-wich-x11 ck-launch-session startplasma-x11
Esc --> a --> a

*******************************************************************

For XFCE, Plasma:

sysrs dbus_enable=YES
sysrs kld_list+=i915
ee /etc/sysctl.conf
net.local.stream.recvspace=65536
net.local.stream.sendspace=65536
Esc --> a --> a
```

Handbook FreeBSD:

[Handbook](https://docs.freebsd.org/ru/books/handbook/)

Но лучше пользоваться английской версией:

[FreeBSD Handbook | FreeBSD Documentation Portal](https://docs.freebsd.org/en/books/handbook/))

Поиск портов(программ) FreeBSD:

[Search_ports](https://ports.freebsd.org/cgi/ports.cgi?query=&stype=all&sektion=all)



