**Домашка 1:**

0. проверяем текущую версию ядра:
```
danila@kds-otus01:~$ uname -r 
6.8.0-59-generic
```
1. Добавляем репозиторий:
```
danila@kds-otus01:~$ sudo add-apt-repository ppa:cappelikan/ppa 
```

2. Обновляем список доступных пактов: 
```
danila@kds-otus01:~$ sudo apt update
```

3. Устанавливаем пакет mainline kernels:
```
danila@kds-otus01:~$ sudo apt install mainline 
```

4. Проверяем доступную версию ядра: 
```
danila@kds-otus01:~$ mainline check 
mainline 1.4.13
Updating Kernels...
Latest update: 6.14.4 
Latest point update: 6.8.12 
mainline: done 
```

5. Обновляем ядро до последней доступной версии: 
```
danila@kds-otus01:~$ sudo mainline install-latest 
```

6. Перезагружаем виртуальную машину: 
```
danila@kds-otus01:~$ sudo reboot 
```

7. Проверяем текущую версию ядра: 
```
danila@kds-otus01:~$ uname -r 
6.14.4-061404-generic
```


**Домашка 2:**

1. С правами суперпользователя запускаем утилиту gdisk и создаем таблицу разделов GUID:
```
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
```

2. Создаем 5 разделов по 100 МБ каждый, после чего проверяем результат:
```
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
```

3. Выходим из утилиты gdisk с сохранением:
```
Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdb.
The operation has commdadm pleted successfully.
danila@kds-otus01:~$
```

4. Выводим список доступных блочных устройств:
```
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
```

5. Создаем на разделе sdb5 файловую систему:
```
danila@kds-otus01:/$ sudo mkfs.ext4 /dev/sdb5
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
```

6. Создаем каталог /mnt/distr и монтируем к нему раздел /dev/sdb5:
```
danila@kds-otus01:/$ sudo mkdir /mnt/distr && sudo mount /dev/sdb5 /mnt/distr
danila@kds-otus01:/$
```

7. Создаем из разделов sdb1 и sdb2 массив raid1, создаем файловую систему, каталог подключения и монтируем созданный raid1. Проверяем:
```
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
```

8. Сохраним данные о массиве и точках монтирования в конфигурационных файлах. Обновим initramfs:
```
danila@kds-otus01:~$ sudo sh -c 'mdadm --detail --scan >> /etc/mdadm/mdadm.conf'
danila@kds-otus01:~$ sudo sh -c 'echo "/dev/disk/by-uuid/c8ccf4b3-bb0e-404a-8401-268be216b75e /mnt/distr ext4 defaults 0 2" >> /etc/fstab'
danila@kds-otus01:~$ sudo sh -c 'echo "/dev/md127 /mnt/raid1 ext4 defaults 0 2" >> /etc/fstab'
danila@kds-otus01:/$ sudo update-initramfs -u
update-initramfs: Generating /boot/initrd.img-6.14.4-061404-generic
```
9. Создаем файл для проверки работоспособности массива. С помощью mdadm удаляем раздел sdb1 и проверяем работоспособность массива:
```
danila@kds-otus01:/$ sudo touch /mnt/raid1/test.txt
danila@kds-otus01:/$ sudo mdadm /dev/md127 --fail /dev/sdb1
mdadm: set /dev/sdb1 faulty in /dev/md127
danila@kds-otus01:/$ cat /proc/mdstat
Personalities : [raid1] [linear] [raid0] [raid6] [raid5] [raid4] [raid10]
md127 : active raid1 sdb2[1] sdb1[0](F)
      101376 blocks super 1.2 [2/1] [_U]
```

10. Добавим рабочий раздел в массив и проверим состояние. После чего удалим сбойный раздел:
```
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
```


**Домашка 3**

1. Пакет lvm2 уде установлен в системе, LVM настроен:
```
danila@kds-otus01:~$ lsblk -f
NAME                      FSTYPE            FSVER            LABEL                           UUID                                   FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1
├─sda2                    ext4              1.0                                              a40566c6-01cb-4d01-9595-0c73e227b1a7      1.6G    10% /boot
└─sda3                    LVM2_member       LVM2 001                                         FWDBDo-LlJe-Quue-5bAJ-Lfw7-0I9W-GWOWPq
  └─ubuntu--vg-ubuntu--lv ext4              1.0                                              5295f539-3d2d-44cd-8fab-c05beb98f11f     11.1G    35% /
sdb
├─sdb1                    linux_raid_member 1.2              kds-otus01:127                  9570d2eb-ffa5-aefe-4262-740ddc31050a
├─sdb2                    linux_raid_member 1.2              kds-otus01:127                  9570d2eb-ffa5-aefe-4262-740ddc31050a
│ └─md127                 ext4              1.0                                              cf4255e3-fa1e-4255-b469-f974439623f0     81.8M     0% /mnt/raid1
├─sdb3                    linux_raid_member 1.2              kds-otus01:127                  9570d2eb-ffa5-aefe-4262-740ddc31050a
│ └─md127                 ext4              1.0                                              cf4255e3-fa1e-4255-b469-f974439623f0     81.8M     0% /mnt/raid1
├─sdb4                    ext4              1.0                                              183d03dc-78ed-48d7-8bc2-06ed61991865
└─sdb5                    ext4              1.0                                              c8ccf4b3-bb0e-404a-8401-268be216b75e     82.7M     0% /mnt/distr
sdc
sdd
sr0                       iso9660           Joliet Extension Ubuntu-Server 24.04.2 LTS amd64 2025-02-16-22-49-22-00

```

