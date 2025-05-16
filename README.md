# otus
**Домашка 1:**

0. проверяем текущую версию ядра:
danila@kds-otus01:~$ uname -r 
6.8.0-59-generic

1. Добавляем репозиторий:
danila@kds-otus01:~$ sudo add-apt-repository ppa:cappelikan/ppa 

2. Обновляем список доступных пактов: 
danila@kds-otus01:~$ sudo apt update

3. Устанавливаем пакет mainline kernels:
danila@kds-otus01:~$ sudo apt install mainline 

4. Проверяем доступную версию ядра: 
danila@kds-otus01:~$ mainline check 
mainline 1.4.13
Updating Kernels...
Latest update: 6.14.4 
Latest point update: 6.8.12 
mainline: done 

5. Обновляем ядро до последней доступной версии: 
danila@kds-otus01:~$ sudo mainline install-latest 

6. Перезагружаем виртуальную машину: 
danila@kds-otus01:~$ sudo reboot 

7. Проверяем текущую версию ядра: 
danila@kds-otus01:~$ uname -r 
6.14.4-061404-generic

**Домашка 2:**

1. С правами суперпользователя запускаем утилиту gdisk и создаем таблицу разделов GUID:

danila@kds-otus01:~$ gdisk /dev/sdb 
GPT fdisk (gdisk) version 1.0.10

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Command (? for help): o
This option deletes all partitions and creates a new protective MBR.
Proceed? (Y/N): y


2. Создаем 5 разделов по 100 МБ каждый, после чего проверяем результат:

Command (? for help): n
Partition number (1-128, default 1):
First sector (34-2097118, default = 2048) or {+-}size{KMGTP}:
Last sector (2048-2097118, default = 2095103) or {+-}size{KMGTP}: +100M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'

..

Command (? for help): n
Partition number (5-128, default 5):
First sector (34-2097118, default = 821248) or {+-}size{KMGTP}:
Last sector (821248-2097118, default = 2095103) or {+-}size{KMGTP}: +100M
Current type is 8300 (Linux filesystem)
Hex code or GUID (L to show codes, Enter = 8300):
Changed type of partition to 'Linux filesystem'

