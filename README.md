# Otus Homework 3. RAID

### Цель домашнего задания
Научиться использовать утилиту для управления программными RAID-массивами в Linux 
### Описание домашнего задания
- добавить в Vagrantfile еще дисков;
- сломать/починить raid;
- собрать R0/R5/R10 на выбор;
- прописать собранный рейд в конф, чтобы рейд собирался при загрузке;
- создать GPT раздел и 5 партиций.

- Доп. задание * : Vagrantfile, который сразу собирает систему с подключенным рейдом и смонтированными разделами. После перезагрузки стенда разделы должны автоматически примонтироваться.

## Выполнение
С помощью **Vagrant**  создается виртуальная машина с ОС Alma Linux 9, имеющая 5 дополнительных жестких дисков. Бокс был загружен из Vagrant Cloud, соотвественно выполнение команды **vagrant up** должно осуществляться при ***подключенном VPN-соединении***.
Настройка RAID-массива осуществляется с помощью **Ansible**. Так как **Ansible** запускается не с хоста, на котором установлен **Vagrant**, в **Vagrantfile** добавлен скрипт, копирующий SSH ключ на созданные ВМ. Ansible сервер и ВМ находятся в одной подсети.

*Ansible Playbook* запускается с ролью *raid*, которая выполняет следующие задания:
- установка **mdadm**
- создание RAID-10, используя диски *sdb, sdc, sdd, sde*
- создание файла конфигурации **mdadm.conf**
- перевод диска *sdd* в режим **Failed**
- замена диска *sdd* на *sdf*
- создание GPT раздела и 5 партиций на RAID массиве
- создание файловой системы *ext4* на каждом разделе
- редактирование файла */etc/fstab* и монтирование партиций в соответсвующие директории в */mnt/*
- при каждом изменении RAID массива выполняются команды `cat /proc/mdstat` и `lsblk` результат которых записывается в текстовые файлы

В результате выполнения Ansible Playbook мы получим настроенный RAID-10 массив, на котором созданы 5 партиций, автоматически монтируемые в каталог */mnt/* при перезагрузке.
Также в домашнем каталоге пользователя **vagrant** будут находиться файлы *mdstat.txt* и *lsblk.txt*, отражающие выполнения соотвествующих команд при каждом изменении RAID.

Вывод файлов привожу ниже: 

### mdadm.txt

    1. RAID10 was created:

    Personalities : [raid10]
    md127 : active raid10 sde[3] sdd[2] sdc[1] sdb[0]
        200704 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]

    unused devices: <none>
    __________________________________________________

    2. Disk sdd was failed

    Personalities : [raid10]
    md127 : active raid10 sde[3] sdd[2](F) sdc[1] sdb[0]
        200704 blocks super 1.2 512K chunks 2 near-copies [4/3] [UU_U]

    unused devices: <none>
    __________________________________________________

    3. Disk sdd was changed to sdf

    Personalities : [raid10]
    md127 : active raid10 sdf[4] sde[3] sdc[1] sdb[0]
        200704 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]

    unused devices: <none>


### lsblk.txt

    1. RAID10 was created:

    NAME                     MAJ:MIN RM  SIZE RO TYPE   MOUNTPOINTS
    sda                        8:0    0  128G  0 disk
    ├─sda1                     8:1    0    1G  0 part   /boot
    └─sda2                     8:2    0  127G  0 part
    ├─almalinux_alma9-root 253:0    0   70G  0 lvm    /
    └─almalinux_alma9-swap 253:1    0    2G  0 lvm    [SWAP]
    sdb                        8:16   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    sdc                        8:32   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    sdd                        8:48   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    sde                        8:64   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    sdf                        8:80   0  100M  0 disk
    __________________________________________________

    2. Disk sdd was failed

    NAME                     MAJ:MIN RM  SIZE RO TYPE   MOUNTPOINTS
    sda                        8:0    0  128G  0 disk
    ├─sda1                     8:1    0    1G  0 part   /boot
    └─sda2                     8:2    0  127G  0 part
    ├─almalinux_alma9-root 253:0    0   70G  0 lvm    /
    └─almalinux_alma9-swap 253:1    0    2G  0 lvm    [SWAP]
    sdb                        8:16   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    sdc                        8:32   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    sdd                        8:48   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    sde                        8:64   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    sdf                        8:80   0  100M  0 disk
    __________________________________________________

    3. Disk sdd was changed to sdf

    NAME                     MAJ:MIN RM  SIZE RO TYPE   MOUNTPOINTS
    sda                        8:0    0  128G  0 disk
    ├─sda1                     8:1    0    1G  0 part   /boot
    └─sda2                     8:2    0  127G  0 part
    ├─almalinux_alma9-root 253:0    0   70G  0 lvm    /
    └─almalinux_alma9-swap 253:1    0    2G  0 lvm    [SWAP]
    sdb                        8:16   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    sdc                        8:32   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    sdd                        8:48   0  100M  0 disk
    sde                        8:64   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    sdf                        8:80   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    __________________________________________________

    4. Partitions

    NAME                     MAJ:MIN RM  SIZE RO TYPE   MOUNTPOINTS
    sda                        8:0    0  128G  0 disk
    ├─sda1                     8:1    0    1G  0 part   /boot
    └─sda2                     8:2    0  127G  0 part
    ├─almalinux_alma9-root 253:0    0   70G  0 lvm    /
    └─almalinux_alma9-swap 253:1    0    2G  0 lvm    [SWAP]
    sdb                        8:16   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    ├─md127p1              259:0    0 39.2M  0 part
    ├─md127p2              259:1    0   38M  0 part
    ├─md127p3              259:2    0   40M  0 part
    ├─md127p4              259:3    0   39M  0 part
    └─md127p5              259:4    0   39M  0 part
    sdc                        8:32   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    ├─md127p1              259:0    0 39.2M  0 part
    ├─md127p2              259:1    0   38M  0 part
    ├─md127p3              259:2    0   40M  0 part
    ├─md127p4              259:3    0   39M  0 part
    └─md127p5              259:4    0   39M  0 part
    sdd                        8:48   0  100M  0 disk
    sde                        8:64   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    ├─md127p1              259:0    0 39.2M  0 part
    ├─md127p2              259:1    0   38M  0 part
    ├─md127p3              259:2    0   40M  0 part
    ├─md127p4              259:3    0   39M  0 part
    └─md127p5              259:4    0   39M  0 part
    sdf                        8:80   0  100M  0 disk
    └─md127                    9:127  0  196M  0 raid10
    ├─md127p1              259:0    0 39.2M  0 part
    ├─md127p2              259:1    0   38M  0 part
    ├─md127p3              259:2    0   40M  0 part
    ├─md127p4              259:3    0   39M  0 part
    └─md127p5              259:4    0   39M  0 part
