# **Загрузка системы**

## **Домашнее задание**

**Работа с загрузчиком**
- Попасть в систему без пароля несколькими способами
- Установить систему с LVM, после чего переименовать VG
- Добавить модуль в initrd

- (*). Сконфигурировать систему без отдельного раздела с /boot, а только с LVM
Репозиторий с пропатченым grub: https://yum.rumyantsev.com/centos/7/x86_64/
PV необходимо инициализировать с параметром --bootloaderareasize 1m
Критерии оценки: Описать действия, описать разницу между методами получения шелла в процессе загрузки.
Где получится - используем script, где не получается - словами или копипастой описываем действия. 

**Установить систему с LVM**

Установила систему с образа CentOS-7-x86_64-Minimal-1804.iso на VM в VirtualBox с именем CentOS. На приложенном Screenshot from 2019-04-14 15-27-55.png видно как размечен диск:/boot - раздел sda1, root - /, swap - [SWAP].

**Попасть в систему без пароля несколькими способами**

**Способ 1. init=/bin/sh**
- **В конце строки начинающейся с linux16 добавляем init=/bin/sh и нажимаем сtrl-x для загрузки в систему**.
- **Попадаем в shell без пароля. Приветствие**: sh-4.2#
- **Но есть один нюанс. Рутовая файловая система при этом монтируется в режиме Read-Only. Чтобы перемонтировать ее в режим Read-Write воспользуемся
командой:**

sh-4.2# mount -o remount,rw

**Воспользуемся следующей командой и увидим результаты монтирования:**

sh-4.2# mount | grep root 

/dev/mapper/centos-root on /type xfs (**rw**,realtime,attr2,inode64,noquota)

**Способ 2. rd.break**

- **В конце строки начинающейся с linux16 добавляем rd.break и нажимаем сtrl-x для загрузки в систему.**
- **Попадаем в emergency mode. Наша корневая файловая система смонтирована опять же в режиме Read-Only, но мы не в ней. Далее будет пример как попасть в нее и поменять пароль администратора:**

Entering emergency mode. Exit the shell to continue.

Type "jornalctl" to view system logs.

You might want to save "/run/initframfs/rdsosreport.txt" to USB stick or /boot after mounting them and attach it to a bug report.

switch_root:/#

switch_root:/# mount -o remount,rw /sysroot

switch_root:/# chroot /sysroot

sh-4.2# passwd root

Changing password for user root.

New password:

BAD PASSWORD: The password is shorter than 7 characters

Retype new password:

passwd: all authentication token updated successfully.

sh-4.2# touch /.autorelabel

**После перезагрузилась и зашла успешно под новым паролем:**

localhost login: root

Password:

Last login: Sun Apr 14 09:57:48 on tty1

[root@localhost ~]#

**Способ 3. rw init=/sysroot/bin/sh**
В строке начинающейся с linux16 заменяем ro на rw init=/sysroot/bin/sh и нажимаем сtrl-xдля загрузки в систему. От прошлого примера отличается только тем, что файловая система сразу смонтирована в режим Read-Write:

Entering emergency mode. Exit the shell to continue.

Type "jornalctl" to view system logs.

You might want to save "/run/initframfs/rdsosreport.txt" to USB stick or /boot after mounting them and attach it to a bug report.

:/#

**На системе с LVM переименовываем VG**

**Вначале посмотрим текущее состояние системы:**

[root@localhost ~]# vgs

  VG     #PV #LV #SN Attr   VSize   VFree

  centos   1   2   0 wz--n- <7.00g     0

**Интересует вторая строка с именем centos**

Приступим к переименованию:

[root@localhost ~]# vgrename centos OtusRoot

Volume group "centos" successfully renamed to "OtusRoot"

**Далее правим /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. Везде заменяем старое название на новое.**

#

#/etc/fstab

#Created by anaconda on Sun Apr 14 03:46:55 2019

#

#Accessible filesystems, by reference, are maintained under '/dev/disk'

#See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info