Command (? for help): p
Disk /dev/sdb: 2097152 sectors, 1024.0 MiB
Model: QEMU HARDDISK
Sector size (logical/physical): 512/4096 bytes
Disk identifier (GUID): 445E263E-27DB-4876-A68C-1F5230670240
Partition table holds up to 128 entries
Main partition table begins at sector 2 and ends at sector 33
First usable sector is 34, last usable sector is 2097118
Partitions will be aligned on 2048-sector boundaries
Total free space is 1073085 sectors (524.0 MiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048          206847   100.0 MiB   8300  Linux filesystem
   2          206848          411647   100.0 MiB   8300  Linux filesystem
   3          411648          616447   100.0 MiB   8300  Linux filesystem
   4          616448          821247   100.0 MiB   8300  Linux filesystem
   5          821248         1026047   100.0 MiB   8300  Linux filesystem


3. Выходим из утилиты gdisk с сохранением:

Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has commdadm pleted successfully.
danila@kds-otus01:~$


4. Выводим список доступных блочных устройств:

danila@kds-otus01:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                         8:0    0   40G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part /boot
└─sda3                      8:3    0   38G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   19G  0 lvm  /
sdb                         8:16   0    1G  0 disk
├─sdb1                      8:17   0  100M  0 part
├─sdb2                      8:18   0  100M  0 part
├─sdb3                      8:19   0  100M  0 part
├─sdb4                      8:20   0  100M  0 part
└─sdb5                      8:21   0  100M  0 part
sr0                        11:0    1    3G  0 rom


5. Создаем на разделе sdb5 файловую систему:

danila@kds-otus01:/$ sudo mkfs.ext4 /dev/sdb5
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done


6. Создаем каталог /mnt/distr и монтируем к нему раздел /dev/sdb5:

danila@kds-otus01:/$ sudo mkdir /mnt/distr && sudo mount /dev/sdb5 /mnt/distr
danila@kds-otus01:/$


7. Создаем из разделов sdb1 и sdb2 массив raid1, создаем файловую систему, каталог подключения и монтируем созданный raid1. Проверяем:

danila@kds-otus01:~$ sudo mdadm --create /dev/md127 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdb2
[sudo] password for danila:
mdadm: /dev/sdb1 appears to contain an ext2fs file system
       size=102400K  mtime=Thu Jan  1 03:00:00 1970
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: /dev/sdb2 appears to contain an ext2fs file system
       size=102400K  mtime=Thu Jan  1 03:00:00 1970
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md127 started.
danila@kds-otus01:~$ sudo mkdir /mnt/raid1
danila@kds-otus01:~$ sudo mkfs.ext4 /dev/md127
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done
Creating filesystem with 25344 4k blocks and 25344 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

danila@kds-otus01:~$ sudo mount /dev/md1 /mnt/raid1
danila@kds-otus01:~$ sudo mdadm --detail /dev/md127
[sudo] password for danila:
/dev/md127:
           Version : 1.2
     Creation Time : Wed May 14 16:08:51 2025
        Raid Level : raid1
        Array Size : 101376 (99.00 MiB 103.81 MB)
     Used Dev Size : 101376 (99.00 MiB 103.81 MB)
      Raid Devices : 2r
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Wed May 14 16:28:13 2025
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : kds-otus01:127  (local to host kds-otus01)
              UUID : 9570d2eb:ffa5aefe:4262740d:dc31050a
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       1       8       18        1      active sync   /dev/sdb2
danila@kds-otus01:~$ lsblk
NAME                      MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINTS
sda                         8:0    0   40G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    2G  0 part  /boot
└─sda3                      8:3    0   38G  0 part
  └─ubuntu--vg-ubuntu--lv 252:0    0   19G  0 lvm   /
sdb                         8:16   0    1G  0 disk
├─sdb1                      8:17   0  100M  0 part
│ └─md127                   9:127  0   99M  0 raid1
├─sdb2                      8:18   0  100M  0 part
│ └─md127                   9:127  0   99M  0 raid1
├─sdb3                      8:19   0  100M  0 part
├─sdb4                      8:20   0  100M  0 part
└─sdb5                      8:21   0  100M  0 part  /mnt/distr
sr0                        11:0    1    3G  0 rom


8. Сохраним данные о массиве и точках монтирования в конфигурационных файлах. Обновим initramfs:

danila@kds-otus01:~$ sudo sh -c 'mdadm --detail --scan >> /etc/mdadm/mdadm.conf'
danila@kds-otus01:~$ sudo sh -c 'echo "/dev/disk/by-uuid/c8ccf4b3-bb0e-404a-8401-268be216b75e /mnt/distr ext4 defaults 0 2" >> /etc/fstab'
danila@kds-otus01:~$ sudo sh -c 'echo "/dev/md127 /mnt/raid1 ext4 defaults 0 2" >> /etc/fstab'
danila@kds-otus01:/$ sudo update-initramfs -u
update-initramfs: Generating /boot/initrd.img-6.14.4-061404-generic

9. Создаем файл для проверки работоспособности массива. С помощью mdadm удаляем раздел sdb1 и проверяем работоспособность массива:

danila@kds-otus01:/$ sudo touch /mnt/raid1/test.txt
danila@kds-otus01:/$ sudo mdadm /dev/md127 --fail /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md127
danila@kds-otus01:/$ cat /proc/mdstat
Personalities : [raid1] [linear] [raid0] [raid6] [raid5] [raid4] [raid10]
md127 : active raid1 sdb2[1] sdb1[0](F)
      101376 blocks super 1.2 [2/1] [_U]


10. Добавим рабочий раздел в массив и проверим состояние. После чего удалим сбойный раздел:

danila@kds-otus01:/$ sudo mdadm /dev/md127 --add /dev/sdb3
mdadm: added /dev/sdb3
danila@kds-otus01:/$ cat /proc/mdstat
Personalities : [raid1] [linear] [raid0] [raid6] [raid5] [raid4] [raid10]
md127 : active raid1 sdb3[2] sdb2[1] sdb1[0](F)
      101376 blocks super 1.2 [2/2] [UU]

unused devices: <none>
danila@kds-otus01:/$ sudo mdadm --detail /dev/md127
/dev/md127:
           Version : 1.2
     Creation Time : Wed May 14 18:56:44 2025
        Raid Level : raid1
        Array Size : 101376 (99.00 MiB 103.81 MB)
     Used Dev Size : 101376 (99.00 MiB 103.81 MB)
      Raid Devices : 2
     Total Devices : 3
       Persistence : Superblock is persistent

       Update Time : Wed May 14 19:34:07 2025
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 1
     Spare Devices : 0

Consistency Policy : resync

              Name : kds-otus01:127  (local to host kds-otus01)
              UUID : 9570d2eb:ffa5aefe:4262740d:dc31050a
            Events : 38

    Number   Major   Minor   RaidDevice State
       2       8       19        0      active sync   /dev/sdb3
       1       8       18        1      active sync   /dev/sdb2

       0       8       17        -      faulty   /dev/sdb1
danila@kds-otus01:/$ sudo mdadm /dev/md127 --remove /dev/sdb1
mdadm: hot removed /dev/sdb1 from /dev/md127