2. Cоздаем physical volume, volume group test-vg и logical volume otus:
```
danila@kds-otus01:~$ sudo pvcreate /dev/sdc
[sudo] password for danila:
  Physical volume "/dev/sdc" successfully created.
danila@kds-otus01:~$ sudo vgcreate test-vg /dev/sdc
  Volume group "test-vg" successfully created
danila@kds-otus01:~$ sudo lvcreate test-vg -n otus -L 100M
  Logical volume "otus" created.

```

3. Создаем на logical volume файловую систему EXT4, создаем точку монтирования /mnt/exercise3 и подключаем раздел с последующим занесением в fstab:
```
danila@kds-otus01:~$ sudo mkfs.ext4 /dev/mapper/test--vg-otus
mke2fs 1.47.0 (5-Feb-2023)
Discarding device blocks: done
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done
Writing inode tables: done
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done
danila@kds-otus01:~$ sudo mkdir /mnt/exercise3
danila@kds-otus01:~$ sudo mount /dev/mapper/test--vg-otus /mnt/exercise3
danila@kds-otus01:~$ sudo sh -c 'echo "/dev/disk/by-uuid/ab8535a2-d9e3-4d37-a99b-fd99acdc971b /mnt/exercise3 ext4 defaults 0 2" >> /etc/fstab'
```

4. Создаем physical volume на диске sdd и расширяем volume group test-vg на него:
```
danila@kds-otus01:~$ sudo pvcreate /dev/sdd
[sudo] password for danila:
  Physical volume "/dev/sdd" successfully created.
danila@kds-otus01:~$ sudo vgextend test-vg /dev/sdd
  Volume group "test-vg" successfully extended
```

5. Добавляем logical volume otus дополнительные 200 МБ дискового пространства и проверяем:
```
danila@kds-otus01:~$ sudo lvextend /dev/mapper/test--vg-otus -L +200M
  Size of logical volume test-vg/otus changed from 100.00 MiB (25 extents) to 300.00 MiB (75 extents).
  Logical volume test-vg/otus successfully resized.
danila@kds-otus01:~$ sudo resize2fs /dev/mapper/test--vg-otus
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/mapper/test--vg-otus is mounted on /mnt/exercise3; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/mapper/test--vg-otus is now 76800 (4k) blocks long.
danila@kds-otus01:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              392M  808K  391M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   19G  6.5G   12G  37% /
tmpfs                              2.0G     0  2.0G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/md127                          89M   24K   82M   1% /mnt/raid1
/dev/sda2                          2.0G  188M  1.7G  11% /boot
/dev/sdb5                           90M   24K   83M   1% /mnt/distr
tmpfs                              392M   16K  392M   1% /run/user/1000
/dev/mapper/test--vg-otus          278M   24K  265M   1% /mnt/exercise3
```

6. Проверяем состояние pv, vg и lv:
```
danila@kds-otus01:~$ sudo pvdisplay
  --- Physical volume ---
  PV Name               /dev/sdc
  VG Name               test-vg
  PV Size               1.00 GiB / not usable 4.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              255
  Free PE               180
  Allocated PE          75
  PV UUID               2TPxXW-ue5U-BjPQ-jwMW-F8UP-RfGJ-6a5A0H

  --- Physical volume ---
  PV Name               /dev/sdd
  VG Name               test-vg
  PV Size               1.00 GiB / not usable 4.00 MiB
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              255
  Free PE               255
  Allocated PE          0
  PV UUID               dxairZ-MDVo-wmuh-ytpw-CpSt-tB7R-GNPVrf

  --- Physical volume ---
  PV Name               /dev/sda3
  VG Name               ubuntu-vg
  PV Size               <38.00 GiB / not usable 0
  Allocatable           yes
  PE Size               4.00 MiB
  Total PE              9727
  Free PE               4864
  Allocated PE          4863
  PV UUID               FWDBDo-LlJe-Quue-5bAJ-Lfw7-0I9W-GWOWPq

danila@kds-otus01:~$ sudo vgdisplay
  --- Volume group ---
  VG Name               test-vg
  System ID
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  4
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               1.99 GiB
  PE Size               4.00 MiB
  Total PE              510
  Alloc PE / Size       75 / 300.00 MiB
  Free  PE / Size       435 / <1.70 GiB
  VG UUID               x9Wgds-tpwL-3a2T-FRYk-2TIm-MRWk-ouolHN

  --- Volume group ---
  VG Name               ubuntu-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <38.00 GiB
  PE Size               4.00 MiB
  Total PE              9727
  Alloc PE / Size       4863 / <19.00 GiB
  Free  PE / Size       4864 / 19.00 GiB
  VG UUID               EZ48E9-uDW1-bFi5-NRSS-Bi3G-ohCX-EDMaU2

danila@kds-otus01:~$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/test-vg/otus
  LV Name                otus
  VG Name                test-vg
  LV UUID                cPucL9-LVe0-ez0B-tsC2-0GNJ-hyYe-e8cq6e
  LV Write Access        read/write
  LV Creation host, time kds-otus01, 2025-05-19 18:04:57 +0300
  LV Status              available
  # open                 1
  LV Size                300.00 MiB
  Current LE             75
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:1

  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                Lkkx1d-E2wF-FlRS-cYwm-AIvU-xFE9-5QZMMk
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2025-05-07 09:26:34 +0300
  LV Status              available
  # open                 1
  LV Size                <19.00 GiB
  Current LE             4863
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           252:0

```

**Домашка 4**

