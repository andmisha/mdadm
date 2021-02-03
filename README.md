# mdadm by andmisha

### Было сделано для выполнения домашнего задания:
#### 1) Взял исходный Vagrantfile из репозитория https://github.com/erlong15/otus-linux (оставил 4 диска)

#### 2) Создал новый репозиторий для домашней работы:
- GitHub - https://github.com/andmisha/mdadm

#### 3) Склонировал репозиторий к себе
```
git clone https://github.com/andmisha/mdadm C:\OTUS\mdadm
```
#### 4) Скопировал в папку C:\OTUS\mdadm Vagrantfile

#### 5) Выполнил vagrant up и vagrant ssh

#### 6) Выполнил lsblk
```
[vagrant@localhost ~]$ lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdd               8:48   0  250M  0 disk
sdb               8:16   0  250M  0 disk
sde               8:64   0  250M  0 disk
sdc               8:32   0  250M  0 disk
sda               8:0    0   10G  0 disk
├─sda2            8:2    0    9G  0 part
│ ├─centos-swap 253:1    0    1G  0 lvm  [SWAP]
│ └─centos-root 253:0    0    8G  0 lvm  /
└─sda1            8:1    0    1G  0 part /boot
```
#### 7) Выполнил fdisk
```
[vagrant@localhost ~]$ sudo fdisk -l | grep Disk
Disk /dev/sde: 262 MB, 262144000 bytes, 512000 sectors
Disk /dev/sdb: 262 MB, 262144000 bytes, 512000 sectors
Disk /dev/sdd: 262 MB, 262144000 bytes, 512000 sectors
Disk /dev/sdc: 262 MB, 262144000 bytes, 512000 sectors
Disk /dev/sda: 10.7 GB, 10737418240 bytes, 20971520 sectors
Disk label type: dos
Disk identifier: 0x000a1b1f
Disk /dev/mapper/centos-root: 8585 MB, 8585740288 bytes, 16769024 sectors
Disk /dev/mapper/centos-swap: 1073 MB, 1073741824 bytes, 2097152 sectors
```
#### 8) Установил mdadm
```
[vagrant@localhost ~]$ sudo yum install mdadm
```
#### 9) Занулил суперблок на всех дисках
```
sudo mdadm --zero-superblock --force /dev/sd{b,c,d,e}
```
#### 10) Создал RAID5 и 4-ех дисков
```
[vagrant@localhost ~]$ sudo mdadm --create --verbose /dev/md0 -l 5 -n 4 /dev/sd{b,c,d,e}
mdadm: layout defaults to left-symmetric
mdadm: layout defaults to left-symmetric
mdadm: chunk size defaults to 512K
mdadm: size set to 253952K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```
#### 11) Проверил статус RAID
```
[vagrant@localhost ~]$ cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 sde[4] sdd[2] sdc[1] sdb[0]
      761856 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/4] [UUUU]
```
#### 12) Проверил с помощью mdadm
```
[vagrant@localhost ~]$ mdadm -D /dev/md0
mdadm: must be super-user to perform this action
[vagrant@localhost ~]$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Feb  3 21:05:00 2021
        Raid Level : raid5
        Array Size : 761856 (744.00 MiB 780.14 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Feb  3 21:05:02 2021
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : fb725e3c:605b38ef:251278a5:d70f8779
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       4       8       64        3      active sync   /dev/sde
```
#### 13) Проверил метаданные RAID
```
[vagrant@localhost ~]$ sudo mdadm --detail --scan --verbose
ARRAY /dev/md0 level=raid5 num-devices=4 metadata=1.2 name=localhost.localdomain:0 UUID=fb725e3c:605b38ef:251278a5:d70f8779
   devices=/dev/sdb,/dev/sdc,/dev/sdd,/dev/sde
```
#### 14) Создал конфиг для ОС
Конфиг не создавался даже с sudo (постоянно ошибка -bash: /etc/mdadm/mdadm.conf: Permission denied), помогло запуск sudo mc и создание конфига в ручную
#### 15) Перезагрузил ВМ для теста, после перезагрузки выполнил sudo mdadm -D /dev/md0
```
[vagrant@localhost ~]$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Feb  3 21:05:00 2021
        Raid Level : raid5
        Array Size : 761856 (744.00 MiB 780.14 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Feb  3 21:05:02 2021
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : fb725e3c:605b38ef:251278a5:d70f8779
            Events : 18

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       4       8       64        3      active sync   /dev/sde
```
#### 16) Вывел из строя диск /dev/sdc
```
[vagrant@localhost ~]$ sudo mdadm /dev/md0 --fail /dev/sdc
mdadm: set /dev/sdc faulty in /dev/md0
```
#### 17) Проверил статус RAID
```
[vagrant@localhost ~]$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Feb  3 21:05:00 2021
        Raid Level : raid5
        Array Size : 761856 (744.00 MiB 780.14 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Feb  3 21:19:13 2021
             State : clean, degraded
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : fb725e3c:605b38ef:251278a5:d70f8779
            Events : 20

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       -       0        0        1      removed
       2       8       48        2      active sync   /dev/sdd
       4       8       64        3      active sync   /dev/sde

       1       8       32        -      faulty   /dev/sdc
```
#### 18) Удалил диск из RAID
```
[vagrant@localhost ~]$ sudo mdadm /dev/md0 --remove /dev/sdc
mdadm: hot removed /dev/sdc from /dev/md0
```
#### 19) Проверил статус RAID
```
[vagrant@localhost ~]$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Feb  3 21:05:00 2021
        Raid Level : raid5
        Array Size : 761856 (744.00 MiB 780.14 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 4
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Wed Feb  3 21:20:16 2021
             State : clean, degraded
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : fb725e3c:605b38ef:251278a5:d70f8779
            Events : 21

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       -       0        0        1      removed
       2       8       48        2      active sync   /dev/sdd
       4       8       64        3      active sync   /dev/sde
 ```
 #### 20) Добавил диск в RAID
 ```
 [vagrant@localhost ~]$ sudo mdadm /dev/md0 --add /dev/sdc
mdadm: added /dev/sdc
```
#### 21) Проверил статус RAID
```
[vagrant@localhost ~]$ sudo cat /proc/mdstat
Personalities : [raid6] [raid5] [raid4]
md0 : active raid5 sdc[5] sdd[2] sde[4] sdb[0]
      761856 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/4] [UUUU]

unused devices: <none>
[vagrant@localhost ~]$ sudo mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Feb  3 21:05:00 2021
        Raid Level : raid5
        Array Size : 761856 (744.00 MiB 780.14 MB)
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Wed Feb  3 21:21:06 2021
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : left-symmetric
        Chunk Size : 512K

Consistency Policy : resync

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : fb725e3c:605b38ef:251278a5:d70f8779
            Events : 40

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       5       8       32        1      active sync   /dev/sdc
       2       8       48        2      active sync   /dev/sdd
       4       8       64        3      active sync   /dev/sde
 ```
