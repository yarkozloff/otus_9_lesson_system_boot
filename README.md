# Работа с загрузчиком

Описание/Пошаговая инструкция выполнения домашнего задания:
- Попасть в систему без пароля несколькими способами.
- Установить систему с LVM, после чего переименовать VG.
- Добавить модуль в initrd. 

- (*). Сконфигурировать систему без отдельного раздела с /boot, а только с LVM Репозиторий с пропатченым grub: https://yum.rumyantsev.com/centos/7/x86_64/ PV необходимо инициализировать с параметром --bootloaderareasize 1m

## Попасть в систему без пароля несколькими способами.
## Установить систему с LVM, после чего переименовать VG.
Среда выполнения: VMWare Workstation 15 Player, Ubuntu 20.04.3 с Vagrant. Локально загруженный бокс centos7, с выделенным диском (для LVM).
По предыдущему занятию подготовим LVM. https://github.com/yarkozloff/otus_3_lesson_lvm
Проверим состояние
```
[vagrant@lvm ~]$ sudo vgs
  VG         #PV #LV #SN Attr   VSize   VFree
  VolGroup00   1   1   0 wz--n- <10.00g    0
```
Переименовываем VG:
```
[vagrant@lvm ~]$ sudo vgrename VolGroup00 VagrantRoot
  Volume group "VolGroup00" successfully renamed to "VagrantRoot"
```
Пробуем ребутнуться, чтобы убедиться, что изменения не применились


## Добавить модуль в initrd.