#

/dev/mapper/OtusRoot-root /                       xfs     defaults        0 0

UUID=07a6a72f-7d85-4715-a702-a0309feea76c /boot                   xfs     defaults        0 0

/dev/mapper/OtusRoot-swap swap                    swap    defaults        0 0

[root@localhost ~]# vi /etc/default/grub 

GRUB_TIMEOUT=5

GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"

GRUB_DEFAULT=saved

GRUB_DISABLE_SUBMENU=true

GRUB_TERMINAL_OUTPUT="console"

GRUB_CMDLINE_LINUX="crashkernel=auto rd.lvm.lv=OtusRoot/root rd.lvm.lv=OtusRoot/swap rhgb quiet"

GRUB_DISABLE_RECOVERY="true"

**В /boot/grub2/grub.cfg из всего править нужно только следующую строку**

[root@localhost ~]# vi /boot/grub2/grub.cfg

...
        linux16 /vmlinuz-3.10.0-862.el7.x86_64 root=/dev/mapper/OtusRoot-root ro crashkernel=auto rd.lvm.lv=OtusRoot/root rd.lvm.lv=OtusRoot/swap rhgb quiet quiet LANG=eng_US.UTF-8

        initrd16 /initramfs-3.10.0-862.2.3.el7.x86_64.img
...

**Пересоздаем initrd image, чтобы он знал новое название Volume Group**

[root@localhost ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

...

*** Creating image file done ***

*** Creating initramfs image file '/boot/initramfs-3.10.0-862.el7.x86_64.img' done ***

Более подробный результат вывода команды приложен в скриншоте Screenshot from 2019-04-15 05-07-48.png.

Перезагружаемся и проверяем успешную загрузку с новым именем:

[root@localhost ~]# vgs

  VG       #PV #LV #SN Attr   VSize   VFree

  OtusRoot   1   2   0 wz--n- <7.00g     0

**Добавить модуль в initrd**

Скрипты модулей хранятся в каталоге /usr/lib/dracut/modules.d/. Для того чтобы добавить свой модуль создаем там папку с именем 01test:

[root@localhost ~]# mkdir /usr/lib/dracut/modules.d/01test

В нее поместим два скрипта:

1.module-setup.sh - который устанавливает модуль и вызывает скрипт test.sh

2.test.sh - собственно сам вызываемый скрипт, в нём у нас рисуется пингвинчик.

[root@localhost ~]# ls -a

. .. module-setup.sh test.sh

[root@localhost ~]# module-setup.sh

#!/bin/bash

check() {

    return 0

}

depends() {

    return 0

}

install() {

    inst_hook cleanup 00 "${moddir}/test.sh"

}

[root@localhost ~]# vi test.sh 

#!/bin/bash

exec 0<>/dev/console 1<>/dev/console 2<>/dev/console

cat <<'msgend'

Hello! You are in dracut module!
 ___________________

< I'm dracut module >

 -------------------

   \

    \

        .--.

       |o_o |

       |:_/ |

      //   \ \

     (|     | )

    /'\_   _/`\

    \___)=(___/

msgend

sleep 10

echo " contining...."

**Пересобираем образ initrd  и видим пингвинчика :=)**

[root@localhost ~]# mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)

[root@localhost ~]# dracut -f -v

**Можно проверить/посмотреть какие модули загружены в образ:**

[root@localhost ~]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test

test

**Далее отредактируем grub.cfg убрав опции rghb и quiet:**

...
        linux16 /vmlinuz-3.10.0-862.el7.x86_64 root=/dev/mapper/OtusRoot-root ro crashkernel=auto rd.lvm.lv=OtusRoot/root rd.lvm.lv=OtusRoot/swap LANG=eng_US.UTF-8

        initrd16 /initramfs-3.10.0-862.2.3.el7.x86_64.img
...

**После перезагрузки и после паузы на 10 секунд появился пингвин в выводе терминала. Результат приложен в Screenshot from 2019-04-15 06-09-07.png.**