#### 22) Создал GPT раздел на RAID
```
sudo parted -s /dev/md0 mklabel gpt
```
#### 23) Создал партиции
```
[vagrant@localhost ~]$ sudo parted /dev/md0 mkpart primary ext4 0% 25%
Information: You may need to update /etc/fstab.

[vagrant@localhost ~]$ sudo parted /dev/md0 mkpart primary ext4 25% 50%
Information: You may need to update /etc/fstab.

[vagrant@localhost ~]$ sudo parted /dev/md0 mkpart primary ext4 50% 75%
Information: You may need to update /etc/fstab.

[vagrant@localhost ~]$ sudo parted /dev/md0 mkpart primary ext4 75% 100%
```
#### 24) Выполнил lsblk для просмотра созданных партиций
```
[vagrant@localhost ~]$ lsblk
NAME            MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sdd               8:48   0   250M  0 disk
└─md0             9:0    0   744M  0 raid5
  ├─md0p4       259:1    0 184.5M  0 md
  ├─md0p2       259:3    0   186M  0 md
  ├─md0p3       259:0    0   186M  0 md
  └─md0p1       259:2    0 184.5M  0 md
sdb               8:16   0   250M  0 disk
└─md0             9:0    0   744M  0 raid5
  ├─md0p4       259:1    0 184.5M  0 md
  ├─md0p2       259:3    0   186M  0 md
  ├─md0p3       259:0    0   186M  0 md
  └─md0p1       259:2    0 184.5M  0 md
sde               8:64   0   250M  0 disk
└─md0             9:0    0   744M  0 raid5
  ├─md0p4       259:1    0 184.5M  0 md
  ├─md0p2       259:3    0   186M  0 md
  ├─md0p3       259:0    0   186M  0 md
  └─md0p1       259:2    0 184.5M  0 md
sdc               8:32   0   250M  0 disk
└─md0             9:0    0   744M  0 raid5
  ├─md0p4       259:1    0 184.5M  0 md
  ├─md0p2       259:3    0   186M  0 md
  ├─md0p3       259:0    0   186M  0 md
  └─md0p1       259:2    0 184.5M  0 md
sda               8:0    0    10G  0 disk
├─sda2            8:2    0     9G  0 part
│ ├─centos-swap 253:1    0     1G  0 lvm   [SWAP]
│ └─centos-root 253:0    0     8G  0 lvm   /
└─sda1            8:1    0     1G  0 part  /boot
```
#### 25) Выполнил создание файловых систем
```
[vagrant@localhost ~]$ for i in $(seq 1 4);do sudo mkfs.ext4 /dev/md0p$i; done
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=1024 (log=0)
Fragment size=1024 (log=0)
Stride=512 blocks, Stripe width=1536 blocks
47232 inodes, 188928 blocks
9446 blocks (5.00%) reserved for the super user
First data block=1
Maximum filesystem blocks=33816576
24 block groups
8192 blocks per group, 8192 fragments per group
1968 inodes per group
Superblock backups stored on blocks:
        8193, 24577, 40961, 57345, 73729

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```
#### 26) Выполнил монтирование всех партиций
```
[vagrant@localhost ~]$ for i in $(seq 1 4); do sudo mount /dev/md0p$i /raid/part$i; done
```
#### 27) Выполнил lsblk для проверки монтирования партиций
```
[vagrant@localhost /]$ lsblk
NAME            MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sdd               8:48   0   250M  0 disk
└─md0             9:0    0   744M  0 raid5
  ├─md0p4       259:1    0 184.5M  0 md    /raid/part4
  ├─md0p2       259:3    0   186M  0 md    /raid/part2
  ├─md0p3       259:0    0   186M  0 md    /raid/part3
  └─md0p1       259:2    0 184.5M  0 md    /raid/part1
sdb               8:16   0   250M  0 disk
└─md0             9:0    0   744M  0 raid5
  ├─md0p4       259:1    0 184.5M  0 md    /raid/part4
  ├─md0p2       259:3    0   186M  0 md    /raid/part2
  ├─md0p3       259:0    0   186M  0 md    /raid/part3
  └─md0p1       259:2    0 184.5M  0 md    /raid/part1
sde               8:64   0   250M  0 disk
└─md0             9:0    0   744M  0 raid5
  ├─md0p4       259:1    0 184.5M  0 md    /raid/part4
  ├─md0p2       259:3    0   186M  0 md    /raid/part2
  ├─md0p3       259:0    0   186M  0 md    /raid/part3
  └─md0p1       259:2    0 184.5M  0 md    /raid/part1
sdc               8:32   0   250M  0 disk
└─md0             9:0    0   744M  0 raid5
  ├─md0p4       259:1    0 184.5M  0 md    /raid/part4
  ├─md0p2       259:3    0   186M  0 md    /raid/part2
  ├─md0p3       259:0    0   186M  0 md    /raid/part3
  └─md0p1       259:2    0 184.5M  0 md    /raid/part1
sda               8:0    0    10G  0 disk
├─sda2            8:2    0     9G  0 part
│ ├─centos-swap 253:1    0     1G  0 lvm   [SWAP]
│ └─centos-root 253:0    0     8G  0 lvm   /
└─sda1            8:1    0     1G  0 part  /boot
```
