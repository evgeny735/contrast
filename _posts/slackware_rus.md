#### Русификация Slackware

>*KDE5 Plasma можно русифицировать в настройках(графический интерфейс)*

Изменение локали делают внутри файла `/etc/profile.d/lang.sh` и `/etc/profile.d/lang.csh`

#### Пример содежимого
 
>*Коментируется строчка импортирующая Английский язык и добавляется строчка 
для импорта Русского языка*.

---

Меняем содержимое файла:

`nano /etc/profile.d/lang.sh`
```
#!/bin/sh
Set the system locale.  (no, we don’t have a menu for this ;-)
For a list of locales which are supported by this machine, type:
Установите системный языковой стандарт.  (нет, у нас нет меню для этого;-)
Чтобы получить список языковых стандартов, поддерживаемых данным устройством, введите:
locale -a
en_US is the Slackware default locale:
en_US - это языковой стандарт Slackware по умолчанию:
#export LANG=en_US
export LANG=ru_RU.UTF-8
chmod +x lang.sh
```
---

Далее, меняем содержимое файла:

`nano /etc/profile.d/lang.csh`

Тут всё так же, как и в примере выше.

---
```
#!/bin/csh
Set the system locale.  (no, we don’t have a menu for this ;-)
For a list of locales which are supported by this machine, type:
locale -a
en_US is the Slackware default locale:
#setenv LANG en_US
setenv LANG ru_RU.UTF-8
chmod +x lang.csh
```
---
#### Как исправить показ кирилицы(крякозябры) при подключении USB дисков, в консоли.

>*В Slackware 15 это уже исправлено*

`nano /etc/rc.d/rc.font`

```
#!/bin/sh
setfont -v Cyr_a8x16.psfu.gz
for i in 1 2 3 4 5 6;do
echo -ne "\033%G" >/dev/tty$i
done
chmod +x rc.font
```
---

Далее необходимо перезагрузить иксы (`CTRL+alt+BACKSPACE`) или компьютер (по желанию).