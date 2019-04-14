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


