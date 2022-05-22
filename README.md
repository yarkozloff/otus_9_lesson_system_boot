# Работа с загрузчиком

Описание/Пошаговая инструкция выполнения домашнего задания:
- Попасть в систему без пароля несколькими способами.
- Установить систему с LVM, после чего переименовать VG.
- Добавить модуль в initrd. 

- (*). Сконфигурировать систему без отдельного раздела с /boot, а только с LVM Репозиторий с пропатченым grub: https://yum.rumyantsev.com/centos/7/x86_64/ PV необходимо инициализировать с параметром --bootloaderareasize 1m

## Попасть в систему без пароля несколькими способами.
1) В конце строки начинающейся с linux16 добавляем init=/bin/sh (чтобы попасть в режим Read-Write, перезаписать ro на rw) и нажимаем сtrl-x для загрузки в систему
Можно сменить пароль и перезагрузиться:

![image](https://user-images.githubusercontent.com/69105791/169695015-6985816c-4358-489c-ab98-3d65f094ab2a.png)

2) В конце строки начинающейся с linux16 добавляем rd.break и нажимаем сtrl-x для загрузки в систему.

Попадаем в emergency mode. Далее попадаем в файловую системы и меняем пароль администратора:
![image](https://user-images.githubusercontent.com/69105791/169694689-715fcfc5-7f57-4fd5-8558-45777708a562.png)

После чего можно перезагружаться и заходить в систему с новым паролем. 

3) В строке начинающейся с linux16 заменяем ro на rw init=/sysroot/bin/sh и нажимаем сtrl-x для загрузки в систему

![image](https://user-images.githubusercontent.com/69105791/169695077-ac8f6dbc-04a6-4ee8-b9b4-897b7cd705bc.png)

Аналогичным образом можно сменить пароль и перезагрузиться с новым:
![image](https://user-images.githubusercontent.com/69105791/169695175-434336dd-9a6d-4ae1-a16c-682b12341d23.png)


## Установить систему с LVM, после чего переименовать VG.
Среда выполнения: VMWare Workstation 15 Player. Установленный iso образ с Centos7 с LVM:
```
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   20G  0 disk
├─sda1            8:1    0 1020M  0 part /boot
└─sda2            8:2    0   19G  0 part
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sr0              11:0    1 1024M  0 rom
```
Проверим состояние:
```
[root@localhost ~]# vgs
  VG     #PV #LV #SN Attr   VSize  VFree
  centos   1   2   0 wz--n- 19.00g    0
```
Переименовываем VG:
```
[root@localhost ~]# vgrename centos OtusRoot
  Volume group "centos" successfully renamed to "OtusRoot"
```
Далее правим /etc/fstab, /etc/default/grub, /boot/grub2/grub.cfg. Везде заменяем старое название на новое. Измененные файлы в репозитории

Пересоздаем initrd image, чтобы он знал новое название Volume Group
```
mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```
Ребутаемся, проверяем:
```
[root@localhost ~]# vgs
  VG       #PV #LV #SN Attr   VSize  VFree
  OtusRoot   1   2   0 wz--n- 19.00g    0
```
## Добавить модуль в initrd.
Скрипты модулей хранятся в каталоге /usr/lib/dracut/modules.d/. Для того чтобы добавить свой модуль создаем там папку с именем 01test.
Скрипт который уствнавливает модуль и вызывает скрипт test.sh:
```
[root@localhost ~]# cat /usr/lib/dracut/modules.d/01test/module_setup.sh
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
```
Скрипт, который рисует пингвинчика:
```
[root@localhost ~]# cat /usr/lib/dracut/modules.d/01test/test.sh
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
echo " continuing...."
```
Пересобираем образ initrd
```
mkinitrd -f -v /boot/initramfs-$(uname -r).img $(uname -r)
```
Проверим загруженный модуль:
```
[root@localhost ~]# lsinitrd -m /boot/initramfs-$(uname -r).img | grep test
test
```
Ребутаемся, жмем клавишку "e", стираем опции  rghb и quiet, затем жмем "ctrl X"
Наблюдаем загрузку системы с пингвином:

![image](https://user-images.githubusercontent.com/69105791/169694347-9f0ce8cb-9a95-4c99-b89f-a51e5291362e.png)

